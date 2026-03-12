# Frontend Test Setup And Migration Reference

## Migration Checklist

### Assess

- Package manager and workspace layout
- Frontend framework and bundler
- Existing test runner and assertions
- Current DOM env setup
- Existing mock strategy
- CI commands and coverage expectations

### Choose target defaults

- Runner: `vitest`
- Component tests: `@testing-library/react` or framework equivalent
- Interactions: `@testing-library/user-event`
- Network mocks: `msw`
- E2E: `@playwright/test`
- Matchers: `@testing-library/jest-dom`

## Typical Files

- `vitest.config.ts`
- `src/test/setup.ts`
- `src/test/msw/server.ts`
- `src/test/msw/handlers.ts`
- `playwright.config.ts`
- `package.json` scripts

## Suggested Script Shape

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "typecheck": "tsc --noEmit"
  }
}
```

## Suggested Vitest Setup

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
  },
});
```

```ts
import '@testing-library/jest-dom/vitest';
```

## Rollout Strategy

### Greenfield

- Add runner, setup, scripts
- Add one component integration test
- Add MSW only if API-backed UI exists now

### Partial setup

- Reuse working pieces
- Normalize helpers and setup locations
- Upgrade weakest patterns first

### Jest to Vitest migration

- Port config with smallest behavior change
- Replace Jest globals and mocks carefully
- Migrate custom matchers and setup imports
- Run focused suites during conversion

### Brittle suite cleanup

- Delete low-signal snapshots
- Remove implementation-detail assertions
- Replace `fireEvent` with `userEvent` where realistic
- Collapse duplicate render helpers

## Verification

- `typecheck` passes
- Representative component tests pass
- MSW setup works in success and error states
- E2E command isolated from unit/integration command
- CI uses current commands only
- Team-facing docs mention preferred style
