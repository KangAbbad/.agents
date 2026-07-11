# Backend Endpoint Code Pattern Guidelines

Use this document when adding or refactoring backend endpoints.

## Goal

Keep endpoint code consistent, composable, testable, and predictable by enforcing one implementation pattern across routes.

## Canonical Route Module Structure

Each resource route should follow this shape:

```text
src/routes/{resource}/
├── route.ts
├── schemas.ts
├── lib/
│   └── *.ts
├── repositories/
│   └── *.ts
├── services/
│   ├── get.ts
│   ├── post.ts
│   ├── patch.ts
│   ├── delete.ts
│   └── {sub-resource}/
│       └── route.ts
└── types/
    ├── common.ts
    ├── create.ts
    ├── update.ts
    ├── query.ts
    ├── filter.ts
    └── dto.ts
```

## Layer Responsibilities

### `route.ts` (resource entrypoint)

Responsibilities:

- Create resource `Hono<AppEnv>()` router.
- Mount method services and sub-resource routes only.
- Export route type.

Do not:

- Put business logic, repository calls, or response mapping here.

Example pattern:

```ts
const resourceRoute = new Hono<AppEnv>()
resourceRoute.route('/', getRoute)
resourceRoute.route('/', postRoute)
resourceRoute.route('/', patchRoute)
resourceRoute.route('/', deleteRoute)
resourceRoute.route('/:slug/members', memberRoute)
```

### `services/*` (request orchestration)

Responsibilities:

- Declare HTTP method handlers (`get.ts`, `post.ts`, `patch.ts`, `delete.ts`).
- Apply middleware (`authMiddleware`, cache, validators).
- Parse validated input from `c.req.valid(...)`.
- Coordinate service flow (permission checks, repository calls, cache invalidation).
- Return final API response envelope.

Do not:

- Write SQL or Drizzle query builders directly.
- Duplicate reusable parsing/normalization logic that belongs in `lib/`.
- Accept raw user input without `customValidator`.

### `repositories/*` (data access only)

Responsibilities:

- Execute Drizzle queries only.
- Return DB/domain objects needed by services.
- Keep methods focused and single-purpose.

Do not:

- Throw HTTP errors directly for route semantics.
- Read request context (`c`, headers, params).
- Access the database without `runWithRLS` for tenant-scoped data.

### `lib/*` (resource-scoped helpers)

Responsibilities:

- Resource-specific helper logic used by services/repositories.
- Identifier resolution (`resolve*`), normalizers, mappers, derived fields.

Do not:

- Access request context directly unless helper explicitly accepts required values as args.

### `types/*` (contract definitions)

Responsibilities:

- Input/output/query/filter schemas and inferred types.
- Shared per-resource literals and unions.

Do not:

- Mix route handler code into `types` files.

## Required Endpoint Conventions

### 1) Validation at route boundary

Every request input must be validated with `customValidator` + Zod schema:

- `json` body -> `types/create.ts` or `types/update.ts`
- `query` -> `types/query.ts`
- `param` -> `types/query.ts` / `types/update.ts`

Example:

```ts
customValidator('json', CreateResourceRequestBodySchema)
customValidator('param', ResourceDetailParamsDTOSchema)
```

### 2) Auth before protected handlers

- Use `authMiddleware` for protected endpoints before validators/handler logic.
- Use `optionalAuthMiddleware` for endpoints that read public data but optionally personalize for authenticated users.

### 3) Route params and identifier priority

- Prefer `slug` first when available.
- Prefer actual domain/public identifiers such as `code`, `symbol`, or `number` next.
- Treat primitive `id` as the last fallback only when the user explicitly requests UUID-only behavior or no domain/public identifier exists.
- Prefer actual identifier names (`slug`, `code`, `symbol`, `number`) when known.
- Use `lookup` or `<resource>Lookup` for mutation/detail lookup values that can accept slug/code/id fallback.
- Avoid generic names that conflict with framework primitives like `key` and `ref`.
- Avoid `*Id` unless the value is guaranteed to be an internal UUID/DB id.
- For fields that can represent global/org scope, prefer purpose names like `organizationScope`.
- Apply identifier priority to BE route params, variable names, service/repository args, schemas, API response/integration naming, cache keys, and API test collections.
- Use names like `:slug`, `:code`, `:number`, or the actual domain identifier; avoid making primitive `id` the public route contract.
- Backend may resolve the public identifier to an internal UUID server-side for mutations, version checks, uniqueness checks, deletes, restores, and cache invalidation.
- Keep the public API contract domain identifier first even when internal mutation logic uses the resolved UUID.
- Keep parameter naming consistent across `types/query.ts`, services, repositories, FE contracts, test collections, and cache helpers.

### 3a) Identifier priority across the API/form contract

The same priority rule governs the backend request body schema AND the frontend form that submits to it. If the backend defines a field as text, the form emits text; if the backend defines a field as a database id, the form emits the database id. Mismatches cause 400s and force refactors on both sides. Inspect the backend request body schema before changing a form field.

| Backend schema says | Frontend form sends | What to display to the user | React `key` prop |
| --- | --- | --- | --- |
| `z.string()` (text) | the text value | a label | the unique id |
| `z.uuid()` (database id) | the database id | a label | the unique id |

Common mistake: sending a database id when the request body schema expects text. Sending text when the schema expects a database id. Either breaks the contract.

Common mistake: keying dropdown options on a non-unique field. Two records can share the same display value. React emits duplicate-key warnings and the user can pick the wrong row. Always key on a field that is unique across records.

For the FE-side companion rule, see the FE CRUD boilerplate docs (`overview.md` → Identifier Rules, `input-fields.md` → Select Field).

### 4) Response envelope shape

Use stable response shapes:

- Detail/create/update: `{ data: ... }`
- List: `{ data: [...], meta: { page, limit, total, totalPages } }` when paginated
- Delete: `{ data: null, message: '...' }` (preferred convention)

Do not mix envelope shapes for equivalent endpoints.

### 5) Error handling

Use centralized error classes from your project's error module (e.g., `lib/errors`):

- `BadRequestError`
- `UnauthorizedError`
- `ForbiddenError`
- `NotFoundError`
- `ConflictError`

Rules:

- Throw explicit errors in services when business conditions fail.
- Never return HTTP 200 for failures.
- Keep not-found wording resource-specific (`new NotFoundError('Outlet')`).

### 6) Timing instrumentation

For non-trivial operations use Hono timing markers:

```ts
startTime(c, 'resource-action')
// operation
endTime(c, 'resource-action')
```

Use descriptive names (`outlet-list-query`, `resource-update`).

### 7) Cache strategy for GET + invalidation on mutation

When caching is enabled for a resource:

- `GET /resource` should use versioned list cache keys by user scope.
- `GET /resource/:slug` / `:code` / `:number` / actual domain identifier should use detail cache keys scoped by user.
- `POST/PATCH/DELETE` should bump list cache version before responding.
- Detail cache invalidation can run in `waitUntil` when non-blocking.

### 8) RLS and permissions

Rules:

- Execute DB work via `runWithRLS` from context for tenant/user scoping.
- Distinguish authentication vs authorization checks.
- Verify ownership/member role before update/delete-sensitive operations.

### 9) Query/filter/sort standards

Collection endpoints should support bounded reads:

- `page` (>=1)
- `limit` (>=1, <=100)
- `search` (optional)
- `sortBy` and `sortDirection`
- Optional date filters (`createdAfter`, `createdBefore`, etc.) if needed

`types/filter.ts` should hold reusable `SortBySchema` and `SortDirectionSchema`.

## Endpoint Family Patterns

The codebase implements several endpoint families. Each one has its own conventions on top of the canonical structure.

### Standard CRUD (e.g. tenant-owned editable resources)

Use when the resource maps to a primary table users own and edit.

Required layout:

- Services split per HTTP verb: `get.ts`, `post.ts`, `patch.ts`, `delete.ts`.
- Detail endpoints accept `:slug`, `:code`, `:number`, or another domain/public identifier shape; primitive `:id` is last fallback only.
- List endpoints implement pagination + filter + sort.
- Mutations bump list cache version and invalidate detail cache.

Do:

- Keep response field names stable across list/detail/create/update.
- Soft-delete via `deletedAt`.
- Use ownership/membership check before update/delete.

Don’t:

- Hard-delete tenant-owned rows.
- Treat primitive `id` as the primary public contract when `slug`, `code`, or another domain identifier exists.
- Use the raw path param directly in update/delete/restore SQL after resolving by slug/code; always mutate using the resolved record internally.

### Master Data with Ownership Scope (global + tenant/user scoped resources)

Use when a resource has both global (system-owned) rows and per-user/per-outlet rows.

Required layout:

- Distinguish `createdBy: null` (master) from user-owned rows.
- Read endpoints expose union of global + scoped data.
- Repositories must guard global rows from any mutation.

Do:

- Add explicit guards at repository entry points to refuse update/delete on `createdBy: null`.
- Reject creation of duplicate names within a single scope.

Don’t:

- Allow PATCH/DELETE to proceed on master rows even with admin claims unless explicitly designed.

### Read-Only Lookups (static selector/dropdown resources)

Use when data is largely static and primarily used for selects/dropdowns.

Required layout:

- Services contain only `get.ts`.
- No mutation routes mounted.
- Public or `optionalAuthMiddleware` is acceptable.

Do:

- Cache aggressively when payload is large.
- Keep response shape stable so FE selectors can rely on it.

Don’t:

- Add mutation endpoints to lookup routes; create a separate management route module instead.

### Sub-Resource Routes (e.g. `/{parent}/:slug/{child}`)

Use when a resource only makes sense scoped under a parent resource.

Required layout:

- Sub-route mounted from parent `route.ts`.
- Sub-route uses parent identifier from path before continuing its own validation.
- Reuse parent identifier resolver (`lib/resolve{Parent}.ts`).

Do:

- Always re-check membership/ownership of the parent inside sub-route services.
- Bump parent list cache when sub-resource changes affect parent visibility/state.

Don’t:

- Embed sub-resource logic into parent service files.
- Allow sub-resource access without verifying parent context.

### Action-Oriented Endpoints (workflow/state-transition actions beyond CRUD)

Use when a workflow mutates multiple tables atomically and goes beyond CRUD verbs.

Required layout:

- One service file per action verb (`processPayment.ts`, `processRefund.ts`).
- Service composes repositories + side effects (cache, ledger, notification).
- DB writes that must be atomic should run inside a transaction wrapper.

Do:

- Keep action verbs in URL path only when CRUD verbs cannot express the operation.
- Validate state transitions explicitly (e.g. cannot refund a transaction that is unpaid).

Don’t:

- Bundle unrelated state transitions into the same endpoint.
- Skip idempotency considerations for financially impactful actions.

### Auth Endpoints (e.g. `auth/sign-in`, `auth/sign-up`, `auth/refresh`, `auth/oauth`)

Use when interacting with Supabase Auth or session lifecycle.

Required layout:

- One service file per flow (`signIn.ts`, `signUp.ts`, `refresh.ts`, `oauth.ts`, `me.ts`).
- HttpOnly cookies set via `setSessionCookies`.
- Status code semantics: `200` sign-in/refresh, `201` sign-up, `200` sign-out (with optional message).

Do:

- Strip secrets from any returned payload.
- Treat token refresh as best-effort; surface explicit errors when failures change session state.

Don’t:

- Echo raw access tokens in non-cookie responses without an explicit reason.
- Mix sign-up and sign-in branching inside a single handler.

### Financial Ledger Endpoints (money/points equivalent state changes)

Use when operations record money- or points-equivalent state changes.

Required layout:

- Strict transaction boundaries in repositories.
- Hold/capture/release pattern when applicable.
- Mutation history retained; no destructive updates.

Do:

- Validate balance/eligibility before write.
- Add idempotency keys when retries are possible.

Don’t:

- Mutate ledger rows without explicit reason recorded.
- Allow negative balance unless contract permits it.

### Upload Endpoints (binary ingestion endpoints)

Use when accepting binary content.

Required layout:

- `services/post.ts` only.
- Accept `multipart/form-data` and stream to storage.
- Return canonical URL or storage reference.

Do:

- Validate mime/type, size, and optional checksum.
- Authenticate uploader before storage call.

Don’t:

- Persist file metadata into business tables in the same handler unless contract demands it.

### Webhook/External Callback Endpoints (e.g. payment provider callbacks if added later)

Required layout:

- Verify provider signature first.
- Idempotent handler keyed by event id.

Do:

- Log raw payload for forensic review.

Don’t:

- Trust event payload without verifying signature.

## Advanced Integrity Standards

These standards apply when endpoints have higher risk (financial impact, high write concurrency, external integrations, compliance-sensitive data) and should be the default for new critical endpoints.

### Concurrency Control (Read-Modify-Write)

Do:

- Require optimistic locking for mutable critical resources (`version`, `updatedAt`, or `If-Match`/ETag).
- Return `409 Conflict` or `412 Precondition Failed` when client writes stale data.

Don’t:

- Overwrite concurrent updates silently.

### Idempotency and Retries

Do:

- Require `Idempotency-Key` for non-idempotent `POST` actions (payments, ledger writes, invitations, side-effectful workflows).
- Persist key + response hash for a defined replay window.

Don’t:

- Execute duplicate writes when client/network retries the same request.

### PATCH Semantics (`null` vs omitted)

Do:

- Define per-field update semantics explicitly:
  - omitted => no change
  - `null` => clear value (only for nullable fields)
- Implement body mappers/normalizers in services/lib for explicit intent.

Don’t:

- Treat empty string, `null`, and omitted as equivalent without contract definition.

### Soft-Delete Integrity

Do:

- Define cascade rules for each parent resource (what is soft-deleted/restored with parent).
- Use partial unique indexes for active rows (`... where deleted_at is null`).

Don’t:

- Leave cascading behavior implicit.
- Break uniqueness checks due to mixed active/deleted rows.

### DTO Projection and Redaction

Do:

- Map repository objects into explicit API DTOs in services.
- Redact internal/sensitive columns before responding.

Don’t:

- Return raw DB rows directly to API clients.

### Authorization Matrix

Do:

- Document endpoint-level access matrix (public, authenticated, member, owner, admin).
- Enforce authorization server-side regardless of UI restrictions.

Don’t:

- Assume hidden frontend controls are sufficient authorization.

### State Machine and Transition Rules

Do:

- Define allowed status transitions for stateful resources.
- Return explicit errors for invalid transitions.

Don’t:

- Allow arbitrary status changes that bypass workflow constraints.

### Pagination Stability

Do:

- Use deterministic sort + tie-breaker (e.g. `createdAt`, then `id`) for list endpoints.
- Prefer cursor pagination for high-churn datasets; use offset pagination for simpler low-churn lists.

Don’t:

- Return unstable page ordering that causes duplicate/missing records across pages.

### Bulk Operation Policy

Do:

- Define max batch size, transaction behavior, and partial-failure response shape.
- Return per-item status when partial success is allowed.

Don’t:

- Run unbounded bulk writes in a single request.

### Observability and Correlation

Do:

- Accept/propagate `X-Correlation-ID` (or generate when missing).
- Use structured logs with endpoint, actor, tenant scope, and action outcome.
- Record audit trails for sensitive writes.

Don’t:

- Log sensitive payload fields in plaintext.

### Rate Limiting and Abuse Protection

Do:

- Set endpoint-class rate limits (auth, uploads, webhooks, high-cost reads, financial writes).
- Define response behavior for throttling (`429`).

Don’t:

- Use one global throttle policy for all endpoint classes.

### External Callback/Webhook Security

Do:

- Verify signatures and replay windows.
- Process events idempotently using external event ids.

Don’t:

- Trust callback payloads without cryptographic verification.

### Data Privacy and Retention

Do:

- Define anonymize-vs-delete policy per resource.
- Document retention periods and purge mechanisms for PII-bearing data.

Don’t:

- Retain personally identifiable data indefinitely without policy.

### Versioning and Deprecation Lifecycle

Do:

- Define what constitutes a breaking change.
- Use versioned route groups for breaking contract changes.
- Provide deprecation windows and migration notes.

Don’t:

- Ship breaking response contract changes silently.

### Contract Governance

Do:

- Keep OpenAPI or equivalent contract docs in sync with implementation.
- Add CI checks for schema/contract drift on changed endpoints.

Don’t:

- Treat API docs as optional after implementation.

### Performance Budgets and SLOs

Do:

- Define target latency/error budgets per endpoint family.
- Monitor query count and prevent N+1 patterns in repositories.

Don’t:

- Wait for production incidents to define endpoint performance expectations.

### Incident and Rollback Readiness

Do:

- Define rollback strategy and feature kill-switch for risky endpoint changes.
- Document safe fallback behavior under dependency outages.

Don’t:

- Release high-risk endpoint changes without rollback planning.

## Naming Standards

- Schemas: `CreateXRequestBodySchema`, `UpdateXRequestBodySchema`, `XListQueryDTOSchema`
- Types: `CreateXRequestBody`, `XListQueryDTO`
- Services routers: `getRoute`, `postRoute`, `patchRoute`, `deleteRoute`
- Repositories: verb-first single-purpose names (`findXList`, `findXById`, `updateX`, `deleteX`)

## Definition of Done (Code Pattern)

Before merging a new endpoint implementation:

- [ ] `route.ts` is mount-only
- [ ] method-specific logic is in `services/*`
- [ ] DB access only in `repositories/*`
- [ ] all params/body/query validated with Zod + `customValidator`
- [ ] auth + authorization checks implemented in services
- [ ] public route contracts prioritize `slug`/`code`/domain identifiers before primitive `id`
- [ ] request body schemas match the public identifier the FE form will emit (`z.string()` for human-readable fields like `unit`, `z.uuid()` only when the FE resolves to an internal UUID) — coordinate with FE before changing either side
- [ ] multi-identifier mutations resolve once and use the resolved record internally
- [ ] response envelope follows project convention
- [ ] error classes from `lib/errors` used
- [ ] list/detail cache + mutation invalidation strategy implemented if cache is used
- [ ] Typecheck command passes
- [ ] API test collection (e.g., Bruno, Hoppscotch, Postman) updated for new/changed endpoint behavior

## Testing Requirements

For every new or changed endpoint:

1. Update or add API test requests in your project's API test collection (e.g., Bruno, Hoppscotch, Postman).
2. Cover success + failure cases:
   - unauthorized
   - forbidden
   - validation failure
   - not found
3. Run the scoped API test flow locally and record result in the related plan task.

## Anti-Patterns to Avoid

- Fat `route.ts` files with repository calls inline.
- Inconsistent param names (`id` in path, `idOrSlug` in schema, mixed usage).
- Prioritizing primitive `id` while `slug`, `code`, or another domain identifier exists.
- Picking a more complex identifier type than the use case needs. Use text when the schema is for human-readable input; use a database id only when the operation specifically requires server-side record lookup.
- Resolving by slug/code, then using the raw path param for mutation SQL, uniqueness exclusion, version checks, restore, or delete.
- Unbounded list endpoints without pagination support.
- Multiple response envelope styles for same resource family.
- Business logic hidden in repositories.
- Silent fallback behavior that masks contract bugs.
- Mutating master/global rows by accident through generic repository methods.
- Calling repositories directly from handlers without going through the service layer.
- Returning raw DB rows without intentional response shaping.

## Reference Implementation

Use standard resource modules (e.g., `routes/{resource}/`) as the primary reference for endpoint code pattern implementation. Cross-check implementation across different endpoint families (Master Data, Sub-Resource, Action-Oriented, Ledger, Upload) when building endpoints with those specific concerns.
