---
title: ARIA Live Regions
impact: HIGH
tags: [accessibility, aria, screen-reader, dynamic-content]
---

# ARIA Live Regions

Use ARIA live regions to announce dynamic content changes to screen readers.

## Why

Screen readers don't automatically announce content that changes after page load. Live regions tell assistive technology to announce updates.

## Live Region Types

| Attribute               | When to Use                                              |
| ----------------------- | -------------------------------------------------------- |
| `aria-live="polite"`    | Non-urgent updates, wait for user to finish current task |
| `aria-live="assertive"` | Urgent updates that interrupt the user (use sparingly)   |
| `role="status"`         | Status messages (implicitly `aria-live="polite"`)        |
| `role="alert"`          | Error messages (implicitly `aria-live="assertive"`)      |

## Common Use Cases

### Form Validation Errors

```tsx
// Error container - announced immediately
{
  error && (
    <p role="alert" className="text-failure-600">
      {error}
    </p>
  );
}
```

### Loading States

```tsx
// Status updates - announced politely
<div role="status" aria-live="polite">
  {isLoading ? t("Loading...") : t("Loaded {{count}} results", { count })}
</div>
```

### Toast Notifications

```tsx
function Toast({ message, variant }: ToastProps) {
  return (
    <div
      role={variant === "error" ? "alert" : "status"}
      aria-live={variant === "error" ? "assertive" : "polite"}
    >
      {message}
    </div>
  );
}
```

### Search Results Count

```tsx
<Region>
  <Heading className="sr-only">{t("Search results")}</Heading>
  <p role="status" aria-live="polite">
    {t("{{count}} results found", { count: results.length })}
  </p>
  <ResultsList results={results} />
</Region>
```

### Form Submission Status

```tsx
function SubmitButton({ isSubmitting, isSuccess }: Props) {
  return (
    <>
      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? t("Saving...") : t("Save")}
      </Button>
      <div role="status" aria-live="polite" className="sr-only">
        {isSubmitting && t("Saving your changes...")}
        {isSuccess && t("Changes saved successfully")}
      </div>
    </>
  );
}
```

## Bad Patterns

### Missing Live Region for Dynamic Content

```tsx
// Bad - screen reader won't announce this
function SearchResults({ results }) {
  return (
    <div>
      <p>{results.length} results</p>
      {results.map((r) => (
        <Result key={r.id} {...r} />
      ))}
    </div>
  );
}

// Good - announced to screen readers
function SearchResults({ results }) {
  return (
    <div>
      <p role="status" aria-live="polite">
        {t("{{count}} results", { count: results.length })}
      </p>
      {results.map((r) => (
        <Result key={r.id} {...r} />
      ))}
    </div>
  );
}
```

### Overusing assertive

```tsx
// Bad - every update interrupts the user
<div aria-live="assertive">
  {t("{{count}} items in cart", { count })}
</div>

// Good - polite for non-urgent updates
<div aria-live="polite">
  {t("{{count}} items in cart", { count })}
</div>
```

## Rules

1. Use `role="alert"` for error messages that need immediate attention
2. Use `role="status"` or `aria-live="polite"` for non-urgent updates
3. Keep announcements concise - don't announce entire paragraphs
4. The live region must exist in the DOM before content changes
5. Use `assertive` sparingly - only for critical, time-sensitive information
6. Consider combining with `sr-only` for announcements that don't need visual display
