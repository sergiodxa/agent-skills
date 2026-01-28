---
title: Avoid navigate(-1) for In-App Back Links
impact: MEDIUM
tags: [navigation, links, ux]
---

# Avoid navigate(-1) for In-App Back Links

Prefer explicit back targets stored in navigation state instead of `navigate(-1)`.

## Why

- Browser history isn’t scoped to your app
- `navigate(-1)` can send users out of your app
- Back links should stay within known routes

## Pattern

Pass an explicit back target when you link to a detail route:

```tsx
import { Link, useLocation, useNavigate } from "react-router";

function useCurrentURL() {
  let location = useLocation();
  return location.pathname + location.search;
}

function useBackNavigation() {
  let navigate = useNavigate();
  let location = useLocation();

  return (event: React.MouseEvent<HTMLAnchorElement>) => {
    let back = location.state?.back;
    if (back) {
      event.preventDefault();
      navigate(back);
    }
  };
}

export function ListItemLink({ id }: { id: string }) {
  let back = useCurrentURL();

  return (
    <Link to={`/items/${id}`} state={{ back }}>
      View
    </Link>
  );
}

export function BackLink() {
  let onClick = useBackNavigation();

  return (
    <Link to="/items" onClick={onClick}>
      Back
    </Link>
  );
}
```

## Rules

1. Avoid `navigate(-1)` for in-app back links
2. Pass explicit `state.back` when linking into detail views
3. Use a fallback `to` for cases without back state
