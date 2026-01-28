---
title: Form Pending States
impact: MEDIUM
tags: [forms, loading, ux]
---

# Form Pending States

Show loading states with useNavigation or fetcher.state. Use useSpinDelay to avoid flicker.

## Why

- Users need feedback that action is processing
- Prevents double-submission
- Quick actions shouldn't flash loading state (flicker)

## Basic Fetcher Pending State

```tsx
function SubmitButton() {
  let fetcher = useFetcher();
  let isPending = fetcher.state !== "idle";

  return (
    <Button type="submit" isDisabled={isPending}>
      {isPending ? "Submitting..." : "Submit"}
    </Button>
  );
}
```

## With Spinner and useSpinDelay

Avoid flicker for fast operations:

```tsx
import { useSpinDelay } from "spin-delay";

function SubmitButton() {
  let fetcher = useFetcher();

  // Only show spinner if pending for >50ms
  let isPending = useSpinDelay(fetcher.state !== "idle", { delay: 50 });

  return (
    <Button type="submit" isDisabled={isPending} className="relative">
      {isPending && (
        <div className="absolute inset-0 center">
          <Spinner />
        </div>
      )}
      <span className={isPending ? "invisible" : ""}>Submit</span>
    </Button>
  );
}
```

## useNavigation for Form Component

```tsx
import { Form, useNavigation } from "react-router";

function CreateForm() {
  let navigation = useNavigation();

  // Check if THIS form is submitting
  let isSubmitting =
    navigation.state === "submitting" && navigation.formAction === "/items/new";

  return (
    <Form method="post" action="/items/new">
      <input name="title" disabled={isSubmitting} />
      <Button type="submit" isDisabled={isSubmitting}>
        {isSubmitting ? "Creating..." : "Create"}
      </Button>
    </Form>
  );
}
```

## Disable Form During Submission

```tsx
function EditForm() {
  let fetcher = useFetcher();
  let isPending = fetcher.state !== "idle";

  return (
    <fetcher.Form method="post">
      <fieldset disabled={isPending}>
        <input name="name" />
        <input name="email" />
        <Button type="submit">{isPending ? "Saving..." : "Save"}</Button>
      </fieldset>
    </fetcher.Form>
  );
}
```

## Prevent Modal Close During Submission

```tsx
function EditModal({ isOpen, onClose }) {
  let fetcher = useFetcher();
  let isPending = fetcher.state !== "idle";

  return (
    <Modal
      isOpen={isOpen}
      onOpenChange={(open) => {
        // Don't allow close while submitting
        if (!open && isPending) return;
        if (!open) onClose();
      }}
    >
      <fetcher.Form method="post">{/* Form content */}</fetcher.Form>
    </Modal>
  );
}
```
