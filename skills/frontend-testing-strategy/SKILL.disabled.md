---
name: frontend-testing-strategy
description: Frontend-first testing strategy for JavaScript and TypeScript apps using testing trophy principles, Vitest, Testing Library, MSW, and Playwright. Use when designing test strategy, writing frontend tests, choosing between unit/integration/E2E coverage, or replacing brittle implementation-detail tests.
risk: unknown
source: community
date_added: "2026-02-27"
---

# Frontend Testing Strategy

Use this skill to build frontend test suites that maximize confidence per line of test code.

## Use this skill when

- Planning test coverage for React or other browser-based UI
- Writing or refactoring Vitest, Testing Library, MSW, or Playwright tests
- Deciding whether a scenario should be static analysis, integration, unit, or E2E
- Replacing brittle snapshots or implementation-detail assertions

## Core strategy

- Follow testing trophy, not test pyramid
- Make static analysis first line: TypeScript strict mode, lint rules, CI checks
- Make integration tests largest layer: render real components, real providers, realistic network mocks, real user interactions
- Keep unit tests for pure logic, transformations, pricing rules, parsers, custom hooks
- Keep E2E suite small: only critical business journeys and cross-page behavior
- Optimize for confidence, realism, speed, maintainability

## Default tool stack

- `Vitest` for unit and integration
- `@testing-library/*` plus `user-event` for behavior-driven component tests
- `MSW` for network-level API mocking
- `Playwright` for E2E and visual regression
- `TypeScript` strict mode plus `ESLint` for static analysis

## Workflow

1. Classify each requested test:
   - Static analysis issue -> improve types, schemas, lint, narrowing
   - Pure logic or hook -> unit test
   - UI flow within rendered app surface -> integration test
   - Multi-page, auth, checkout, booking, onboarding, payments -> E2E
2. Prefer behavioral assertions:
   - Query by role, label, text, placeholder
   - Use `userEvent`, not internal method calls
   - Assert visible outcomes, requests, state transitions user can observe
3. Mock at seams, not internals:
   - Prefer MSW over mocking `fetch` directly
   - Avoid mocking React internals, component state, third-party library behavior
4. Keep suite lean:
   - Delete redundant tests
   - Avoid broad snapshot coverage
   - Treat flaky tests as defects

## Rules of thumb

- Test behavior, not implementation details
- Prefer one valuable integration test over several shallow unit tests
- Use `data-testid` only when semantic queries fail
- Avoid assertions on CSS classes unless style itself is contract; use screenshot coverage for visual regressions
- Do not chase coverage numbers at expense of signal

## Deliverables

- Recommend test mix before writing code when strategy unclear
- Explain why each test belongs in its layer
- Add minimal setup needed for Vitest, Testing Library, MSW, Playwright
- End with verification steps and any gaps not covered

## References

- Open `REFERENCE.md` for decision guides, tooling defaults, anti-patterns, and verification checklist
- Open `EXAMPLES.md` for integration, unit, hook, MSW, E2E, and setup templates
