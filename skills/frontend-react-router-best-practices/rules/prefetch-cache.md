---
title: Short Cache for Prefetch Requests
impact: LOW
tags: [performance, caching]
---

# Short Cache for Prefetch Requests

Detect prefetch requests and return a short-lived cache header.

## Pattern

```ts
import { isPrefetch } from "remix-utils/is-prefetch";
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let headers = new Headers();
  if (isPrefetch(request)) {
    headers.set("Cache-Control", "private, max-age=5, smax-age=0");
  }
  return data(await getData(), { headers });
}
```

## Rules

1. Use short caches only for prefetch requests
2. Avoid long cache on data that changes per user
