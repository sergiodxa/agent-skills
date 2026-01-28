---
title: Revalidation Patterns
impact: LOW
tags: [loader, revalidation, real-time, polling]
---

# Revalidation Patterns

Use `useRevalidator` to refresh loader data based on user activity or time intervals.

## Why

- Keep dashboard data fresh without full page refresh
- Update data when user returns to the tab
- Provide near-real-time updates without WebSockets
- React Router automatically revalidates after actions, but not on focus/interval

## Basic Revalidation

```tsx
import { useRevalidator } from "react-router";
import { useEffect } from "react";

export default function Component() {
  let { revalidate } = useRevalidator();

  useEffect(() => {
    let id = setInterval(revalidate, 30000); // Every 30 seconds
    return () => clearInterval(id);
  }, [revalidate]);

  // ... render with loader data
}
```

## Smart Revalidation

Only revalidate when it makes sense - don't waste bandwidth or server resources.

### Pause When Tab is Hidden

```tsx
import { useSyncExternalStore } from "react";

function useVisibilityState() {
  return useSyncExternalStore(
    (callback) => {
      document.addEventListener("visibilitychange", callback);
      return () => document.removeEventListener("visibilitychange", callback);
    },
    () => document.visibilityState,
    () => "visible" as const,
  );
}

export default function Component() {
  let { revalidate } = useRevalidator();
  let visibilityState = useVisibilityState();

  useEffect(() => {
    if (visibilityState === "hidden") return; // Don't poll hidden tabs
    let id = setInterval(revalidate, 30000);
    return () => clearInterval(id);
  }, [revalidate, visibilityState]);
}
```

### Pause When Offline

```tsx
function useOnlineStatus() {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener("online", callback);
      window.addEventListener("offline", callback);
      return () => {
        window.removeEventListener("online", callback);
        window.removeEventListener("offline", callback);
      };
    },
    () => navigator.onLine,
    () => true,
  );
}

export default function Component() {
  let { revalidate } = useRevalidator();
  let isOnline = useOnlineStatus();
  let visibilityState = useVisibilityState();

  useEffect(() => {
    if (!isOnline) return; // Offline
    if (visibilityState === "hidden") return; // Hidden tab

    let id = setInterval(revalidate, 30000);
    return () => clearInterval(id);
  }, [revalidate, isOnline, visibilityState]);
}
```

### Revalidate on Focus

Refresh data when user returns to the tab:

```tsx
export default function Component() {
  const { revalidate } = useRevalidator();

  useEffect(() => {
    function onFocus() {
      revalidate();
    }

    window.addEventListener("focus", onFocus);
    window.addEventListener("visibilitychange", () => {
      if (document.visibilityState === "visible") onFocus();
    });

    return () => {
      window.removeEventListener("focus", onFocus);
      window.removeEventListener("visibilitychange", onFocus);
    };
  }, [revalidate]);
}
```

### Revalidate on Reconnect

Refresh when internet connection is restored:

```tsx
export default function Component() {
  const { revalidate } = useRevalidator();

  useEffect(() => {
    window.addEventListener("online", revalidate);
    return () => window.removeEventListener("online", revalidate);
  }, [revalidate]);
}
```

## Combined Hook

Create a reusable hook for common patterns:

```tsx
interface UseSmartRevalidationOptions {
  interval?: number; // Polling interval in ms (0 = disabled)
  onFocus?: boolean; // Revalidate when tab gains focus
  onReconnect?: boolean; // Revalidate when coming back online
}

function useSmartRevalidation({
  interval = 0,
  onFocus = false,
  onReconnect = false,
}: UseSmartRevalidationOptions = {}) {
  let { revalidate } = useRevalidator();
  let isOnline = useOnlineStatus();
  let visibilityState = useVisibilityState();

  // Interval polling
  useEffect(() => {
    if (interval <= 0) return;
    if (!isOnline) return;
    if (visibilityState === "hidden") return;

    let id = setInterval(revalidate, interval);
    return () => clearInterval(id);
  }, [revalidate, interval, isOnline, visibilityState]);

  // On focus
  useEffect(() => {
    if (!onFocus) return;
    window.addEventListener("focus", revalidate);
    return () => window.removeEventListener("focus", revalidate);
  }, [revalidate, onFocus]);

  // On reconnect
  useEffect(() => {
    if (!onReconnect) return;
    window.addEventListener("online", revalidate);
    return () => window.removeEventListener("online", revalidate);
  }, [revalidate, onReconnect]);
}

// Usage
export default function Dashboard() {
  useSmartRevalidation({
    interval: 30000, // Poll every 30s
    onFocus: true, // Refresh on tab focus
    onReconnect: true, // Refresh when back online
  });

  const { data } = useLoaderData<typeof loader>();
  // ...
}
```

## Caveats

### Scroll Position Issues

For feed-like UIs, revalidation can disrupt scroll position if new items are added. Consider:

- Showing "New items available" button instead of auto-updating
- Only revalidating when scrolled to top

### Server Load

Short intervals with many users can create high server load. Consider:

- Longer intervals (30s+ instead of 1s)
- WebSockets for truly real-time needs
- Server-Sent Events for push updates

### Save-Data Mode

Respect users who have enabled data-saving:

```tsx
const saveData = navigator.connection?.saveData ?? false;

useEffect(() => {
  if (saveData) return; // Don't poll on save-data mode
  // ... polling logic
}, [saveData /* ... */]);
```

## Rules

1. Don't revalidate hidden tabs - wastes resources
2. Don't revalidate when offline - will fail
3. Use longer intervals (30s+) to avoid server overload
4. Consider on-focus revalidation instead of polling for most cases
5. Respect save-data mode preferences
6. For truly real-time needs, consider WebSockets or SSE instead
