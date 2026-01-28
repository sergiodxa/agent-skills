---
title: Stack Layout Utilities
impact: CRITICAL
tags: [layout, flex, utilities]
---

# Stack Layout Utilities

Use custom stack utilities instead of raw flex classes.

## Why

- More semantic and readable
- Consistent across codebase
- Shorter class names
- Defined in tailwind.config.js

## Stack Classes

| Class             | Equivalent                            | Description                |
| ----------------- | ------------------------------------- | -------------------------- |
| `v-stack`         | `flex flex-col`                       | Vertical stack             |
| `h-stack`         | `flex flex-row`                       | Horizontal stack           |
| `v-stack-reverse` | `flex flex-col-reverse`               | Reversed vertical          |
| `h-stack-reverse` | `flex flex-row-reverse`               | Reversed horizontal        |
| `z-stack`         | Grid overlay                          | Overlapping centered stack |
| `center`          | `flex items-center justify-center`    | Center both axes           |
| `spacer`          | `flex-1`                              | Flexible space filler      |
| `circle`          | `aspect-square rounded-full shrink-0` | Perfect circle             |

## Utility Definitions

```css
@utility v-stack {
  display: flex;
  flex-direction: column;
}

@utility v-stack-reverse {
  display: flex;
  flex-direction: column-reverse;
}

@utility h-stack {
  display: flex;
  flex-direction: row;
}

@utility h-stack-reverse {
  display: flex;
  flex-direction: row-reverse;
}

@utility z-stack {
  display: grid;
  align-items: center;
  justify-items: center;

  & > * {
    grid-area: 1 / 1 / 1 / 2;
  }
}

@utility center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@utility spacer {
  flex: 1 1 auto;
}

@utility circle {
  aspect-ratio: 1 / 1;
  border-radius: 9999px;
  flex-shrink: 0;
}
```

## Pattern

```tsx
// Bad
<div className="flex flex-col gap-4">
  <header className="flex flex-row items-center justify-between">
  <main className="flex-1">
  <footer>
</div>

// Good
<div className="v-stack gap-4">
  <header className="h-stack items-center justify-between">
  <main className="spacer">
  <footer>
</div>
```

## z-stack for Overlays

Stack elements on top of each other, centered:

```tsx
// Avatar with status indicator
<div className="z-stack">
  <img src={avatar} className="size-12 circle" />
  <div className="size-3 circle bg-success-500 self-end justify-self-end" />
</div>

// Image with overlay text
<div className="z-stack">
  <img src={background} />
  <h2 className="text-white text-2xl">Overlay Title</h2>
</div>
```

## Combining with Gap

```tsx
<div className="v-stack gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<div className="h-stack gap-2 items-center">
  <Icon />
  <span>Label</span>
</div>
```

## Combining with Alignment

```tsx
// Vertical stack, horizontally centered
<div className="v-stack items-center gap-4">

// Horizontal stack, vertically centered
<div className="h-stack items-center gap-2">

// Horizontal stack, space between
<div className="h-stack items-center justify-between">
```

## Center Utility

```tsx
// Center content in container
<div className="center h-screen">
  <div>Centered content</div>
</div>

// Center icon in button
<button className="center size-10 rounded-full bg-teal-500">
  <Icon />
</button>
```

## Spacer Utility

Push elements apart:

```tsx
<header className="h-stack items-center px-4">
  <Logo />
  <spacer className="spacer" />
  <UserMenu />
</header>

// Or use a div
<div className="h-stack">
  <div>Left</div>
  <div className="spacer" />
  <div>Right</div>
</div>
```
