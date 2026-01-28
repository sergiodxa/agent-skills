---
title: Prefer Gaps Over Margins
impact: HIGH
tags: [layout, spacing, flex, grid]
---

# Prefer Gaps Over Margins

Use `gap-*` on the parent container instead of `m-*`/`mt-*` on children when spacing siblings.

## Why

- Parent controls layout; children stay reusable
- Avoids margins inside components that break encapsulation
- No “last item” exceptions or conditional class logic
- Easier to switch layout direction at breakpoints
- Avoids margin-collapsing surprises

## Pattern

```tsx
// Bad: child margins, special-case last item
<div>
  {items.map((item, index) => (
    <Item
      key={item.id}
      className={index === items.length - 1 ? "" : "mb-4"}
    />
  ))}
</div>

// Good: parent gap controls spacing
<div className="flex flex-col gap-4">
  {items.map((item) => (
    <Item key={item.id} />
  ))}
</div>
```

## Component Encapsulation

Avoid margins inside components. Instead, let parents decide spacing:

```tsx
// Bad: component defines its own spacing
function Card() {
  return <div className="mb-4 rounded-lg border p-4" />;
}

// Good: parent controls spacing
function Card() {
  return <div className="rounded-lg border p-4" />;
}

<div className="v-stack gap-4">
  <Card />
  <Card />
</div>
```

## Responsive Layouts

```tsx
// Switch direction without touching children
<div className="flex flex-col gap-4 md:flex-row">
  <Item />
  <Item />
  <Item />
</div>
```

## Rules

1. Use `gap-*` for spacing between siblings in flex/grid
2. Avoid margins inside components; let parents control spacing
3. Keep margins for one-off external offsets only
4. Prefer `gap-*` for lists, stacks, and repeating content
