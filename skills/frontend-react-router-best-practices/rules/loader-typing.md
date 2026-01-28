---
title: Proper TypeScript Typing for Loaders
impact: MEDIUM
tags: [loader, typescript, typing]
---

# Proper TypeScript Typing for Loaders

Use Route.LoaderArgs and `typeof loader` with useLoaderData for type safety.

## Why

Proper typing catches errors at compile time and provides autocomplete for loader data in components.

## Bad

```tsx
// No types - no autocomplete, no compile-time checks
export async function loader({ request, params }) {
  return data({ user: await getUser(params.id) });
}

export default function Component() {
  let loaderData = useLoaderData(); // any type
  return <div>{loaderData.user.name}</div>; // No type safety
}
```

## Good

```tsx
import { data } from "react-router";
import { useLoaderData } from "react-router";

export async function loader({ request, params }: Route.LoaderArgs) {
  return data({ user: await getUser(params.id) });
}

export default function Component() {
  const { user } = useLoaderData<typeof loader>();
  return <div>{user.name}</div>; // Type-safe
}
```

## With Status Codes

```tsx
import { data } from "react-router";

export async function loader({ request, params }: Route.LoaderArgs) {
  let user = await getUser(params.id);
  if (!user) {
    throw data({ message: "User not found" }, { status: 404 });
  }
  return data({ user });
}

export default function Component() {
  const { user } = useLoaderData<typeof loader>();
  // user is properly typed from the return type
}
```

Note: Always use `data()` for loader returns. `json()` is deprecated with Single Fetch.

## With SerializeFrom for Shared Types

When passing loader data to other components:

```tsx
import type { SerializeFrom } from "react-router";

type LoaderData = SerializeFrom<typeof loader>;

function UserCard({ user }: { user: LoaderData["user"] }) {
  return <div>{user.name}</div>;
}
```
