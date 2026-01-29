---
title: Stream Updates with SSE
impact: MEDIUM
tags: [sse, streaming]
---

# Stream Updates with SSE

Use `eventStream` in a resource route and `useEventSource` in the UI for live updates.

## Pattern

```ts
// app/routes/sse.time.ts
import { eventStream } from "remix-utils/sse/server";

export async function loader({ request }: Route.LoaderArgs) {
  return eventStream(request.signal, (send) => {
    let intervalId = setInterval(() => {
      send({ event: "time", data: new Date().toISOString() });
    }, 1000);
    return () => clearInterval(intervalId);
  });
}
```

```tsx
import { useEventSource } from "remix-utils/sse/react";

export function Clock() {
  let time = useEventSource("/sse/time", { event: "time" });
  if (!time) return null;
  return <time dateTime={time}>{time}</time>;
}
```

## Rules

1. Put SSE in a resource route
2. Clean up intervals on disconnect
3. Reuse one connection per URL/event
