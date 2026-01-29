---
title: Use getClientIPAddress Safely
impact: LOW
tags: [http, headers]
---

# Use getClientIPAddress Safely

Extract client IP from trusted proxy headers when you need it.

## Pattern

```ts
import { getClientIPAddress } from "remix-utils/get-client-ip-address";

export async function loader({ request }: Route.LoaderArgs) {
  let ip = getClientIPAddress(request) ?? "unknown";
  return { ip };
}
```

## Rules

1. Expect `null` in local development
2. Only trust headers from known proxies
