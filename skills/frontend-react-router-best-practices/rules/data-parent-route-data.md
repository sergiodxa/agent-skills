---
title: Access Parent Route Data
impact: MEDIUM
tags: [data-loading, routes, hooks]
---

# Access Parent Route Data

Use `useRouteLoaderData` or `useMatches` to access data from parent routes.

## Why

- Avoids prop drilling through nested routes
- No need to refetch data already loaded by parent
- Type-safe access to parent loader data
- Works with React Router's nested routing model

## useRouteLoaderData

Access a specific parent route's data by route ID:

```tsx
import { useRouteLoaderData } from "react-router";
import type { loader as parentLoader } from "~/routes/_layout/route";

export default function ChildRoute() {
  // Route ID matches the file path without extension
  let parentData =
    useRouteLoaderData<typeof parentLoader>("routes/_layout");

  if (!parentData) return null;

  return <div>Welcome, {parentData.user.name}</div>;
}
```

## useMatches

Access all matched routes and their data:

```tsx
import { useMatches } from "react-router";

export default function Component() {
  let matches = useMatches();

  // Find a specific match by ID or pathname
  let layoutMatch = matches.find((m) => m.id === "routes/_layout");
  let layoutData = layoutMatch?.data as LayoutLoaderData | undefined;

  return <div>{layoutData?.user.name}</div>;
}
```

## When to Use Which

| Scenario                            | Use                      |
| ----------------------------------- | ------------------------ |
| Known parent route, need typed data | `useRouteLoaderData`     |
| Search through multiple routes      | `useMatches`             |
| Access route handle metadata        | `useMatches`             |
| Component used in multiple places   | `useMatches` with search |

## Don't Use Outlet Context

```tsx
// Bad: Outlet context
export default function Parent() {
  let data = useLoaderData<typeof loader>();
  return <Outlet context={data} />;
}

function Child() {
  let data = useOutletContext<ParentData>();
}

// Good: useRouteLoaderData
function Child() {
  let data = useRouteLoaderData<typeof parentLoader>("routes/parent");
}
```

## Common Parent Routes

```tsx
// Access root loader data
const rootData = useRouteLoaderData<typeof rootLoader>("root");

// Access authenticated layout data
const layoutData =
  useRouteLoaderData<typeof layoutLoader>("routes/_layout");

// Access settings layout data
const settingsData =
  useRouteLoaderData<typeof settingsLoader>("routes/_.settings");
```
