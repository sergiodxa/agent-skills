---
title: Single Fetch Migration
impact: HIGH
tags: [streaming, single-fetch, data-loading, migration]
---

# Single Fetch Migration

Migrate from defer() to data() pattern for Single Fetch. Promises automatically stream.

## Why

- Single Fetch is enabled (`v3_singleFetch: true` in remix.config)
- `data()` is the new standard, `defer()` is deprecated
- Promises in `data()` response automatically stream
- Cleaner API, no special handling needed

## Migration Pattern

### Before (defer)

```tsx
import { defer } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let criticalData = await getCriticalData();

  return defer({
    critical: criticalData,
    lazy: getLazyData(), // Promise - must use defer for streaming
  });
}
```

### After (data with Single Fetch)

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let criticalData = await getCriticalData();

  return data({
    critical: criticalData,
    lazy: getLazyData(), // Promise - automatically streams with Single Fetch
  });
}
```

## Example: Mixing Awaited and Streamed Data

```tsx
import { data } from "react-router";

export async function loader({ request, context }: Route.LoaderArgs) {
  let client = await authenticate(request);

  // Critical data - await before returning
  const [profile, settings] = await Promise.all([
    queryProfile(client),
    querySettings(client),
  ]);

  return data({
    profile,
    settings,
    // Non-critical data - streams automatically
    activities: queryActivities(client),
    notifications: queryNotifications(client),
    recommendations: queryRecommendations(client),
  });
}
```

## Component Usage

The component usage with `<Await>` and `<Suspense>` stays the same:

```tsx
import { Await, useLoaderData } from "react-router";
import { Suspense } from "react";

export default function Component() {
  const { profile, activities } = useLoaderData<typeof loader>();

  return (
    <div>
      {/* Critical data renders immediately */}
      <h1>{profile.name}</h1>

      {/* Streamed data with Suspense */}
      <Suspense fallback={<ActivitySkeleton />}>
        <Await resolve={activities}>
          {(data) => <ActivityFeed activities={data} />}
        </Await>
      </Suspense>
    </div>
  );
}
```

## When to Await vs Stream

```tsx
return data({
  // AWAIT: Critical for initial render, SEO, or needed by other data
  user: await getUser(request), // Await - needed for auth
  title: await getPageTitle(), // Await - needed for SEO

  // STREAM: Non-critical, below fold, or slow queries
  comments: getComments(postId), // Stream - below fold
  recommendations: getRecommendations(), // Stream - non-critical
  analytics: getAnalytics(), // Stream - slow query
});
```

## Non-Streaming (All Data Awaited)

When all data is critical and needs to be sent at once:

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
  return data({ a, b, c }); // All data sent at once, no streaming
}
```

Note: Always use `data()` - `json()` is deprecated with Single Fetch.
