---
title: Localize the 404 Route
impact: MEDIUM
tags: [i18n, routes]
---

# Localize the 404 Route

Provide a not-found route so the middleware runs and the i18n instance is configured.

## Pattern

```tsx
// app/routes/not-found.tsx
import { data, Link } from "react-router";
import { useTranslation } from "react-i18next";

export async function loader() {
  return data(null, { status: 404 });
}

export default function Component() {
  let { t } = useTranslation("notFound");
  return (
    <div>
      <h1>{t("title")}</h1>
      <p>{t("description")}</p>
      <Link to="/">{t("backToHome")}</Link>
    </div>
  );
}
```

## Rules

1. Return a 404 response in the loader
2. Use `useTranslation` with a not-found namespace
3. Ensure the 404 route is registered so middleware runs
