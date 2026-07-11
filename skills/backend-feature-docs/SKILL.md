---
name: backend-feature-docs
description: Documents backend feature implementation by analyzing routes, handlers, services, schemas, database models, migrations, permissions, jobs, events, and tests. Use when asked to create or update backend feature docs, map how an API/backend feature works, or produce implementation documentation from existing backend code.
---

# Backend Feature Docs

## Goal

Create accurate, actionable documentation grounded in the codebase. Preserve details that help engineers modify, test, debug, operate, or extend the feature.

## Workflow

1. Identify scope
   - Determine feature name, API boundary, route group, domain module, job, event flow, or database-backed capability.
   - If scope is ambiguous, ask one focused clarification question.

2. Inspect implementation
   - Find routes/controllers, handlers, middleware, services, repositories, schemas, types, database tables, migrations, queues/jobs, events, integrations, and tests.
   - Trace request or trigger flow from entry point through validation, auth, business logic, persistence, side effects, response, and error handling.
   - Note dependencies, environment variables by name only, feature flags, tenant scoping, permissions, transactions, idempotency, retries, observability, and external services.

3. Verify claims
   - Cite code references as `file_path:line_number`.
   - Do not infer behavior not supported by code.
   - Mark uncertain, missing, or unverified behavior explicitly.
   - Do not expose secrets, credentials, private tokens, or environment-specific IDs.

4. Produce documentation using this structure:

```md
# [Feature Name]

## Overview

Brief purpose, business outcome, and backend responsibilities.

## Entry Points

- HTTP routes, RPC methods, webhooks, jobs, queue consumers, cron tasks, event listeners, or CLI commands that start the feature.
- Include method/path/event names and code references.

## Request, Trigger, or Input Contract

Document parameters, body/query/path fields, headers, event payloads, validation rules, defaults, and required/optional fields.

## Response or Output Contract

Document status codes, response shape, emitted events, queued jobs, persisted records, returned errors, and side effects.

## Implementation Map

- Routes/controllers/handlers
- Middleware/auth/guards
- Services/domain logic
- Repositories/data access
- Schemas/types/validators
- Database tables/migrations
- Jobs/queues/events/webhooks
- External integrations
- Tests

## Execution Flow

1. Step-by-step flow from entry point to completion.
2. Include alternate paths for validation failure, auth failure, missing data, conflicts, retries, rollback, and external service failure.

## Data Model and Persistence

Document tables, columns, relations, indexes, constraints, transactions, soft deletes, audit fields, and tenant/org scoping.

## Authorization and Security

Document authentication, roles/permissions, ownership checks, RLS/policies when present, rate limits, input sanitization, secret handling, and security gaps. State `None found` only after inspection.

## Errors and Edge Cases

List error codes/messages, exception mapping, retries, idempotency behavior, concurrency handling, fallbacks, and known gaps.

## Observability and Operations

Document logs, metrics, traces, alerts, background job visibility, configuration/env vars by name only, and operational risks.

## Testing Notes

Document existing unit, integration, contract, migration, or API collection coverage and gaps. Include commands only if already present in project docs/scripts.

## Known Gaps or Risks

List missing coverage, unclear behavior, edge cases, migration risks, data consistency risks, or code smells found during analysis. Do not invent future work.
```

## Rules

- Keep docs concise but complete enough for another engineer to safely change the feature.
- Prefer bullets, tables, contracts, and code references over prose.
- Never hardcode or print secrets, credentials, environment-specific IDs, private URLs, tokens, database URLs, or keys.
- Do not add unrelated docs or files.
- If creating a file, create only the requested doc unless the user asks for more.
- If editing existing docs, preserve local style and headings where practical.
