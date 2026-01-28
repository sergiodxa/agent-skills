---
title: className Prop Types
impact: MEDIUM
tags: [classname, typescript, components]
---

# className Prop Types

Use proper TypeScript types for className props from `~/lib/cn`.

## Why

- Type safety for className props
- Support for multi-element className records
- Consistent API across components
- IDE autocomplete for className keys

## Types

```tsx
import type { ClassName, ClassNameRecord } from "~/lib/cn";
```

### ClassName

For components with a single styleable element:

```tsx
type Props = {
  className?: ClassName;
};

function Button({ className }: Props) {
  return <button className={cn("base", className)} />;
}

// Usage
<Button className="mt-4" />
<Button className={["mt-4", isLarge && "text-lg"]} />
```

### ClassNameRecord

For components with multiple styleable elements:

```tsx
type Props = {
  className?: ClassNameRecord<"root" | "label" | "input" | "error">;
};

function TextField({ className }: Props) {
  return (
    <div className={cn("v-stack gap-1", className?.root)}>
      <label className={cn("text-sm font-medium", className?.label)}>
        {label}
      </label>
      <input className={cn("rounded-lg border px-3 py-2", className?.input)} />
      {error && (
        <p className={cn("text-sm text-failure-600", className?.error)}>
          {error}
        </p>
      )}
    </div>
  );
}

// Usage
<TextField
  className={{
    root: "w-full",
    label: "text-neutral-600",
    input: "border-failure-500",
  }}
/>;
```

## Common Patterns

### Modal with Multiple Parts

```tsx
type Props = {
  className?: ClassNameRecord<
    "overlay" | "container" | "header" | "body" | "footer"
  >;
};

function Modal({ className, children }: Props) {
  return (
    <div className={cn("fixed inset-0 bg-neutral-900/50", className?.overlay)}>
      <div className={cn("bg-white rounded-xl", className?.container)}>
        <header className={cn("p-4 border-b", className?.header)}>
          {title}
        </header>
        <div className={cn("p-4", className?.body)}>{children}</div>
        <footer className={cn("p-4 border-t", className?.footer)}>
          {actions}
        </footer>
      </div>
    </div>
  );
}
```

### Card Component

```tsx
type Props = {
  className?: ClassNameRecord<"root" | "header" | "body">;
};

function Card({ className, title, children }: Props) {
  return (
    <div className={cn("rounded-xl bg-white shadow", className?.root)}>
      {title && (
        <div className={cn("px-4 py-3 border-b", className?.header)}>
          <h3 className="font-semibold">{title}</h3>
        </div>
      )}
      <div className={cn("p-4", className?.body)}>{children}</div>
    </div>
  );
}
```

### List Item

```tsx
type Props = {
  className?: ClassNameRecord<"root" | "icon" | "content" | "action">;
};

function ListItem({ className, icon, children, onAction }: Props) {
  return (
    <div className={cn("h-stack items-center gap-3 p-3", className?.root)}>
      {icon && <div className={cn("shrink-0", className?.icon)}>{icon}</div>}
      <div className={cn("grow min-w-0", className?.content)}>{children}</div>
      {onAction && (
        <button
          className={cn("shrink-0", className?.action)}
          onClick={onAction}
        >
          Action
        </button>
      )}
    </div>
  );
}
```

## When to Use Which

| Scenario                          | Type                   |
| --------------------------------- | ---------------------- |
| Single wrapper element            | `ClassName`            |
| Component with internal structure | `ClassNameRecord<...>` |
| Forwarding to child component     | Match child's type     |
