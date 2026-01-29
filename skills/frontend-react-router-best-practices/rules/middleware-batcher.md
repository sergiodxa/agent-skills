---
title: Batcher Middleware
impact: HIGH
tags: [middleware, performance]
---

# Batcher Middleware

Use `createBatcherMiddleware` for request-scoped deduping of expensive calls.

## Why

- Avoids duplicate DB/API calls within one request
- Keeps code composable across loaders/actions
- Simplifies request-level caching

## Pattern

```ts
// app/middleware/batcher.server.ts
import { createBatcherMiddleware } from "remix-utils/middleware/batcher";
import { getContext } from "~/middleware/context-storage.server";

export const [batcherMiddleware, getBatcherFromContext] =
  createBatcherMiddleware();

export function getBatcher() {
  return getBatcherFromContext(getContext());
}
```

```ts
export const middleware: Route.MiddlewareFunction[] = [batcherMiddleware];

export async function loader() {
  let batcher = getBatcher();
  let result = await batcher.batch("key", async () => getData());
  return data({ result });
}
```

## Rules

1. Use a stable batch key per request
2. Prefer batching to ad-hoc in-memory caches
