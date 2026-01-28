---
title: Abort Async Work in Loaders and Actions
impact: HIGH
tags: [loaders, actions, fetch, performance]
---

# Abort Async Work in Loaders and Actions

Use `request.signal` to abort in-flight async work when the client cancels a navigation.

## Why

- Prevent wasted work when the user navigates away
- Reduce backend load for requests that will never be used
- Keep loaders responsive under rapid navigation

## Fetch Calls

```ts
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let response = await fetch("https://example.com/api", {
    signal: request.signal,
  });

  return data(await response.json());
}
```

## Parallel Requests

```ts
export async function loader({ request }: Route.LoaderArgs) {
  let [a, b] = await Promise.all([
    fetch(urlA, { signal: request.signal }),
    fetch(urlB, { signal: request.signal }),
  ]);

  return data({ a: await a.json(), b: await b.json() });
}
```

## AbortError Handling

```ts
export async function loader({ request }: Route.LoaderArgs) {
  try {
    let response = await fetch(url, { signal: request.signal });
    return data(await response.json());
  } catch (error) {
    if (error instanceof Error && error.name === "AbortError") {
      return new Response(null, { status: 204 });
    }
    throw error;
  }
}
```

## Non-Fetch Async Work

If an API doesn't accept an `AbortSignal`, check manually between steps:

```ts
export async function action({ request }: Route.ActionArgs) {
  let first = await doStepOne();
  if (request.signal.aborted) throw new Error("AbortError");

  let second = await doStepTwo();
  if (request.signal.aborted) throw new Error("AbortError");

  return data({ first, second });
}
```

## Database Transactions

If the action touches your database, wrap the steps in a transaction so an abort can roll back partial work:

```ts
export async function action({ request }: Route.ActionArgs) {
  return db.transaction(async (trx) => {
    let a = await createA(trx);
    if (request.signal.aborted) throw new Error("AbortError");

    let b = await createB(trx, a.id);
    if (request.signal.aborted) throw new Error("AbortError");

    return data({ a, b });
  });
}
```

If the action triggers external side effects (fetching other APIs), rolling back may be difficult or impossible — design with idempotency or compensating actions in mind.

## Rules

1. Always pass `request.signal` to fetch calls in loaders/actions
2. Handle `AbortError` explicitly when you catch errors
3. For non-fetch async work, check `request.signal.aborted` between steps
4. Use DB transactions to roll back partial work in actions
5. Be cautious in actions that trigger side effects on external services
