---
title: No Default Exports
impact: MEDIUM
impactDescription: consistency and refactoring safety
tags: javascript, modules, exports, conventions
---

## No Default Exports

Use named exports for all modules. Avoid default exports.

### Why

1. **Consistent imports** - Same name everywhere, easier to search/refactor
2. **Better IDE support** - Auto-imports work reliably
3. **Explicit naming** - Forces meaningful names at export site
4. **Easier refactoring** - Rename symbol, all imports update

### Pattern

```typescript
// Bad: default export
export default function formatCurrency(amount: number) {
  return `$${amount.toFixed(2)}`;
}

// Bad: default export class
export default class UserService {
  // ...
}

// Good: named export
export function formatCurrency(amount: number) {
  return `$${amount.toFixed(2)}`;
}

// Good: named export class
export class UserService {
  // ...
}
```

### Components

```tsx
// Bad: default export component
export default function UserCard({ user }: Props) {
  return <div>{user.name}</div>;
}

// Good: named export component
export function UserCard({ user }: Props) {
  return <div>{user.name}</div>;
}
```

### Importing

```typescript
// With named exports - consistent name everywhere
import { UserCard } from "~/components/user-card";
import { formatCurrency } from "~/lib/format";

// With default exports - name can vary (bad)
import UserCard from "~/components/user-card";
import Card from "~/components/user-card"; // Same component, different name!
```

### Exception: Remix Route Components

Remix requires a default export for the route component. Name it `Component`:

```tsx
// app/routes/_.users/route.tsx
export async function loader() {
  // ...
}

export async function action() {
  // ...
}

// Exception: Remix requires default export for route component
// Always name it "Component"
export default function Component() {
  let data = useLoaderData<typeof loader>();
  return <div>{/* ... */}</div>;
}
```

### Re-exporting

```typescript
// Bad: re-export as default
export { UserCard as default } from "./user-card";

// Good: named re-export
export { UserCard } from "./user-card";

// Good: barrel file with named exports
export { UserCard } from "./user-card";
export { UserList } from "./user-list";
export { UserAvatar } from "./user-avatar";
```
