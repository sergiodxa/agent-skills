---
title: Use Safe Redirect Helpers
impact: MEDIUM
tags: [security, redirects]
---

# Use Safe Redirect Helpers

Protect user-driven redirects with `safeRedirect` and prefer `redirectBack` for post actions.

## Pattern

```ts
import { redirect } from "react-router";
import { safeRedirect } from "remix-utils/safe-redirect";

export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);
  let redirectTo = url.searchParams.get("redirectTo");
  return redirect(safeRedirect(redirectTo, "/"));
}
```

```ts
import { redirectBack } from "remix-utils/redirect-back";

export async function action({ request }: Route.ActionArgs) {
  throw redirectBack(request, { fallback: "/" });
}
```

## Rules

1. Never redirect to raw user input
2. Use `safeRedirect` for query-based redirects
3. Use `redirectBack` for post/put/delete actions
