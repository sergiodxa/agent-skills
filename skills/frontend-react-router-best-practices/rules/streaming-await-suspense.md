---
title: Await with Suspense
impact: MEDIUM
tags: [streaming, suspense, await, ux]
---

# Await with Suspense

Use Await component with Suspense fallback for streamed data.

## Why

- Await handles promise resolution from loader
- Suspense provides loading UI while promise resolves
- Enables streaming without blocking initial render
- Error boundaries catch rejected promises

## Basic Pattern

```tsx
import { Await, useLoaderData } from "react-router";
import { Suspense } from "react";

export default function Component() {
  const { criticalData, streamedData } = useLoaderData<typeof loader>();

  return (
    <div>
      {/* Critical data renders immediately */}
      <h1>{criticalData.title}</h1>

      {/* Streamed data with loading state */}
      <Suspense fallback={<LoadingSkeleton />}>
        <Await resolve={streamedData}>
          {(data) => <DataDisplay data={data} />}
        </Await>
      </Suspense>
    </div>
  );
}
```

## With Error Handling

```tsx
<Suspense fallback={<Skeleton />}>
  <Await resolve={streamedData} errorElement={<ErrorMessage />}>
    {(data) => <DataDisplay data={data} />}
  </Await>
</Suspense>
```

## Multiple Streamed Values

```tsx
export default function Component() {
  const { profile, activities, recommendations } =
    useLoaderData<typeof loader>();

  return (
    <div>
      <ProfileHeader profile={profile} />

      {/* Each streamed value gets its own Suspense boundary */}
      <Suspense fallback={<ActivitySkeleton />}>
        <Await resolve={activities}>
          {(data) => <ActivityFeed activities={data} />}
        </Await>
      </Suspense>

      <Suspense fallback={<RecommendationSkeleton />}>
        <Await resolve={recommendations}>
          {(data) => <Recommendations items={data} />}
        </Await>
      </Suspense>
    </div>
  );
}
```

## Nested Suspense for Prioritization

```tsx
// Outer content loads first, inner content streams in
<Suspense fallback={<PageSkeleton />}>
  <Await resolve={mainContent}>
    {(content) => (
      <MainContent data={content}>
        {/* Nested streaming for secondary content */}
        <Suspense fallback={<SidebarSkeleton />}>
          <Await resolve={sidebarData}>
            {(sidebar) => <Sidebar data={sidebar} />}
          </Await>
        </Suspense>
      </MainContent>
    )}
  </Await>
</Suspense>
```

## With Promise.all for Combined Data

When multiple promises should resolve together:

```tsx
// Loader
return data({
  // Combine related promises
  widgetData: Promise.all([badges, privateStocks]),
});

// Component
<Suspense fallback={<WidgetSkeleton />}>
  <Await resolve={widgetData}>
    {([badges, privateStocks]) => (
      <Widgets badges={badges} stocks={privateStocks} />
    )}
  </Await>
</Suspense>;
```

## Skeleton Components

Create skeleton components that match the final UI layout:

```tsx
function ActivitySkeleton() {
  return (
    <div className="v-stack gap-4">
      {[1, 2, 3].map((i) => (
        <div key={i} className="h-stack gap-3 animate-pulse">
          <div className="w-10 h-10 bg-neutral-200 rounded-full" />
          <div className="v-stack gap-2 flex-1">
            <div className="h-4 bg-neutral-200 rounded w-3/4" />
            <div className="h-3 bg-neutral-200 rounded w-1/2" />
          </div>
        </div>
      ))}
    </div>
  );
}
```
