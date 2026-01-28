---
title: E2E Test Structure
impact: HIGH
tags: [testing, e2e, playwright]
---

# E2E Test Structure

Structure E2E tests around user flows, not technical implementation.

## File Location

E2E tests live in the `e2e` package, NOT in `frontend`:

```
e2e/
└── tests/
    ├── utils.ts              # Test utilities, helpers
    ├── global.setup.ts       # Global setup
    ├── orders.spec.ts        # Order flows
    ├── account.spec.ts       # Account setup flow
    ├── login.spec.ts         # Authentication
    ├── home.spec.ts          # Home page
    └── profile.spec.ts       # Profile management
```

## Basic Test Structure

```typescript
import { test, expect } from "@playwright/test";
import { addAccountBalance, createTestingAccount } from "./utils";

test.describe("Orders", () => {
  test.beforeEach(async ({ page, context }) => {
    // Create test account with mock signin
    await createTestingAccount(page, { account_status: "active" });

    // Get account_id from cookies
    let cookies = await context.cookies();
    let account_id = cookies.find((cookie) => cookie.name === "account_id").value;

    // Set up test data
    await addAccountBalance({ account_id, amount: 10000, replaceBalance: true });
  });

  test("place order with default values", async ({ page }) => {
    await page.goto("/catalog");

    // Search for item
    await page
      .getByPlaceholder("Search by name or ID")
      .fill("example");

    await page.getByRole("heading", { name: "Example Item" }).click();
    await page.getByRole("link", { name: "Buy" }).first().click();

    // Step 1: Amount
    await expect(
      page.getByRole("heading", { name: "Enter the order details" }),
    ).toBeVisible();
    await page.getByRole("button", { name: "Next" }).click();

    // Step 2: Share info
    await page.waitForURL((url) => url.searchParams.get("step") === "share");
    await page.getByRole("button", { name: "Next" }).click();

    // Step 3: Notes
    await page.waitForURL((url) => url.searchParams.get("step") === "notes");
    await page.getByRole("button", { name: "Skip" }).click();

    // Step 4: Confirm
    await page.waitForURL((url) => url.searchParams.get("step") === "confirm");
    await page.getByRole("button", { name: "Submit" }).click();

    // Assert success
    await expect(page.getByAltText("Thank you")).toBeVisible();
  });
});
```

## Key Utilities

Import helpers from `./utils`:

```typescript
import {
  createTestingAccount, // Create test user via mock signin
  addAccountBalance, // Add balance to account
  addAccountBalanceToEmail, // Add balance by email
  getAccountIdByEmail, // Get account ID from email
  logout, // Log out user
  createMockResource, // Create test resource
  createRecordForUser, // Create test record
} from "./utils";
```

### createTestingAccount

Creates a test user via mock signin endpoint:

```typescript
let user = await createTestingAccount(page, {
  email: "test@example.com", // Optional, random if not provided
  name: "John Doe", // Optional
  slug: "john-doe", // Optional
  accountName: "Example Account", // Optional
  account_status: "active", // "active" | "pending" | null
  account_step: "profile", // Optional
  account_source: "web", // Optional
});

// Returns: { email, name, slug, accountName, account_status }
```

### addFundBalance

Adds balance to an account (requires account_id from cookies):

```typescript
let cookies = await context.cookies();
let account_id = cookies.find((c) => c.name === "account_id").value;

await addFundBalance({
  account_id,
  amount: 10000, // Amount in cents
  replaceBalance: true, // Clear existing balance first
});
```

## Waiting Patterns

### Wait for URL changes

```typescript
// Wait for specific URL
await page.waitForURL(/\/home/);

// Wait for query param
await page.waitForURL((url) => url.searchParams.get("step") === "confirm");

// Wait for URL with timeout
await expect(page).toHaveURL(/\/home/, { timeout: 30000 });
```

### Wait for elements

```typescript
// Wait for element to be visible
await page.getByRole("heading", { name: "Success" }).waitFor();

// Wait for element to appear then assert
await expect(page.getByAltText("Thank you")).toBeVisible();

// Wait for element to disappear
await expect(page.getByText("Loading")).not.toBeVisible();
```

## Test Isolation

Each test should be independent:

```typescript
test.describe("Feature", () => {
  // Set up fresh state before each test
  test.beforeEach(async ({ page, context }) => {
    await createTestingAccount(page, { account_status: "active" });
    // ... additional setup
  });

  test("scenario 1", async ({ page }) => {
    // Test runs with fresh state
  });

  test("scenario 2", async ({ page }) => {
    // Test runs with fresh state, independent of scenario 1
  });
});
```

## Rules

1. E2E tests go in `e2e/tests/`, not `frontend/`
2. Use `createTestingAccount` for test user setup
3. Use `addAccountBalance` for setting up test balances
4. Use descriptive test names that explain the user scenario
5. Each test should be independent - use `beforeEach` for setup
6. Wait for URL changes with `waitForURL` or `toHaveURL`
7. Use role-based selectors when possible
