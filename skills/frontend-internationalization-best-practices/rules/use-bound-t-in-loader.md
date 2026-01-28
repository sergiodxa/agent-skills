---
title: Use the Bound t() in Loaders
impact: MEDIUM
tags: [i18n, loaders]
---

# Use the Bound t() in Loaders

Use the middleware instance’s bound `t()` in loaders and keep UI hooks for components.

## Pattern

```ts
import { getInstance } from "~/middleware/i18next";

export async function loader({ context }: Route.LoaderArgs) {
  let t = getInstance(context).t;
  return { title: t("title"), description: t("description") };
}
```

```tsx
import { useTranslation } from "react-i18next";

export default function Component() {
  let { t } = useTranslation();
  return <h1>{t("title")}</h1>;
}
```

## Rules

## Namespaced t()

If you need a specific namespace, use `getFixedT` with the current locale and namespace:

```ts
export async function loader({ context }: Route.LoaderArgs) {
  let i18n = getInstance(context);
  let t = i18n.getFixedT(i18n.language, "notFound");
  return { title: t("title") };
}
```

## Rules

1. Use the bound `t()` from `getInstance(context)` in loaders
2. Use `getFixedT(locale, namespace)` when you need a specific namespace
3. Use `useTranslation` in components for client rendering
4. Avoid mixing different locales in the same loader
