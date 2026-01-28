---
title: Avoid Data Fetching Waterfalls
impact: HIGH
tags: [loader, performance, data-fetching, waterfalls]
---

# Avoid Data Fetching Waterfalls

Fetch all data in loaders. Never fetch data in components with useEffect or useFetcher on mount.

## Why

- Waterfalls add latency: each fetch waits for the previous one
- Component-level fetching causes loading spinners inside already-loaded pages
- Loaders run in parallel at the route level, components run sequentially
- Server-side fetching is faster (closer to data sources, no round trip)

## The Golden Rule

**All data fetching happens in loaders. Components only render data.**

## Bad: Fetching in Components

```tsx
// BAD: Component fetches its own data
function UserProfile() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch("/api/user")
      .then((r) => r.json())
      .then(setUser);
  }, []);

  if (!user) return <Spinner />;
  return <div>{user.name}</div>;
}

// BAD: useFetcher on mount
function UserProfile() {
  let fetcher = useFetcher();

  useEffect(() => {
    fetcher.load("/api/user");
  }, []);

  if (!fetcher.data) return <Spinner />;
  return <div>{fetcher.data.name}</div>;
}
```

## Good: Fetching in Loaders

```tsx
// route.tsx
export async function loader({ request }: Route.LoaderArgs) {
  let user = await getUser(request);
  return data({ user });
}

export default function Component() {
  const { user } = useLoaderData<typeof loader>();
  return <div>{user.name}</div>;
}
```

## Parent-Child Route Data Sharing

Fetch data in each loader that needs it. API clients share data between loaders via request-level caching, so there's no duplicate network request.

```tsx
// routes/dashboard.tsx (parent)
export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);
  let user = await getUser(client); // Cached per request
  return data({ user });
}

// routes/dashboard.settings.tsx (child)
export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);
  // Same getUser call - uses cached result, no extra network request
  let user = await getUser(client);
  let settings = await getSettings(client, user.id);
  return data({ user, settings });
}

export default function Component() {
  const { user, settings } = useLoaderData<typeof loader>();
  return <SettingsForm user={user} settings={settings} />;
}
```

For UI-only access to parent data (no loader logic needed), use `useRouteLoaderData`:

```tsx
// routes/dashboard.notifications.tsx (child) - only needs user for display
export default function Component() {
  const { user } =
    useRouteLoaderData<typeof dashboardLoader>("routes/dashboard");
  return <NotificationPrefs userName={user.name} />;
}
```

## Parallel Fetching in Loaders

When you need multiple pieces of data, fetch them in parallel:

```tsx
export async function loader({ request }: Route.LoaderArgs) {
  let user = await requireUser(request);

  // Parallel fetches - total time = slowest query
  let [orders, accountBalance, recommendations] = await Promise.all([
    getOrders(user.id),
    getAccountBalance(user.accountId),
    getRecommendations(user.id),
  ]);

  return data({ user, orders, accountBalance, recommendations });
}
```

## Streaming Non-Critical Data

For slow, non-critical data, use streaming:

```tsx
export async function loader({ request }: Route.LoaderArgs) {
  let user = await requireUser(request);

  // Critical data - awaited
  let accountBalance = await getAccountBalance(user.accountId);

  // Non-critical data - streamed (don't await)
  let recommendations = getRecommendations(user.id);

  return data({ user, accountBalance, recommendations });
}

export default function Component() {
  const { balance, recommendations } = useLoaderData<typeof loader>();

  return (
    <div>
      <BalanceCard balance={balance} />

      <Suspense fallback={<RecommendationsSkeleton />}>
        <Await resolve={recommendations}>
          {(data) => <Recommendations data={data} />}
        </Await>
      </Suspense>
    </div>
  );
}
```

## When useFetcher is Appropriate

useFetcher is for **user-initiated actions**, not initial data loading:

```tsx
// GOOD: User clicks to load more
function LoadMoreButton({ page }) {
  let fetcher = useFetcher();

  return (
    <fetcher.Form method="get" action="/api/orders">
      <input type="hidden" name="page" value={page + 1} />
      <Button type="submit">
        {fetcher.state === "loading" ? "Loading..." : "Load More"}
      </Button>
    </fetcher.Form>
  );
}

// GOOD: User submits form
function CheckoutForm() {
  let fetcher = useFetcher();

  return (
    <fetcher.Form method="post" action="/checkout">
      {/* form fields */}
    </fetcher.Form>
  );
}
```

## Rules

1. All initial data fetching happens in loaders
2. Never use useEffect + fetch for data loading
3. Never use useFetcher.load() on component mount
4. Fetch data in each loader that needs it (API clients cache per request)
5. Use `useRouteLoaderData` only for UI-only access to parent data
6. Use `Promise.all` for parallel independent fetches
7. Use streaming (Suspense + Await) for slow, non-critical data
8. useFetcher is only for user-initiated actions (forms, load more, etc.)
