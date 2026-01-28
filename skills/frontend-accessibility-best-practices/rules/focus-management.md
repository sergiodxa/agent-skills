---
title: Focus Management
impact: HIGH
tags: [accessibility, focus, keyboard]
---

# Focus Management

Manage focus visibility and trapping for keyboard users.

## Focus Visibility

Always show a visible focus indicator for keyboard users.

### Use focus-visible

The `focus-visible` pseudo-class shows focus only for keyboard navigation, not mouse clicks:

```tsx
// Good - focus ring only shows for keyboard users
<button className="focus-visible:ring-2 focus-visible:ring-teal-600">
  Click me
</button>

// With react-aria data attributes
<Button className="data-[focus-visible]:ring-2 data-[focus-visible]:ring-teal-600">
  Click me
</Button>
```

### Bad - Removing Focus Outlines

```tsx
// Bad - removes focus indicator entirely
<button className="focus:outline-none">Click me</button>

// Bad - outline:none without replacement
button:focus {
  outline: none;
}
```

### Good - Custom Focus Styles

```tsx
// Good - custom focus style that's still visible
<button className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-teal-600 focus-visible:ring-offset-2">
  Click me
</button>
```

## Focus Trapping

Trap focus within modals and dialogs so keyboard users can't Tab out.

### React Aria Handles This

Modal and Dialog from react-aria-components automatically trap focus:

```tsx
import { Modal, Dialog } from "react-aria-components";

function MyModal({ isOpen, onClose }) {
  return (
    <Modal isOpen={isOpen} onOpenChange={onClose}>
      <Dialog>
        {/* Focus is automatically trapped here */}
        <Heading slot="title">Modal Title</Heading>
        <p>Modal content...</p>
        <Button onPress={onClose}>Close</Button>
      </Dialog>
    </Modal>
  );
}
```

### Manual Focus Trapping

If not using react-aria, implement focus trapping:

```tsx
import { useFocusTrap } from "@mantine/hooks"; // or similar

function Modal({ children, onClose }) {
  let focusTrapRef = useFocusTrap();

  return (
    <div ref={focusTrapRef} role="dialog" aria-modal="true">
      {children}
    </div>
  );
}
```

## Focus Restoration

Return focus to the trigger element when a modal closes:

```tsx
// react-aria handles this automatically
<DialogTrigger>
  <Button>Open Dialog</Button>
  <Modal>
    <Dialog>{/* When closed, focus returns to the trigger button */}</Dialog>
  </Modal>
</DialogTrigger>
```

## Programmatic Focus

Move focus to important content:

```tsx
function SearchResults({ results, error }) {
  let errorRef = useRef<HTMLDivElement>(null);
  let resultsRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (error) {
      errorRef.current?.focus();
    } else if (results.length > 0) {
      resultsRef.current?.focus();
    }
  }, [error, results]);

  return (
    <>
      {error && (
        <div ref={errorRef} tabIndex={-1} role="alert">
          {error}
        </div>
      )}
      <div ref={resultsRef} tabIndex={-1}>
        {/* Results */}
      </div>
    </>
  );
}
```

## Skip Links

Add skip links for keyboard users to bypass navigation:

```tsx
function Layout({ children }) {
  return (
    <>
      <a
        href="#main-content"
        className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 focus:z-50 focus:bg-white focus:p-4"
      >
        {t("Skip to main content")}
      </a>
      <Header />
      <Main id="main-content">{children}</Main>
    </>
  );
}
```

## Rules

1. Never remove focus outlines without providing an alternative
2. Use `focus-visible:` for focus styles (not `focus:`)
3. Use react-aria-components for modals/dialogs - they handle focus trapping
4. Return focus to trigger element when closing modals
5. Use `tabIndex={-1}` for elements that receive programmatic focus
6. Consider adding skip links for pages with significant navigation
7. Test focus management by navigating with Tab and Shift+Tab
