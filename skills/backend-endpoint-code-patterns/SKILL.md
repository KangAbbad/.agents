---
name: backend-endpoint-code-patterns
description: Provides standardization guidelines for backend Hono endpoint development. Use when creating new API endpoints, refactoring existing routes, or reviewing endpoint implementation against architectural patterns.
---

# Backend Endpoint Code Pattern Guidelines

## Core Principles

- **Single Implementation Pattern**: Enforce Route -> Service -> Repository -> Type structure.
- **Route handlers (`route.ts`) are mount-only**: They should never contain business logic, repository calls, or response mapping.
- **Service layer (`services/`) orchestrates**: Handles auth, validation, cache, and business rules.
- **Repositories (`repositories/`) access DB**: Strict isolation of Drizzle queries.
- **DTO Projection**: Services must map DB objects to stable API DTOs; do not expose internal DB models.
- **Identifier Priority**: Prefer `slug` first, then actual domain/public identifiers such as `code`, `symbol`, or `number`, then primitive `id` only as the last fallback when explicitly requested or no other identifier exists. Prefer actual identifier names (`slug`, `code`, `symbol`, `number`) when known. Use `lookup` or `<resource>Lookup` for mutation/detail lookup values that accept slug/code/id fallback. Avoid generic names that conflict with framework primitives like `key` and `ref`. Avoid `*Id` unless the value is guaranteed to be an internal UUID/DB id. For fields that can represent global/org scope, prefer purpose names like `organizationScope`. Apply this to route params, variables, schemas, services, repositories, test collections, cache keys, and response/API integration naming. Backend may resolve the public identifier to an internal UUID server-side, but the public contract stays domain identifier first.
  - **This rule applies to BOTH the API contract (request/response schemas) AND the form contract.** The contract on the API side and the form side must match. When the backend defines a field as text, the form sends text. When the backend defines it as a database id, the form sends the database id. Pick the identifier type based on what the request body schema actually requires, not on what the resource table has columns for. See the FE CRUD boilerplate docs (`overview.md` → Identifier Rules, `input-fields.md` → Select Field) for the FE-side companion rule.
- **Environment Configuration**: Do not hardcode environment/config variables such as `ALLOWED_ORIGINS`; store them in `.env.local`, `.dev.vars`, or the appropriate environment file.
- **Secrets**: Never hardcode secrets.
- **RLS Enforcement**: Always execute database work through `runWithRLS`.

## Definition of Done (Code Pattern)

- [ ] `route.ts` is mount-only.
- [ ] Method-specific logic is in `services/*`.
- [ ] DB access only in `repositories/*`.
- [ ] All inputs validated with Zod + `customValidator`.
- [ ] Auth + authorization enforced in services.
- [ ] Route params, variables, service/repository args, resolvers, response/API integration naming, test collections, and cache keys use `slug` first, then actual domain/public identifiers like `code`/`symbol`/`number`, then primitive `id` only as last fallback or when explicitly requested. Fallback lookup values use `lookup` or `<resource>Lookup`; `*Id` means guaranteed internal UUID/DB id only.
- [ ] Request body schemas on the backend match the type of identifier the frontend sends. Coordinate before changing either side.
- [ ] If server-side mutation needs an internal UUID, resolve from the public identifier once, then use the resolved record internally for mutation/version/uniqueness logic without changing the public contract.
- [ ] Response envelope follows stable API shape (`{ data, meta }` or `{ data, message }`).
- [ ] Error classes from `lib/errors` used.
- [ ] List/detail cache invalidation implemented if cache is active.
- [ ] API test collection (e.g., Bruno, Hoppscotch, Postman) updated for verification.

## Advanced Integrity Standards

For enterprise-grade API design:

- **Optimistic Locking**: Use `If-Match`/`version` for concurrent updates.
- **Idempotency**: Use `Idempotency-Key` for financial mutations.
- **Soft-Delete Uniqueness**: Use partial indexes `where deleted_at is null` for unique constraints.
- **Tracing**: Pass `X-Correlation-ID` throughout service chain for logs.
- **Schema Evolution**: Handle `PATCH` semantics (omit vs null) explicitly in converters.

## Endpoint Families

- **CRUD**: Split by verb (`get.ts`, `post.ts`, `patch.ts`, `delete.ts`).
- **Master Data**: Guard global records (`createdBy: null`) from mutations.
- **Sub-Resources**: Reuse parent identifier resolution logic.
- **Actions**: Atomic workflow mutations outside standard CRUD.
- **Auth**: Lifecycle managed via HttpOnly cookies.

## Full Reference

Use [REFERENCE.md](REFERENCE.md) as the source of truth for complete endpoint conventions, endpoint family patterns, integrity standards, testing requirements, anti-patterns, and reference implementation guidance.
