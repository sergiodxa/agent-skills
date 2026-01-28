---
title: Route Import Restrictions
impact: HIGH
tags: [routes, imports, architecture]
---

# Route Import Restrictions

Avoid importing from route files. Routes can import their own modules or non-route modules, but other code should not import from routes.

## Why

- Routes are entry points, not shared modules
- Prevents circular dependencies
- Keeps route code isolated and refactorable
- Clear dependency direction: shared code → routes, not routes → routes

## Rules

### Routes Can Import

```tsx
// app/routes/_.users/route.tsx

// Good: import from own route folder
import { queryUsers } from "./queries.server";
import { actionSchema } from "./schemas.server";
import { UserCard } from "./components/user-card";

// Good: import from shared modules
import { Button } from "~/components/ui/button";
import { formatCurrency } from "~/lib/format";
import { authenticate } from "~/lib/auth.server";

// Good: import from components (non-route)
import { PageHeader } from "~/components/page-header";
```

### Routes Should NOT Import From Other Routes

```tsx
// app/routes/_.orders/route.tsx

// Bad: importing from another route
import { UserCard } from "~/routes/users/components/user-card";

// Bad: importing queries from another route
import { queryUser } from "~/routes/users/queries.server";

// Good: move shared code to a shared location
import { UserCard } from "~/components/user-card";
import { queryUser } from "~/lib/users.server";
```

### Exception: Loader/Action Types for Type Inference

Components may import `loader` or `action` types for `useFetcher` type inference:

```tsx
// app/components/order-form.tsx
import type { action } from "~/routes/api.orders/route";

type Props = {
  itemId: string;
};

export function OrderForm({ itemId }: Props) {
  // Type inference for fetcher data
  let fetcher = useFetcher<typeof action>();

  return (
    <fetcher.Form method="post" action="/api/orders">
      {/* ... */}
    </fetcher.Form>
  );
}
```

```tsx
// app/components/search-results.tsx
import type { loader } from "~/routes/api.search/route";

export function SearchResults() {
  let fetcher = useFetcher<typeof loader>();

  // fetcher.data is typed based on loader return
  return (
    <div>
      {fetcher.data?.results.map((result) => (
        <SearchResultItem key={result.id} result={result} />
      ))}
    </div>
  );
}
```

### What to Move to Shared Locations

| If you need to share... | Move to...                     |
| ----------------------- | ------------------------------ |
| Components              | `app/components/`              |
| Hooks                   | `app/hooks/`                   |
| Utilities               | `app/utils/`                   |
| Data fetching           | `app/services/`                |
| Types                   | `app/types.ts` or `app/types/` |
| Zod schemas             | `app/deserializers/`           |

## Architecture

```
app/
├── components/      # Shared components (can be imported anywhere)
├── hooks/           # Shared hooks
├── services/        # Shared server logic
├── utils/           # Shared utilities
├── ui/              # UI primitives
└── routes/
    ├── _.users/
    │   ├── route.tsx         # Can import from above folders
    │   ├── queries.server.ts # Route-specific, not shared
    │   └── components/       # Route-specific components
    └── _.orders/
        ├── route.tsx         # Cannot import from _.users/
        └── ...
```
