# Frontend E2E Testing — Reference

Detailed code patterns and worked examples to complement [SKILL.md](SKILL.md).

## Reading test tokens from `.env.local`

A small helper avoids pulling in `dotenv` just for tests:

```ts
import { readFileSync } from 'node:fs'
import { resolve } from 'node:path'

const ENV_FILE = resolve(process.cwd(), '.env.local')

function readEnvVar(name: string): string {
  const raw = readFileSync(ENV_FILE, 'utf8')
  const match = raw.match(new RegExp(`^${name}=(.*)$`, 'm'))
  if (!match) throw new Error(`Missing ${name} in .env.local`)
  return match[1].trim()
}
```

## Auth bootstrap pattern

```ts
import { expect, test, type BrowserContext } from '@playwright/test'

const ACCESS = readEnvVar('SB_ACCESS_TOKEN_TEST')
const REFRESH = readEnvVar('SB_ACCESS_REFRESH_TEST')

async function applyAuthCookies(context: BrowserContext) {
  await context.addCookies([
    {
      name: 'sb_access_token',
      value: ACCESS,
      domain: 'localhost',
      path: '/',
      httpOnly: true,
      sameSite: 'Lax',
    },
    {
      name: 'sb_refresh_token',
      value: REFRESH,
      domain: 'localhost',
      path: '/',
      httpOnly: true,
      sameSite: 'Lax',
    },
  ])
}

test.describe('Outlets list', () => {
  test.beforeEach(async ({ context }) => {
    await applyAuthCookies(context)
  })
  // tests...
})
```

Cookie attributes (name, domain, `httpOnly`, `sameSite`) must match the backend's session cookie config. Find them in the BE source — typically a single helper file like `lib/sessionCookies.ts` or middleware that calls `setCookie`.

## Seeding throwaway data via the real API

When a test must create data (e.g., to verify deletion), use Playwright's `request` API with its own cookie jar so refresh `Set-Cookie` headers propagate:

```ts
import { request as playwrightRequest } from '@playwright/test'

async function createThrowawayResource(name: string) {
  const ctx = await playwrightRequest.newContext({
    storageState: {
      cookies: [
        {
          name: 'sb_access_token',
          value: ACCESS,
          domain: 'localhost',
          path: '/',
          httpOnly: true,
          sameSite: 'Lax',
          expires: -1,
          secure: false,
        },
        {
          name: 'sb_refresh_token',
          value: REFRESH,
          domain: 'localhost',
          path: '/',
          httpOnly: true,
          sameSite: 'Lax',
          expires: -1,
          secure: false,
        },
      ],
      origins: [],
    },
  })

  const post = () => ctx.post(`${API_BASE}/resources`, { data: { name, /* ... */ } })

  let response = await post()
  if (response.status() === 401) {
    // Mirror the FE's fetchWithAuth: refresh once, then retry.
    const refresh = await ctx.post(`${API_BASE}/auth/refresh`)
    if (!refresh.ok()) throw new Error(`Refresh failed: ${refresh.status()}`)
    response = await post()
  }
  if (!response.ok()) throw new Error(`Create failed: ${response.status()} ${await response.text()}`)

  const body = (await response.json()) as { data: { id: string; slug: string } }
  await ctx.dispose()
  return body.data
}
```

The `storageState` cookie jar is the key piece — `extraHTTPHeaders: { Cookie }` is fixed and won't pick up `Set-Cookie` from `/auth/refresh`.

## Network timing controls — never mock payloads

### Pattern 1: delay a real GET to make a loading state observable

```ts
await page.route(/\/api\/v1\/resources\?[^/]*page=1[^/]*$/, async (route) => {
  if (route.request().method() !== 'GET') return route.continue()
  await new Promise((r) => setTimeout(r, 800))
  await route.continue() // pass through to the real backend
})
```

### Pattern 2: gate a destructive mutation so the loading UI is observable, then abort

```ts
let releaseDelete: () => void = () => {}
const gate = new Promise<void>((resolve) => { releaseDelete = resolve })

await page.route(/\/api\/v1\/resources\/[^/?]+$/, async (route) => {
  if (route.request().method() !== 'DELETE') return route.continue()
  await gate
  await route.abort('aborted') // outlet preserved, mutation surfaces as error in FE
})

// ... open dialog, click confirm ...
await expect(confirmButton).toBeDisabled()
await expect(confirmButton.locator('svg.animate-spin')).toBeVisible()
releaseDelete()
await expect(confirmButton).toHaveCount(0) // dialog closes via mutation onSettled
```

### Anti-pattern: don't do this

```ts
// ❌ Fabricates reality. The test passes, but the real backend behaviour is never exercised.
await page.route('**/api/v1/resources', async (route) => {
  await route.fulfill({ status: 200, body: JSON.stringify({ data: [...] }) })
})
```

## Anchoring assertions to network events

```ts
test('search debounces then fetches once', async ({ page }) => {
  await page.goto('/resources')
  await expect(page.locator('[data-slot="card"]').first()).toBeVisible()

  const searchInput = page.getByPlaceholder('Search...')
  const searchRequest = page.waitForRequest((req) => req.url().includes('search='))
  await searchInput.fill('Resource 25')

  // Spinner shows during the debounce window (no request fired yet).
  await expect(page.locator('[aria-label="Searching"]')).toBeVisible()

  // The debounce settles, the request finally fires, and we anchor to it.
  await searchRequest

  // After the response, the spinner is gone and the result is rendered.
  await expect(page.locator('[aria-label="Searching"]')).toHaveCount(0)
  await expect(page.getByRole('heading', { name: 'Resource 25' })).toBeVisible()
})
```

## Polling numeric expectations

For asynchronous values that change in steps (card counts, list lengths), use `expect.poll`:

```ts
const cards = page.locator('[data-slot="card"]')
await expect(cards.first()).toBeVisible()

await page.waitForResponse((res) => res.url().includes('page=1') && res.url().includes('limit=10'))
const firstPageCount = await cards.count()

const nextPageRequest = page.waitForRequest((req) => req.url().includes('page=2'))
await cards.nth(firstPageCount - 1).scrollIntoViewIfNeeded()
await nextPageRequest

await expect.poll(async () => cards.count(), { timeout: 5000 }).toBeGreaterThan(firstPageCount)
```

## Scoping CSS selectors

Class-based selectors often match across components that share styling (the same `bg-destructive` Tailwind class is used by both row delete buttons AND the dialog confirm button in shadcn). Always scope:

```ts
// ❌ Matches both row buttons AND the dialog "Hapus" button
const deleteButtons = page.locator('button.bg-destructive')

// ✅ Scoped to row cards only
const rowDeleteButtons = page.locator('[data-slot="card"] button.bg-destructive')
```

## Running and debugging

```bash
# Run all suites (parallel by default)
bun run test:e2e

# One feature
bun run test:e2e app/routes/{feature}/e2e

# By name
bun run test:e2e --grep "infinite scroll"

# Single worker for sequential debugging
bun run test:e2e --workers=1

# Debug UI (interactive mode)
bun run test:e2e:ui

# Capture trace on first retry only
# (configured in playwright.config.ts: use: { trace: 'on-first-retry' })
```

When a test fails, inspect the trace zip in `test-results/.../trace.zip` via `npx playwright show-trace`.

## Common failure modes & fixes

| Symptom                                               | Likely cause                                                            | Fix                                                                                                  |
| ----------------------------------------------------- | ----------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Loading-state assertion fails intermittently          | Real API resolved before assertion                                      | Gate the response with `route` + delay or controlled promise                                         |
| Test passes locally, fails in parallel mode           | Shared dev-backend data conflict, or IndexedDB leak across tests        | Use unique names, clean up created data, reset IndexedDB in `beforeEach`                             |
| Test passes but the feature is broken in production   | `route.fulfill` returned a fabricated payload that hid the real bug     | Replace with `route.continue()` and assert against real backend behaviour                            |
| Mutation succeeds but list still shows the old value  | FE refetched stale browser-cached response (`Cache-Control: max-age=N`) | BE issue: change `Cache-Control` to `private, no-cache` so the browser revalidates                   |
| `getByRole('button', { name: 'X' })` matches too many | Dialog confirm shares button text with row action                       | Scope: `card.getByRole('button', { name: 'X' })` or `page.getByRole('alertdialog').getByRole(...)`   |
| `text-decoration` test for a class selector fails     | Component doesn't actually apply that class — implementation drifted    | Switch to a behavioural assertion (visible text, role, `toBeDisabled`) rather than a CSS-class probe |

## When mocks ARE acceptable

Use `route.fulfill` with fabricated payloads only for:

- **Third-party services** that are unsafe or expensive to call from CI: payment processors (Stripe), email senders (Resend, Postmark), SMS providers, AI APIs.
- **Webhook receivers** when simulating provider-side events.

Always document the reason inline:

```ts
// External payment provider — never call from CI.
await page.route('**/api.stripe.com/**', async (route) => {
  await route.fulfill({ status: 200, body: JSON.stringify({ status: 'succeeded' }) })
})
```
