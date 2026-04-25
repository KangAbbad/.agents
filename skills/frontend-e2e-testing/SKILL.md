---
name: frontend-e2e-testing
description: Execution-level rules for Playwright end-to-end tests in real applications. Use when writing new Playwright specs, reviewing e2e tests, deciding between mocks and real API calls, setting up test authentication, choosing locators, debugging flaky e2e tests, or designing the colocation/structure of a feature's e2e suite. Complements the higher-level `frontend-testing-strategy` skill — that one decides what to test; this one decides how to write the spec without it being flaky, fake, or destructive.
---

# Frontend E2E Testing (Playwright)

Use this skill when authoring or reviewing Playwright specs. It encodes the rules that prevent the four failure modes E2E suites usually drift into: **flaky timing**, **fake responses that hide real bugs**, **broken auth bootstrap**, and **destructive tests that pollute the dev backend**.

## Use this skill when

- Writing a new `*.spec.ts` for a feature
- Reviewing a teammate's Playwright PR
- Deciding whether to mock an endpoint or hit the real backend
- Setting up authentication that doesn't go through the UI on every test
- Debugging a flaky test or a passing test that hides a real bug
- Choosing between `page.locator(css)` and accessibility queries

## Core principles

1. **Colocate specs with the feature they test.** A spec that exercises one route belongs next to that route, not in a global `e2e/` dump. Top-level e2e folders are reserved for cross-feature flows (auth bootstrap, navigation across unrelated routes).
2. **Real API over mocks.** Mocks fabricate reality and let real bugs slip through. Use Playwright's `page.route` only to **delay** or **abort** real responses, never to substitute fake payloads. Acceptable mocks: third-party services that are unsafe or expensive to call from CI (payments, email).
3. **Auth via cookies, not UI login.** Read the seeded test tokens from `.env.local` (or equivalent) and inject them with `context.addCookies` in `beforeEach`. UI login per test is slow and brittle.
4. **Anchor assertions to network events, not time.** Use `waitForRequest`, `waitForResponse`, or `expect.poll` — never `setTimeout`/`page.waitForTimeout`.
5. **Don't mutate persistent data.** For destructive flows, `route.abort()` after observing the loading state, or create a throwaway resource and clean up.

## Quick start

```ts
import { expect, test, type BrowserContext } from '@playwright/test'
import { readFileSync } from 'node:fs'

const accessToken = readEnvVar('SB_ACCESS_TOKEN_TEST') // helper: fs.readFileSync('.env.local')
const refreshToken = readEnvVar('SB_ACCESS_REFRESH_TEST')

async function applyAuthCookies(context: BrowserContext) {
  await context.addCookies([
    { name: 'sb_access_token', value: accessToken, domain: 'localhost', path: '/', httpOnly: true, sameSite: 'Lax' },
    { name: 'sb_refresh_token', value: refreshToken, domain: 'localhost', path: '/', httpOnly: true, sameSite: 'Lax' },
  ])
}

test.describe('Outlets list', () => {
  test.beforeEach(async ({ context }) => {
    await applyAuthCookies(context)
  })

  test('debounces search and shows a spinner', async ({ page }) => {
    await page.goto('/outlets')
    await expect(page.locator('[data-slot="card"]').first()).toBeVisible()

    const searchRequest = page.waitForRequest((req) => req.url().includes('search='))
    await page.getByPlaceholder('Cari outlet...').fill('zzz')
    await expect(page.locator('[aria-label="Searching"]')).toBeVisible()
    await searchRequest
    await expect(page.locator('[aria-label="Searching"]')).toHaveCount(0)
  })
})
```

## Workflows

### Writing a new spec

1. Determine **scope**: single feature → colocate (`{feature}/e2e/{feature}.spec.ts`). Cross-feature → top-level `e2e/`.
2. **Set up auth** in `beforeEach` via `context.addCookies` (cookie names must match the BE's session cookie config).
3. **Write happy path first** with real API. No `page.route` yet.
4. **Add timing controls only when needed**: a loading state needs to be observable → gate the GET with `route.continue()` after a delay; a destructive mutation should not commit → gate the DELETE/POST with a controlled promise then `route.abort()`.
5. **Anchor every assertion** to a request/response event, never to wall-clock time.
6. **Run the spec under `--workers=2+`** locally to flush out parallel-state bugs before pushing.

### Reviewing an existing spec

Apply the [pre-merge checklist](#pre-merge-checklist). The most common failures are:

- Fabricated `route.fulfill` payloads that should be `route.continue()`.
- `page.waitForTimeout(N)` instead of `waitForRequest`.
- Class-based selectors (`button.bg-destructive`) that match unintended elements (e.g., dialog buttons sharing the same variant) when scoped too broadly.
- Destructive deletes that actually run against the dev backend.

## Locator priority

Prefer accessible queries so refactors don't break tests. Drop down only when the level above can't disambiguate:

| Need                   | Use                                                            |
| ---------------------- | -------------------------------------------------------------- |
| A button by label      | `page.getByRole('button', { name: 'Hapus' })`                  |
| A form field           | `page.getByLabel('Nama *')`                                    |
| A search input         | `page.getByPlaceholder('Cari outlet...')`                      |
| A text content node    | `page.getByText('Tidak ada hasil')`                            |
| A primitive slot       | `page.locator('[data-slot="card"]')` (shadcn/Radix primitives) |
| A loading spinner      | `confirmButton.locator('svg.animate-spin')`                    |

Class selectors (`button.bg-destructive`) are an escape hatch. **Always scope** them (`[data-slot="card"] button.bg-destructive`) so they don't accidentally match dialog buttons or other shared-style elements.

## Race-condition guards

- **Wait for the API call**, not for time:

  ```ts
  const searchRequest = page.waitForRequest((req) => req.url().includes('search='))
  await searchInput.fill('Outlet 25')
  await searchRequest
  ```

- **Gate the response** when verifying a loading state — a real API often resolves in <100 ms, so the loading UI vanishes before the assertion fires:

  ```ts
  let releaseDelete: () => void = () => {}
  const gate = new Promise<void>((resolve) => { releaseDelete = resolve })
  await page.route(/\/api\/outlets\/[^/?]+$/, async (route) => {
    if (route.request().method() !== 'DELETE') return route.continue()
    await gate
    await route.abort('aborted') // or route.continue() depending on test intent
  })
  ```

- **Use `expect.poll`** for asynchronous numeric assertions:

  ```ts
  await expect.poll(async () => cards.count(), { timeout: 5000 }).toBeGreaterThan(firstPageCount)
  ```

## Data hygiene

E2E tests run against a shared development backend. Treat persistent data as precious:

- Prefer **read-only** flows. Verify behaviour without creating noise.
- For destructive actions (delete, cancel, refund), `route.abort()` after observing the loading state so no real record is touched.
- If a test must create data, **create a uniquely-named throwaway** via direct API call in setup, then either delete it in teardown or rely on the test's own delete flow.
- **Never rely on the previous test's leftover data.** Each test should be re-runnable in isolation.
- Reset client-side caches in `beforeEach` if the test needs a cold cache (e.g., `indexedDB.deleteDatabase('your-app-cache')`).

## Coverage scope

Aim for **happy path + the explicit user-facing edge cases** the feature was built for, not exhaustive permutations:

- Render: required UI shows up under a real fetch.
- Loading: skeletons / spinners appear during the gated request.
- Empty / error states: triggered by realistic input (e.g., search query that returns 0).
- Async flows specific to the feature (debounce, infinite scroll, optimistic updates, post-mutation cache invalidation).

Unit/integration tests belong elsewhere (`frontend-testing-strategy` skill); E2E is for cross-cutting browser behaviour.

## Pre-merge checklist

- [ ] Spec is colocated with the feature it tests (or top-level `e2e/` only if cross-feature).
- [ ] No fabricated API responses; any `page.route` calls `route.continue()` or `route.abort()`.
- [ ] Auth set via cookies, not UI login.
- [ ] Locators use roles/labels/`data-slot`; class selectors are scoped if used.
- [ ] Loading-state assertions are anchored to a gated request, not `waitForTimeout`.
- [ ] Destructive actions don't leave residue in the dev backend.
- [ ] Spec passes in parallel mode (`--workers=2+`).

## References

- [Code snippets and full examples](REFERENCE.md)
- The companion `frontend-testing-strategy` skill for the strategic layer (test trophy, choosing levels).
