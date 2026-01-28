---
title: Touch Target Sizes
impact: MEDIUM
tags: [accessibility, touch, mobile]
---

# Touch Target Sizes

Ensure interactive elements are large enough to tap accurately on touch devices.

## Why

- Users with motor impairments need larger tap targets
- Fat finger problem on mobile devices
- WCAG 2.5.5 requires 44x44px minimum for Level AAA

## Minimum Sizes

| Level    | Minimum Size | Use Case                          |
| -------- | ------------ | --------------------------------- |
| WCAG AA  | 24x24px      | Minimum acceptable                |
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
// Good - padding makes the whole area tappable
<nav>
  {links.map((link) => (
    <Link
      key={link.href}
      to={link.href}
      className="block py-3 px-4" // Full-width tappable area
    >
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

## Bad Patterns

### Too Small

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

## Expanding Click Area

Make the clickable area larger than the visible element:

```tsx
// Technique 1: Padding
<button className="p-4 -m-4"> {/* Negative margin compensates */}
  <SmallIcon />
</button>

// Technique 2: Pseudo-element (in CSS)
.small-button {
  position: relative;
}
.small-button::before {
  content: '';
  position: absolute;
  inset: -8px; /* Expands clickable area */
}
```

## Rules

1. All touch targets should be at least 44x44px
2. Leave at least 8px spacing between adjacent targets
3. Use padding to expand tap area, not just visual size
4. Icon-only buttons need explicit width/height
5. Make entire list items/cards tappable, not just text
6. Test on actual touch devices, not just browser dev tools
