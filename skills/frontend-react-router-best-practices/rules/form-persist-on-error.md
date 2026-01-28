---
title: Persist Form Inputs on Validation Error
impact: MEDIUM
tags: [form, validation, ux, error-handling]
---

# Persist Form Inputs on Validation Error

Return field values from the action when validation fails, then use `defaultValue` to repopulate inputs.

## Why

- Users shouldn't have to re-enter all data when only one field is wrong
- Required for progressive enhancement (no-JS form submissions)
- With JS enabled, `<Form>` preserves inputs naturally, but returning fields ensures consistency
- Better UX than showing errors with empty fields

## The Problem

When a form submission fails validation, the page reloads (in no-JS) or re-renders. Without preserving field values, users lose their input:

```tsx

// Action only returns errors - fields are lost on no-JS
export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();
  let result = schema.safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return data({ errors: result.error.flatten() }, { status: 400 });
  }

  // ...
}
```

## Solution: Return Fields with Errors

```tsx
import { data, redirect } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();

  let fields = {
    email: formData.get("email")?.toString() ?? "",
    name: formData.get("name")?.toString() ?? "",
  };

  let result = schema.safeParse(fields);

  if (!result.success) {
    return data(
      {
        errors: result.error.flatten().fieldErrors,
        fields, // Return the submitted values
      },
      { status: 400 },
    );
  }

  await createUser(result.data);
  throw redirect("/success");
}
```

## Component: Use defaultValue

```tsx
import { Form, useActionData } from "react-router";

export default function SignupForm() {
  let actionData = useActionData<typeof action>();

  return (
    <Form method="post">
      <div>
        <label htmlFor="name">Name</label>
        <input id="name" name="name" defaultValue={actionData?.fields?.name} />
        {actionData?.errors?.name && (
          <p className="text-failure-600">{actionData.errors.name[0]}</p>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          defaultValue={actionData?.fields?.email}
        />
        {actionData?.errors?.email && (
          <p className="text-failure-600">{actionData.errors.email[0]}</p>
        )}
      </div>

      <button type="submit">Sign Up</button>
    </Form>
  );
}
```

## With useFetcher

Same pattern works with fetcher forms:

```tsx
function InlineForm() {
  let fetcher = useFetcher<typeof action>();

  return (
    <fetcher.Form method="post">
      <input name="email" defaultValue={fetcher.data?.fields?.email} />
      {fetcher.data?.errors?.email && <p>{fetcher.data.errors.email[0]}</p>}
      <button type="submit">Submit</button>
    </fetcher.Form>
  );
}
```

## Combined with Reset on Success

This pattern complements `form-reset-on-success.md`:

- **On error**: Persist fields so user can correct mistakes
- **On success**: Reset form to clear all fields

```tsx

export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();
  let fields = { title: formData.get("title")?.toString() ?? "" };

  let result = schema.safeParse(fields);

  if (!result.success) {
    // Error: return fields for persistence
    return data(
      { ok: false, errors: result.error.flatten().fieldErrors, fields },
      { status: 400 },
    );
  }

  await createItem(result.data);
  // Success: return ok for reset trigger (no fields needed)
  return data({ ok: true });
}
```

## Handling Sensitive Fields

Don't return sensitive fields like passwords:

```tsx
let fields = {
  email: formData.get("email")?.toString() ?? "",
  // Don't include password in returned fields
};

// Validate including password, but don't return it
let result = schema.safeParse({
  ...fields,
  password: formData.get("password")?.toString() ?? "",
});
```

## Rules

1. Always return submitted field values when returning validation errors
2. Use `defaultValue` (not `value`) to allow user editing
3. Don't return sensitive fields (passwords, tokens)
4. Return `{ errors, fields }` on error, `{ ok: true }` on success
5. Works with both `<Form>` and `fetcher.Form`
6. Essential for progressive enhancement (no-JS support)
