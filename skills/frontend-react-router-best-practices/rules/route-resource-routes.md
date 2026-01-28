---
title: Resource Routes
impact: MEDIUM
tags: [routes, api, resource-routes]
---

# Resource Routes

Use resource routes for API-like endpoints without UI.

## Why

- Clean API endpoints within React Router
- No component rendering overhead
- Can return any response type (JSON, files, redirects)
- Reusable across the app with fetcher

## What Makes a Resource Route

A resource route has **no default export** (no component):

```tsx
import { data } from "react-router";

// routes/api.search.tsx - RESOURCE ROUTE
export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);
  let query = url.searchParams.get("q");
  let results = await search(query);
  return data({ results });
}

// No default export = resource route
```

## Common Use Cases

### 1. Search Endpoints

```tsx
import { data } from "react-router";

// routes/api.search.tsx
export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);
  let query = url.searchParams.get("q") || "";
  let limit = Number(url.searchParams.get("limit")) || 10;

  let results = await search(query, limit);

  return data({ results });
}
```

Usage with fetcher:

```tsx
function SearchWidget() {
  let fetcher = useFetcher<typeof searchLoader>();

  return (
    <fetcher.Form action="/api/search" method="get">
      <input name="q" onChange={(e) => fetcher.submit(e.currentTarget.form)} />
      {fetcher.data?.results.map((item) => (
        <SearchItem key={item.id} item={item} />
      ))}
    </fetcher.Form>
  );
}
```

### 2. File Downloads

```tsx
// routes/api.reports.$reportId.download.tsx
export async function loader({ params }: Route.LoaderArgs) {
  let report = await getReport(params.reportId);
  let pdf = await generatePDF(report);

  return new Response(pdf, {
    headers: {
      "Content-Type": "application/pdf",
      "Content-Disposition": `attachment; filename="${report.name}.pdf"`,
    },
  });
}
```

### 3. Image Generation

```tsx
// routes/api.og-image.tsx
export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);
  let title = url.searchParams.get("title") || "Default Title";

  let image = await generateOGImage({ title });

  return new Response(image, {
    headers: {
      "Content-Type": "image/png",
      "Cache-Control": "public, max-age=31536000",
    },
  });
}
```

### 4. Webhooks

```tsx
import { data } from "react-router";

// routes/api.webhooks.provider.tsx
export async function action({ request }: Route.ActionArgs) {
  let payload = await request.text();
  let signature = request.headers.get("x-signature");

  let event = verifyWebhook(payload, signature);

  switch (event.type) {
    case "order.completed":
      await handleOrderComplete(event.data);
      break;
    // ... other events
  }

  return data({ received: true });
}
```

### 5. Form Actions Without Navigation

```tsx
import { data } from "react-router";

// routes/api.toggle-favorite.tsx
export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  let itemId = z.string().parse(formData.get("itemId"));
  let favorited = formData.get("favorited") === "true";

  let count = await toggleFavorite(client, itemId, favorited);

  return data({ count });
}
```

## Naming Convention

```
api.search.tsx               -> /api/search
api.webhooks.provider.tsx    -> /api/webhooks/provider
api.reports.$id.download.tsx -> /api/reports/:id/download
```

## Type-Safe Fetcher

```tsx
import type { loader as searchLoader } from "~/routes/api.search";

function SearchComponent() {
  let fetcher = useFetcher<typeof searchLoader>();

  // fetcher.data is typed from loader return type
  let results = fetcher.data?.results ?? [];
}
```
