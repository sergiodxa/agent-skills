---
title: Avoid Testing React Components in Isolation
impact: HIGH
tags: [testing, react, components]
---

# Avoid Testing React Components in Isolation

Don't write unit tests for React components. Test them through E2E tests or not at all.

## Why

- Component tests often test implementation details
- They break when you refactor without changing behavior
- They require complex mocking (context, hooks, providers)
- They don't provide confidence that the feature actually works
- E2E tests cover components naturally as part of user flows

## Bad: Component Unit Tests

```typescript
// BAD: Testing component rendering
import { render, screen } from "@testing-library/react";
import { OrderCard } from "./order-card";

describe("OrderCard", () => {
  test("renders order amount", () => {
    render(<OrderCard amount={100} itemName="Example Item" />);
    expect(screen.getByText("$100")).toBeInTheDocument();
    expect(screen.getByText("Red Cross")).toBeInTheDocument();
  });

  test("shows pending state", () => {
    render(<OrderCard amount={100} itemName="Example Item" status="pending" />);
    expect(screen.getByText("Pending")).toBeInTheDocument();
  });
});
```

Why this is bad:

- Tests that props render correctly - this is React's job
- Doesn't test that the component works in context
- Will break if you rename props or restructure JSX
- Mocking providers is tedious and error-prone

## Good: E2E Test That Covers the Component

```typescript
// GOOD: E2E test that naturally tests OrderCard
import { test, expect } from "@playwright/test";

test("order history shows pending orders", async ({ page }) => {
  // Create a pending order via API or test setup
  await createTestOrder({ status: "pending" });

  await page.goto("/orders");

  // The OrderCard component is tested implicitly
  await expect(page.getByText("$100")).toBeVisible();
  await expect(page.getByText("Example Item")).toBeVisible();
  await expect(page.getByText("Pending")).toBeVisible();
});
```

## When Component Tests Might Be Acceptable

Only consider component tests for:

1. **Highly reusable UI library components** (Button, Input, Modal)
   - But even then, prefer visual regression tests or Storybook

2. **Complex isolated logic** (but extract it to a hook and test that)

```typescript
// ACCEPTABLE: Testing a reusable Badge component variants
// But only if it's a shared UI component used everywhere
describe(Badge.name, () => {
  test('renders the "success" variant classes', () => {
    render(<Badge variant="success">Text</Badge>);
    expect(screen.getByText("Text")).toHaveClass("bg-success-100");
  });
});
```

## Extract Logic to Testable Hooks

If a component has complex logic, extract it:

```typescript
// BAD: Complex logic in component, tested via component test
function OrderForm() {
  let [amount, setAmount] = useState(0);
  let [fee, setFee] = useState(0);

  useEffect(() => {
    // Complex fee calculation
    let baseFee = amount * 0.029 + 0.3;
    let adjustedFee = amount > 1000 ? baseFee * 0.9 : baseFee;
    setFee(adjustedFee);
  }, [amount]);

  // ...
}

// GOOD: Extract logic to hook, test the hook
function useOrderFee(amount: number) {
  return useMemo(() => {
    let baseFee = amount * 0.029 + 0.3;
    return amount > 1000 ? baseFee * 0.9 : baseFee;
  }, [amount]);
}

// Or even better: pure function
function calculateOrderFee(amount: number): number {
  let baseFee = amount * 0.029 + 0.3;
  return amount > 1000 ? baseFee * 0.9 : baseFee;
}

// Test the pure function
describe("calculateOrderFee", () => {
  test("calculates standard fee", () => {
    expect(calculateOrderFee(100)).toBe(3.2);
  });

  test("applies discount for large orders", () => {
    expect(calculateOrderFee(2000)).toBe(52.47);
  });
});
```

## Rules

1. Don't write unit tests for React components
2. Test components through E2E tests as part of user flows
3. If component has complex logic, extract to a hook or pure function
4. UI library components can have minimal variant tests
5. Never test that props render correctly - that's React's job
6. If you need to mock providers/context, write an E2E test instead
