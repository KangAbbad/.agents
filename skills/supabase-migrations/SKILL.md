---
name: supabase-migrations
description: Write safe, idempotent SQL migrations for multi-tenant Supabase backends. Use when creating database schema, adding tables, implementing RLS policies, or writing seed data migrations.
---

# Supabase Migrations Skill

## Quick Start

Create a feature-based migration:

```sql
-- supabase/migrations/features/<feature>/001_<change>.sql
CREATE TABLE IF NOT EXISTS public.orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES public.tenants(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);

ALTER TABLE public.orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Service role manages orders"
  ON public.orders FOR ALL TO service_role
  USING (TRUE) WITH CHECK (TRUE);

DROP TRIGGER IF EXISTS update_orders_updated_at ON public.orders;
CREATE TRIGGER update_orders_updated_at
  BEFORE UPDATE ON public.orders
  FOR EACH ROW
  EXECUTE FUNCTION public.update_updated_at_column();
```

## Workflows

### Writing a New Table Migration

1. **Determine folder**: Create `supabase/migrations/features/<feature>/` if needed
2. **Number sequentially**: Use `001_`, `002_`, etc. within the folder
3. **Define schema**: Use standard table shape (see REFERENCE.md)
4. **Add RLS**: Enable RLS + required policies
5. **Add triggers**: `updated_at` trigger for mutable tables
6. **Add indexes**: FK indexes + soft-delete partial indexes
7. **Verify idempotency**: All `CREATE` statements use `IF NOT EXISTS`

### Writing a Seed Data Migration

1. **Use `ON CONFLICT DO NOTHING`** for global defaults
2. **Use function + trigger** for per-tenant defaults
3. **Test re-run safety**: Migration should succeed if run twice

### Writing an Audit/History Table

1. **Omit `updated_at` and `deleted_at`**
2. **Add `AFTER UPDATE` trigger** to parent table
3. **Restrict policies**: Service role only, no user INSERT/UPDATE/DELETE

## Key Patterns

| Pattern | When | Example |
|---|---|---|
| Soft delete | Mutable business data | `deleted_at TIMESTAMPTZ` |
| Partial unique index | Uniqueness ignoring deleted | `WHERE deleted_at IS NULL` |
| Denormalized tenant_id | High-volume child tables | Audit tables, ledger entries |
| RLS helper functions | Membership/role checks | `is_tenant_member()`, `has_tenant_role()` |
| Atomic counter | Race-safe sequences | `INSERT ... ON CONFLICT DO UPDATE ... RETURNING` |
| Idempotency key | Financial operations | Unique partial index on `(scope_id, idempotency_key)` |

## Idempotency Checklist

Before finalizing:

- [ ] All `CREATE TABLE/INDEX/TRIGGER` use `IF NOT EXISTS`
- [ ] `DROP TRIGGER IF EXISTS` before `CREATE TRIGGER`
- [ ] Seed inserts use `ON CONFLICT`
- [ ] RLS enabled + service role policy exists
- [ ] Soft-delete filters in read policies
- [ ] Partial unique indexes use `WHERE deleted_at IS NULL`
- [ ] No hard deletes on business tables

## Common Mistakes

- Reusing migration numbers in same folder
- Missing service-role RLS policy
- Unique index without `WHERE deleted_at IS NULL`
- Non-idempotent seed inserts
- Duplicated RLS logic instead of helper functions

## References

See [REFERENCE.md](REFERENCE.md) for:
- Standard table shape template
- RLS policy examples
- Trigger patterns
- Execution order guidelines
