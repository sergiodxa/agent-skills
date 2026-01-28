---
title: Client-Side Validation with clientAction
impact: MEDIUM
tags: [action, validation, clientAction, progressive-enhancement]
---

# Client-Side Validation with clientAction

Use `clientAction` to validate forms on the client before hitting the server. This provides instant feedback while maintaining server-side validation as the source of truth.

## Why

- Instant validation feedback without server roundtrip
- Progressive enhancement: works without JS (server validates)
- Same Zod schema can validate on both client and server
- Reduces unnecessary server requests for invalid data

## Progressive Enhancement Layers

Build validation in layers, each enhancing the previous:

1. **Server-side validation** (always required - never trust the client)
2. **HTML5 validation** (`required`, `type="email"`, `minLength`, etc.)
3. **clientAction validation** (instant Zod validation after JS loads)

## Implementation

### Server Action (Source of Truth)

```tsx
// schemas.server.ts
import { z } from "zod";

export const createPostSchema = z.object({
  title: z.string().min(1, "Title is required").max(100),
  content: z.string().min(1, "Content is required"),
});

// route.tsx
export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();

  let result = createPostSchema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return data(
      { ok: false, errors: result.error.flatten().fieldErrors },
      { status: 400 },
    );
  }

  await createPost(result.data);
  return data({ ok: true });
}
```

### Client Action (Fast Feedback)

```tsx
// route.tsx
export async function clientAction({
  request,
  serverAction,
}: Route.ClientActionArgs) {
  let formData = await request.clone().formData();

  // Validate on client first
  let result = createPostSchema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    // Return errors without hitting server
    return data(
      { ok: false, errors: result.error.flatten().fieldErrors },
      { status: 400 },
    );
  }

  // Validation passed, call server action
  return serverAction<typeof action>();
}
```

### Form Component

```tsx
export default function Component() {
  let actionData = useActionData<typeof action>();

  return (
    <Form method="post">
      <div>
        <label htmlFor="title">Title</label>
        <input
          id="title"
          name="title"
          required              {/* HTML5 validation */}
          maxLength={100}       {/* HTML5 validation */}
          aria-invalid={actionData?.errors?.title ? true : undefined}
          aria-describedby={actionData?.errors?.title ? "title-error" : undefined}
        />
        {actionData?.errors?.title && (
          <p id="title-error" role="alert">{actionData.errors.title[0]}</p>
        )}
      </div>

      <div>
        <label htmlFor="content">Content</label>
        <textarea
          id="content"
          name="content"
          required
          aria-invalid={actionData?.errors?.content ? true : undefined}
        />
        {actionData?.errors?.content && (
          <p role="alert">{actionData.errors.content[0]}</p>
        )}
      </div>

      <button type="submit">Create Post</button>
    </Form>
  );
}
```

## How It Works

1. **Without JS**: Form submits to server, server validates, returns errors
2. **With JS + invalid data**: clientAction validates, returns errors instantly (no server call)
3. **With JS + valid data**: clientAction validates, passes, calls serverAction

## Sharing Schemas

Keep schemas in a shared location so both client and server use the same validation:

```
routes/
  _.posts.new/
    schemas.ts          # Shared schema (not .server.ts!)
    route.tsx           # Uses schema in action and clientAction
```

```tsx
// schemas.ts (no .server suffix - needs to run on client too)
import { z } from "zod";

export const createPostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1),
});
```

## With i18n

For translated error messages, you may need different approaches for client vs server:

```tsx
// Server action - has access to i18n
export async function action({ request }: Route.ActionArgs) {
  let t = await i18next.getFixedT(request);
  let schema = createPostSchema(t); // Schema factory with translations
  // ...
}

// Client action - simpler approach
export async function clientAction({
  request,
  serverAction,
}: Route.ClientActionArgs) {
  let result = createPostSchemaBasic.safeParse(/* ... */);
  if (!result.success) {
    // Return generic errors, server will return translated ones if needed
    return data(
      { ok: false, errors: result.error.flatten().fieldErrors },
      { status: 400 },
    );
  }
  return serverAction<typeof action>();
}
```

## Rules

1. Always validate on the server - clientAction is an enhancement, not a replacement
2. Use the same Zod schema for client and server when possible
3. Keep schemas in non-.server.ts files if clientAction needs them
4. Add HTML5 validation attributes for no-JS fallback
5. clientAction should call `serverAction()` after validation passes
6. Return the same error shape from both client and server validation
