---
title: Only Route Component Calls Data Hooks
impact: MEDIUM
tags: [data-loading, components, architecture]
---

# Only Route Component Calls Data Hooks

Only the route's default export component should call `useLoaderData` and `useActionData`. Child components receive data via props.

## Why

- Clear data flow from route to children
- Components are reusable and testable
- Easier to understand where data comes from
- Avoids hidden dependencies in child components

## Pattern

```tsx
// route.tsx
export async function loader({ request }: Route.LoaderArgs) {
  let client = await authenticate(request);
  let items = await getItems(client);
  return data({ items });
}

export async function action({ request }: Route.ActionArgs) {
  // ... handle action
  return data({ errors: [] });
}

// Only the route component calls the hooks
export default function Component() {
  const { items } = useLoaderData<typeof loader>();
  let actionData = useActionData<typeof action>();

  return (
    <div>
      {actionData?.errors && <ErrorList errors={actionData.errors} />}
      <ItemList items={items} />
    </div>
  );
}
```

## Child Components Receive Props

```tsx
// components/item-list.tsx
interface ItemListProps {
  items: Item[];
}

// Component receives data as props, doesn't call useLoaderData
export function ItemList({ items }: ItemListProps) {
  return (
    <ul>
      {items.map((item) => (
        <ItemCard key={item.id} item={item} />
      ))}
    </ul>
  );
}

// components/item-card.tsx
interface ItemCardProps {
  item: Item;
}

export function ItemCard({ item }: ItemCardProps) {
  return <li>{item.name}</li>;
}
```

## Exception: Deeply Nested Data

For deeply nested components that need parent route data, use `useRouteLoaderData`:

```tsx
// Deeply nested component needing user from parent layout
function UserAvatar() {
  // This is acceptable for layout-level data
  let data = useRouteLoaderData<typeof profileLoader>("routes/_._profile");
  return <Avatar src={data?.user.avatar} />;
}
```

## Benefits

```tsx
// Components are easily testable
import { render } from "@testing-library/react";
import { ItemList } from "./item-list";

test("renders items", () => {
  let items = [{ id: "1", name: "Test" }];
  render(<ItemList items={items} />);
  // No need to mock useLoaderData
});
```
