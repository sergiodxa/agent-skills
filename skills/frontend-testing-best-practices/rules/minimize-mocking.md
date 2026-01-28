---
title: Minimize Mocking
impact: HIGH
tags: [testing, mocking, strategy]
---

# Minimize Mocking

Keep mocks simple and minimal. If you need complex mocking, write an E2E test instead.

## Why

- Mocks can diverge from real implementations
- Complex mocks are hard to maintain
- Mocks test your mock, not your code
- Over-mocking leads to false confidence

## The Mock Smell Test

If your test setup looks like this, write an E2E test:

```typescript
// BAD: Too many mocks = write an E2E test
vi.mock("~/lib/auth");
vi.mock("~/lib/transactions");
vi.mock("~/hooks/useUser");
vi.mock("~/hooks/useCart");
vi.mock("@remix-run/react", () => ({
  useNavigate: () => vi.fn(),
  useLoaderData: () => mockLoaderData,
}));

describe("CheckoutPage", () => {
  // This test provides false confidence
});
```

## Acceptable Mocking

### 1. MSW for API Calls (Simple Cases)

```typescript
import { mockServer, http, HttpResponse } from "~/lib/test-utils";

beforeEach(() => {
  mockServer.use(
    http.get("/api/user", () => HttpResponse.json({ id: 1, name: "John" })),
  );
});

test("loader returns user data", async () => {
  let response = await loader({ request, params: {}, context: {} });
  let data = await response.json();
  expect(data.user.name).toBe("John");
});
```

### 2. Fake Timers for Time-Based Logic

```typescript
import { vi, beforeAll, afterAll } from "vitest";

beforeAll(() => {
  vi.useFakeTimers();
  vi.setSystemTime(new Date("2025-01-15"));
});

afterAll(() => {
  vi.useRealTimers();
});

test("isExpired returns true for past dates", () => {
  expect(isExpired(new Date("2025-01-01"))).toBe(true);
});
```

### 3. Environment Variables

```typescript
test("uses production API in production", async () => {
  vi.stubEnv("NODE_ENV", "production");
  const { apiUrl } = await import("./config");
  expect(apiUrl).toBe("https://api.example.com");
  vi.unstubAllEnvs();
});
```

## Unacceptable Mocking

### Mocking React Hooks

```typescript
// BAD: Mocking hooks
vi.mock("react", async () => ({
  ...(await vi.importActual("react")),
  useState: vi.fn(),
  useEffect: vi.fn(),
}));
```

### Mocking Remix Functions

```typescript
// BAD: Mocking Remix internals
vi.mock("@remix-run/react", () => ({
  useLoaderData: () => ({ user: { name: "John" } }),
  useActionData: () => null,
  useNavigation: () => ({ state: "idle" }),
}));
```

### Mocking Multiple Services

```typescript
// BAD: If you need all these mocks, write an E2E test
vi.mock("~/lib/auth");
vi.mock("~/lib/transactions");
vi.mock("~/lib/notifications");
```

## Decision Tree

```
Can I test this with no mocks?
  → Yes: Do that
  → No: Continue...

Can I test this with just MSW (1-2 endpoints)?
  → Yes: Integration test with MSW
  → No: Continue...

Would I need to mock Remix, React, or 3+ services?
  → Yes: Write an E2E test instead
```

## Rules

1. Zero mocks is the ideal - test pure functions
2. MSW is acceptable for simple API mocking (1-2 endpoints)
3. Fake timers are acceptable for time-based logic
4. Never mock React, Remix, or third-party UI libraries
5. If you need 3+ mocks, write an E2E test
6. Complex mock setup is a code smell - refactor or use E2E
7. Mocks should be simple enough to verify correctness at a glance
