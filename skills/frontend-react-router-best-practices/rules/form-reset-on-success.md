---
title: Reset Form on Success
impact: MEDIUM
tags: [form, fetcher, ux]
---

# Reset Form on Success

Reset uncontrolled form inputs after a successful action submission using a ref and effect.

## Why

- Uncontrolled inputs don't clear automatically after submission
- React doesn't re-mount inputs when staying on the same route
- Users expect forms to clear after successful submission
- Prevents accidental re-submission of the same data

## The Problem

When using uncontrolled inputs (no `value` prop), the input values persist after submission:

```tsx
// Form stays filled after submission - bad UX
export default function Component() {
  return (
    <Form method="post">
      <input name="title" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

## Solution: Reset with Ref and Effect

### With Form

```tsx
import { Form, useActionData, useNavigation } from "react-router";
import { useEffect, useRef } from "react";

export default function Component() {
  let formRef = useRef<HTMLFormElement>(null);
  let navigation = useNavigation();
  let actionData = useActionData<typeof action>();

  useEffect(
    function resetFormOnSuccess() {
      if (navigation.state === "idle" && actionData?.ok) {
        formRef.current?.reset();
      }
    },
    [navigation.state, actionData],
  );

  return (
    <Form method="post" ref={formRef}>
      <input name="title" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

### With useFetcher

```tsx
import { useFetcher } from "react-router";
import { useEffect, useRef } from "react";

function CreateItemForm() {
  let formRef = useRef<HTMLFormElement>(null);
  let fetcher = useFetcher<typeof action>();

  useEffect(
    function resetFormOnSuccess() {
      if (fetcher.state === "idle" && fetcher.data?.ok) {
        formRef.current?.reset();
      }
    },
    [fetcher.state, fetcher.data],
  );

  return (
    <fetcher.Form method="post" ref={formRef}>
      <input name="title" />
      <button type="submit">Create</button>
    </fetcher.Form>
  );
}
```

## Action Response Pattern

Return a success indicator from your action:

```tsx
export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();

  try {
    await createItem(formData);
    return data({ ok: true });
  } catch (error) {
    return data({ ok: false, error: "Failed to create" }, { status: 400 });
  }
}
```

Or use a status literal:

```tsx
return data({ status: "success" as const });

// In component
if (fetcher.state === "idle" && fetcher.data?.status === "success") {
  formRef.current?.reset();
}
```

## Alternative: Key-Based Reset

For simpler cases, use a key to force React to remount the form:

```tsx
function CreateItemForm() {
  let [key, setKey] = useState(0);
  let fetcher = useFetcher<typeof action>();

  useEffect(() => {
    if (fetcher.state === "idle" && fetcher.data?.ok) {
      setKey((k) => k + 1); // Remounts entire form
    }
  }, [fetcher.state, fetcher.data]);

  return (
    <fetcher.Form method="post" key={key}>
      <input name="title" />
      <button type="submit">Create</button>
    </fetcher.Form>
  );
}
```

Note: This remounts the entire form, which may reset focus. Use the ref approach for better UX.

## Rules

1. Use `formRef.current?.reset()` to clear uncontrolled inputs
2. Check both `state === "idle"` and success indicator before resetting
3. Return a success indicator (`ok: true` or `status: "success"`) from actions
4. Prefer ref-based reset over key-based remounting
5. Don't reset on error - user may want to correct their input
