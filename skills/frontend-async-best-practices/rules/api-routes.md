---
title: Prevent Waterfall Chains in API Routes
impact: CRITICAL
impactDescription: 2-10x improvement
tags: api-routes, waterfalls, parallelization
---

## Prevent Waterfall Chains in API Routes

In API routes and loaders, start independent operations immediately, even if you don't await them yet.

**Incorrect (config waits for auth, data waits for both):**

```typescript
export async function loader({ request }: LoaderFunctionArgs) {
  let session = await auth();
  let config = await fetchConfig();
  let data = await fetchData(session.user.id);
  return json({ data, config });
}
```

**Correct (auth and config start immediately):**

```typescript
export async function loader({ request }: LoaderFunctionArgs) {
  let sessionPromise = auth();
  let configPromise = fetchConfig();
  let session = await sessionPromise;
  let [config, data] = await Promise.all([
    configPromise,
    fetchData(session.user.id),
  ]);
  return json({ data, config });
}
```

For operations with more complex dependency chains, use `better-all` to automatically maximize parallelism (see Dependency-Based Parallelization).
