---
title: Unit Test Structure
impact: MEDIUM
tags: [testing, vitest, unit-tests]
---

# Unit Test Structure

Structure unit tests for pure functions and utilities.

## When to Write Unit Tests

Only write unit tests for:

- Pure utility functions (`formatCurrency`, `parseDate`)
- Data transformations and validators
- Complex algorithms
- Custom hooks with non-trivial logic (rare)

## Basic Structure

```typescript
import { describe, test, expect } from "vitest";
import { formatCurrency, formatPercentage } from "./format";

describe("formatCurrency", () => {
  test("formats positive amounts with two decimals", () => {
    expect(formatCurrency(1234.5)).toBe("$1,234.50");
  });

  test("formats zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });

  test("formats negative amounts", () => {
    expect(formatCurrency(-100)).toBe("-$100.00");
  });
});

describe("formatPercentage", () => {
  test("formats decimal as percentage", () => {
    expect(formatPercentage(0.25)).toBe("25%");
  });
});
```

## Parameterized Tests

Use arrays for testing multiple inputs:

```typescript
import { describe, test, expect } from "vitest";
import { slugify } from "./string";

const testCases: [string, string][] = [
  ["Hello World", "hello-world"],
  ["Multiple   Spaces", "multiple-spaces"],
  ["Special @#$ Characters", "special-characters"],
  ["Already-slugified", "already-slugified"],
  ["", ""],
];

describe("slugify", () => {
  test.each(testCases)('slugify("%s") returns "%s"', (input, expected) => {
    expect(slugify(input)).toBe(expected);
  });
});
```

## Testing Error Cases

```typescript
import { describe, test, expect } from "vitest";
import { parseAmount } from "./parse";

describe("parseAmount", () => {
  test("throws for invalid input", () => {
    expect(() => parseAmount("not-a-number")).toThrow("Invalid amount");
  });

  test("throws for negative amounts", () => {
    expect(() => parseAmount("-100")).toThrow("Amount must be positive");
  });
});
```

## Testing Async Functions

```typescript
import { describe, test, expect } from "vitest";
import { fetchUserData } from "./api";

describe("fetchUserData", () => {
  test("returns user data for valid ID", async () => {
    let user = await fetchUserData("user-123");
    expect(user).toEqual({
      id: "user-123",
      name: expect.any(String),
    });
  });

  test("throws for non-existent user", async () => {
    await expect(fetchUserData("invalid")).rejects.toThrow("User not found");
  });
});
```

## File Naming and Location

Tests are co-located with source files:

```
app/
├── utils/
│   ├── format.ts
│   ├── format.test.ts      # Unit test
│   ├── string.ts
│   └── string.test.ts
```

## Test Naming

Use descriptive names that explain the scenario:

```typescript
// Bad: vague names
test("works", () => {});
test("test 1", () => {});
test("formatCurrency", () => {});

// Good: describes behavior
test("formats amount with thousand separators", () => {});
test("returns empty string for null input", () => {});
test("throws when amount exceeds maximum", () => {});
```

## Rules

1. Test file goes next to source file: `foo.ts` → `foo.test.ts`
2. Use `describe` to group related tests
3. Use descriptive test names that explain the expected behavior
4. Test edge cases: null, undefined, empty, negative, max values
5. Use `test.each` for parameterized tests with multiple inputs
6. Keep tests focused - one assertion per behavior
