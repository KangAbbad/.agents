---
name: design-rest-api
description: Designs and reviews REST API contracts for consistency, predictability, security, and developer experience. Use when designing new endpoints, reviewing route handlers, refactoring API contracts, or checking REST conventions in Hono, TypeScript, or backend services.
---

# Design REST API Skill

## Quick Start

Use this skill to design or review API endpoints against repository API rules.

1. Identify resource and client use case
2. Choose resource-first path and HTTP method
3. Define request schema and response contract
4. Check status codes, errors, authn/authz, security
5. Add pagination/filter/sort for collections
6. Keep contract stable and client-focused

## Workflows

### Designing a New Endpoint

1. Start with noun-based resource path
2. Prefer `/api/v1/...` for shared/public APIs
3. Choose method:
   - `GET` read
   - `POST` create
   - `PUT` full replacement
   - `PATCH` partial update
   - `DELETE` remove
4. Choose delete success contract intentionally:
   - Prefer `200` with lightweight message body when frontend needs server-provided confirmation text
   - Use `204` only when endpoint intentionally returns no body
5. Define Zod request validation at route boundary
6. Define success response shape with consistent field naming
7. Define structured error response
8. Add authn/authz rules
9. Add pagination/filter/sort if endpoint returns collections
10. Confirm response is client-oriented, not raw table dump

### Reviewing an Existing Endpoint

1. Check path for verbs or action-heavy naming
2. Check method semantics match behavior
3. Check status codes for success and failure paths
4. Check payload naming consistency
5. Check error shape consistency
6. Check authn/authz separation
7. Check sensitive fields are excluded
8. Check list endpoints for pagination and query-based filters
9. Check handler/service/repository responsibilities stay separated

## Review Checklist

- [ ] Resource-first path
- [ ] Correct HTTP method
- [ ] `PATCH` for partial update, `PUT` only for full replacement
- [ ] Meaningful status codes
- [ ] Consistent `camelCase` payloads unless existing contract differs
- [ ] Delete response follows repo preference: `200` with lightweight message body unless empty `204` is intentional
- [ ] Structured error contract
- [ ] Zod validation present
- [ ] Authn/authz enforced server-side
- [ ] Pagination/filter/sort for collections
- [ ] No leaked sensitive/internal fields
- [ ] Response shaped for client needs, not database schema

## Output Patterns

When proposing endpoint designs, include:

- Method + path
- Purpose
- Request schema summary
- Success response shape
- Error cases + status codes
- Authn/authz notes
- Pagination/filter/sort rules if applicable

## References

- [API Design Reference](REFERENCE.md)
- Source rules: `/Users/kangabbad/Documents/MyProject/Hono/qlover-laundry-be/docs/api-design-guidelines.md`
