---
title: Structure Locales by Language
impact: HIGH
tags: [i18n, locales, structure]
---

# Structure Locales by Language

Organize translations under `app/locales/{lng}` and re-export them in a single resource map.

## Why

- Keeps locale resources explicit and discoverable
- Enables type-safe parity checks between languages
- Makes server and client loaders consistent

## Pattern

```ts
// app/locales/en/translation.ts
export default {
  title: "Example",
  description: "A React Router + remix-i18next example",
};

// app/locales/en/index.ts
import type { ResourceLanguage } from "i18next";
import translation from "./translation";

export default { translation } satisfies ResourceLanguage;
```

```ts
// app/locales/es/translation.ts
export default {
  title: "Ejemplo",
  description: "Un ejemplo de React Router + remix-i18next",
} satisfies typeof import("~/locales/en/translation").default;

// app/locales/es/index.ts
import type { ResourceLanguage } from "i18next";
import translation from "./translation";

export default { translation } satisfies ResourceLanguage;
```

```ts
// app/locales/index.ts
import type { Resource } from "i18next";
import en from "./en";
import es from "./es";

export default { en, es } satisfies Resource;
```

## Rules

1. Keep `en` as the default source of truth for keys
2. Use `satisfies` to enforce parity between locales
3. Export a single `resources` object for middleware
