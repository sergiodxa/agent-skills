---
title: Session Middleware
impact: HIGH
tags: [middleware, session]
---

# Session Middleware

Use `createSessionMiddleware` to keep a single session instance per request.

## Why

- Avoids multiple session reads/writes in one request
- Centralizes commit logic
- Keeps loaders/actions consistent

## Pattern

```ts
// app/middleware/session.server.ts
import { createCookieSessionStorage } from "react-router";
import { createSessionMiddleware } from "remix-utils/middleware/session";

let sessionStorage = createCookieSessionStorage({
  cookie: { name: "session", path: "/", sameSite: "lax" },
});

export const [sessionMiddleware, getSession] =
  createSessionMiddleware(sessionStorage);
```

### Optimize Commits

Customize commit behavior so you only write when session data changes:

```ts
import { dequal } from "dequal";

export const [sessionMiddleware, getSession] = createSessionMiddleware(
  sessionStorage,
  (previous, next) => !dequal(previous, next),
);
```

You can also commit only when specific fields change:

```ts
export const [sessionMiddleware, getSession] = createSessionMiddleware(
  sessionStorage,
  (previous, next) => previous.user?.id !== next.user?.id,
);
```

```ts
export const middleware: Route.MiddlewareFunction[] = [sessionMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  let session = await getSession(context);
  session.set("user", { id: "123" });
  return data({ ok: true });
}
```

## Rules

1. Use session middleware on routes that read/write session
2. Use `getSession(context)` inside loaders/actions
3. Customize commit logic to avoid unnecessary cookie writes
