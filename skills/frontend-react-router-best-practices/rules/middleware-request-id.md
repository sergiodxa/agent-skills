---
title: Request ID Middleware
impact: MEDIUM
tags: [middleware, logging]
---

# Request ID Middleware

Generate a request ID and store it in context for logging/correlation.

## Why

- Correlates logs across loaders/actions
- Supports upstream request IDs
- Makes debugging distributed systems easier

## Pattern

```ts
// app/middleware/request-id.server.ts
import { createRequestIDMiddleware } from "remix-utils/middleware/request-id";

export const [requestIDMiddleware, getRequestID] =
  createRequestIDMiddleware();
```

```ts
export const middleware: Route.MiddlewareFunction[] = [requestIDMiddleware];

export async function loader() {
  let requestId = getRequestID();
  log.info({ requestId }, "request");
  return data({ ok: true });
}
```

## Rules

1. Add request IDs at the root middleware
2. Reuse upstream IDs when available
