---
title: Hoist RegExp Creation
impact: LOW-MEDIUM
impactDescription: avoids recreation
tags: javascript, regexp, optimization, memoization
---

## Hoist RegExp Creation

Don't create RegExp inside loops or frequently-called functions. Hoist to module scope or cache the result.

**Incorrect (new RegExp every call):**

```typescript
function highlightMatches(text: string, query: string) {
  let regex = new RegExp(`(${query})`, "gi"); // Created every call
  return text.split(regex);
}

// In a loop - creates regex 1000 times
items.forEach((item) => {
  let matches = highlightMatches(item.text, searchQuery);
});
```

**Correct (hoist static patterns):**

```typescript
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const URL_REGEX = /https?:\/\/[^\s]+/g;

function isValidEmail(email: string) {
  return EMAIL_REGEX.test(email);
}
```

**Correct (cache dynamic patterns):**

```typescript
const regexCache = new Map<string, RegExp>();

function getHighlightRegex(query: string): RegExp {
  if (!regexCache.has(query)) {
    regexCache.set(query, new RegExp(`(${escapeRegex(query)})`, "gi"));
  }
  return regexCache.get(query)!;
}

function highlightMatches(text: string, query: string) {
  let regex = getHighlightRegex(query);
  return text.split(regex);
}
```

**Warning (global regex has mutable state):**

Global regex (`/g`) has mutable `lastIndex` state:

```typescript
const regex = /foo/g;
regex.test("foo"); // true, lastIndex = 3
regex.test("foo"); // false, lastIndex = 0 (unexpected!)
```

Reset `lastIndex` before reuse, or create a new regex for each use when using `/g` flag.
