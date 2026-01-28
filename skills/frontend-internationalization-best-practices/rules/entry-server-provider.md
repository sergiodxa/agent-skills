---
title: Reuse i18next Instance on the Server
impact: CRITICAL
tags: [i18n, server, entry]
---

# Reuse i18next Instance on the Server

Wrap SSR with `I18nextProvider` using the instance created by the middleware.

## Pattern

```tsx
// app/entry.server.tsx
import type { EntryContext, RouterContextProvider } from "react-router";
import { ServerRouter } from "react-router";
import { I18nextProvider } from "react-i18next";
import { getInstance } from "~/middleware/i18next";

export default function handleRequest(
  request: Request,
  responseStatusCode: number,
  responseHeaders: Headers,
  entryContext: EntryContext,
  routerContext: RouterContextProvider,
) {
  return (
    <I18nextProvider i18n={getInstance(routerContext)}>
      <ServerRouter context={entryContext} url={request.url} />
    </I18nextProvider>
  );
}
```

## Rules

1. Use `getInstance(routerContext)` from the middleware
2. Avoid creating a new i18next instance on the server
