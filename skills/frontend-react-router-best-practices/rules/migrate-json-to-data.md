---
title: Migrate from json() to data()
impact: HIGH
tags: [migration, single-fetch, data-loading]
---

# Migrate from json() to data()

Replace `json()` with `data()` from `react-router`. The `json()` helper is deprecated with Single Fetch.

## Why

- `json()` is deprecated with Single Fetch enabled
- `data()` is the new standard for all loader and action responses
- `data()` supports automatic promise streaming
- Consistent API for responses with or without status codes/headers

## Migration Pattern

### Loaders

```tsx
// Before
import { json } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return json({ items });
}

// After
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return data({ items });
}
```

### Actions

```tsx
// Before
import { json } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();
  let result = await processForm(formData);
  return json({ success: true, result });
}

// After
import { data } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  let formData = await request.formData();
  let result = await processForm(formData);
  return data({ success: true, result });
}
```

### With Status Codes

```tsx
// Before
import { json } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  try {
    // ... validation
  } catch (error) {
    if (error instanceof z.ZodError) {
      return json({ errors: error.issues }, { status: 400 });
    }
    throw error;
  }
}

// After
import { data } from "react-router";

export async function action({ request }: Route.ActionArgs) {
  try {
    // ... validation
  } catch (error) {
    if (error instanceof z.ZodError) {
      return data({ errors: error.issues }, { status: 400 });
    }
    throw error;
  }
}
```

### With Headers

```tsx
// Before
import { json } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return json(
    { items },
    {
      headers: {
        "Cache-Control": "max-age=300",
      },
    },
  );
}

// After
import { data } from "react-router";

export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return data(
    { items },
    {
      headers: {
        "Cache-Control": "max-age=300",
      },
    },
  );
}
```

### Throwing Errors

```tsx
// Before
import { json } from "react-router";

export async function loader({ params }: Route.LoaderArgs) {
  let item = await getItem(params.id);
  if (!item) {
    throw json({ message: "Not found" }, { status: 404 });
  }
  return json({ item });
}

// After
import { data } from "react-router";

export async function loader({ params }: Route.LoaderArgs) {
  let item = await getItem(params.id);
  if (!item) {
    throw data({ message: "Not found" }, { status: 404 });
  }
  return data({ item });
}
```

## Important: Always Use data()

Do NOT return raw objects from loaders or actions:

```tsx
// Bad: raw object return
export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return { items }; // Don't do this
}

// Good: always use data()
export async function loader({ request }: Route.LoaderArgs) {
  let items = await getItems();
  return data({ items });
}
```
