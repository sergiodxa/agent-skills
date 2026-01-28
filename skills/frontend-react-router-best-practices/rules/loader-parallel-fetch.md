---
title: Parallel Data Fetching in Loaders
impact: HIGH
tags: [loader, performance, data-fetching]
---

# Parallel Data Fetching in Loaders

Use Promise.all for parallel data fetching when queries don't depend on each other.

## Why

Sequential fetches add up latency. If you have 3 queries each taking 100ms, sequential = 300ms, parallel = 100ms.

## Bad

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let user = await getUser(request);
  let posts = await getPosts(user.id);
  let comments = await getComments(user.id);
  let notifications = await getNotifications(user.id);
  return data({ user, posts, comments, notifications });
}
```

Each await blocks the next query. Total time = sum of all query times.

## Good

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let user = await getUser(request); // Need user.id first

  const [posts, comments, notifications] = await Promise.all([
    getPosts(user.id),
    getComments(user.id),
    getNotifications(user.id),
  ]);

  return data({ user, posts, comments, notifications });
}
```

Authenticate first (required), then run independent queries in parallel.

## When to Use Sequential

Only use sequential when one query depends on another's result:

```tsx
// Sequential is correct here - need user.id for posts
const user = await getUser(request);
const posts = await getPosts(user.id);

// But these can be parallel since they both just need user.id
const [comments, likes] = await Promise.all([
  getComments(user.id),
  getLikes(user.id),
]);
```
