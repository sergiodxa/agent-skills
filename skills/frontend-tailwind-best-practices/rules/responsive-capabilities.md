---
title: Design for Capabilities, Not Device Labels
impact: MEDIUM
tags: [responsive, input, pointers]
---

# Design for Capabilities, Not Device Labels

Target input capabilities (pointer, hover) and viewport ranges instead of assuming “mobile” or “desktop.”

## Why

- Many devices support both touch and mouse
- Screen size doesn’t equal input capability
- Capability-based styles age better than device lists

## Pattern

Use pointer/hover variants to adjust targets and affordances:

```tsx
// Larger targets on coarse pointers (touch)
<button className="h-10 w-10 pointer-coarse:h-12 pointer-coarse:w-12">
  <Icon />
</button>

// Hover effects only when hover is supported
<button className="bg-gray-900 text-white hover:bg-gray-800">
  Primary
</button>
```

Use breakpoints for layout clusters, not specific devices:

```tsx
<div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
  {items.map((item) => (
    <Card key={item.id} />
  ))}
</div>
```

## Rules

1. Avoid “mobile vs desktop” assumptions in UI behavior
2. Use `pointer-coarse`/`pointer-fine` for target sizes
3. Keep hover-only affordances behind hover-capable variants
4. Place breakpoints in layout clusters, not exact device widths
