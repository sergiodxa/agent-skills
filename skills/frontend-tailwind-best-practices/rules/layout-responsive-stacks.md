---
title: Responsive Stack Layouts
impact: MEDIUM
tags: [layout, responsive, stacks]
---

# Responsive Stack Layouts

Switch layout direction at breakpoints using stack utilities with responsive prefixes.

## Why

- Common pattern for mobile-first layouts
- Sidebar layouts that stack on mobile
- Card grids that become lists on mobile

## Pattern

```tsx
// Mobile: vertical, Desktop: horizontal
<div className="v-stack lg:h-stack gap-4">
  <main className="grow">Main content</main>
  <aside className="shrink-0 lg:w-80">Sidebar</aside>
</div>
```

## Common Layouts

### Page with Sidebar

```tsx
<div className="v-stack lg:h-stack gap-6">
  <main className="grow v-stack gap-4">{/* Main content */}</main>
  <aside className="shrink-0 w-full lg:max-w-xs v-stack gap-4">
    {/* Sidebar content */}
  </aside>
</div>
```

### Card Grid to List

```tsx
// Grid on desktop, stack on mobile
<div className="v-stack md:h-stack md:flex-wrap gap-4">
  {items.map(item => (
    <Card key={item.id} className="md:w-[calc(50%-0.5rem)] lg:w-[calc(33.333%-0.67rem)]" />
  ))}
</div>

// Or use CSS grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
```

### Header Navigation

```tsx
<header className="h-stack items-center justify-between">
  <Logo />

  {/* Desktop nav */}
  <nav className="h-stack gap-4 max-md:hidden">
    <Link to="/about">About</Link>
    <Link to="/contact">Contact</Link>
  </nav>

  {/* Mobile menu button */}
  <button className="md:hidden">
    <MenuIcon />
  </button>
</header>
```

### Form Layout

```tsx
// Side-by-side on desktop, stacked on mobile
<div className="v-stack md:h-stack gap-4">
  <Input label="First Name" className="md:w-1/2" />
  <Input label="Last Name" className="md:w-1/2" />
</div>
```

## Reverse on Breakpoint

```tsx
// Normal order on mobile, reversed on desktop
<div className="v-stack lg:h-stack-reverse">
  <Content /> {/* First on mobile, second on desktop */}
  <Sidebar /> {/* Second on mobile, first on desktop */}
</div>
```

## With Grow and Shrink

```tsx
<div className="v-stack lg:h-stack gap-4">
  {/* Takes remaining space */}
  <div className="grow min-w-0">
    <Content />
  </div>

  {/* Fixed width, doesn't shrink */}
  <div className="shrink-0 lg:w-64">
    <Sidebar />
  </div>
</div>
```
