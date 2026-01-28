---
title: Layout-Aware Error Boundary
impact: MEDIUM
tags: [error-handling, error-boundary, ux]
---

# Layout-Aware Error Boundary

Implement ErrorBoundary with useRouteError and handle different error types.

## Why

- Catches errors from loader, action, and component
- Provides user-friendly error messages
- Maintains layout context when possible
- Handles both Response errors and JavaScript errors

## Basic Pattern

```tsx
import { useRouteError, isRouteErrorResponse } from "react-router";

export function ErrorBoundary() {
  let error = useRouteError();

  // Handle HTTP Response errors (404, 500, etc.)
  if (isRouteErrorResponse(error)) {
    return (
      <div className="p-8 text-center">
        <h1 className="text-2xl font-bold">
          {error.status} {error.statusText}
        </h1>
        <p className="mt-2 text-neutral-600">{error.data}</p>
      </div>
    );
  }

  // Handle JavaScript errors
  return (
    <div className="p-8 text-center">
      <h1 className="text-2xl font-bold">Something went wrong</h1>
      <p className="mt-2 text-neutral-600">
        {error instanceof Error ? error.message : "Unknown error occurred"}
      </p>
    </div>
  );
}
```

## With Layout Preservation

For nested routes, the error boundary replaces only the errored route's content, preserving parent layouts:

```tsx
// routes/_._profile/route.tsx - Layout route
export default function ProfileLayout() {
  return (
    <div className="flex">
      <Sidebar />
      <main>
        <Outlet /> {/* Child errors render here */}
      </main>
    </div>
  );
}

// routes/_._profile.settings/route.tsx - Child route
export async function loader() {
  throw new Error("Failed to load settings");
}

export function ErrorBoundary() {
  let error = useRouteError();

  // This renders INSIDE the ProfileLayout
  return (
    <div className="p-8">
      <h1>Settings Error</h1>
      <p>{error instanceof Error ? error.message : "Unknown error"}</p>
      <Link to="/profile">Back to Profile</Link>
    </div>
  );
}
```

## Typed Error Responses

```tsx
import { data } from "react-router";

// In loader/action - throw typed errors
export async function loader({ params }: Route.LoaderArgs) {
  let item = await getItem(params.id);

  if (!item) {
    throw data({ message: "Item not found", id: params.id }, { status: 404 });
  }

  return data({ item });
}

// In ErrorBoundary - handle typed data
export function ErrorBoundary() {
  let error = useRouteError();

  if (isRouteErrorResponse(error)) {
    if (error.status === 404) {
      return (
        <div>
          <h1>Not Found</h1>
          <p>{error.data.message}</p>
          <p>Looking for ID: {error.data.id}</p>
        </div>
      );
    }
  }

  // ... handle other errors
}
```

## Common Error Status Handling

```tsx
export function ErrorBoundary() {
  let error = useRouteError();

  if (isRouteErrorResponse(error)) {
    switch (error.status) {
      case 400:
        return <BadRequestError message={error.data} />;
      case 401:
        return <UnauthorizedError />;
      case 403:
        return <ForbiddenError />;
      case 404:
        return <NotFoundError />;
      case 500:
        return <ServerError />;
      default:
        return <GenericError status={error.status} />;
    }
  }

  // Log unexpected errors
  console.error(error);

  return (
    <div>
      <h1>Unexpected Error</h1>
      <p>Please try again later.</p>
    </div>
  );
}
```

## Development vs Production

```tsx
export function ErrorBoundary() {
  let error = useRouteError();

  return (
    <div className="p-8">
      <h1>Error</h1>

      {process.env.NODE_ENV === "development" && error instanceof Error && (
        <pre className="mt-4 p-4 bg-red-50 text-red-900 overflow-auto">
          {error.stack}
        </pre>
      )}

      {process.env.NODE_ENV === "production" && (
        <p>Something went wrong. Please try again.</p>
      )}
    </div>
  );
}
```
