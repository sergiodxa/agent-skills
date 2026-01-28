---
title: Build a Language Switcher
impact: HIGH
tags: [i18n, locale, cookies]
---

# Build a Language Switcher

Persist locale in cookie or session so users keep their preference across requests.

## Pattern

```tsx
// app/routes/_index.tsx
import { data } from "react-router";
import type { Route } from "./+types/_index";
import { localeCookie, getLocale } from "~/middleware/i18next";

export async function loader({ context }: Route.LoaderArgs) {
  let locale = getLocale(context);
  return data(
    { locale },
    { headers: { "Set-Cookie": await localeCookie.serialize(locale) } },
  );
}
```

```tsx
export function LanguageSwitcher() {
  return (
    <div>
      <a href="/?lng=en">English</a>
      <a href="/?lng=es">Espanol</a>
    </div>
  );
}
```

## Rules

1. Store locale in cookie/session when user switches languages
2. Keep URLs stable; use query params or UI state for the switch
3. Prefer cookie/session over DB lookups on each request
