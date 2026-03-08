# Refactor Code Reference

## TypeScript Patterns

### 1. Enums → Literal Unions

**Forbidden:**

```typescript
// ❌ TypeScript enum
enum Status {
  Pending = "pending",
  Active = "active",
}

// ❌ z.enum
const StatusSchema = z.enum(["pending", "active"]);
```

**Correct:**

```typescript
// ✅ Define const array, derive type, use z.literal union
const STATUS_LIST = ["pending", "active"] as const;
type Status = (typeof STATUS_LIST)[number];
const StatusSchema = z.union([z.literal("pending"), z.literal("active")]);
```

### 2. Any → Unknown + Validation

**Forbidden:**

```typescript
// ❌ Never use any
const processData = (data: any) => { ... }
```

**Correct:**

```typescript
// ✅ Use unknown for unvalidated data
const rawData: unknown = await fetchUserData();
const validationResult = UserDataSchema.safeParse(rawData);
if (validationResult.success) {
  const userData: UserData = validationResult.data;
}
```

### 3. Naming Conventions

| Pattern     | Avoid                       | Use                     |
| ----------- | --------------------------- | ----------------------- |
| Functions   | `handleSubmitForm`          | `submitForm`            |
| Functions   | `onSubmitForm`              | `submitForm`            |
| Functions   | `goToHomePage`              | `navigateToHomePage`    |
| Functions   | `openDeleteModal`           | `showDeleteModal`       |
| Functions   | `deleteItem`                | `deleteProduct`         |
| Collections | `users`                     | `userList`              |
| Collections | `products`                  | `productList`           |
| Types       | `type Product`              | `type ProductDTO`       |
| Types       | `type CreateProductPayload` | `type CreateProductDTO` |

**Exception:** Backend field names must match API response exactly.

### 3a. "Item" Terminology Rule (Critical)

Generic terms like `item`, `items`, `transactionItems` are **forbidden** in TypeScript/backend code because they're not descriptive. However, database table names in SQL migrations are **allowed** (and required) to match the actual database schema.

**❌ NOT Allowed - TypeScript/backend code:**
```typescript
// Variable/constant names
const transactionItems = pgTable("transaction_items", { ... })
const items = await db.query.items.findMany()
function deleteItem(id: string) { ... }

// Type names
type TransactionItemDTO { ... }
type CreateItemInput = { ... }

// Property names
transaction.items.map(item => ...)
```

**✅ Allowed - Database table names (must match SQL schema):**
```typescript
// Table name in pgTable MUST match the actual database table
export const transactionArticles = pgTable("transaction_items", { ... })
// The first argument "transaction_items" is the REAL table name - keep as-is
// The constant name transactionArticles is the CODE reference - must be descriptive
```

**✅ Correct - Descriptive TypeScript names:**
```typescript
// Use specific entity names
export const transactionArticles = pgTable("transaction_items", { ... })
const articles = await db.query.transactionArticles.findMany()
function deleteTransactionArticle(id: string) { ... }

// Property names
type TransactionDTO {
  articles?: TransactionArticleDTO[]  // ✅ Clear what this contains
}
```

**Key Distinction:**
- **Database layer** (SQL migrations, pgTable first argument): Use table name exactly as defined in SQL
- **Application layer** (TypeScript variables, functions, types): Never use generic "item" - always use specific domain terms

### 4. No Barrel Files

**Forbidden:**

```typescript
// ❌ Barrel file
// components/BulkInputSelect/index.ts
export { BulkInputSelect } from "./BulkInputSelect.component";
import { BulkInputSelect } from "@/components/BulkInputSelect";
```

**Correct:**

```typescript
// ✅ Direct import
import { BulkInputSelect } from "@/components/BulkInputSelect/BulkInputSelect.component";
```

### 5. No Namespace Imports

**Forbidden:**

```typescript
// ❌ Namespace import
import * as React from "react";
React.useState(0);
```

**Correct:**

```typescript
// ✅ Named imports
import { useState } from "react";
useState(0);
```

### 6. Use Zod/v4-mini

**Forbidden:**

```typescript
// ❌ Standard zod
import { z } from "zod";
const UserSchema = z.object({
  name: z.string().nullish(),
});
```

**Correct:**

```typescript
// ✅ zod/v4-mini
import { z } from "zod/v4-mini";
const UserSchema = z.object({
  name: z.nullish(z.string()),
});
```

### 7. Schema-First Types

**Forbidden:**

```typescript
// ❌ Raw type without schema
type CreateUserDTO = { name: string; email: string };
```

**Correct:**

```typescript
// ✅ Schema first, then infer
const CreateUserSchema = z.object({ name: z.string(), email: z.string() });
type CreateUserDTO = z.infer<typeof CreateUserSchema>;
```

### 8. Prefer Type Over Interface

```typescript
// ✅ Use type for shapes
type User = { id: string; name: string };
```

### 9. Descriptive Aliases for Hooks

**Correct:**

```typescript
// ✅ Descriptive aliases when destructuring
const { isLoading: isCustomerLoading, create: createCustomer } =
  useCustomerForm();
```

### 10. NEVER Use suppressHydrationWarning

**Forbidden:**

```typescript
// ❌ Never suppress hydration warnings
<html suppressHydrationWarning>
<body suppressHydrationWarning>
<div suppressHydrationWarning>
```

**Fix root causes instead:**

- Use `useEffect` for client-only code
- Ensure server/client render same initial content
- Use dynamic imports with `ssr: false`
- Make timestamps consistent between server/client

---

## API Contracts

### 1. Frontend vs Backend Boundaries

**Cannot Change (Backend Contract):**

- Schema field names
- Endpoint URLs
- HTTP methods
- Request/response structure
- Query parameter names

**Can Change (Frontend Code):**

- Schema/type names
- Service/hook function names
- Component names
- Local variable names

### 2. Field Name Handling

**Correct approach:**

```typescript
// Schema matches backend
const ProductDataDTOSchema = z.object({
  item_id: z.string(),
  item_name: z.string(),
});

// Alias when consuming
const { item_id: productId, item_name: productName } = data?.data ?? {};
```

### 3. Service Layer Naming

**Mutation operations** (use `{Op}{Entity}` pattern):

- `CreateCustomerParamsDTOSchema`
- `CreateCustomerRequestBodyDTOSchema`
- `CreateCustomerDataDTOSchema`
- `CreateCustomerResponseDTOSchema`

**Read operations** (use `{Entity}{Op}` pattern):

- `CustomerListParamsDTOSchema`
- `CustomerListDataDTOSchema`
- `CustomerListResponseDTOSchema`

### 4. Argument Patterns

| Args | Pattern              | Example                                            |
| ---- | -------------------- | -------------------------------------------------- |
| 1    | Direct               | `async (body: Body)`                               |
| 2+   | Object destructuring | `async ({ params, body }: { params: P; body: B })` |

**Correct:**

```typescript
// 1 arg = direct
export const deleteEntity = async (params: P) => { ... }

// 2+ args = object
export const updateEntity = async ({ params, body }: { params: P; body: B }) => { ... }
```

### 5. Service File Structure

**Service files** (lowercase): `post.ts`, `put.ts`, `patch.ts`, `delete.ts`, `get.ts`

**Hook files** (PascalCase): `useCreate{Entity}.tsx`, `useUpdate{Entity}.tsx`, `useDelete{Entity}.tsx`

**Two-layer pattern:**

1. **Service Layer** (`/services/{operation}.ts`) - API calls + Zod validation
2. **Hook Layer** (`/hooks/api/use{Operation}{Entity}.tsx`) - mutations + cache + feedback

```typescript
// 1. PARAMS SCHEMA
export const CreateCustomerParamsDTOSchema = z.object({ ... })
export type CreateCustomerParamsDTO = z.infer<typeof CreateCustomerParamsDTOSchema>

// 2. REQUEST BODY SCHEMA
export const CreateCustomerRequestBodyDTOSchema = z.object({ ... })
export type CreateCustomerRequestBodyDTO = z.infer<typeof CreateCustomerRequestBodyDTOSchema>

// 3. RESPONSE DATA SCHEMA
export const CreateCustomerDataDTOSchema = z.object({ ... })
export type CreateCustomerDataDTO = z.infer<typeof CreateCustomerDataDTOSchema>

// 4. RESPONSE WRAPPER
export const CreateCustomerResponseDTOSchema = httpResponseSchemaBuilder(CreateCustomerDataDTOSchema)
export type CreateCustomerResponseDTO = z.infer<typeof CreateCustomerResponseDTOSchema>

// 5. API FUNCTION
export const createCustomer = async (body: CreateCustomerRequestBodyDTO) => {
  return await apiCall(url, CreateCustomerResponseDTOSchema, {
    method: 'POST',
    body: JSON.stringify(body),
  })
}
```

### 6. Common Pitfalls

❌ **Inconsistent naming**

```typescript
export const createItem = async (...)  // Generic
export const CreateCustomerBodySchema = z.object(...) // Missing "RequestBody"
```

✅ **Follow conventions**

```typescript
export const createCustomer = async (...)
export const CreateCustomerRequestBodyDTOSchema = z.object(...)
```

❌ **Changing backend field names**

```typescript
export const Schema = z.object({ customerId: z.string() }); // Backend sends customer_id
```

✅ **Match backend, alias in frontend**

```typescript
export const Schema = z.object({ customer_id: z.string() });
const { customer_id: customerId } = data;
```

### 7. AI Reminder Checklist

Before implementing:

- [ ] Do schema FIELD NAMES match backend response? ⚠️ CRITICAL
- [ ] Am I only changing frontend naming (types/functions)? ✅ SAFE
- [ ] Does API use PUT or PATCH? ⚠️
- [ ] Is argument pattern correct? (1 arg=direct, 2+=object) ⚠️

**Key distinction:**

- ✅ Frontend code (names) → Change freely
- ⚠️ API contract (schema fields, endpoints) → Must match backend

---

## Drizzle Patterns

### 1. Relational Queries Over Manual JOINs

**Forbidden:**

```typescript
// ❌ Manual JOIN
const memberList = await db
  .select({ id: outletMembers.id, profile: { id: profiles.id } })
  .from(outletMembers)
  .leftJoin(profiles, eq(outletMembers.userId, profiles.id));
```

**Correct:**

```typescript
// ✅ Relational query
const memberList = await db.query.outletMembers.findMany({
  where: eq(outletMembers.outletId, outletId),
  with: { user: true },
});
```

### 2. Define Relations in Schema

```typescript
// ✅ Define relations
export const outletsRelations = relations(outlets, ({ one, many }) => ({
  owner: one(profiles, {
    fields: [outlets.ownerId],
    references: [profiles.id],
  }),
  members: many(outletMembers),
}));
```

### 3. Repository Pattern

**File structure:**

```
src/routes/contacts/repositories/
├── findContactById.ts
├── findContactBySlug.ts
├── findContactList.ts
├── createContact.ts
├── updateContact.ts
└── softDeleteContact.ts
```

**Naming conventions:**
| Operation | Pattern | Example |
|-----------|---------|---------|
| Find one | `find{Entity}By{Field}` | `findOutletById` |
| Find one + relations | `find{Entity}By{Field}With{Relations}` | `findOutletByIdWithOwner` |
| Find many | `find{Entity}List` | `findOutletList` |
| Create | `create{Entity}` | `createOutlet` |
| Update | `update{Entity}` | `updateOutlet` |
| Soft delete | `softDelete{Entity}` | `softDeleteOutlet` |

**Repository function pattern:**

```typescript
// Receive db as parameter
export function findOutletById(db: Database, id: string) {
  return db.query.outlets.findFirst({ where: eq(outlets.id, id) });
}

// Usage in route
const outlet = await runWithRLS((db) => findOutletById(db, outletId));
```

### 4. Type Inference

```typescript
// ✅ Infer base types from schema
export type ProfileDTO = typeof profiles.$inferSelect;
export type CreateProfileDTO = typeof profiles.$inferInsert;

// ✅ Define types for entities with relations
export type OutletWithOwner = OutletDTO & { owner: ProfileDTO | null };
```

### 5. Transactions

```typescript
// ✅ Transaction for atomic operations
export async function createOutletWithOwner(db: Database, data: Input) {
  return db.transaction(async (tx) => {
    const [outlet] = await tx.insert(outlets).values({ ... }).returning()
    if (!outlet) throw new Error('Failed to create outlet')
    await tx.insert(outletMembers).values({ outletId: outlet.id, ... })
    return outlet
  })
}
```

### 6. Selecting Specific Columns

```typescript
// ✅ When you need specific columns only
const memberList = await db.query.outletMembers.findMany({
  where: eq(outletMembers.outletId, outletId),
  columns: {
    id: true,
    role: true,
    isActive: true,
  },
  with: {
    user: {
      columns: {
        id: true,
        fullName: true,
        avatarUrl: true,
      },
    },
  },
});
```

### 7. RLS-Aware Queries

When using Row Level Security, pass the RLS executor to repository functions.

```typescript
// Route handler
outletRoute.get("/:id", async (c) => {
  const { id } = c.req.valid("param");
  const runWithRLS = c.get("runWithRLS");

  // RLS is applied automatically via the executor
  const outlet = await runWithRLS((db) => findOutletByIdWithOwner(db, id));

  if (!outlet) {
    throw new HTTPException(404, { message: "Outlet not found" });
  }

  return c.json({ data: outlet });
});
```

### 8. Database Connection Pattern

**Forbidden:**

```typescript
// ❌ Creating connection inside repository
export async function findOutletById(id: string) {
  const db = createDb(process.env.DATABASE_URL)
  return db.query.outlets.findFirst({ ... })
}
```

**Correct:**

```typescript
// ✅ Receive db as parameter
export function findOutletById(db: Database, id: string) {
  return db.query.outlets.findFirst({ where: eq(outlets.id, id) });
}
```

---

## React Component Patterns

### 1. JSX Property Ordering

**Correct order:** Required props → Component props → className → style → Event handlers

```tsx
// ✅ Correct ordering
<SomeComponent
  placeholder="Enter name"
  value={name}
  disabled={isLoading}
  icon={<IconComponent />}
  customComponent={<CustomComponent />}
  className="tailwind-classes-here"
  style={styleObject}
  onChange={handleChange}
  onSubmit={handleSubmit}
/>
```

### 2. Self-Contained Component Logic

Components should own their business logic via custom hooks. Don't pass handlers that just forward from parent's hooks.

**Forbidden:**

```typescript
// ❌ Parent forwards handler
const { saveData } = useMyLogic()
<MyComponent onSave={saveData} />
```

**Correct:**

```typescript
// ✅ Component owns its logic
<MyComponent />

export function MyComponent() {
  const { saveData } = useMyComponentLogic()
  // ...
}
```

**Decision rule:** Is parent just forwarding a handler from its hook?

- YES → Component should own it (move to hook)
- NO → Pass as prop (parent has legitimate reason)

**Props ARE correct for:**

- Display data: `<List details={details} />`
- Generic UI components: `<Button onClick={handler} />`
- Read-only state: `isLoading`, `disabled`

**Never pass handlers that:**

- Come from component's own business domain
- Are derived from store/state actions
- Define component's core behavior

### 3. Extract Conditional Data to Variables

**Forbidden:**

```tsx
// ❌ Conditional logic in JSX
<p>
  {latestPurchase
    ? format(new Date(latestPurchase.createdAt), "dd/MM/yyyy")
    : "N/A"}
</p>
```

**Correct:**

```tsx
// ✅ Extract to variables before JSX
const formattedDate = latestPurchase
  ? format(new Date(latestPurchase.createdAt), "dd/MM/yyyy")
  : "N/A";

return <p>{formattedDate}</p>;
```

### 4. Icon Import Rules

**Forbidden:**

```typescript
import { Plus } from "lucide-react";
```

**Correct:**

```typescript
import { PlusIcon } from "lucide-react";
```

Always use the `Icon` suffix for lucide imports.

### 5. TailwindCSS Rules

**Conditional classes:**

```tsx
// ✅ Use clsx for conditional className
import { clsx } from 'clsx'

className={clsx(
  'base-classes',
  isActive && 'bg-blue-500',
  isDisabled && 'opacity-50'
)}
```

**Layout utilities:**

- Use `gap-*` for grid/flexbox layouts
- Use `space-y-*`/`space-x-*` for simple stacks
- **Never mix both on the same container**

**Text color for primary data:**

```tsx
// ❌ Wrong: Primary data with text-gray-900
<p className="text-gray-900 font-bold">Important price</p>

// ✅ Correct: Let it default to black
<p className="font-bold">Rp 50.000</p>
```

**Ant Design important mark:**

```tsx
// ✅ Correct: Important mark at the end
<Text className="text-left!">...</Text>

// ❌ Wrong: Important mark at start
<Text className="!text-left">...</Text>
```

### 6. Ant Design Button Component

**Forbidden:**

```tsx
// ❌ Never use icon prop
<Button icon={<PlusIcon />}>Add Item</Button>
```

**Correct:**

```tsx
// ✅ Pass icon as child
<Button>
  <PlusIcon />
  <span>Add Item</span>
</Button>
```

---

## Migration Patterns

**Schema Ownership:**

- **Drizzle** → TypeScript types only (table definitions)
- **Supabase SQL** → All DDL: tables, columns, RLS, indexes, constraints

**Workflow:**

1. Create SQL migration: `supabase/migrations/{sequence}_{description}.sql`
2. Run SQL in Supabase SQL Editor
3. Update `src/routes/*/schemas.ts` to match
4. Update Hoppscotch collection for API changes

**Timestamp rule:** Always use `withTimezone: true` for timestamps.

---

## Complete Refactor Checklist

### TypeScript

- [ ] No `enum` or `z.enum` — use literal unions
- [ ] No `any` types — use `unknown` + validation
- [ ] No `handle` prefix on functions
- [ ] No `on` prefix on internal functions (props only)
- [ ] Use `navigateTo` for navigation
- [ ] Use `show/hide` for modals
- [ ] No generic "item" terminology
- [ ] Use "List" suffix for collections
- [ ] Use "DTO" suffix for API types
- [ ] Descriptive aliases when destructuring hooks
- [ ] No `suppressHydrationWarning`
- [ ] No barrel files (index.ts)
- [ ] No namespace imports (`import * as React`)
- [ ] Use `zod/v4-mini`
- [ ] Schema-first types with `z.infer`
- [ ] Prefer `type` over `interface`

### API Contracts

- [ ] Schema field names match backend exactly
- [ ] Endpoint URLs unchanged
- [ ] HTTP methods unchanged
- [ ] Mutation operations: `{Op}{Entity}` pattern
- [ ] Read operations: `{Entity}{Op}` pattern
- [ ] Argument pattern: 1 arg=direct, 2+=object destructuring
- [ ] Service + Hook layer separation

### Drizzle ORM

- [ ] Relations defined for all foreign keys
- [ ] Using `db.query.*` with `with` instead of manual JOINs
- [ ] Repository functions in separate files
- [ ] Types defined for entities with relations
- [ ] Multi-table operations in transactions
- [ ] Database instance passed as parameter
- [ ] RLS executor used for user-scoped queries

### React Components

- [ ] JSX property order: Required → Components → className → style → Handlers
- [ ] Components own their logic via custom hooks (not prop drilling)
- [ ] Conditional data extracted to variables before JSX
- [ ] Icon imports use Icon suffix (`PlusIcon` not `Plus`)
- [ ] `clsx` used for conditional className
- [ ] No mixing `gap-*` with `space-y-*`/`space-x-*`
- [ ] Primary data uses default black (no `text-gray-900`)
- [ ] Ant Design important mark at end (`text-left!` not `!text-left`)
- [ ] Ant Design Button icons passed as children (not icon prop)
