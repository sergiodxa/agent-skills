---
title: Responsive Typography
impact: MEDIUM
tags: [responsive, typography, text]
---

# Responsive Typography

Scale text sizes responsively across breakpoints.

## Why

- Larger screens can accommodate larger text
- Improves readability at different distances
- Maintains visual hierarchy across devices

## Pattern

```tsx
// Heading that scales
<h1 className="text-2xl md:text-3xl lg:text-4xl font-bold">

// Body text that scales
<p className="text-sm md:text-base lg:text-lg">
```

## Font Scale Reference

| Class       | Size | Usage                       |
| ----------- | ---- | --------------------------- |
| `text-xs`   | 12px | Captions, labels            |
| `text-sm`   | 14px | Secondary text, mobile body |
| `text-base` | 16px | Body text                   |
| `text-lg`   | 18px | Large body, subheadings     |
| `text-xl`   | 20px | Small headings              |
| `text-2xl`  | 24px | Section headings            |
| `text-3xl`  | 30px | Page headings               |
| `text-4xl`  | 36px | Hero headings               |

## Common Patterns

### Page Title

```tsx
<h1 className="text-2xl md:text-3xl lg:text-4xl font-bold text-neutral-900">
  Page Title
</h1>
```

### Section Heading

```tsx
<h2 className="text-xl md:text-2xl font-semibold text-neutral-900">
  Section Title
</h2>
```

### Card Title

```tsx
<h3 className="text-lg md:text-xl font-medium text-neutral-900">Card Title</h3>
```

### Body Text

```tsx
<p className="text-sm md:text-base text-neutral-600 leading-relaxed">
  Body content that's readable on all devices.
</p>
```

### Small Text / Captions

```tsx
<span className="text-xs md:text-sm text-neutral-500">Updated 2 hours ago</span>
```

## Line Height with Text Size

When text size changes, line height often needs adjustment:

```tsx
// Explicit line height per breakpoint
<p className="text-sm leading-5 md:text-base md:leading-6 lg:text-lg lg:leading-7">

// Or use relative line heights
<p className="text-sm md:text-base lg:text-lg leading-relaxed">
```

## Font Weight with Size

Larger text often needs different weight:

```tsx
// Hero text: larger = lighter weight acceptable
<h1 className="text-3xl font-bold md:text-4xl md:font-semibold">

// Keep it simple when possible
<h1 className="text-2xl md:text-4xl font-bold">
```

## Truncation

```tsx
// Single line truncation
<p className="truncate">Long text that will be truncated...</p>

// Multi-line clamp
<p className="line-clamp-2 md:line-clamp-3">
  Text clamped to 2 lines on mobile, 3 on tablet+
</p>
```

## Anti-Patterns

```tsx
// Bad: hardcoded pixels
<p style={{ fontSize: "14px" }}>

// Bad: too many breakpoint changes
<p className="text-xs sm:text-sm md:text-base lg:text-lg xl:text-xl">

// Good: 2-3 breakpoints max
<p className="text-sm md:text-base lg:text-lg">
```
