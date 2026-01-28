---
title: Use Semantic HTML Landmarks
impact: HIGH
tags: [accessibility, semantic-html, landmarks]
---

# Use Semantic HTML Landmarks

Use semantic HTML elements to create page landmarks that screen readers can navigate.

## Why

- Screen reader users can jump between landmarks (main, nav, header, footer)
- Provides document outline without relying on visual layout
- Better SEO and machine readability

## Landmark Elements

| Element     | Purpose                      | ARIA Role                      |
| ----------- | ---------------------------- | ------------------------------ |
| `<main>`    | Primary page content         | `main`                         |
| `<nav>`     | Navigation links             | `navigation`                   |
| `<header>`  | Introductory content         | `banner` (when top-level)      |
| `<footer>`  | Footer content               | `contentinfo` (when top-level) |
| `<aside>`   | Tangentially related content | `complementary`                |
| `<section>` | Thematic grouping            | `region` (when labeled)        |
| `<article>` | Self-contained content       | `article`                      |

## Bad

```tsx
// Generic divs provide no semantic meaning
function Page() {
  return (
    <div className="page">
      <div className="header">...</div>
      <div className="sidebar">...</div>
      <div className="content">...</div>
      <div className="footer">...</div>
    </div>
  );
}
```

## Good

```tsx
import { Main } from "~/components/heading";

function Page() {
  return (
    <>
      <header>
        <nav aria-label={t("Primary")}>...</nav>
      </header>

      <Main>
        <article>...</article>
        <aside>...</aside>
      </Main>

      <footer>...</footer>
    </>
  );
}
```

## Multiple Navigation Regions

When you have multiple `<nav>` elements, label them:

```tsx
<header>
  <nav aria-label={t("Primary site")}>
    {/* Main navigation */}
  </nav>
</header>

<aside>
  <nav aria-label={t("Table of contents")}>
    {/* Page navigation */}
  </nav>
</aside>

<footer>
  <nav aria-label={t("Footer")}>
    {/* Footer links */}
  </nav>
</footer>
```

## Rules

1. Every page should have exactly one `<main>` element (use `Main` component)
2. Use `<header>` for page/section headers, not just `<div className="header">`
3. Use `<nav>` for navigation, always with `aria-label` when multiple exist
4. Use `<footer>` for page/section footers
5. Use `<article>` for self-contained content (blog posts, cards, comments)
6. Use `<aside>` for sidebars, related content, or call-outs
7. Use `<section>` with Region component for labeled content sections
