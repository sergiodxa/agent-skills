---
title: Choose a Namespace Strategy
impact: HIGH
tags: [i18n, namespaces]
---

# Choose a Namespace Strategy

Use a single namespace for small apps, and multiple namespaces for large apps.

## Why

- Single namespace is simplest for small codebases
- Multiple namespaces keep large apps modular and faster to load
- Makes route-level translations easier to reason about

## Single Namespace (Small Apps)

```ts
// app/locales/en/translation.ts
export default {
  home: {
    title: "Home",
    description: "Welcome",
  },
  settings: {
    title: "Settings",
  },
};
```

## Multiple Namespaces (Large Apps)

```ts
// app/locales/en/common.ts
export default { appName: "Example" };

// app/locales/en/home.ts
export default { title: "Home" };

// app/locales/en/index.ts
import type { ResourceLanguage } from "i18next";
import common from "./common";
import home from "./home";

export default { common, home } satisfies ResourceLanguage;
```

## Rules

1. Use a single `translation` namespace for small apps
2. Use per-route namespaces (`home`, `notFound`, etc.) for large apps
3. Keep namespace names stable to simplify caching
