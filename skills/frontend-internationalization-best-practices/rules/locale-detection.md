---
title: Prefer Cookie/Session Locale with DB Source of Truth
impact: CRITICAL
tags: [i18n, locale, detection]
---

# Prefer Cookie/Session Locale with DB Source of Truth

Use cookie or session for fast locale access, and fall back to DB when needed.

## Why

- Avoid DB lookups on every request
- Keep a stable locale across server and client
- Still honor user preferences stored in DB

## Pattern

```ts
// app/middleware/i18next.ts
export const [i18nextMiddleware, getLocale] = createI18nextMiddleware({
  detection: {
    supportedLanguages: ["es", "en"],
    fallbackLanguage: "en",
    cookie: localeCookie,
    async findLocale(request) {
      let locale = await getLocaleFromSession(request);
      if (locale) return locale;

      let userLocale = await getLocaleFromDatabase(request);
      return userLocale ?? "en";
    },
  },
  i18next: { resources },
});
```

## Optional: Pathname Locale

If you want `/en/...` routes, you can use `findLocale` to read the first path segment. This is optional and not recommended as the default storage.

```ts
findLocale(request) {
  let locale = new URL(request.url).pathname.split("/")[1];
  return locale;
}
```

## Rules

1. Read locale from cookie/session for speed
2. Use DB as the source of truth when present
3. Only use pathname locales if your routing strategy requires it
