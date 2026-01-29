---
title: Use Sec-Fetch Headers to Guard Mutations
impact: HIGH
tags: [security, csrf]
---

# Use Sec-Fetch Headers to Guard Mutations

Use `Sec-Fetch-*` headers to reject cross-site mutation requests in middleware.

## Why

- Browsers send `Sec-Fetch-*` headers for navigations and form submissions
- You can block cross-site requests before actions run
- Complements CSRF defenses without extra tokens

## Pattern

```ts
// app/middleware/sec-fetch.server.ts
import { fetchSite } from "remix-utils/sec-fetch";

export const secFetchMiddleware: Route.MiddlewareFunction = async ({ request }) => {
  let method = request.method.toUpperCase();
  if (method === "GET" || method === "HEAD") return;

  let site = fetchSite(request);
  if (site === "cross-site" || site === "none") {
    throw new Response("Forbidden", { status: 403 });
  }
};
```

## Rules

1. Use Sec-Fetch checks only for state-changing methods
2. Reject `cross-site` and `none` for mutations
3. Apply as middleware so actions never run on bad requests
