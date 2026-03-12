# Frontend Testing Strategy Examples

## Integration Test Pattern

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect } from 'vitest';
import { SearchForm } from './SearchForm';

describe('SearchForm', () => {
  it('submits query and displays results', async () => {
    const user = userEvent.setup();

    render(<SearchForm />);

    await user.type(screen.getByRole('searchbox'), 'react hooks');
    await user.click(screen.getByRole('button', { name: /search/i }));

    expect(
      await screen.findByText(/results for "react hooks"/i)
    ).toBeInTheDocument();
  });
});
```

## MSW Pattern

```ts
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

export const server = setupServer(
  http.get('/api/user/:id', ({ params }) => {
    return HttpResponse.json({
      id: params.id,
      name: 'Umesh Malik',
      role: 'engineer',
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Unit Test Pattern

```ts
import { describe, it, expect } from 'vitest';
import { calculateDiscount, formatCurrency } from './utils';

describe('formatCurrency', () => {
  it('formats USD with two decimals', () => {
    expect(formatCurrency(1234.5, 'USD')).toBe('$1,234.50');
  });
});
```

## Hook Test Pattern

```ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { useDebounce } from './useDebounce';

describe('useDebounce', () => {
  beforeEach(() => vi.useFakeTimers());
  afterEach(() => vi.useRealTimers());

  it('updates after delay', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 300),
      { initialProps: { value: 'hello' } }
    );

    rerender({ value: 'world' });
    act(() => vi.advanceTimersByTime(300));

    expect(result.current).toBe('world');
  });
});
```

## E2E Pattern

```ts
import { test, expect } from '@playwright/test';

test('user can complete checkout flow', async ({ page }) => {
  await page.goto('/products');
  await page.click('[data-testid="add-to-cart-1"]');
  await expect(page.locator('.cart-count')).toHaveText('1');

  await page.click('text=Checkout');
  await expect(page).toHaveURL('/checkout');

  await page.fill('#email', 'test@example.com');
  await page.fill('#address', '123 Test St');
  await page.click('button:text("Place Order")');

  await expect(page.locator('h1')).toHaveText('Order Confirmed');
});
```

## Vitest Setup Template

```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.test.{ts,tsx}'],
    coverage: {
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'src/test/'],
    },
  },
});
```

```ts
import '@testing-library/jest-dom/vitest';
```
