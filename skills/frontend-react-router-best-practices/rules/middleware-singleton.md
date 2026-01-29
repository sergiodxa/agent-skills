---
title: Singleton Middleware
impact: MEDIUM
tags: [middleware, caching]
---

# Singleton Middleware

Create a per-request singleton for caches or shared services.

## Why

- Ensures one instance per request
- Avoids global singletons leaking across users
- Simplifies per-request caches

## Pattern

```ts
// app/middleware/singleton.server.ts
import { createSingletonMiddleware } from "remix-utils/middleware/singleton";

export const [singletonMiddleware, getSingleton] =
  createSingletonMiddleware({
    instantiator: () => new RequestCache(),
  });
```

```ts
export const middleware: Route.MiddlewareFunction[] = [singletonMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  let cache = getSingleton(context);
  let value = await cache.get("key");
  return data({ value });
}
```

## Rules

1. Use singleton middleware for request-scoped caches
2. Avoid global singletons for per-request data
