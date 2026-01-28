---
title: Affordance Classes
impact: HIGH
tags: [css, layering, utilities, components]
---

# Affordance Classes

Create element-agnostic visual patterns (affordances) with Tailwind `@utility`, `@apply`, and `@variant`.

## Why

- Decouple appearance from element choice (`button`, `label`, `a`, `summary`)
- Keep a single source of truth for interactive styles
- Preserve Tailwind tree-shaking and IntelliSense
- Let utilities override affordances without specificity fights

## Pattern

Define affordances with `@utility` so they are tree-shakeable and show up in IntelliSense. Use `:where()` for zero specificity and `@variant` for readable states:

```css
@utility ui-button {
  :where(&) {
    @apply inline-flex items-center gap-2 rounded-md px-4 py-2 text-sm font-semibold;
    @apply bg-primary text-primary-foreground shadow-sm;

    @variant hover {
      @apply bg-primary/90;
    }

    @variant focus-visible {
      @apply outline-2 outline-offset-2 outline-primary;
    }
  }
}

@utility ui-input {
  :where(&) {
    @apply block w-full rounded-md border border-neutral-300 bg-white px-3 py-2;
    @apply text-neutral-900;

    @variant focus-visible {
      @apply border-primary outline-2 outline-offset-2 outline-primary;
    }
  }
}
```

## Usage

```tsx
// Label styled like a button
<label className="ui-button" htmlFor="document-upload">
  Choose file
</label>

// Utilities can still override
<button className="ui-button bg-red-600 hover:bg-red-500">Delete</button>

// Input affordance on any element that needs to look typeable
<input className="ui-input" />
<textarea className="ui-input" />
```

## Rules

1. Use a `ui-` (or similar) prefix to signal affordance classes
2. Use `@utility` so affordances are tree-shakeable and discoverable
3. Wrap styles in `:where()` to keep specificity at zero
4. Use `@variant` blocks for readable state styles
