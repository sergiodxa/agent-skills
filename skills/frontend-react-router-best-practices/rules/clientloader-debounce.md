---
title: Debounce Loaders and Actions with clientLoader/clientAction
impact: MEDIUM
tags: [clientLoader, clientAction, debounce, performance]
---

# Debounce Loaders and Actions with clientLoader/clientAction

Use `clientLoader` and `clientAction` to debounce at the route level instead of in components.

## Why

- Centralizes debounce logic in the route, not scattered across components
- Automatically cancels pending requests when a new one starts
- Works with `<Form>` and `fetcher.Form` without custom `onSubmit` handlers
- Makes debounce timing easy to change or remove
- Route controls _when_ its logic runs, not just _what_ it does

## Pattern: Debounce a Loader

For search inputs that trigger on every keystroke:

```tsx
import { setTimeout } from "node:timers/promises";
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let url = new URL(request.url);
  let query = url.searchParams.get("query") ?? "";
  let results = await searchItems(query);
  return data({ results });
}

export async function clientLoader({
  request,
  serverLoader,
}: Route.ClientLoaderArgs) {
  // Debounce by 500ms - if user types again, request.signal aborts this
  return await setTimeout(500, serverLoader, { signal: request.signal });
}

// Required to enable clientLoader
clientLoader.hydrate = true;
```

The `clientLoader` is called on every keystroke, but `serverLoader` only runs after 500ms of inactivity. If the same fetcher calls `clientLoader` again before the delay completes, the previous request is aborted via `request.signal`.

## Pattern: Debounce an Action

For actions that fire frequently, like scroll position tracking:

```tsx
import { setTimeout } from "node:timers/promises";
import { data } from "react-router";

export async function action({ request, params }: Route.ActionArgs) {
  let formData = await request.formData();
  let scrollY = Number(formData.get("scrollY"));
  await updateReadProgress(params.postId, scrollY);
  return data({ ok: true });
}

export async function clientAction({
  request,
  serverAction,
}: Route.ClientActionArgs) {
  // Debounce by 50ms for scroll events
  return await setTimeout(50, serverAction, { signal: request.signal });
}
```

## How It Works

1. User triggers loader/action (typing, scrolling, etc.)
2. `clientLoader`/`clientAction` starts a `setTimeout`
3. If triggered again before timeout completes, `request.signal` aborts the pending request
4. Only the last request after the debounce period calls the server

## Component Usage

No special handling needed in components:

```tsx
export default function SearchPage() {
  let { results } = useLoaderData<typeof loader>();
  let [searchParams, setSearchParams] = useSearchParams();

  return (
    <div>
      <input
        type="search"
        defaultValue={searchParams.get("query") ?? ""}
        onChange={(e) => setSearchParams({ query: e.target.value })}
      />
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

The route handles debouncing - the component just updates search params normally.

## When to Use

| Scenario            | Debounce Time |
| ------------------- | ------------- |
| Search/autocomplete | 300-500ms     |
| Form auto-save      | 1000-2000ms   |
| Scroll tracking     | 50-100ms      |
| Resize handling     | 100-200ms     |

## Rules

1. Use `clientLoader`/`clientAction` for route-level debouncing
2. Pass `{ signal: request.signal }` to `setTimeout` for automatic cancellation
3. Import `setTimeout` from `node:timers/promises` (returns a Promise)
4. Set `clientLoader.hydrate = true` when using clientLoader
5. Prefer route-level debounce over component-level `useDebounce` hooks
