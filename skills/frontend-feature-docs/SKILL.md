---
name: frontend-feature-docs
description: Documents frontend feature implementation by analyzing routes, components, hooks, state, API usage, validation, and user flows. Use when asked to create or update frontend feature docs, map how a UI feature works, or produce implementation documentation from existing frontend code.
---

# Frontend Feature Docs

## Trigger

Use this skill when the user asks to document a frontend feature, explain an implemented UI flow, audit feature implementation, or produce developer-facing docs from frontend code.

## Goal

Create accurate, actionable documentation grounded in the codebase. Preserve implementation details that help engineers modify, test, debug, or extend the feature.

## Workflow

1. Identify scope
   - Determine feature name, route/page, user flow, or component boundary.
   - If scope is ambiguous, ask one focused clarification question.

2. Inspect implementation
   - Find route files, page components, feature components, hooks, stores, schemas, API clients, and tests.
   - Trace data flow from UI events to state updates, API calls, persistence, and rendered output.
   - Note dependencies, feature flags, auth/permissions, forms, validation, loading states, empty states, and error handling.

3. Verify claims
   - Cite code references as `file_path:line_number`.
   - Do not infer behavior not supported by code.
   - Mark uncertain or missing behavior explicitly.

4. Produce documentation using this structure:

```md
# [Feature Name]

## Overview

Brief purpose, target users, and business/user outcome.

## Entry Points

- Routes, navigation links, buttons, or components that expose the feature.
- Include code references.

## User Flow

1. Step-by-step flow from first interaction to completion.
2. Include alternate paths for cancel, error, empty, or permission-denied states.

## Implementation Map

- Pages/routes
- Components
- Hooks/state stores
- API clients/endpoints
- Schemas/types
- Tests

## Data Flow

Describe inputs, local state, server data, mutations, cache invalidation, persistence, and rendered outputs.

## Validation and Error Handling

Document field validation, disabled states, toast/messages, API errors, retries, fallbacks, and known gaps.

## Permissions and Feature Flags

Document auth requirements, role checks, organization/tenant scoping, and feature flags. State `None found` only after inspection.

## UI States

- Loading
- Empty
- Success
- Error
- Disabled/read-only
- Responsive or accessibility-specific behavior when present

## Testing Notes

Document existing unit, integration, browser, or manual test coverage and gaps. Include commands only if already present in project docs/scripts.

## Known Gaps or Risks

List missing coverage, unclear behavior, edge cases, or code smells found during analysis. Do not invent future work.
```

## Rules

- Keep docs concise but complete enough for another engineer to modify the feature.
- Prefer bullets, tables, and code references over prose.
- Do not hardcode secrets, environment-specific IDs, private URLs, tokens, or credentials.
- Do not add unrelated docs or files.
- If creating a file, create only the requested doc unless the user asks for more.
- If editing existing docs, preserve local style and headings where practical.
