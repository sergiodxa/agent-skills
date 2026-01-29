---
title: Route Organization
impact: MEDIUM
tags: [routes, organization, file-structure]
---

# Route Organization

Use folder routes with colocated files for complex routes.

## Why

- Keeps related code together
- Easier to navigate large codebases
- Clear separation of concerns
- Server-only code stays separate

## File Structure

```
routes/
  # Simple routes - single file
  login.tsx
  logout.tsx

  # Complex routes - folder with colocated files
  _.my-feature/
    route.tsx            # Loader, action, default export
    schemas.server.ts    # Zod schemas for validation
    queries.server.ts    # Data fetching functions
    actions.server.ts    # Action handlers (optional)
    components/          # Route-specific components
      header.tsx
      item-card.tsx
    assets/              # Route-specific assets
      banner.png

  # Layout routes
  _._app/
    route.tsx            # Layout with Outlet
    components/
      sidebar.tsx

  # Nested routes under layout
  _._app.dashboard/
    route.tsx
    queries.server.ts
    components/
      header.tsx
      activity-feed.tsx
```

## Naming Conventions

| Pattern                     | Meaning                | URL                      |
| --------------------------- | ---------------------- | ------------------------ |
| `_.projects/route.tsx`      | Nested under \_ layout | `/projects`              |
| `_._profile/route.tsx`      | Layout route           | (no URL)                 |
| `_._profile.home/route.tsx` | Child of layout        | `/home`                  |
| `_.projects_.$projectId/`   | Dynamic segment        | `/projects/:projectId`   |
| `api.search.items.tsx`      | Resource route (no UI) | `/api/search/items`      |

## File Naming

```
route.tsx           # Required - loader, action, component
queries.server.ts   # Data fetching (server-only)
actions.server.ts   # Mutation handlers (server-only)
schemas.server.ts   # Zod schemas for params, search, form data
*.server.ts         # Any server-only code
components/         # Route-specific components
hooks/              # Route-specific hooks
assets/             # Images, etc.
```

## Example: Complex Route

```
routes/_.items_.$itemId_.edit/
  route.tsx
  schemas.server.ts
  queries.server.ts
  actions.server.ts
  components/
    toolbar.tsx
    image-gallery.tsx
```

### route.tsx

```tsx
import { data } from "react-router";
import { queryItem, queryToolbar } from "./queries.server";
import { Toolbar } from "./components/toolbar";
import { ImageGallery } from "./components/image-gallery";
import { currentUser } from "~/lib/authorize.server";

export async function loader({ request, params }: Route.LoaderArgs) {
  let user = currentUser();
  let itemId = z.string().parse(params.itemId);

  let [item, toolbar] = await Promise.all([
    queryItem(user.id, itemId),
    queryToolbar(user.id, itemId),
  ]);

  return data({ item, toolbar });
}

export async function action({ request, params }: Route.ActionArgs) {
  // ... uses actions.server.ts
}

export default function Component() {
  const { item, toolbar } = useLoaderData<typeof loader>();

  return (
    <div>
      <Toolbar {...toolbar} />
      <ImageGallery images={item.images} />
      {/* ... */}
    </div>
  );
}
```

### schemas.server.ts

Keep all Zod schemas for the route in one place. Use factory functions that receive `TFunction` for translatable error messages:

```tsx
import type { TFunction } from "i18next";
import { z } from "zod";

// URL params validation (no i18n needed - internal errors)
export const paramsSchema = z.object({
  itemId: z.string(),
});

// Search params validation (no i18n needed - internal errors)
export const searchParamsSchema = z.object({
  page: z.coerce.number().optional().default(1),
  sort: z.enum(["newest", "oldest"]).optional().default("newest"),
  query: z.string().nullish(),
});

// Form data validation - factory function for i18n error messages
export function actionSchema(t: TFunction) {
  return z.discriminatedUnion("intent", [
    z.object({
      intent: z.literal("update"),
      title: z.string().min(1, t("Title is required")),
      amount: z.coerce
        .number({ message: t("Amount must be a number") })
        .positive(t("Amount must be positive"))
        .max(10000, t("Amount cannot exceed {{max}}", { max: 10000 })),
      description: z.string().optional(),
    }),
    z.object({
      intent: z.literal("delete"),
    }),
  ]);
}
```

### queries.server.ts

```tsx
export async function queryItem(userId: string, itemId: string) {
  let item = await fetchItem(userId, itemId);
  return {
    id: item.id,
    title: item.title,
    images: item.images,
  };
}
```

### route.tsx using schemas

```tsx
import {
  paramsSchema,
  searchParamsSchema,
  actionSchema,
} from "./schemas.server";
import { queryItem } from "./queries.server";
import { getInstance } from "~/middleware/i18next";
import { currentUser } from "~/lib/authorize.server";

export async function loader({ request, params }: Route.LoaderArgs) {
  let user = currentUser();
  let { itemId } = paramsSchema.parse(params);

  let url = new URL(request.url);
  let { page, sort } = searchParamsSchema.parse(
    Object.fromEntries(url.searchParams),
  );

  let item = await queryItem(user, itemId);
  return data({ item, page, sort });
}

export async function action({ request, params }: Route.ActionArgs) {
  let user = currentUser();
  let t = getInstance(context).t;
  let { itemId } = paramsSchema.parse(params);
  let formData = await request.formData();

  // Pass t to schema factory for translated error messages
  let body = actionSchema(t).parse(Object.fromEntries(formData.entries()));

  if (body.intent === "update") {
    await updateItem(user, itemId, body);
    throw redirect(`/items/${itemId}`);
  }

  if (body.intent === "delete") {
    await deleteItem(user, itemId);
    throw redirect("/items");
  }
}
```
