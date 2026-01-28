---
title: Prefer E2E Tests Over Unit Tests
impact: HIGH
tags: [testing, e2e, strategy]
---

# Prefer E2E Tests Over Unit Tests

Write E2E tests as your default testing strategy. Only write unit tests for pure functions and utilities.

## Why

- E2E tests provide real confidence - they test what users actually do
- E2E tests catch integration issues that unit tests miss
- E2E tests don't require mocking, so they're more maintainable
- Refactoring doesn't break E2E tests (implementation changes, behavior stays)

## Decision Flow

```
Is it a pure function with no dependencies?
  → Yes: Unit test (Vitest)
  → No: Continue...

Is it a loader/action with simple API calls?
  → Yes: Integration test with MSW (Vitest)
  → No: Continue...

Does it involve user interaction, routing, or complex state?
  → Yes: E2E test (Playwright)

Would testing it require complex mocking?
  → Yes: E2E test (Playwright)
```

## Examples

### E2E Test (Preferred for User Flows)

```typescript
// e2e/orders.spec.ts
import { test, expect } from "@playwright/test";

test("user can place an order", async ({ page }) => {
  await page.goto("/catalog");

  await page.getByLabel("Quantity").fill("1");
  await page.getByLabel("Item").selectOption("item-123");
  await page.getByRole("button", { name: "Buy" }).click();

  await expect(page.getByText("Thanks for your order")).toBeVisible();
});
```

### Unit Test (For Pure Functions Only)

```typescript
// app/utils/format.test.ts
import { describe, test, expect } from "vitest";
import { formatCurrency } from "./format";

describe("formatCurrency", () => {
  test("formats dollars with two decimals", () => {
    expect(formatCurrency(1234.5)).toBe("$1,234.50");
  });

  test("handles zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });
});
```

### Integration Test (For Loaders with Simple Mocking)

```typescript
// app/routes/profile/route.test.ts
import { describe, test, expect, beforeEach } from "vitest";
import { loader } from "./route";
import { mockServer, http, HttpResponse } from "~/lib/test-utils";

beforeEach(() => {
  mockServer.use(
    http.get("/api/user", () => HttpResponse.json({ name: "John" })),
  );
});

describe("profile loader", () => {
  test("returns user data", async () => {
    let response = await loader({
      request: new Request("http://test"),
      params: {},
      context: {},
    });
    let data = await response.json();
    expect(data.user.name).toBe("John");
  });
});
```

## What NOT to Unit Test

```typescript
// BAD: Don't unit test React components
describe("OrderForm", () => {
  test("renders form fields", () => {
    render(<OrderForm />);
    // This doesn't provide real confidence
  });
});

// BAD: Don't unit test with complex mocks
describe("CheckoutFlow", () => {
  test("processes checkout", async () => {
    vi.mock("~/lib/transactions");
    vi.mock("~/lib/analytics");
    vi.mock("~/hooks/useCart");
    // If you need this many mocks, write an E2E test
  });
});
```

## Rules

1. Default to E2E tests for anything involving user interaction
2. Use unit tests only for pure functions with no dependencies
3. If a unit test requires more than one mock, consider E2E instead
4. Don't unit test React components - test them via E2E
5. Integration tests (Vitest + MSW) are acceptable for loaders/actions
6. Measure confidence, not coverage
