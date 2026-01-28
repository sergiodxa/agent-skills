---
title: Redirect After Mutation
impact: MEDIUM
tags: [action, redirect, forms]
---

# Redirect After Mutation

Redirect after successful mutations to prevent form resubmission.

## Why

- Prevents duplicate submissions on page refresh
- Provides clear feedback that action completed
- Follows Post/Redirect/Get (PRG) pattern
- Browser back button works as expected

## Pattern

```tsx

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  let formData = await request.formData();

  let data = schema.parse({ ... });
  await createItem(client, data);

  // Redirect after successful mutation
  throw redirect("/items");
}
```

## Use `throw redirect()` Not `return redirect()`

```tsx
// Good: throw ensures code stops executing
await createItem(client, data);
throw redirect("/success");

// Works but less explicit
await createItem(client, data);
return redirect("/success");
```

## Redirect with Flash Messages

Use session flash for success/error messages:

```tsx
import { commitSession, getSession } from "~/lib/session.server";

export async function action({ request }: Route.ActionArgs) {
  let session = await getSession(request);

  try {
    await createItem(client, data);

    session.flash("success", "Item created successfully");

    throw redirect("/items", {
      headers: {
        "Set-Cookie": await commitSession(session),
      },
    });
  } catch (error) {
    // Let redirects bubble up - redirect() returns a Response
    if (error instanceof Response) throw error;

    // Handle actual errors...
    return data({ errors: ["Something went wrong"] }, { status: 500 });
  }
}
```

## Re-throw Redirects in Catch Blocks

When using `throw redirect()` inside a try block, the redirect is caught by the catch block since `redirect()` returns a `Response`. Always re-throw it:

```tsx
// Bad: redirect gets swallowed by catch
try {
  await createItem(client, data);
  throw redirect("/success");
} catch (error) {
  return data({ errors: ["Failed"] }, { status: 500 });
}

// Good: re-throw Response to let redirect bubble up
try {
  await createItem(client, data);
  throw redirect("/success");
} catch (error) {
  if (error instanceof Response) throw error;
  return data({ errors: ["Failed"] }, { status: 500 });
}
```

## When NOT to Redirect

Some actions return data instead of redirecting:

```tsx
import { data } from "react-router";

// In-place updates with fetcher - return data, no redirect
export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();
  let intent = formData.get("intent");

  if (intent === "like") {
    let postId = z.string().parse(formData.get("postId"));
    let likes = await toggleLike(postId);
    return data({ likes }); // Return updated count
  }

  throw new Error(`Unknown intent: ${intent}`);
}
```

## Redirect Patterns

```tsx
// Simple redirect
throw redirect("/items");

// Redirect with preserved search params
const url = new URL(request.url);
throw redirect(`/items${url.search}`);

// Redirect to dynamic path
throw redirect(`/items/${newItem.id}`);

// Redirect with object (React Router)
throw redirect({ pathname: "/home" });

// Redirect with returnTo param
const url = new URL(request.url);
const returnTo = url.searchParams.get("returnTo") || "/home";
throw redirect(returnTo);
```
