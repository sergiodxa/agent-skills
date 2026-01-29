---
title: Use CORS Helper for API Routes
impact: MEDIUM
tags: [cors, security]
---

# Use CORS Helper for API Routes

Apply CORS headers with `cors()` in loaders/actions that serve cross-origin clients.

## Pattern

```ts
import { cors } from "remix-utils/cors";
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let response = data(await getData());
  return await cors(request, response);
}
```

## Rules

1. Use `cors()` only on routes meant for cross-origin access
2. Keep CORS configuration explicit (origin/methods/headers)
