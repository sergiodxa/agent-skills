---
title: URL Parameter Validation
impact: MEDIUM
tags: [loader, validation, zod, params]
---

# URL Parameter Validation

Validate URL params with zod at the start of loaders.

## Why

URL params are user input. Validate early to fail fast with clear errors and ensure type safety.

## Bad

```tsx
import { data } from "react-router";

export async function loader({ params }: Route.LoaderArgs) {
  // params.id is string | undefined - could be missing or invalid
  let item = await getItem(params.id);
  return data({ item });
}
```

## Good

```tsx
import { data } from "react-router";
import { z } from "zod";

export async function loader({ params }: Route.LoaderArgs) {
  let itemId = z.string().parse(params.itemId);
  let item = await getItem(itemId);
  return data({ item });
}
```

## With Multiple Params

```tsx
import { data } from "react-router";
import { z } from "zod";

export async function loader({ params }: Route.LoaderArgs) {
  const { userId, postId } = z
    .object({
      userId: z.string(),
      postId: z.string().uuid(),
    })
    .parse(params);

  let post = await getPost(userId, postId);
  return data({ post });
}
```

## With Search Params

```tsx
import { data } from "react-router";
import { z } from "zod";

export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);

  const { page, limit, sort } = z
    .object({
      page: z.coerce.number().min(1).default(1),
      limit: z.coerce.number().min(1).max(100).default(20),
      sort: z.enum(["newest", "oldest", "popular"]).default("newest"),
    })
    .parse(Object.fromEntries(url.searchParams));

  let items = await getItems({ page, limit, sort });
  return data({ items, page, limit, sort });
}
```
