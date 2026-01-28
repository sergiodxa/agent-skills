---
title: Working with Color Schemes
impact: MEDIUM
tags: [colors, theming, dark-mode]
---

# Working with Color Schemes

Use class-based color schemes (`light`, `dark`, `system`) with a custom Tailwind `dark` variant.

## Why

- No flash of incorrect scheme on first paint
- Works with system preference and explicit user choice
- Keeps styles consistent with `dark:` utilities

## Tailwind Setup

Override the `dark` variant to support `.dark` and `.system` classes:

## Pattern

```css
@custom-variant dark {
  &:where(.dark *, .dark) {
    @slot;
  }

  &:where(.system *, .system) {
    @media (prefers-color-scheme: dark) {
      @slot;
    }
  }
}
```

## Usage

```tsx
// Base styles
<button className="rounded-full bg-gray-900 px-4 py-2 text-white dark:bg-gray-100 dark:text-gray-900">
  Toggle
</button>
```

Apply `light`, `dark`, or `system` on the root element and Tailwind will resolve `dark:` based on the class.
