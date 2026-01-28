---
title: Prefetch Data for Fetcher Usage
impact: LOW
tags: [prefetch, fetcher, performance]
---

# Prefetch Data for Fetcher Usage

Use `<PrefetchPageLinks>` to preload data for routes that will be loaded via `fetcher.load()`.

## Why

- `<Link prefetch="intent">` only works for navigation, not fetcher calls
- Modals, drawers, and expandable sections often load data on demand
- Preloading this data improves perceived performance
- Data is ready when the user clicks, making the UI feel instant

## The Problem

When using `fetcher.load()` to load data (e.g., for a modal), the data fetch only starts when the user triggers the action:

```tsx
function ItemDetails({ itemId }: { itemId: string }) {
  let fetcher = useFetcher<typeof resourceLoader>();

  return (
    <>
      <button onClick={() => fetcher.load(`/api/items/${itemId}`)}>
        View Details
      </button>
      {fetcher.data && <Modal data={fetcher.data} />}
    </>
  );
}
```

The user clicks, waits for data to load, then sees the modal.

## Solution: PrefetchPageLinks

Render `<PrefetchPageLinks>` to start preloading the data immediately:

```tsx
import { useFetcher, PrefetchPageLinks } from "react-router";

function ItemDetails({ itemId }: { itemId: string }) {
  let fetcher = useFetcher<typeof resourceLoader>();

  return (
    <>
      {/* Preload data for this resource route */}
      <PrefetchPageLinks page={`/api/items/${itemId}`} />

      <button onClick={() => fetcher.load(`/api/items/${itemId}`)}>
        View Details
      </button>
      {fetcher.data && <Modal data={fetcher.data} />}
    </>
  );
}
```

When the component renders, it starts prefetching. By the time the user clicks, data may already be cached.

## How It Works

`<PrefetchPageLinks>` renders `<link rel="prefetch">` tags in the document head:

```html
<link
  rel="prefetch"
  as="fetch"
  href="/api/items/123?_data=routes/api.items.$itemId"
/>
```

The browser prefetches the data in the background. When `fetcher.load()` is called, it uses the cached response.

## Use Cases

### Modal Data Preloading

```tsx
function UserRow({ user }: { user: User }) {
  let [isOpen, setIsOpen] = useState(false);
  let fetcher = useFetcher<typeof userDetailsLoader>();

  return (
    <tr>
      <PrefetchPageLinks page={`/api/users/${user.id}/details`} />
      <td>{user.name}</td>
      <td>
        <button
          onClick={() => {
            fetcher.load(`/api/users/${user.id}/details`);
            setIsOpen(true);
          }}
        >
          View Profile
        </button>
      </td>
      {isOpen && fetcher.data && (
        <UserProfileModal
          user={fetcher.data}
          onClose={() => setIsOpen(false)}
        />
      )}
    </tr>
  );
}
```

### Expandable Sections

```tsx
function AccordionItem({ sectionId, title }: Props) {
  let [isExpanded, setIsExpanded] = useState(false);
  let fetcher = useFetcher<typeof sectionLoader>();

  return (
    <div>
      <PrefetchPageLinks page={`/api/sections/${sectionId}`} />
      <button
        onClick={() => {
          if (!fetcher.data) fetcher.load(`/api/sections/${sectionId}`);
          setIsExpanded((isExpanded) => !isExpanded);
        }}
      >
        {title}
      </button>
      {isExpanded && fetcher.data && <div>{fetcher.data.content}</div>}
    </div>
  );
}
```

## Conditional Prefetching

Only prefetch when likely to be used:

```tsx
function ItemCard({ item, isHovered }: Props) {
  let fetcher = useFetcher<typeof detailsLoader>();

  return (
    <div>
      {/* Only prefetch when user hovers */}
      {isHovered && <PrefetchPageLinks page={`/api/items/${item.id}`} />}
      <button onClick={() => fetcher.load(`/api/items/${item.id}`)}>
        Details
      </button>
    </div>
  );
}
```

## When to Use

| Scenario                    | Use                        |
| --------------------------- | -------------------------- |
| Navigation to another page  | `<Link prefetch="intent">` |
| Modal/drawer data loading   | `<PrefetchPageLinks>`      |
| Expandable section content  | `<PrefetchPageLinks>`      |
| Data loaded on button click | `<PrefetchPageLinks>`      |

## Rules

1. Use `<PrefetchPageLinks page="/path">` for `fetcher.load()` prefetching
2. Use `<Link prefetch="intent">` for navigation prefetching
3. Consider conditional rendering to avoid prefetching data that won't be used
4. The `page` prop should match the URL passed to `fetcher.load()`
5. Works with resource routes that export a `loader`
