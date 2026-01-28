---
title: Const vs Let Usage
impact: MEDIUM
impactDescription: consistency and intent clarity
tags: javascript, variables, naming, conventions
---

## Const vs Let Usage

Use `const` at module level and `let` inside functions/blocks.

### Module Level: Always `const`

```typescript
// Single value constants - UPPER_SNAKE_CASE
const MAX_RETRIES = 3;
const DEFAULT_TIMEOUT = 5000;
const API_BASE_URL = "/api/v1";

// Objects, arrays, Maps, Sets - camelCase
const userCache = new Map<string, User>();
const allowedRoles = new Set(["admin", "editor"]);
const defaultConfig = { timeout: 5000, retries: 3 };

// Regex patterns - UPPER_SNAKE_CASE
const EMAIL_PATTERN = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const SLUG_PATTERN = /^[a-z0-9-]+$/;
```

### Inside Functions/Blocks: Always `let`

```typescript
function processUsers(users: User[]) {
  // Always use let inside functions, even for values that don't change
  let total = 0;
  let activeCount = 0;
  let result = [];

  for (let user of users) {
    let isActive = user.status === "active";
    if (isActive) {
      activeCount += 1;
      result.push(user);
    }
    total += 1;
  }

  let summary = { total, activeCount };
  return { result, summary };
}
```

### Why This Convention

1. **Clear scope distinction** - `const` signals module-level, `let` signals local
2. **Easier refactoring** - No need to change `const` to `let` when you need to reassign
3. **Consistent codebase** - One rule to follow, no debates about const vs let locally
4. **UPPER_SNAKE_CASE** - Instantly identifies true constants (primitives that never change)

### Never Use `var`

```typescript
// Bad
var count = 0;
var users = [];

// Good
let count = 0;
let users = [];
```

### Summary

| Scope                                 | Keyword                | Naming             |
| ------------------------------------- | ---------------------- | ------------------ |
| Module-level primitive constants      | `const`                | `UPPER_SNAKE_CASE` |
| Module-level objects/arrays/Maps/Sets | `const`                | `camelCase`        |
| Module-level functions                | `function` declaration | `camelCase`        |
| Inside functions/methods/blocks       | `let`                  | `camelCase`        |
