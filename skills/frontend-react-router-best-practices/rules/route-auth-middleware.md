---
title: Auth Middleware + Route Authorization
impact: HIGH
tags: [auth, middleware, loaders, actions]
---

# Auth Middleware + Route Authorization

Authenticate in middleware and authorize inside each loader/action.

## Why

- Authentication is a cross-cutting concern (session → user) and belongs in middleware
- Authorization is per-route/per-action and should be explicit at the call site
- Centralizes checks while keeping route intent clear

## Pattern

### Contexts

```ts
// app/context.ts
import { createContext } from "react-router";
import type { User } from "~/lib/users.server";

export const userContext = createContext<User | null>(null);
```

### Context storage middleware

Use `remix-utils` to store the router context and request in AsyncLocalStorage so authorization helpers don't need `context` or `request` arguments.

```ts
// app/middleware/context-storage.server.ts
import { createContextStorageMiddleware } from "remix-utils/middleware/context-storage";

export const [contextStorageMiddleware, getContext, getRequest] =
  createContextStorageMiddleware();
```

### Session middleware (remix-utils)

```ts
// app/middleware/session.server.ts
import { createCookie, createCookieSessionStorage } from "react-router";
import { createSessionMiddleware } from "remix-utils/middleware/session";

let sessionStorage = createCookieSessionStorage({
  cookie: createCookie("session", { path: "/", sameSite: "lax" }),
});

export const [sessionMiddleware, getSession] =
  createSessionMiddleware(sessionStorage);
```

### Auth middleware (loads user into context)

```ts
// app/middleware/auth.server.ts
import { getSession } from "~/middleware/session.server";
import { getUserById } from "~/lib/users.server";
import { userContext } from "~/context";

export function authenticate({ context }: MiddlewareFunctionArgs<Response>) {
  let session = getSession(context);
  let userId = session?.get("userId");
  if (userId) context.set(userContext, await getUserById(userId));
}
```

### Authorization helpers (per loader/action)

```ts
// app/lib/authorize.server.ts
import { redirect } from "react-router";
import { userContext } from "~/context";
import { getContext, getRequest } from "~/middleware/context-storage.server";
import { getSession } from "~/middleware/session.server";

function getUser() {
  return getContext().get(userContext);
}

export function anonymous() {
  let user = getUser();
  if (user) throw redirect("/");
}

export function currentUser() {
  let user = getUser();
  if (user) return user;
  let request = getRequest();
  let returnTo = new URL(request.url).pathname;
  throw redirect(`/login?returnTo=${encodeURIComponent(returnTo)}`);
}

export function onboarded() {
  let user = currentUser();
  return Boolean(user.onboardingComplete);
}

export function role(name: string) {
  let user = currentUser();
  return Boolean(user.role === name);
}

export function permission(name: string) {
  let user = currentUser();
  return Boolean(user && user.permissions?.includes(name));
}

export function feature(flag: string) {
  let user = currentUser();
  return Boolean(user && user.featureFlags?.includes(flag));
}

export function mfa(maxAgeMs = 60 * 60 * 1000) {
  let session = getSession(getContext());
  if (!session) return false;

  let verifiedAt = session.get("mfaVerifiedAt");
  if (!verifiedAt) return false;

  let verifiedAtMs =
    typeof verifiedAt === "number"
      ? verifiedAt
      : Date.parse(String(verifiedAt));
  if (Number.isNaN(verifiedAtMs)) return false;

  return Date.now() - verifiedAtMs <= maxAgeMs;
}
```

### Use in routes

```ts
// app/routes/settings/route.tsx
import { contextStorageMiddleware } from "~/middleware/context-storage.server";
import { sessionMiddleware } from "~/middleware/session.server";
import { authenticate } from "~/middleware/auth.server";
import {
  anonymous,
  currentUser,
  feature,
  mfa,
  onboarded,
  permission,
  role,
} from "~/lib/authorize.server";

export const middleware: Route.MiddlewareFunction[] = [
  contextStorageMiddleware,
  sessionMiddleware,
  authenticate,
];

export async function loader() {
  let user = currentUser();
  if (onboarded()) return data({ user });
  throw redirect("/onboarding");
}

export async function action() {
  let user = currentUser();

  if (!permission("account:delete")) {
    throw redirect("/forbidden");
  }

  if (!mfa(5 * 60 * 1000)) {
    throw redirect("/confirm-2fa");
  }

  // ... handle mutation
  return null;
}
```

## Rules

1. Always load session/user in middleware, not inside loaders/actions
2. Use context storage middleware so helpers don't need args
3. Use `currentUser()` for gated pages and `anonymous()` for auth pages
4. Use `role()`/`permission()` for privileged actions
5. Enforce `feature()`, `onboarded()`, and `mfa()` when required
