---
title: Colocate Data Queries
impact: MEDIUM
tags: [loader, organization, queries]
---

# Colocate Data Queries

Keep data fetching functions in colocated `queries.server.ts` files next to route files.

## Why

- Keeps loaders clean and focused on orchestration
- Makes queries reusable across loader and action
- Server-only code stays separate from client code
- Easier to test queries in isolation

## File Structure

```
routes/
  my-feature/
    queries.server.ts    # Data fetching functions
    actions.server.ts    # Mutation functions (optional)
    route.tsx            # Loader, action, component
    components/          # Route-specific components
      header.tsx
      item-card.tsx
```

## queries.server.ts

```tsx
// queries.server.ts
import type { APIClient } from "~/lib/api.server";

export async function queryItems(client: APIClient) {
  let items = await client.items.list();
  return items.map((item) => ({
    id: item.id,
    title: item.title,
    slug: item.slug,
  }));
}

export async function queryFeaturedItems(client: APIClient) {
  return client.items.featured();
}

export async function querySuggestedItems(
  client: APIClient,
  existingItems: { id: string }[],
) {
  let suggestions = await client.items.suggested();
  let existingIds = new Set(existingItems.map((i) => i.id));
  return suggestions.filter((s) => !existingIds.has(s.id));
}
```

## route.tsx

```tsx
// route.tsx
import { data } from "react-router";
import {
  queryItems,
  queryFeaturedItems,
  querySuggestedItems,
} from "./queries.server";

export async function loader({ request, context }: Route.LoaderArgs) {
  let client = await authenticate(request);

  const [featuredItems, items] = await Promise.all([
    queryFeaturedItems(client),
    queryItems(client),
  ]);

  let suggestedItems = await querySuggestedItems(client, items);

  return data({ featuredItems, items, suggestedItems });
}
```

## Benefits

1. **Loader stays focused**: Orchestrates auth, parallel fetching, response
2. **Queries are testable**: Can unit test query functions directly
3. **Reusable**: Same query can be used in loader and action
4. **Clear separation**: `.server.ts` suffix ensures server-only bundling
