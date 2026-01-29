---
title: Protect Public Forms with Honeypot
impact: MEDIUM
tags: [forms, spam, security]
---

# Protect Public Forms with Honeypot

Use `remix-utils` honeypot to block basic spam bots on public forms.

## Pattern

```ts
// app/middleware/honeypot.server.ts
import { createHoneypotMiddleware } from "remix-utils/middleware/honeypot";

export const [honeypotMiddleware, getHoneypotInputProps] =
  createHoneypotMiddleware();
```

```tsx
// app/root.tsx
import { HoneypotProvider } from "remix-utils/honeypot/react";

export const middleware: Route.MiddlewareFunction[] = [honeypotMiddleware];

export async function loader() {
  let honeypotInputProps = await getHoneypotInputProps();
  return data({ honeypotInputProps });
}

export default function App({ loaderData }: Route.ComponentProps) {
  return (
    <HoneypotProvider {...loaderData.honeypotInputProps}>
      <Outlet />
    </HoneypotProvider>
  );
}
```

```tsx
// Any public form
import { HoneypotInputs } from "remix-utils/honeypot/react";

<Form method="post">
  <HoneypotInputs />
  {/* form fields */}
</Form>;
```

## Rules

1. Add honeypot inputs to public forms
2. Run honeypot middleware on routes that accept public posts
3. Combine with rate limiting for stronger protection
