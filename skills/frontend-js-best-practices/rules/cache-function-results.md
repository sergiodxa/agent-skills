---
title: Cache Repeated Function Calls
impact: MEDIUM
impactDescription: avoid redundant computation
tags: javascript, cache, memoization, performance
---

## Cache Repeated Function Calls

Use a module-level Map to cache function results when the same function is called repeatedly with the same inputs.

**Incorrect (redundant computation):**

```typescript
function processProjects(projects: Project[]) {
  return projects.map((project) => {
    // slugify() called 100+ times for same project names
    let slug = slugify(project.name);
    return { ...project, slug };
  });
}
```

**Correct (cached results):**

```typescript
// Module-level cache
const slugifyCache = new Map<string, string>();

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!;
  }
  let result = slugify(text);
  slugifyCache.set(text, result);
  return result;
}

function processProjects(projects: Project[]) {
  return projects.map((project) => {
    // Computed only once per unique project name
    let slug = cachedSlugify(project.name);
    return { ...project, slug };
  });
}
```

**Simpler pattern for single-value functions:**

```typescript
let isLoggedInCache: boolean | null = null;

function isLoggedIn(): boolean {
  if (isLoggedInCache !== null) {
    return isLoggedInCache;
  }

  isLoggedInCache = document.cookie.includes("auth=");
  return isLoggedInCache;
}

// Clear cache when auth changes
function onAuthChange() {
  isLoggedInCache = null;
}
```

**With LRU limits (prevent memory leaks):**

```typescript
const MAX_CACHE_SIZE = 1000;

function cachedSlugify(text: string): string {
  if (slugifyCache.has(text)) {
    return slugifyCache.get(text)!;
  }

  // Prevent unbounded growth
  if (slugifyCache.size >= MAX_CACHE_SIZE) {
    let firstKey = slugifyCache.keys().next().value;
    slugifyCache.delete(firstKey);
  }

  let result = slugify(text);
  slugifyCache.set(text, result);
  return result;
}
```

Use a Map (not React hooks) so it works everywhere: utilities, event handlers, loaders, etc.
