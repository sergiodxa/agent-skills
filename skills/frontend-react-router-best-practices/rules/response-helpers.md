---
title: Use Response Helpers for Resource Routes
impact: LOW
tags: [responses, routes]
---

# Use Response Helpers for Resource Routes

Use `remix-utils/responses` to return typed resource responses.

## Pattern

```ts
import { html, javascript, stylesheet, xml, txt, notModified } from "remix-utils/responses";

export async function loader() {
  if (shouldUseCache) return notModified();
  return html("<h1>Hello</h1>");
}
```

## Rules

1. Use helpers to set proper content types
2. Return `notModified()` for 304 cache hits
