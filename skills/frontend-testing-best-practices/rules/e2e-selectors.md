---
title: E2E Test Selectors
impact: MEDIUM
tags: [testing, e2e, playwright, selectors]
---

# E2E Test Selectors

Use accessible queries and data-testid for reliable element selection.

## Selector Priority

Prefer selectors in this order (most to least preferred):

1. **Role-based** - `getByRole("button", { name: "Submit" })`
2. **Label-based** - `getByLabel("Email")`
3. **Text-based** - `getByText("Welcome")`
4. **Test ID** - `getByTestId("balance")`

## Role-Based Selectors (Preferred)

```typescript
// Buttons
await page.getByRole("button", { name: "Submit" }).click();
await page.getByRole("button", { name: /submit/i }).click(); // Case insensitive

// Links
await page.getByRole("link", { name: "Home" }).click();

// Headings
await expect(page.getByRole("heading", { name: "Dashboard" })).toBeVisible();

// Form elements
await page.getByRole("textbox", { name: "Email" }).fill("test@example.com");
await page.getByRole("checkbox", { name: "Accept terms" }).check();
await page.getByRole("combobox", { name: "Country" }).selectOption("US");
```

## Label-Based Selectors

```typescript
// Form inputs by label
await page.getByLabel("Email").fill("test@example.com");
await page.getByLabel("Password").fill("secret123");

// Partial match
await page.getByLabel(/email/i).fill("test@example.com");
```

## Text-Based Selectors

```typescript
// Exact text
await page.getByText("Welcome back").click();

// Partial/regex
await page.getByText(/welcome/i).click();

// Within a container
await page.locator(".card").getByText("Details").click();
```

## Test ID Selectors (When Needed)

Use `data-testid` when:

- Element has no accessible name
- Multiple similar elements exist
- Testing specific values (like balance display)

```tsx
// In component
<span data-testid="balance">${balance}</span>;

// In test
await expect(page.getByTestId("balance")).toHaveText("$1,234.56");
```

## Bad Selectors

```typescript
// Bad: fragile CSS selectors
await page.locator(".btn-primary").click();
await page.locator("#submit-button").click();
await page.locator("form > div:nth-child(2) > button").click();

// Bad: implementation details
await page.locator("[class*='Button_primary']").click();
```

## Combining Selectors

```typescript
// Filter within results
await page
  .getByRole("listitem")
  .filter({ hasText: "Red Cross" })
  .getByRole("button", { name: "Donate" })
  .click();

// Within a specific region
await page
  .getByRole("region", { name: "Order history" })
  .getByRole("row")
  .first()
  .click();
```

## Waiting for Elements

```typescript
// Wait for element to be visible
await expect(page.getByRole("heading", { name: "Success" })).toBeVisible();

// Wait for element to disappear
await expect(page.getByText("Loading...")).not.toBeVisible();

// Wait for specific count
await expect(page.getByRole("listitem")).toHaveCount(5);
```

## Rules

1. Prefer role-based selectors - they test accessibility too
2. Use label selectors for form inputs
3. Use `data-testid` only when no accessible selector exists
4. Never use CSS class or generated ID selectors
5. Use `filter()` to narrow down multiple matches
6. Always use `await expect()` for assertions
