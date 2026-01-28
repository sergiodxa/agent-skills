---
title: Migrate from namedAction to Intent Pattern
impact: MEDIUM
tags: [migration, action, forms]
---

# Migrate from namedAction to Intent Pattern

Replace `namedAction` from remix-utils with `z.discriminatedUnion` for type-safe intent validation.

## Why

- No external dependency needed
- Type-safe validation of intent and its associated fields
- Better TypeScript inference after parsing
- Single validation step for all form data

## Migration Pattern

### Before (namedAction)

```tsx
import { data } from "react-router";
import { namedAction } from "remix-utils/named-action";

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  return namedAction(formData, {
    async create() {
      let validated = createSchema.parse({ ... });
      await createItem(client, validated);
      return data({ success: true });
    },

    async update() {
      let validated = updateSchema.parse({ ... });
      await updateItem(client, validated);
      return data({ success: true });
    },

    async delete() {
      let id = z.string().parse(formData.get("id"));
      await deleteItem(client, id);
      return data({ success: true });
    },
  });
}
```

### After (z.discriminatedUnion)

```tsx
import { data, redirect } from "react-router";
import { z } from "zod";

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  let body = z
    .discriminatedUnion("intent", [
      z.object({
        intent: z.literal("create"),
        title: z.string().min(1),
        amount: z.coerce.number().positive(),
      }),
      z.object({
        intent: z.literal("update"),
        id: z.string(),
        title: z.string().min(1),
      }),
      z.object({
        intent: z.literal("delete"),
        id: z.string(),
      }),
    ])
    .parse(Object.fromEntries(formData.entries()));

  if (body.intent === "create") {
    await createItem(client, body); // body is typed with title, amount
    throw redirect("/items");
  }

  if (body.intent === "update") {
    await updateItem(client, body); // body is typed with id, title
    throw redirect("/items");
  }

  if (body.intent === "delete") {
    await deleteItem(client, body.id); // body is typed with id
    throw redirect("/items");
  }
}
```

## Forms Stay the Same

No changes needed to form markup - both patterns use the same `intent` field:

```tsx
<fetcher.Form method="post">
  <input type="hidden" name="intent" value="create" />
  <input name="title" />
  <button type="submit">Create</button>
</fetcher.Form>
```

## Button with Intent

Use button's name/value attributes for intent instead of hidden input:

```tsx
<fetcher.Form method="post">
  <input type="hidden" name="id" value={item.id} />
  <Button type="submit" name="intent" value="archive">
    Archive
  </Button>
  <Button type="submit" name="intent" value="delete" color="danger">
    Delete
  </Button>
</fetcher.Form>
```

## With Extracted Handlers

### Before

```tsx
// actions.server.ts
import { namedAction } from "remix-utils/named-action";

export function handleActions(formData: FormData, client: ApiClient) {
  return namedAction(formData, {
    async create() { ... },
    async update() { ... },
    async delete() { ... },
  });
}

// route.tsx
export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();
  return handleActions(formData, client);
}
```

### After

```tsx
// actions.server.ts
import type { APIClient } from "~/lib/api.server";
import { Logger } from "~/lib/logger.server";

const logger = Logger.getLogger("routes/items/actions.server");

export async function createItem(
  client: APIClient,
  data: { title: string; amount: number },
) {
  try {
    return await client.items.create(data);
  } catch (error) {
    logger.error(error);
    throw error;
  }
}

export async function deleteItem(client: APIClient, id: string) {
  try {
    await client.items.delete(id);
  } catch (error) {
    logger.error(error);
    throw error;
  }
}

// route.tsx
import { createItem, deleteItem } from "./actions.server";

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  let body = z
    .discriminatedUnion("intent", [
      z.object({
        intent: z.literal("create"),
        title: z.string().min(1),
        amount: z.coerce.number().positive(),
      }),
      z.object({
        intent: z.literal("delete"),
        id: z.string(),
      }),
    ])
    .parse(Object.fromEntries(formData.entries()));

  if (body.intent === "create") {
    await createItem(client, body);
    throw redirect("/items");
  }

  if (body.intent === "delete") {
    await deleteItem(client, body.id);
    throw redirect("/items");
  }
}
```

## Error Handling

Re-throw `Response` objects (redirects), return validation errors:

```tsx
export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  try {
    let body = z
      .discriminatedUnion("intent", [
        z.object({
          intent: z.literal("create"),
          title: z.string().min(1),
        }),
      ])
      .parse(Object.fromEntries(formData.entries()));

    if (body.intent === "create") {
      await createItem(client, body);
      throw redirect("/items");
    }
  } catch (error) {
    // Let redirects bubble up
    if (error instanceof Response) throw error;

    if (error instanceof z.ZodError) {
      return data(
        { errors: error.issues.map(({ message }) => message) },
        { status: 400 },
      );
    }

    throw error; // Re-throw unknown errors
  }
}
```
