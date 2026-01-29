---
title: Logger Middleware
impact: LOW
tags: [middleware, logging]
---

# Logger Middleware

Log request/response details consistently with `createLoggerMiddleware`.

## Why

- Standardizes request logs
- Measures response time
- Reduces ad-hoc logging

## Pattern

```ts
// app/middleware/logger.server.ts
import { createLoggerMiddleware } from "remix-utils/middleware/logger";

export const [loggerMiddleware] = createLoggerMiddleware({
  precision: 2,
});
```

```ts
export const middleware: Route.MiddlewareFunction[] = [loggerMiddleware];
```

## Rules

1. Use logger middleware for consistent request logs
2. Customize format if you need structured logs
