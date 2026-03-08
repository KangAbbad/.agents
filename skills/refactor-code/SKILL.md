---
name: refactor-code
description: Refactor code to follow TypeScript, API contracts, Drizzle ORM, and React component patterns. Use when refactoring, cleaning up code, or applying code standards to existing files. Handles React components, hooks, JSX ordering, and styling rules.
---

# Refactor Code Skill

## Quick Start

Analyze file against standards, then apply fixes:

1. **Check TypeScript patterns** - enums, naming, types, imports, hook aliases, suppressHydrationWarning
2. **Check API contracts** - schema naming, argument patterns, boundaries, service/hook layers
3. **Check Drizzle patterns** - queries, relations, repositories, RLS
4. **Check React component patterns** - JSX ordering, self-contained logic, data extraction, Tailwind/Ant Design rules

## Workflows

### Refactoring a Single File

1. Read the target file
2. Check against patterns in REFERENCE.md
3. Apply fixes in priority order:
   - Critical: Breaking API contracts
   - High: Type safety issues (any, enums, missing validation)
   - Medium: Naming conventions
   - Low: Style/consistency
4. Run typecheck: `bun typecheck`

### Refactoring Multiple Files

1. Use glob to find files: `glob "src/**/*.ts"`
2. Prioritize by:
   - Files with `any` types
   - Files with enums
   - Files with manual JOINs
   - Files with `suppressHydrationWarning`
   - Files with inconsistent naming
   - Files missing service/hook layer separation
   - Files with prop drilling (handlers passed unnecessarily)
   - Files with wrong JSX property ordering
   - Files with conditional logic in JSX
3. Refactor one file at a time
4. Run typecheck after each file

## Pre-Refactor Checklist

Before changing code:

- [ ] Does this match backend API contract? ⚠️
- [ ] Am I preserving field names from backend responses?
- [ ] Is argument pattern correct? (1 arg=direct, 2+=object)
- [ ] Are relations defined for foreign keys?
- [ ] Should this use service layer + hook layer pattern?
- [ ] Is RLS being applied for user-scoped queries?
- [ ] Am I avoiding `suppressHydrationWarning`?
- [ ] Are React components self-contained? (own their logic via hooks)
- [ ] Is JSX property order correct? (required → components → styling → handlers)
- [ ] Is conditional data extracted to variables before JSX?
- [ ] Are icon imports using Icon suffix? (`PlusIcon` not `Plus`)
- [ ] Is `clsx` used for conditional className?
- [ ] Are Ant Design Button icons passed as children (not icon prop)?

## Common Refactors

| Pattern                  | Find                                           | Replace With                                         |
| ------------------------ | ---------------------------------------------- | ---------------------------------------------------- |
| Enums                    | `enum Status` or `z.enum()`                    | `const STATUS_LIST` + `z.union([z.literal(...)])`    |
| Any types                | `: any`                                        | `: unknown` + Zod validation                         |
| Manual JOINs             | `.leftJoin()`                                  | `db.query.*` with `with`                             |
| Generic "item"           | `deleteItem`, `itemList`                       | `deleteProduct`, `productList`                       |
| Plural forms             | `users`, `products`                            | `userList`, `productList`                            |
| handle prefix            | `handleSubmit`                                 | `submitForm`                                         |
| on prefix (internal)     | `onSubmitForm`                                 | `submitForm`                                         |
| Generic destructuring    | `{ isLoading, create }`                        | `{ isLoading: isXLoading, create: createX }`         |
| Barrel files             | `index.ts` exports                             | Direct imports                                       |
| Namespace imports        | `import * as React`                            | Named imports                                        |
| suppressHydrationWarning | `suppressHydrationWarning`                     | Remove, fix root cause                               |
| zod import               | `from 'zod'`                                   | `from 'zod/v4-mini'`                                 |
| Raw types                | `type X = { ... }`                             | Schema + `z.infer<typeof Schema>`                    |
| JSX ordering             | `onClick` before `className`                   | Required → Components → className → style → Handlers |
| Prop drilling            | `onSave={saveData}` from parent                | Component owns logic via custom hook                 |
| Inline conditionals      | `{data ? format() : 'N/A'}`                    | Extract to variable: `const value = data ? ...`      |
| Lucide imports           | `import { Plus }`                              | `import { PlusIcon }`                                |
| Tailwind conditionals    | `className={isActive ? 'bg-blue' : 'bg-gray'}` | Use `clsx` for conditional classes                   |
| Ant Button icon          | `<Button icon={<PlusIcon />}>`                 | `<Button><PlusIcon />...</Button>`                   |

## References

- [TypeScript Patterns](REFERENCE.md#typescript-patterns)
- [API Contracts](REFERENCE.md#api-contracts)
- [Drizzle Patterns](REFERENCE.md#drizzle-patterns)
- [React Component Patterns](REFERENCE.md#react-component-patterns)
