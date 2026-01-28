---
title: Meta Function V2
impact: MEDIUM
tags: [seo, meta, routes]
---

# Meta Function V2

Use meta function with loader data for dynamic SEO and OpenGraph tags.

## Why

- Dynamic titles and descriptions based on page content
- Proper OpenGraph tags for social sharing
- Type-safe access to loader data
- Centralized SEO logic in loader or meta function

## Basic Pattern

```tsx
import { data } from "react-router";

export async function loader({ params }: Route.LoaderArgs) {
  let item = await getItem(params.itemId);
  return data({
    item,
    title: item.name,
    description: item.summary,
  });
}

export const meta: Route.MetaFunction<typeof loader> = ({ data }) => {
  if (!data) return [];

  return [
    { title: data.title },
    { name: "description", content: data.description },
    { property: "og:title", content: data.title },
    { property: "og:description", content: data.description },
  ];
};
```

## With Centralized SEO Helper

Use a helper function to generate consistent meta tags:

```tsx
import { data } from "react-router";
import { seo } from "~/lib/seo.server";

export async function loader({ request }: Route.LoaderArgs) {
  let t = await i18n.getFixedT(request);
  let item = await getItem();

  return data({
    item,
    meta: seo(t, {
      title: t("Page Title - {{name}}", { name: item.name }),
      description: t("Description for {{name}}", { name: item.name }),
      og: {
        title: item.name,
        description: item.summary,
        image: item.imageUrl,
      },
    }),
  });
}

export const meta: Route.MetaFunction<typeof loader> = ({ data }) => data?.meta ?? [];
```

## OpenGraph Tags

```tsx
export const meta: Route.MetaFunction<typeof loader> = ({ data }) => {
  if (!data) return [];

  return [
    { title: data.title },
    { name: "description", content: data.description },

    // OpenGraph
    { property: "og:type", content: "website" },
    { property: "og:title", content: data.title },
    { property: "og:description", content: data.description },
    { property: "og:image", content: data.imageUrl },
    { property: "og:url", content: data.canonicalUrl },

    // Twitter
    { name: "twitter:card", content: "summary_large_image" },
    { name: "twitter:title", content: data.title },
    { name: "twitter:description", content: data.description },
    { name: "twitter:image", content: data.imageUrl },
  ];
};
```

## With Parent Data

Access parent route loader data:

```tsx
import type { loader as parentLoader } from "../_layout/route";

export const meta: Route.MetaFunction<
  typeof loader,
  { "routes/_layout": typeof parentLoader }
> = ({ data, matches }) => {
  let parentData = matches.find((m) => m.id === "routes/_layout")?.data;

  return [{ title: `${data?.item.name} | ${parentData?.siteName}` }];
};
```

## Handling Missing Data

Always handle the case where data might be undefined (error states):

```tsx
export const meta: Route.MetaFunction<typeof loader> = ({ data }) => {
  // Return empty array or default meta when data is missing
  if (!data) {
    return [{ title: "Error" }];
  }

  return [
    { title: data.title },
    { name: "description", content: data.description },
  ];
};
```

## Static Meta

For routes with static meta, you can return a simple array:

```tsx
export const meta: Route.MetaFunction = () => [
  { title: "About Us" },
  { name: "description", content: "Learn more about our company" },
];
```
