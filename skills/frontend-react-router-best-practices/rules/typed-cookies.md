---
title: Use Typed Cookies for Safety
impact: MEDIUM
tags: [cookies, validation]
---

# Use Typed Cookies for Safety

Use `createTypedCookie` to validate cookie values with a schema.

## Pattern

```ts
import { createCookie } from "react-router";
import { createTypedCookie } from "remix-utils/typed-cookie";
import { z } from "zod";

let cookie = createCookie("returnTo", { path: "/" });
let schema = z.string().url().nullable();

export let returnToCookie = createTypedCookie({ cookie, schema });
```

## Rules

1. Always validate cookie payloads with a schema
2. Use `.nullable()` when you need to clear cookies
