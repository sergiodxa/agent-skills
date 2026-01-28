---
title: Touch Target Sizes
impact: MEDIUM
tags: [accessibility, touch, mobile]
---

# Touch Target Sizes

Ensure interactive elements have sufficient target size and spacing across input types.

## Why

- Users with motor impairments need larger tap targets
- Fat finger problem on mobile devices
- WCAG 2.5.5 recommends 44x44px for AAA; WCAG 2.5.8 allows 24x24px minimum if spaced
- Touch targets need spacing to avoid accidental taps

## Minimum Sizes

| Level    | Minimum Size | Use Case                          |
| -------- | ------------ | --------------------------------- |
| WCAG AA  | 24x24px      | Minimum if targets don't overlap  |
| WCAG AAA | 44x44px      | Recommended for all touch targets |
| iOS HIG  | 44x44pt      | Apple's recommendation            |
| Material | 48x48dp      | Google's recommendation           |

## Implementation

### Buttons

```tsx
// Good - explicit minimum size
<Button className="min-h-11 min-w-11 px-4 py-2">
  {t("Submit")}
</Button>

// Icon buttons need explicit sizing
<Button variant="icon" className="h-11 w-11">
  <XMarkIcon className="h-5 w-5" />
  <span className="sr-only">{t("Close")}</span>
</Button>
```

### Links in Lists

```tsx
// Good - padding on the link (target) itself
<nav>
  {links.map((link) => (
    <Link key={link.href} to={link.href} className="block px-4 py-3">
      {link.label}
    </Link>
  ))}
</nav>
```

### Checkboxes and Radios

```tsx
// Good - label wraps input for larger tap area
<label className="flex items-center gap-3 py-2 cursor-pointer">
  <input type="checkbox" className="h-5 w-5" />
  <span>{t("Accept terms")}</span>
</label>
```

## Common Issues

### Too Small or Too Close

```tsx
// Bad - icon button too small
<button className="h-6 w-6">
  <XIcon className="h-4 w-4" />
</button>

// Bad - links too close together
<div className="flex gap-1">
  <a href="/a">A</a>
  <a href="/b">B</a>
  <a href="/c">C</a>
</div>
```

### Adequate Spacing

```tsx
// Good - adequate spacing between targets
<div className="flex gap-4">
  <Button>Option A</Button>
  <Button>Option B</Button>
  <Button>Option C</Button>
</div>
```

## Expanding Target Area

Make the clickable area larger than the visible element:

```tsx
// Technique 1: Padding on the target
<button className="p-3">
  <SmallIcon />
</button>

// Technique 2: Pseudo-element (in CSS)
.small-button {
  position: relative;
}
.small-button::before {
  content: "";
  position: absolute;
  inset: -8px; /* Expands clickable area */
}
```

## Spacing and Context

- Small targets need more spacing between them
- Large targets can sit closer without overlap
- Inline links rely on line height; increase `leading` for readability

## Rules

1. Aim for 44x44px targets; allow 24x24px only with sufficient spacing
2. Increase spacing when targets are small or dense
3. Apply padding to the clickable element itself
4. Icon-only buttons need explicit size (min 44x44)
5. Avoid dead zones between related targets
6. Test on real touch devices, not just DevTools
