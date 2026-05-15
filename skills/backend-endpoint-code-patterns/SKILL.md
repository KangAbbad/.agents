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
- **RLS Enforcement**: Always execute database work through `runWithRLS`.

## Definition of Done (Code Pattern)

- [ ] `route.ts` is mount-only.
- [ ] Method-specific logic is in `services/*`.
- [ ] DB access only in `repositories/*`.
- [ ] All inputs validated with Zod + `customValidator`.
- [ ] Auth + authorization enforced in services.
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
