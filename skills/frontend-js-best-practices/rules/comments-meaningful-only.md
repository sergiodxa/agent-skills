---
title: Meaningful Comments Only
impact: MEDIUM
impactDescription: code readability and maintenance
tags: javascript, comments, documentation, conventions
---

## Meaningful Comments Only

Don't write comments that repeat what the code says. Only comment when adding information the code cannot express.

### Why

1. **Noise reduction** - Meaningless comments make important ones harder to find
2. **Maintenance burden** - Comments that restate code get outdated
3. **Code should be self-documenting** - Good names and structure reduce need for comments
4. **Comments are for "why", not "what"** - Code shows what, comments explain why

### Bad: Meaningless Comments

```typescript
// Bad: restates the code
// Set the user's name
let userName = user.name;

// Bad: obvious from the code
// Loop through the items
for (let item of items) {
  // Process the item
  processItem(item);
}

// Bad: describes the function name
// Calculate the total
function calculateTotal(items: Item[]) {
  // ...
}

// Bad: describes variable assignment
// Create an empty array
let results = [];
```

### Good: Meaningful Comments

#### Business Rules

```typescript
// Transactions under $250 don't require written acknowledgment per policy
if (transaction.amount < 250) {
  return { requiresAcknowledgment: false };
}
```

#### Non-Obvious Behavior

```typescript
// API returns amounts in cents, convert to dollars for display
let displayAmount = apiAmount / 100;
```

#### Edge Cases and Workarounds

```typescript
// Safari doesn't support smooth scrolling in iframes, use instant instead
let behavior = isSafari && isInIframe ? "instant" : "smooth";
```

#### Performance Decisions

```typescript
// Using Map for O(1) lookups instead of array.find() which is O(n)
// This matters because we check membership for every item in the list
let userById = new Map(users.map((u) => [u.id, u]));
```

#### External Dependencies

```typescript
// Copied from lodash/debounce to avoid adding the full dependency
// https://github.com/lodash/lodash/blob/main/debounce.js
function debounce(fn: Function, wait: number) {
  // ...
}
```

#### Intentional Behavior

```typescript
// Intentionally not awaiting - we want fire-and-forget analytics
analytics.track("page_view", { path });
```

#### TODO with Context

```typescript
// TODO(#1234): Remove after migration completes in Q2 2024
let useLegacyApi = featureFlags.useLegacyApi;
```

### JSDoc for Public APIs

Use JSDoc for exported functions, especially utilities:

```typescript
/**
 * Formats a number as USD currency
 * @param amount - The amount in dollars (not cents)
 * @param showDecimals - Whether to show cents (default: false)
 * @returns Formatted string like "$1,234" or "$1,234.56"
 */
export function formatCurrency(amount: number, showDecimals = false): string {
  // ...
}
```

### When to Comment

| Comment when...               | Example                                       |
| ----------------------------- | --------------------------------------------- |
| Business rule isn't obvious   | IRS requirements, legal constraints           |
| Working around a bug          | Browser quirks, API limitations               |
| Code is intentionally unusual | Performance optimization, deliberate no-await |
| External context needed       | Links to specs, ticket numbers                |
| Trade-off was made            | Why this approach over alternatives           |

### When NOT to Comment

| Don't comment...         | Why                              |
| ------------------------ | -------------------------------- |
| What the code does       | Code already says it             |
| Variable assignments     | Name should be clear             |
| Loop iterations          | Standard patterns are understood |
| Function purpose         | Name should convey it            |
| Obvious type conversions | TypeScript shows types           |
