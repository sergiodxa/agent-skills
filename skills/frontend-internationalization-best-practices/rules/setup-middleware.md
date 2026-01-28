---
title: Set Up remix-i18next Middleware
impact: CRITICAL
tags: [i18n, middleware, setup]
---

# Set Up remix-i18next Middleware

Use `createI18nextMiddleware` as the single source of locale detection and i18next configuration.

## Why

- Centralizes locale detection for loaders and actions
- Reuses one i18next instance per request
- Enables type-safe `t()` across the app

## Pattern

```ts
// app/middleware/i18next.ts
import { initReactI18next } from "react-i18next";
import { createCookie } from "react-router";
import { createI18nextMiddleware } from "remix-i18next/middleware";
import resources from "~/locales";
import "i18next";

export const localeCookie = createCookie("lng", {
  path: "/",
  sameSite: "lax",
  secure: process.env.NODE_ENV === "production",
  httpOnly: true,
});

export const [i18nextMiddleware, getLocale, getInstance] =
  createI18nextMiddleware({
    detection: {
      supportedLanguages: ["es", "en"],
      fallbackLanguage: "en",
      cookie: localeCookie,
    },
    i18next: { resources },
    plugins: [initReactI18next],
  });

declare module "i18next" {
  interface CustomTypeOptions {
    defaultNS: "translation";
    resources: typeof resources.en;
  }
}
```

## Rules

1. Use middleware as the only locale detection entry point
2. Keep `supportedLanguages` and `fallbackLanguage` explicit
3. Add `CustomTypeOptions` for typed `t()` in TS
