---
title: Link Prefetch Intent
impact: MEDIUM
tags: [navigation, performance, prefetch]
---

# Link Prefetch Intent

Use `prefetch="intent"` on links for faster perceived navigation.

## Why

- Prefetches route data on hover/focus (before click)
- Makes navigation feel instant
- Only loads data when user shows intent
- Better than `prefetch="render"` (loads immediately) for large pages

## Pattern

```tsx
import { Link } from "react-router";

<Link to="/dashboard" prefetch="intent">
  Dashboard
</Link>;
```

## Prefetch Options

| Value        | When Prefetches | Use Case              |
| ------------ | --------------- | --------------------- |
| `"none"`     | Never           | Rarely visited links  |
| `"intent"`   | Hover/focus     | Most navigation links |
| `"render"`   | On render       | Critical next steps   |
| `"viewport"` | When visible    | Link lists            |

## Examples from Codebase

```tsx
// Navigation links
<Link to="/dashboard" prefetch="intent">
  Dashboard
</Link>

// Login/auth links
<Link to="/login" className="font-medium" prefetch="intent">
  Log in
</Link>

// Notification items
<Link
  to={notification.link}
  prefetch="intent"
  className="block p-3 hover:bg-neutral-50"
>
  {notification.title}
</Link>

// Pagination
<Link
  to={`?page=${page + 1}`}
  prefetch="intent"
  className="px-3 py-2"
>
  Next
</Link>
```

## With LinkButton Component

The LinkButton component should also support prefetch:

```tsx
<LinkButton to="/settings" prefetch="intent">
  Settings
</LinkButton>
```

## When NOT to Use prefetch="intent"

```tsx
// External links - can't prefetch
<a href="https://example.com">External</a>

// Very heavy pages - might waste bandwidth
<Link to="/huge-report" prefetch="none">
  Generate Report
</Link>

// User might not navigate (e.g., in long lists)
<Link to={`/items/${item.id}`} prefetch="viewport">
  {item.name}
</Link>
```

## With NavLink for Active States

```tsx
import { NavLink } from "react-router";

<NavLink
  to="/dashboard"
  prefetch="intent"
  className={({ isActive }) =>
    isActive ? "text-accent-600 font-medium" : "text-neutral-600"
  }
>
  Dashboard
</NavLink>;
```

## Prefetch with State

```tsx
<Link to="/checkout" prefetch="intent" state={{ from: "cart" }}>
  Proceed to Checkout
</Link>
```
