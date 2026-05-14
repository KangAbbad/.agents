# Supabase Migrations Reference

## Standard Table Shape

Every mutable business table follows this structure:

```sql
CREATE TABLE IF NOT EXISTS public.<table_name> (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  tenant_id UUID NOT NULL REFERENCES public.tenants(id) ON DELETE CASCADE,

  -- domain columns here

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ  -- NULL = active, timestamp = soft-deleted
);
```

Exceptions:
- Global lookup tables: omit `tenant_id`
- Immutable audit/ledger tables: omit `updated_at` and `deleted_at`

## Column Conventions

| Purpose | Type | Example |
|---|---|---|
| Primary key | `UUID DEFAULT gen_random_uuid()` | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` |
| Foreign key | `UUID REFERENCES public.<table>(id)` | `tenant_id UUID NOT NULL REFERENCES public.tenants(id)` |
| Timestamps | `TIMESTAMPTZ NOT NULL DEFAULT NOW()` | `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` |
| Soft delete | `TIMESTAMPTZ` nullable | `deleted_at TIMESTAMPTZ` |
| Money/amounts | `INTEGER` | `amount_cents INTEGER NOT NULL CHECK (amount_cents >= 0)` |
| Quantities | `DECIMAL(10,3)` | `weight_kg DECIMAL(10,3) NOT NULL` |
| Status/enum | `TEXT` + `CHECK` | `status TEXT NOT NULL CHECK (status IN ('PENDING', 'DONE'))` |
| Flexible data | `JSONB` | `metadata JSONB DEFAULT '{}'::jsonb` |
| Actor | `UUID REFERENCES public.profiles(id)` | `created_by UUID REFERENCES public.profiles(id)` |

## Naming Conventions

### Tables
- Plural snake_case: `orders`, `order_items`, `tenant_members`
- Junction tables: `<parent>_<child>` or `<entity>_<scope>`
- Audit/history: `<entity>_status_history`, `<entity>_ledger_entries`

### Columns
- FK: `<entity>_id` (e.g., `tenant_id`, `order_id`)
- Timestamps: `created_at`, `updated_at`, `deleted_at`
- Actors: `created_by`, `updated_by`, `performed_by`

### Indexes
```
idx_<table>_<column>
idx_<table>_<col1>_<col2>
<table>_<scope>_unique
```

### Triggers
```
update_<table>_updated_at
log_<table>_<event>
check_<table>_<rule>
```

### Functions
```
update_updated_at_column()
is_tenant_member()
has_tenant_role()
<verb>_<domain>()
```

## RLS Policies

### Service Role Bypass (Required)

```sql
CREATE POLICY "Service role manages <table>"
  ON public.<table>
  FOR ALL
  TO service_role
  USING (TRUE)
  WITH CHECK (TRUE);
```

### Member Read

```sql
CREATE POLICY "<table>_select"
  ON public.<table>
  FOR SELECT
  TO authenticated
  USING (
    deleted_at IS NULL
    AND public.is_tenant_member(tenant_id)
  );
```

### Role-Gated Insert

```sql
CREATE POLICY "<table>_insert"
  ON public.<table>
  FOR INSERT
  TO authenticated
  WITH CHECK (
    public.has_tenant_role(tenant_id, ARRAY['OWNER', 'ADMIN'])
  );
```

### Role-Gated Update

```sql
CREATE POLICY "<table>_update"
  ON public.<table>
  FOR UPDATE
  TO authenticated
  USING (
    deleted_at IS NULL
    AND public.has_tenant_role(tenant_id, ARRAY['OWNER', 'ADMIN'])
  )
  WITH CHECK (
    public.has_tenant_role(tenant_id, ARRAY['OWNER', 'ADMIN'])
  );
```

### Soft Delete (Owner Only)

```sql
CREATE POLICY "<table>_delete"
  ON public.<table>
  FOR UPDATE
  TO authenticated
  USING (
    public.has_tenant_role(tenant_id, ARRAY['OWNER'])
  )
  WITH CHECK (
    public.has_tenant_role(tenant_id, ARRAY['OWNER'])
  );
```

## Trigger Patterns

### Updated At (Required on Mutable Tables)

```sql
DROP TRIGGER IF EXISTS update_<table>_updated_at ON public.<table>;
CREATE TRIGGER update_<table>_updated_at
  BEFORE UPDATE ON public.<table>
  FOR EACH ROW
  EXECUTE FUNCTION public.update_updated_at_column();
```

### Audit / Status History

```sql
CREATE OR REPLACE FUNCTION public.log_<table>_status_change()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.status IS DISTINCT FROM NEW.status THEN
    INSERT INTO public.<table>_status_history (
      <table>_id, tenant_id, from_status, to_status, changed_at
    ) VALUES (
      NEW.id, NEW.tenant_id, OLD.status, NEW.status, NOW()
    );
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

DROP TRIGGER IF EXISTS log_<table>_status_change ON public.<table>;
CREATE TRIGGER log_<table>_status_change
  AFTER UPDATE OF status ON public.<table>
  FOR EACH ROW
  EXECUTE FUNCTION public.log_<table>_status_change();
```

## Soft Delete Patterns

### Partial Unique Index

```sql
CREATE UNIQUE INDEX IF NOT EXISTS <table>_<scope>_unique
  ON public.<table>(tenant_id, slug)
  WHERE deleted_at IS NULL;
```

### Soft Delete Performance Index

```sql
CREATE INDEX IF NOT EXISTS idx_<table>_deleted_at
  ON public.<table>(deleted_at)
  WHERE deleted_at IS NULL;
```

## Data Integrity

### Check Constraints

```sql
-- Enum-like
CHECK (status IN ('PENDING', 'ACTIVE', 'ARCHIVED'))

-- Non-negative
CHECK (amount >= 0)

-- Formula consistency
CHECK (total = subtotal - discount)

-- Geo bounds
CHECK (latitude >= -90 AND latitude <= 90)
```

### Foreign Key Cascade Rules

| Relationship | ON DELETE |
|---|---|
| Owned child | `CASCADE` |
| Optional historical link | `SET NULL` |
| True child row | `CASCADE` |

## Idempotency Patterns

### Seed Data (Global Defaults)

```sql
INSERT INTO public.countries (code, name) VALUES
  ('US', 'United States'),
  ('CA', 'Canada')
ON CONFLICT (code) DO NOTHING;
```

### Per-Tenant Defaults

```sql
CREATE OR REPLACE FUNCTION public.seed_default_payment_methods(p_tenant_id UUID)
RETURNS INTEGER AS $$
BEGIN
  INSERT INTO public.payment_methods (tenant_id, name, is_active)
  SELECT p_tenant_id, name, true
  FROM (VALUES ('Cash'), ('Card'), ('Bank Transfer')) AS defaults(name)
  WHERE NOT EXISTS (
    SELECT 1 FROM public.payment_methods
    WHERE tenant_id = p_tenant_id AND deleted_at IS NULL
  );
  RETURN 1;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION public.trg_seed_default_payment_methods()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM public.seed_default_payment_methods(NEW.id);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER seed_default_payment_methods_after_tenant_insert
  AFTER INSERT ON public.tenants
  FOR EACH ROW
  EXECUTE FUNCTION public.trg_seed_default_payment_methods();
```

### Atomic Counter / Sequence

```sql
INSERT INTO public.invoice_sequences (tenant_id, counter)
VALUES (p_tenant_id, 1)
ON CONFLICT (tenant_id)
DO UPDATE SET counter = public.invoice_sequences.counter + 1
RETURNING counter INTO v_sequence;
```

## Immutable / Audit Tables

Append-only tables for history and ledger:

```sql
CREATE TABLE IF NOT EXISTS public.<entity>_ledger_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES public.tenants(id) ON DELETE CASCADE,
  <entity>_id UUID NOT NULL REFERENCES public.<entity>(id) ON DELETE CASCADE,

  amount_delta INTEGER NOT NULL,
  balance_before INTEGER NOT NULL,
  balance_after INTEGER NOT NULL,

  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
  -- NO updated_at, NO deleted_at
);

ALTER TABLE public.<entity>_ledger_entries ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Service role manages <entity>_ledger_entries"
  ON public.<entity>_ledger_entries
  FOR ALL TO service_role
  USING (TRUE) WITH CHECK (TRUE);

CREATE POLICY "<entity>_ledger_entries_select"
  ON public.<entity>_ledger_entries
  FOR SELECT TO authenticated
  USING (public.is_tenant_member(tenant_id));

-- No INSERT/UPDATE/DELETE policies for authenticated users
```

## Execution Order

Run migrations in dependency order:

1. `core/001_functions.sql` — shared trigger functions
2. Lookup tables (countries, etc.)
3. Auth/profile tables
4. Tenant + membership tables
5. RLS helper functions
6. Business feature tables
7. Ledger/history/audit tables

## Migration Idempotency Checklist

Before finalizing:

- [ ] All `CREATE TABLE/INDEX/TRIGGER` use `IF NOT EXISTS`
- [ ] `DROP TRIGGER IF EXISTS` before `CREATE TRIGGER`
- [ ] Seed inserts use `ON CONFLICT`
- [ ] RLS enabled + service role policy exists
- [ ] Soft-delete filters in read policies
- [ ] Partial unique indexes use `WHERE deleted_at IS NULL`
- [ ] No hard deletes on business tables
- [ ] Comments document table/column purpose
- [ ] FK constraints use appropriate `ON DELETE` rules
- [ ] CHECK constraints validate domain rules

## Common Mistakes

- **Reused migration numbers** in same folder → causes ordering ambiguity
- **Missing service-role policy** → table inaccessible from backend
- **Unique index without `WHERE deleted_at IS NULL`** → blocks reuse of soft-deleted values
- **Hard deletes on business tables** → data loss, audit trail broken
- **Non-idempotent seeds** → migration fails on re-run
- **Duplicated RLS logic** → maintenance burden, inconsistency
- **Running by filename order** when dependency order differs → missing dependencies
- **No `IF NOT EXISTS`** → migration fails on re-run
- **Missing `DROP TRIGGER IF EXISTS`** → trigger creation fails on re-run
