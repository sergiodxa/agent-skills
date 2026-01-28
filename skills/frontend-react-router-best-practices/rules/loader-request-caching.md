---
title: Request-Level Caching in Loaders
impact: HIGH
tags: [loader, caching, performance, deduplication]
---

# Request-Level Caching in Loaders

API clients automatically deduplicate calls within the same request. Fetch the same data in multiple loaders without worrying about duplicate network requests.

## Why

- Nested routes often need the same data (user, permissions, etc.)
- React Router runs all loaders in parallel for a single request
- API clients use request-scoped caching to dedupe identical calls
- No manual coordination needed between loaders

## How It Works

When you call the same API function with the same parameters in multiple loaders during the same request, the API client:

1. Makes the network request on the first call
2. Caches the result for the duration of the request
3. Returns the cached result for subsequent identical calls
4. Clears the cache after the request completes

```tsx
// routes/dashboard.tsx (parent)
export async function loader({ request, context }: Route.LoaderArgs) {
  let client = await authenticate(request, { context });
  let user = await getUser(client); // First call - makes network request
  return data({ user });
}

// routes/dashboard.settings.tsx (child)
export async function loader({ request, context }: Route.LoaderArgs) {
  let client = await authenticate(request, { context });
  let user = await getUser(client); // Same call - returns cached result
  let settings = await getSettings(client, user.id);
  return data({ user, settings });
}
```

Both loaders run in parallel. The `context` contains the batcher (`context.batcher`), so `authenticate` and subsequent API calls share the same request-scoped cache. The second `getUser(client)` call returns instantly from the cache.

## When to Fetch vs Use useRouteLoaderData

**Fetch in loader when:**

- The loader needs the data for its own logic (not just UI)
- You need to transform or combine the data with other data
- The child route can be accessed directly (not always through parent)

```tsx
// Child loader needs user.id to fetch settings
export async function loader({ request, context }: Route.LoaderArgs) {
  let client = await authenticate(request, { context });
  let user = await getUser(client); // Need user.id for next call
  let settings = await getSettings(client, user.id);
  return data({ settings });
}
```

**Use useRouteLoaderData when:**

- You only need the data for rendering (no loader logic)
- The parent route always loads before this route
- You're certain the data exists in the parent

```tsx
// Child only needs user for display
export default function Component() {
  const { user } =
    useRouteLoaderData<typeof dashboardLoader>("routes/dashboard");
  return <WelcomeMessage name={user.name} />;
}
```

## How It Works Internally

Use the `remix-utils` batcher middleware to get a request-scoped batcher:

```ts
// app/middleware/batcher.server.ts
import { createBatcherMiddleware } from "remix-utils/middleware/batcher";

export const [batcherMiddleware, getBatcher] = createBatcherMiddleware();
```

```ts
// app/routes/dashboard/route.tsx
import { batcherMiddleware, getBatcher } from "~/middleware/batcher.server";

export const middleware: Route.MiddlewareFunction[] = [batcherMiddleware];

export async function loader({ context }: Route.LoaderArgs) {
  let batcher = getBatcher(context);
  let result = await batcher.batch("key", async () => {
    return await getData();
  });
  return data({ result });
}
```

Your API client can use the batcher for GET requests automatically:

```typescript
// clients/api.ts
override async get<Type>(path: string, body?: ParamsInput, init?: RequestInit) {
  let url = new URL(path, this.baseURL);
  url.search = snakeCaseSearchParams(body).toString();

  // Batch by [pathname, search] - same URL = same cached result
  return this.batch([url.pathname, url.search], () =>
    this.collectTiming("api", `${url.pathname}${url.search}`, async () => {
      let response = await super.get(url.href, { ...init });
      return await response.text().then(parse<Type>);
    })
  );
}
```

The batcher deduplicates based on the key array `[pathname, search]`:

- Same pathname + search params = returns cached result
- Different pathname or params = makes new request
- Only GET requests are batched (POST/PUT/DELETE always execute)

## Rules

1. Always pass `context` to `api()` or `authenticate()` - enables request-level caching
2. Don't avoid fetching data in nested loaders for "performance" - caching handles it
3. Fetch data in each loader that needs it for logic, not just rendering
4. Use `useRouteLoaderData` only for UI-only access to parent data
5. Trust the API client's request-level caching - identical GET requests are deduped automatically
