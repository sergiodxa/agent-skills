---
title: Route Component Naming
impact: MEDIUM
tags: [routes, components, conventions]
---

# Route Component Naming

Name the default export `Component` in React Router route files.

## Why

- Consistent naming across all routes
- Easy to identify as route component vs regular component
- Matches React Router documentation conventions
- Clear distinction from named exports (loader, action)

## Pattern

```tsx
// app/routes/_.users/route.tsx
import { data } from "react-router";
import { useLoaderData } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  // ...
  return data({ users });
}

export async function action({ request }: Route.ActionArgs) {
  // ...
}

// Always name the default export "Component"
export default function Component() {
  let { users } = useLoaderData<typeof loader>();

  return (
    <div>
      <h1>Users</h1>
      <UserList users={users} />
    </div>
  );
}
```

## Don't Use Route-Specific Names

```tsx
// Bad: route-specific name
export default function UsersPage() {
  // ...
}

// Bad: generic name
export default function Page() {
  // ...
}

// Bad: index name
export default function Index() {
  // ...
}

// Good: always "Component"
export default function Component() {
  // ...
}
```

## With ErrorBoundary

```tsx
// app/routes/_.users/route.tsx
export async function loader() {
  // ...
}

export default function Component() {
  // Main route component
}

// ErrorBoundary is a named export, not default
export function ErrorBoundary() {
  let error = useRouteError();
  // ...
}
```

## Why "Component"?

1. **React Router convention** - Matches the React Router docs
2. **Searchable** - Easy to find all route components
3. **Refactor-safe** - Renaming route folder doesn't require renaming function
4. **Clear purpose** - Instantly identifies the route's UI export
