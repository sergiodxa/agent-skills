---
title: Action Error Handling
impact: HIGH
tags: [action, error-handling, validation]
---

# Action Error Handling

Return validation errors using data(). Re-throw redirects and unknown errors.

## Why

- Validation errors should be displayed to users, not crash the app
- Redirects must be re-thrown to work (they're thrown as Response objects)
- Unknown errors should bubble up to error boundaries

## Pattern

```tsx
import { data } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  await authorize(client, { accountStatus: "active" });

  let formData = await request.formData();

  try {
    // Validation
    let validated = schema.parse({ ... });

    // Mutation
    await performMutation(client, validated);

    // Success - redirect
    throw redirect("/success");
  } catch (error) {
    // Handle zod validation errors
    if (error instanceof z.ZodError) {
      return data({ errors: error.issues.map(({ message }) => message) }, { status: 400 });
    }

    // Handle known business errors
    if (error instanceof Error) {
      return data({ errors: [error.message] }, { status: 400 });
    }

    // Re-throw everything else (redirects, unknown errors)
    throw error;
  }
}
```

## Why `throw redirect()` Instead of `return redirect()`

React Router redirects are Response objects that get thrown. Using `throw` ensures:

1. Code after redirect doesn't execute
2. TypeScript knows the function exits
3. Consistent with React Router's internal behavior

```tsx
// Good
await performMutation();
throw redirect("/success");

// Also works but less clear
await performMutation();
return redirect("/success");
```

## Error Types to Handle

```tsx
try {
  // ... validation and mutation
} catch (error) {
  // 1. Zod validation errors -> return 400
  if (error instanceof z.ZodError) {
    return data(
      { errors: error.issues.map(({ message }) => message) },
      { status: 400 },
    );
  }

  // 2. Custom business errors -> return 400
  if (error instanceof BusinessError) {
    return data({ errors: [error.message] }, { status: 400 });
  }

  // 3. Generic errors -> return 400
  if (error instanceof Error) {
    return data({ errors: [error.message] }, { status: 400 });
  }

  // 4. Everything else (redirects, Response objects) -> re-throw
  throw error;
}
```
