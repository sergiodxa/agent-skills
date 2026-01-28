---
title: Fetcher vs Form
impact: MEDIUM
tags: [forms, fetcher, navigation]
---

# Fetcher vs Form

Use useFetcher for in-place updates (no navigation). Use Form for navigation after submit.

## Why

- `Form`: Triggers navigation, shows loading states, updates URL
- `useFetcher`: No navigation, updates in place, can have multiple pending
- Choosing wrong one creates confusing UX

## When to Use Each

| Use Case                       | Component    |
| ------------------------------ | ------------ |
| Create item and navigate to it | `Form`       |
| Delete item from list          | `useFetcher` |
| Like/favorite toggle           | `useFetcher` |
| Search with URL update         | `Form`       |
| Inline edit                    | `useFetcher` |
| Multi-step wizard              | `Form`       |
| Modal form that closes         | `useFetcher` |

## useFetcher: In-Place Updates

```tsx
// Like button - no navigation needed
function LikeButton({ postId, liked }: { postId: string; liked: boolean }) {
  let fetcher = useFetcher();

  // Optimistic UI
  let isLiked = fetcher.formData
    ? fetcher.formData.get("liked") === "true"
    : liked;

  return (
    <fetcher.Form method="post" action="/api/like">
      <input type="hidden" name="postId" value={postId} />
      <input type="hidden" name="liked" value={String(!isLiked)} />
      <button type="submit">{isLiked ? "Unlike" : "Like"}</button>
    </fetcher.Form>
  );
}
```

## Form: Navigation After Submit

```tsx
// Create form - navigates to new item
function CreatePostForm() {
  let navigation = useNavigation();
  let isSubmitting = navigation.state === "submitting";

  return (
    <Form method="post" action="/posts/new">
      <input name="title" required />
      <textarea name="content" required />
      <Button type="submit" isDisabled={isSubmitting}>
        {isSubmitting ? "Creating..." : "Create Post"}
      </Button>
    </Form>
  );
}
```

## Multiple Fetchers on Same Page

```tsx
// Each item has its own fetcher - they work independently
function ItemList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <ItemRow key={item.id} item={item} />
      ))}
    </ul>
  );
}

function ItemRow({ item }) {
  let fetcher = useFetcher();
  let isDeleting = fetcher.state !== "idle";

  // Hide item immediately when deleting (optimistic)
  if (fetcher.formData?.get("intent") === "delete") {
    return null;
  }

  return (
    <li className={isDeleting ? "opacity-50" : ""}>
      {item.name}
      <fetcher.Form method="post">
        <input type="hidden" name="id" value={item.id} />
        <button type="submit" name="intent" value="delete">
          Delete
        </button>
      </fetcher.Form>
    </li>
  );
}
```

## Fetcher with Custom Action

```tsx
// Submit to different route's action
function QuickOrder({ itemId }: { itemId: string }) {
  let fetcher = useFetcher();

  return (
    <fetcher.Form method="post" action="/api/quick-order">
      <input type="hidden" name="itemId" value={itemId} />
      <input type="number" name="amount" placeholder="Amount" />
      <button type="submit">Buy</button>
    </fetcher.Form>
  );
}
```

## Form with GET (Search)

```tsx
// Search form - updates URL with query
function SearchForm() {
  return (
    <Form method="get" action="/search">
      <input type="search" name="q" placeholder="Search..." />
      <button type="submit">Search</button>
    </Form>
  );
}
// Submitting navigates to /search?q=query
```
