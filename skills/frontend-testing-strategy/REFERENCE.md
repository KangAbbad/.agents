# Frontend Testing Strategy Reference

## Testing Trophy

Choose layers in this order:

1. Static analysis
2. Integration tests
3. Unit tests
4. E2E tests

Goal: maximize confidence per line of test code.

## Layer Selection Guide

### Static analysis

Use when failure should never reach runtime.

- TypeScript strict mode
- ESLint safety rules
- Exhaustiveness checks
- Contract typing

Good targets:

- Impossible states
- Missing null checks
- Unsafe casts
- Broken prop or schema usage

### Integration tests

Default layer for frontend behavior.

Use when test needs rendered UI, routing, providers, forms, async state, or network responses.

Good targets:

- Search flows
- Form validation and submit
- Loading, empty, success, error states
- Permission-gated UI
- Cache and refetch behavior

### Unit tests

Reserve for deterministic logic.

Good targets:

- Formatters
- Parsers
- Reducers
- Pricing rules
- Custom hooks with meaningful behavior

Avoid for:

- Basic rendering
- Framework defaults
- Thin wrappers around libraries

### E2E tests

Use for most expensive regressions.

Good targets:

- Auth
- Checkout
- Booking
- Onboarding
- Payment flows
- Cross-page navigation flows

Keep set small. Treat flakiness as suite bug.

## Default Tooling

| Need | Default |
| --- | --- |
| Unit + integration | Vitest |
| DOM environment | jsdom or happy-dom |
| Queries | Testing Library |
| Interactions | user-event |
| API mocking | MSW |
| E2E + screenshots | Playwright |
| Static analysis | TypeScript strict mode + ESLint |

## Keep vs Avoid

Keep:

- User-visible behavior
- Data transformations and business logic
- Hook behavior with meaningful state transitions
- API contract handling through MSW
- Critical browser journeys

Avoid or delete:

- Private state assertions
- Over-mocked component tests
- Broad snapshot suites
- CSS class assertions without product contract
- Tests for third-party internals
- Tests for constants and trivial pass-through code

## Agent Guidance

1. Inspect current test stack before adding tools.
2. Match repo conventions unless clearly brittle.
3. Explain why chosen layer fits requested change.
4. Prefer smallest realistic setup that proves behavior.
5. Replace shallow or flaky tests instead of stacking more on top.

## Verification Checklist

- Types and lint catch obvious invalid states
- Tests use semantic queries where possible
- Async assertions wait for visible outcomes
- Network mocks live at request boundary
- E2E coverage stays limited to high-value journeys
- New tests avoid implementation details unless unavoidable
