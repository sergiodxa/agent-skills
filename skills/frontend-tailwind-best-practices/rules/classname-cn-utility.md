---
title: Use cn() for className Merging
impact: CRITICAL
tags: [classname, utilities, components]
---

# Use cn() for className Merging

Always use the `cn()` utility to merge classNames in components.

## Why

- Properly merges Tailwind classes (handles conflicts)
- Supports conditional classes with objects
- Accepts arrays, strings, undefined
- External className always wins (applied last)

## Import

```tsx
import { cn } from "~/lib/cn";
```

## Definition

```ts
import type { ClassValue } from "clsx";
import type { CSSProperties } from "react";

import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export type ClassName = ClassValue;
export type ClassNameRecord<Key extends string> = { [K in Key]?: ClassName };

type Style = CSSProperties & { [key: `--${string}`]: string };
export type StyleRecord<Key extends string> = { [K in Key]?: Style };

export function cn(...classes: ClassName[]): string {
  return twMerge(clsx(...classes));
}
```

## Pattern

```tsx
function Button({ className, variant, size, disabled }: Props) {
  return (
    <button
      className={cn(
        // Base classes always applied
        "inline-flex items-center justify-center rounded-lg font-medium",
        // Variant classes (conditional object)
        {
          "bg-teal-500 text-white": variant === "primary",
          "bg-neutral-100 text-neutral-900": variant === "secondary",
          "bg-transparent text-teal-600": variant === "ghost",
        },
        // Size classes
        {
          "px-3 py-1.5 text-sm": size === "sm",
          "px-4 py-2 text-base": size === "md",
          "px-6 py-3 text-lg": size === "lg",
        },
        // State classes
        disabled && "opacity-50 cursor-not-allowed",
        // External className ALWAYS LAST
        className,
      )}
    />
  );
}
```

## Conditional Classes

### Object Syntax (Preferred)

```tsx
className={cn(
  "base",
  {
    "active-class": isActive,
    "disabled-class": isDisabled,
    "error-class": hasError,
  }
)}
```

### Logical AND

```tsx
className={cn(
  "base",
  isActive && "active-class",
  isDisabled && "disabled-class"
)}
```

### Ternary

```tsx
className={cn(
  "base",
  isActive ? "bg-teal-500" : "bg-neutral-100"
)}
```

## External className Last

Always put the `className` prop last so consumers can override:

```tsx
// Component
function Card({ className }: { className?: ClassName }) {
  return (
    <div
      className={cn(
        "rounded-xl bg-white p-4",
        className, // Can override padding, background, etc.
      )}
    />
  );
}

// Usage - override works
<Card className="p-8 bg-neutral-50" />;
```

## Arrays

```tsx
const baseClasses = ["rounded-lg", "font-medium"];
const sizeClasses = size === "lg" ? ["px-6", "py-3"] : ["px-4", "py-2"];

className={cn(baseClasses, sizeClasses, className)}
```

## Handling Undefined

`cn()` safely ignores undefined/null/false values:

```tsx
className={cn(
  "base",
  maybeUndefined,             // Ignored if undefined
  condition && "conditional", // Ignored if false
  className                   // Ignored if not passed
)}
```

## Anti-Patterns

```tsx
// Bad - string concatenation
className={`base ${isActive ? "active" : ""} ${className}`}

// Bad - template literal without cn
className={`base ${className || ""}`}

// Bad - className not last
className={cn(className, "base-classes")}

// Bad - not using cn at all
className="static-classes-only"
```
