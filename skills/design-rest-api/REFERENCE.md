# API Design Reference

Derived from `docs/api-design-guidelines.md` in `qlover-laundry-be`.

## Goal

Design APIs for consistency, predictability, scalability, security, and developer experience. Prefer stable contracts over ad hoc JSON responses.

## Core Rules

### Resource-First URLs

- Use nouns, not verbs
- Let HTTP methods express actions
- Good: `/users`, `/users/:id`, `/outlets/:outletId/members`
- Bad: `/getUsers`, `/createUser`, `/deleteUser/:id`

### HTTP Methods

- `GET` read
- `POST` create
- `PUT` full resource replacement
- `PATCH` partial update
- `DELETE` delete
- Prefer `PATCH` for subset updates
- Use `PUT` only when body represents full updatable resource

### Status Codes

- `200` successful read/update
- `201` created
- `200` delete success with lightweight message body is preferred when frontend needs confirmation text
- `204` deleted/no content is acceptable when no response body is needed
- `400` malformed request
- `401` unauthenticated
- `403` authenticated but not allowed
- `404` resource not found
- `409` conflict
- `422` validation failure
- `500` unexpected server error
- Do not return `200` for failures

### Response Shape Consistency

- Keep one naming convention everywhere
- Prefer `camelCase` in this codebase unless existing contract differs
- Keep success response structure predictable across related endpoints
- Keep field names stable across list/detail/create/update responses
- For delete endpoints, prefer `200` with a lightweight body such as `{ "message": "Article type deleted" }`
- Use `204` only when endpoint intentionally returns no body

### Versioning

- Version public/shared APIs from the start
- Prefer `/api/v1/...`
- Breaking changes go in new versions

### Pagination

- Never return unbounded collections
- Support `page`, `limit`, or cursor equivalents
- Return `data` plus `meta`

Example:

```json
{
  "data": [{ "id": "usr_1", "fullName": "Ada Lovelace" }],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 125
  }
}
```

### Filtering And Sorting

- Use query parameters
- Example: `/users?status=active&sort=name`
- Do not create endpoint variants for every combination
- Validate allowed filter/sort fields

### Authentication vs Authorization

- Authentication = who caller is
- Authorization = what caller can do
- Enforce both on server
- Never trust frontend-only restrictions
- Follow Supabase Auth + role/member authorization patterns in this repo

### Error Contract

- Errors must be structured and predictable
- Include machine-usable code and human message
- Keep top-level shape stable

Example:

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

### Security Baseline

- Validate every request with Zod
- Enforce authorization in handlers/services
- Never expose passwords, tokens, secrets, or internal-only metadata
- Assume HTTPS in deployed environments
- Add rate limiting or abuse protection where relevant
- Treat security as part of design from day one

### Design For Clients, Not Tables

- API responses are contracts, not raw DB dumps
- Do not mirror Drizzle schema blindly
- Shape responses around client use cases
- Hide internal columns
- Transform/combine fields when useful
- Internal schema may change; external contract should stay intentional

### Hono Repository Patterns

- Thin route handlers
- Validate at route boundary
- Business logic in services
- Data access in repositories
- Explicit response mapping when DB shape differs from API contract

## Done Checklist

- Resource-based path
- Correct HTTP method
- Correct status codes
- Consistent response shape
- Structured error shape
- Validation with Zod
- Authn/authz enforced
- Pagination/filter/sort for collections
- No sensitive fields leaked
- Contract designed for clients, not tables
