---
title: Context Storage Middleware
impact: HIGH
tags: [middleware, context]
---

# Context Storage Middleware

Store `context` and `request` in AsyncLocalStorage so helpers can access them without args.

## Why

- Avoids passing `context` through many layers
- Enables helper functions outside loaders/actions
- Composes well with other middleware getters

## Pattern

```ts
// app/middleware/context-storage.server.ts
import { createContextStorageMiddleware } from "remix-utils/middleware/context-storage";

export const [contextStorageMiddleware, getContext, getRequest] =
  createContextStorageMiddleware();
```

```ts
export const middleware: Route.MiddlewareFunction[] = [contextStorageMiddleware];

export function getUserFromContext() {
  let context = getContext();
  return context.get(userContext);
}
```

## Rules

1. Use context storage when you need helpers without args
2. Always include it before middleware that relies on it
