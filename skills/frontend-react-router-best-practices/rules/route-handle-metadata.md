---
title: Route Handle Metadata
impact: MEDIUM
tags: [routes, metadata, handle]
---

# Route Handle Metadata

Use handle export with app-defined handle types for route metadata.

## Why

- Control client-side hydration behavior
- Add external scripts per route
- Configure layout features (banners, CTAs, aside panels)
- Type-safe handle configuration

## Handle Types

Define typed handle interfaces in your app (for example, `Handle`, `LayoutHandle`, `AppHandle`) and use them consistently across routes.

### Handle (Base)

The basic handle for all routes:

```tsx
export const handle: Handle = {
  // Enable client-side JavaScript for this route
  hydrate: true,
};

// Or conditionally based on loader data
export const handle: Handle<typeof loader> = {
  hydrate: (data) => data.needsInteractivity,
};
```

### LayoutHandle

For routes within the main layout:

```tsx
export const handle: LayoutHandle<typeof loader> = {
  hydrate: true,
  stickyHeader: true,
  footerType: "app", // "app" | "public"
  canvasColor: "white", // "white" | "neutral"

  // Dynamic banner above header
  banner: (data) =>
    data.showBanner
      ? {
          title: "Special offer",
          link: "/offer",
        }
      : null,

  // CTA buttons in header
  mainCTA: (data) => ({
    label: "Upgrade",
    to: "/upgrade",
  }),
  secondaryCTA: (data) => ({
    label: "Contact Sales",
    to: "/contact",
  }),
};
```

### AppHandle

For authenticated app routes (extends LayoutHandle):

```tsx
export const handle: AppHandle<typeof loader> = {
  hydrate: true,
  footerType: "app",

  // Render a component in the right aside column
  Aside: (data) => <SidebarWidget data={data} />,
};
```

## Common Patterns

### Hydration Control

```tsx
// Always hydrate (needs JS)
export const handle: Handle = {
  hydrate: true,
};

// Never hydrate (static page)
export const handle: Handle = {
  hydrate: false,
};

// Conditional hydration
export const handle: Handle<typeof loader> = {
  hydrate: (data) => data.hasInteractiveFeatures,
};
```

### External Scripts

```tsx
export const handle: Handle<typeof loader> = {
  hydrate: true,
  scripts: (data) => [
    {
      src: "https://example.com/widget.js",
      async: true,
    },
  ],
};
```

### Layout Configuration

```tsx
export const handle: LayoutHandle = {
  hydrate: true,
  stickyHeader: true,
  canvasColor: "neutral",
  footerType: "public",
};
```

### App Route with Aside

```tsx
export const handle: AppHandle<typeof loader> = {
  hydrate: true,
  Aside: (data) => (
    <aside className="p-4">
      <h2>Related Items</h2>
      {data.relatedItems.map((item) => (
        <ItemCard key={item.id} item={item} />
      ))}
    </aside>
  ),
};
```
