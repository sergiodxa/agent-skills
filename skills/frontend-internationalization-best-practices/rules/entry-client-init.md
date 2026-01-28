---
title: Initialize i18next on the Client
impact: CRITICAL
tags: [i18n, client, entry]
---

# Initialize i18next on the Client

Use the HTML `lang` attribute as the client detection source and load namespaces from `/api/locales`.

## Pattern

```tsx
// app/entry.client.tsx
import Fetch from "i18next-fetch-backend";
import i18next from "i18next";
import { startTransition, StrictMode } from "react";
import { hydrateRoot } from "react-dom/client";
import { I18nextProvider, initReactI18next } from "react-i18next";
import { HydratedRouter } from "react-router/dom";
import I18nextBrowserLanguageDetector from "i18next-browser-languagedetector";

async function main() {
  await i18next
    .use(initReactI18next)
    .use(Fetch)
    .use(I18nextBrowserLanguageDetector)
    .init({
      fallbackLng: "en",
      detection: { order: ["htmlTag"], caches: [] },
      backend: { loadPath: "/api/locales/{{lng}}/{{ns}}" },
    });

  startTransition(() => {
    hydrateRoot(
      document,
      <I18nextProvider i18n={i18next}>
        <StrictMode>
          <HydratedRouter />
        </StrictMode>
      </I18nextProvider>,
    );
  });
}

main().catch((error) => console.error(error));
```

## Rules

1. Detect locale from `htmlTag` only (server already decided)
2. Use `/api/locales/{{lng}}/{{ns}}` for resource loading
3. Keep `fallbackLng` aligned with middleware
