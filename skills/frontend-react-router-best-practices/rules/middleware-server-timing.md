---
title: Server Timing Middleware
impact: LOW
tags: [middleware, performance]
---

# Server Timing Middleware

Add `Server-Timing` headers to measure loader/action performance.

## Why

- Inspect server timings in browser devtools
- Identify slow loaders/actions
- Track performance regressions

## Pattern

```ts
// app/middleware/server-timing.server.ts
import { createServerTimingMiddleware } from "remix-utils/middleware/server-timing";

export const [serverTimingMiddleware, getTimingCollector] =
  createServerTimingMiddleware();
```

```ts
export const middleware: Route.MiddlewareFunction[] = [serverTimingMiddleware];

export async function loader() {
  let timing = getTimingCollector();
  return timing.measure("load-data", "Load data", async () => {
    return data(await getData());
  });
}
```

## Rules

1. Add server timing middleware in root
2. Wrap expensive calls in `measure`
