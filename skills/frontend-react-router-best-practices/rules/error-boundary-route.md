---
title: Route Error Boundaries
impact: MEDIUM
tags: [error-handling, error-boundary, routes]
---

# Route Error Boundaries

Add ErrorBoundary to routes with data fetching to catch loader/action errors gracefully.

## Why

- Prevents entire app from crashing on route errors
- Provides route-specific error UI
- Maintains parent layouts during child errors
- Better UX than generic error page

## Which Routes Need Error Boundaries

Add ErrorBoundary to:

1. Routes with loaders that fetch external data
2. Routes with actions that can fail
3. Routes with required params that might be invalid
4. Parent layout routes (catches child errors)

## Pattern

```tsx
import { data } from "react-router";

export async function loader({ params }: Route.LoaderArgs) {
  let item = await getItem(params.itemId);
  if (!item) {
    throw data({ message: "Item not found" }, { status: 404 });
  }
  return data({ item });
}

export async function action({ request }: Route.ActionArgs) {
  // Action that might fail
}

export default function ItemPage() {
  const { item } = useLoaderData<typeof loader>();
  return <ItemDetails item={item} />;
}

// Catches errors from loader, action, and component
export function ErrorBoundary() {
  let error = useRouteError();

  if (isRouteErrorResponse(error) && error.status === 404) {
    return (
      <div className="p-8 text-center">
        <h1 className="text-xl font-semibold">Item Not Found</h1>
        <p className="mt-2 text-neutral-600">
          The item you're looking for doesn't exist.
        </p>
        <Link to="/items" className="mt-4 inline-block text-accent-600">
          Browse all items
        </Link>
      </div>
    );
  }

  return (
    <div className="p-8 text-center">
      <h1 className="text-xl font-semibold">Error Loading Item</h1>
      <p className="mt-2 text-neutral-600">
        Something went wrong. Please try again.
      </p>
    </div>
  );
}
```

## Error Boundary Hierarchy

```
root.tsx (ErrorBoundary)           <- Catches app-level errors
└── routes/_/route.tsx (ErrorBoundary) <- Catches layout errors
    └── routes/_._profile/route.tsx (ErrorBoundary) <- Catches profile errors
        └── routes/_._profile.settings/route.tsx (ErrorBoundary) <- Catches settings errors
```

Each level catches errors from its children. If a child doesn't have an ErrorBoundary, the error bubbles up.

## Root Error Boundary

The root route should always have an ErrorBoundary as the last line of defense:

```tsx
// root.tsx
export function ErrorBoundary() {
  let error = useRouteError();

  return (
    <html>
      <head>
        <title>Error</title>
        <Meta />
        <Links />
      </head>
      <body>
        <div className="min-h-screen flex items-center justify-center">
          <div className="text-center">
            <h1 className="text-2xl font-bold">Something went wrong</h1>
            {isRouteErrorResponse(error) ? (
              <p>
                {error.status}: {error.statusText}
              </p>
            ) : (
              <p>An unexpected error occurred</p>
            )}
            <a href="/" className="mt-4 inline-block text-accent-600">
              Go home
            </a>
          </div>
        </div>
        <Scripts />
      </body>
    </html>
  );
}
```

## Testing Error Boundaries

```tsx
import { data } from "react-router";

// Trigger 404
throw data({ message: "Not found" }, { status: 404 });

// Trigger 500
throw new Error("Database connection failed");

// Trigger from action
if (!isValid) {
  throw data({ errors: ["Invalid data"] }, { status: 400 });
}
```
