---
title: Screen Reader Only Text (sr-only)
impact: MEDIUM
tags: [accessibility, screen-reader, sr-only]
---

# Screen Reader Only Text (sr-only)

Use the `sr-only` class to provide text for screen readers that is visually hidden.

## Why

- Icon-only buttons need text labels for screen readers
- Visual context (like icons, colors) needs text alternatives
- Some content is clear visually but needs explanation for screen readers

## The sr-only Class

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

## Common Use Cases

### Icon-Only Buttons

```tsx
// Bad - no accessible name
<Button variant="icon" onPress={onClose}>
  <XMarkIcon />
</Button>

// Good - sr-only provides the name
<Button variant="icon" onPress={onClose}>
  <XMarkIcon aria-hidden="true" />
  <span className="sr-only">{t("Close")}</span>
</Button>

// Also good - aria-label
<Button variant="icon" onPress={onClose} aria-label={t("Close")}>
  <XMarkIcon aria-hidden="true" />
</Button>
```

### Visual-Only Table Headers

```tsx
<table>
  <thead className="sr-only">
    <tr>
      <th>{t("Item name")}</th>
      <th>{t("Amount")}</th>
      <th>{t("Date")}</th>
    </tr>
  </thead>
  <tbody>{/* Visual rows with no visible headers */}</tbody>
</table>
```

### Section Headings for Screen Reader Navigation

```tsx
import { Region, Heading } from "~/components/heading";

<Region>
  <Heading className="sr-only">{t("Search results")}</Heading>
  <SearchResultsList />
</Region>;
```

### Currency/Unit Indicators

```tsx
<span className="sr-only" id="currency">{t("Currency USD")}</span>
<input type="number" aria-describedby="currency" />
```

### Contextual Information

```tsx
// Badge that's visually clear but needs context
<Badge variant="success">
  <CheckIcon aria-hidden="true" />
  <span className="sr-only">{t("Status:")}</span>
  {t("Approved")}
</Badge>
```

## When NOT to Use sr-only

### Don't Hide Important Content

```tsx
// Bad - hiding content that should be visible
<Button>
  <span className="sr-only">{t("Submit form")}</span>
</Button>

// Good - visible text
<Button>{t("Submit")}</Button>
```

### Don't Duplicate Visible Text

```tsx
// Bad - redundant
<Button>
  {t("Submit")}
  <span className="sr-only">{t("Submit")}</span>
</Button>

// Good - just the visible text
<Button>{t("Submit")}</Button>
```

## Rules

1. Use `sr-only` for text that provides context missing from visual presentation
2. Always add `aria-hidden="true"` to decorative icons
3. Every interactive element must have an accessible name (visible text, sr-only, or aria-label)
4. Don't use sr-only to hide content that should be visible to all users
5. Don't duplicate visible text with sr-only text
