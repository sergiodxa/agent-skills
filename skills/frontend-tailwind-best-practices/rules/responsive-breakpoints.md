---
title: Responsive Breakpoints
impact: MEDIUM
tags: [responsive, breakpoints, mobile-first]
---

# Responsive Breakpoints

Use responsive prefixes for mobile-first design with Tailwind defaults.

## Available Breakpoints

### Standard (min-width)

| Prefix | Min Width | Description                  |
| ------ | --------- | ---------------------------- |
| `sm:`  | 640px     | Large phones / small tablets |
| `md:`  | 768px     | Tablets                      |
| `lg:`  | 1024px    | Small laptops                |
| `xl:`  | 1280px    | Desktops                     |
| `2xl:` | 1536px    | Extra large screens          |

## Pattern

```tsx
// Mobile-first: base is mobile, add for larger
<div className="px-4 md:px-8 lg:px-15">

// Hide/show with standard breakpoints
<div className="hidden md:block">Desktop only</div>
<div className="md:hidden">Mobile only</div>
```

## Common Responsive Patterns

### Show/Hide Elements

```tsx
// Desktop navigation (hidden on mobile)
<nav className="hidden lg:flex h-stack gap-4">

// Mobile menu button (hidden on desktop)
<button className="lg:hidden">
  <MenuIcon />
</button>

// Show different content per breakpoint
<span className="md:hidden">Mobile text</span>
<span className="hidden md:inline">Desktop text</span>
```

### Responsive Spacing

```tsx
// Padding that increases at breakpoints
<div className="p-4 md:p-6 lg:p-8 xl:p-12">

// Gap that increases
<div className="v-stack gap-4 md:gap-6 lg:gap-8">

// Margin that changes
<section className="mt-8 md:mt-12 lg:mt-16">
```

### Responsive Sizing

```tsx
// Width changes at breakpoints
<aside className="w-full md:w-64 lg:w-80">

// Max-width increases
<div className="max-w-sm md:max-w-md lg:max-w-lg">

// Container width
<div className="w-full px-4 md:px-8 lg:max-w-6xl lg:mx-auto">
```

### Responsive Grid

```tsx
// 1 column mobile, 2 tablet, 3 desktop
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">

// Different gap per breakpoint
<div className="grid grid-cols-2 gap-2 md:gap-4 lg:gap-6">
```

### Responsive Typography

```tsx
// Font size increases
<h1 className="text-2xl md:text-3xl lg:text-4xl xl:text-5xl">

// Line height changes with size
<p className="text-sm leading-5 md:text-base md:leading-6 lg:text-lg lg:leading-7">
```

## Combining Breakpoints

```tsx
// Complex responsive behavior
<div className={cn(
  "v-stack gap-4",           // Base: vertical stack
  "md:h-stack md:gap-6",     // Medium+: horizontal
  "lg:gap-8",                // Large+: bigger gap
  "sm:p-4",                  // Small+: padding
)}>
```

## Testing Breakpoints

When testing responsive designs:

- sm: 640px
- md: 768px (iPad portrait)
- lg: 1024px (iPad landscape, small laptops)
- xl: 1280px (standard desktop)
