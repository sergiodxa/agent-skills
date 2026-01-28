---
title: Sync Locale in Root Loader and HTML
impact: CRITICAL
tags: [i18n, root, html]
---

# Sync Locale in Root Loader and HTML

Return the detected locale from the root loader and keep `<html lang dir>` in sync.

## Why

- Ensures server and client use the same locale
- Sets correct `lang` and `dir` attributes for accessibility
- Keeps i18next client instance aligned with the server

## Pattern

```tsx
import { data, Outlet } from "react-router";
import { useEffect } from "react";
import type { Route } from "./+types/root";
import { useTranslation } from "react-i18next";
import { getLocale, i18nextMiddleware, localeCookie } from "~/middleware/i18next";

export const middleware = [i18nextMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  let locale = getLocale(context);
  return data(
    { locale },
    { headers: { "Set-Cookie": await localeCookie.serialize(locale) } },
  );
}

export function Layout({ children }: { children: React.ReactNode }) {
  let { i18n } = useTranslation();
  return (
    <html lang={i18n.language} dir={i18n.dir(i18n.language)}>
      <body>{children}</body>
    </html>
  );
}

export default function App({ loaderData: { locale } }: Route.ComponentProps) {
  let { i18n } = useTranslation();
  useEffect(() => {
    if (i18n.language !== locale) i18n.changeLanguage(locale);
  }, [i18n, locale]);

  return <Outlet />;
}
```

## Rules

1. Always return `locale` from the root loader
2. Set `lang` and `dir` on `<html>`
3. Sync client i18n language in a root `useEffect`
