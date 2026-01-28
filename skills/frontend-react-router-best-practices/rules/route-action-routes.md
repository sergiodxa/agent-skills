---
title: Action Routes Pattern
impact: HIGH
tags: [routes, actions, resource-routes, reusability]
---

# Action Routes Pattern

Centralize reusable actions in dedicated resource routes under `routes/actions/`. This pattern is for actions triggered from multiple UI routes that need shared authentication, validation, responses, and client-side effects.

## Why

- Actions triggered from multiple places (e.g., "like" button on feed, detail page, modal)
- Consistent authentication, validation, and error handling across all usages
- Client-side effects (toasts, redirects) handled once, not duplicated
- Single source of truth for action logic

## When to Use

Use action routes when:

- The same action is triggered from **multiple UI routes**
- You need **consistent client-side effects** (toasts, redirects) across all usages
- The action has **complex logic** worth centralizing (auth, validation, permissions)

**Don't use** for simple, single-use actions—keep those in the route that renders the form.

## File Structure

```
routes/
├── actions.post-create.ts       # /actions/post-create
├── actions.post-delete.ts       # /actions/post-delete
├── actions.favorite-toggle.ts   # /actions/favorite-toggle
├── actions.subscription-cancel.ts
├── _index.tsx
├── feed.tsx
└── posts.$postId.tsx
```

Naming convention: `actions.noun-verb.ts` using flat routes with dot notation. The `actions.` prefix groups them and creates the `/actions/` URL prefix.

## Server Action with Validation

```tsx
// routes/actions.post-create.ts
import { data, redirect } from "react-router";
import { z } from "zod";

const schema = z.object({
  title: z.string().min(1),
  content: z.string().min(1),
});

export async function action({ request, context }: Route.ActionArgs) {
  let client = await authenticate(request, { context });

  let formData = await request.formData();
  let result = schema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return data({ ok: false, errors: result.error.flatten() }, { status: 400 });
  }

  let post = await createPost(client, result.data);

  return data({ ok: true, post }, { status: 201 });
}

// No default export = resource route
```

## Client-Side Effects with clientAction

Handle toasts and redirects in `clientAction`:

```tsx
// routes/actions.post-create.ts
import { redirect } from "react-router";

export async function clientAction({ serverAction }: Route.ClientActionArgs) {
  let result = await serverAction<typeof action>();

  if (result.ok) {
    toast.success("Post created successfully");
    return redirect(`/posts/${result.post.id}`);
  }

  toast.error("Failed to create post");
  return result;
}
```

This pattern:

1. Calls the server action
2. Shows success toast and redirects on success
3. Shows error toast on failure
4. Returns result for UI to handle (e.g., show field errors)

## Using Action Routes with useFetcher

```tsx
// routes/feed.tsx
import { useFetcher } from "react-router";
import type { action } from "~/routes/actions.post-create";

function CreatePostButton() {
  let fetcher = useFetcher<typeof action>();
  let isPending = fetcher.state !== "idle";

  return (
    <fetcher.Form method="post" action="/actions/post-create">
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Creating..." : "Create Post"}
      </button>
      {fetcher.data?.errors && <FieldErrors errors={fetcher.data.errors} />}
    </fetcher.Form>
  );
}
```

## Authorization in Action Routes

```tsx
import { data } from "react-router";

export async function action({ request, context }: Route.ActionArgs) {
  let client = await authenticate(request, { context });

  // Check permissions
  if (!(await hasActiveSubscription(client))) {
    return data(
      { ok: false, error: "Upgrade to create posts." },
      { status: 403 },
    );
  }

  // ... rest of action
}
```

## Rules

1. Use `actions.noun-verb.ts` naming with flat routes (creates `/actions/` URL prefix)
2. No default export—these are resource routes (no UI)
3. Return `{ ok: true, ...data }` or `{ ok: false, errors }` for consistent response shape
4. Handle client-side effects (toasts, redirects) in `clientAction`
5. Keep single-use actions in the route that renders the form
6. Use this pattern only when actions are reused across multiple UI routes
