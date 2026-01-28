---
title: Serve Locales from a Resource Route
impact: HIGH
tags: [i18n, routes, caching]
---

# Serve Locales from a Resource Route

Expose `/api/locales/:lng/:ns` to load translation resources with caching.

## Pattern

```ts
// app/routes/api.locales.$lng.$ns.ts
import { data } from "react-router";
import { cacheHeader } from "pretty-cache-header";
import { z } from "zod";
import resources from "~/locales";
import type { Route } from "./+types/api.locales.$lng.$ns";

export async function loader({ params }: Route.LoaderArgs) {
  const lng = z
    .enum(Object.keys(resources) as Array<keyof typeof resources>)
    .safeParse(params.lng);

  if (lng.error) return data({ error: lng.error }, { status: 400 });

  const namespaces = resources[lng.data];

  const ns = z
    .enum(Object.keys(namespaces) as Array<keyof typeof namespaces>)
    .safeParse(params.ns);

  if (ns.error) return data({ error: ns.error }, { status: 400 });

  const headers = new Headers();
  if (process.env.NODE_ENV === "production") {
    headers.set(
      "Cache-Control",
      cacheHeader({
        maxAge: "5m",
        sMaxage: "1d",
        staleWhileRevalidate: "7d",
        staleIfError: "7d",
      }),
    );
  }

  return data(namespaces[ns.data], { headers });
}
```

## Rules

1. Validate `lng` and `ns` before returning data
2. Cache locale resources in production
3. Keep the route aligned with client `loadPath`
