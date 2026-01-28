---
title: Stop Using jsonHash
impact: HIGH
tags: [streaming, migration, remix-utils]
---

# Stop Using jsonHash

Replace jsonHash from remix-utils with native Promise.all or data() patterns.

## Why

- `jsonHash` is a workaround for parallel async in loaders
- Native `Promise.all` is clearer and doesn't need a dependency
- `data()` with Single Fetch handles promises natively

## Bad: jsonHash Pattern

```tsx
import { jsonHash } from "remix-utils/json-hash";

export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);

  return jsonHash({
    items: client.items.list(),
    categories: client.categories.list(),
    user: client.users.current(),
  });
}
```

## Good: Promise.all (Non-Streaming)

When all data is needed before render:

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);

  let [items, categories, user] = await Promise.all([
    client.items.list(),
    client.categories.list(),
    client.users.current(),
  ]);

  return data({ items, categories, user });
}
```

## Good: data() with Promises (Streaming)

When some data can stream in later:

```tsx
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);

  // Critical data - await immediately
  let items = await client.items.list();

  return data({
    items,
    // Non-critical data - streams automatically with Single Fetch
    categories: client.categories.list(),
    recommendations: client.recommendations.list(),
  });
}
```

## Migration Examples

### Simple Object Return

```tsx
import { data } from "react-router";

// Before
return jsonHash({
  a: getA(),
  b: getB(),
});

// After
const [a, b] = await Promise.all([getA(), getB()]);
return data({ a, b });
```

### Async Functions in jsonHash

```tsx
import { data } from "react-router";

// Before
return jsonHash({
  async items() {
    let raw = await fetchItems();
    return transformItems(raw);
  },
  async metadata() {
    return fetchMetadata();
  },
});

// After
const [items, metadata] = await Promise.all([
  fetchItems().then(transformItems),
  fetchMetadata(),
]);
return data({ items, metadata });
```

### Mixed Sync and Async

```tsx
import { data } from "react-router";

// Before
return jsonHash({
  asyncData: fetchData(),
  syncData() {
    return computeSync();
  },
});

// After
const asyncData = await fetchData();
const syncData = computeSync();
return data({ asyncData, syncData });
```

## When to Use Which Pattern

| Scenario                        | Pattern                  |
| ------------------------------- | ------------------------ |
| All data critical, fast queries | `Promise.all` + `data()` |
| Some data can load later        | `data()` with promises   |
| Complex async transformations   | `Promise.all` + `data()` |
| Need to show partial UI fast    | `data()` with promises   |
