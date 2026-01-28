---
title: Optimize Revalidation
impact: MEDIUM
tags: [routes, performance, revalidation]
---

# Optimize Revalidation

Use shouldRevalidate to prevent unnecessary data fetching.

## Why

- By default, React Router revalidates all loaders after actions
- Some routes don't need revalidation on every action
- Reduces server load and improves performance
- Prevents unnecessary loading states

## Basic Pattern

```tsx

export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  currentUrl,
  nextUrl,
  formAction,
  formMethod,
  defaultShouldRevalidate,
}) => {
  // Your logic here
  return defaultShouldRevalidate;
};
```

## Common Patterns

### 1. Skip Revalidation for Same Path

```tsx
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  currentUrl,
  nextUrl,
  defaultShouldRevalidate,
}) => {
  // Don't revalidate if only hash changed
  if (currentUrl.pathname === nextUrl.pathname) {
    return false;
  }
  return defaultShouldRevalidate;
};
```

### 2. Skip for Specific Actions

```tsx
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  formAction,
  defaultShouldRevalidate,
}) => {
  // Don't revalidate for "like" actions
  if (formAction?.includes("/api/like")) {
    return false;
  }
  return defaultShouldRevalidate;
};
```

### 3. Only Revalidate for Related Actions

```tsx
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  formAction,
  defaultShouldRevalidate,
}) => {
  // Only revalidate for actions related to this route's data
  let relatedActions = ["/dashboard", "/items", "/settings"];
  if (formAction && !relatedActions.some((a) => formAction.startsWith(a))) {
    return false;
  }
  return defaultShouldRevalidate;
};
```

### 4. Skip for Search Params Changes

```tsx
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  currentUrl,
  nextUrl,
  defaultShouldRevalidate,
}) => {
  // Don't revalidate if only search params changed
  // (loader reads params, so this might vary by use case)
  if (
    currentUrl.pathname === nextUrl.pathname &&
    currentUrl.search !== nextUrl.search
  ) {
    return false;
  }
  return defaultShouldRevalidate;
};
```

### 5. Never Revalidate Static Data

```tsx
// For static marketing pages
export const shouldRevalidate: Route.ShouldRevalidateFunction = () => {
  // Static page data never changes, no need to revalidate
  return false;
};
```

## Function Parameters

```tsx
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  currentUrl, // URL before navigation
  nextUrl, // URL after navigation
  currentParams, // Route params before
  nextParams, // Route params after
  formAction, // Action URL if form submitted
  formMethod, // GET, POST, etc.
  formData, // Form data if available
  actionResult, // Result from action
  defaultShouldRevalidate, // What React Router would do
}) => {
  // ...
};
```

## When NOT to Use

Don't skip revalidation when:

- Data depends on user actions
- Data is frequently updated by others
- Stale data would cause bugs
- You're not sure (use default)

```tsx
// When in doubt, use default
export const shouldRevalidate: Route.ShouldRevalidateFunction = ({
  defaultShouldRevalidate,
}) => {
  return defaultShouldRevalidate;
};
```
