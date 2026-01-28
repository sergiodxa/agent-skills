---
title: Combine Multiple Array Iterations
impact: LOW-MEDIUM
impactDescription: reduces iterations
tags: javascript, arrays, loops, performance
---

## Combine Multiple Array Iterations

Multiple `.filter()` or `.map()` calls iterate the array multiple times. Combine into one loop.

**Incorrect (3 iterations):**

```typescript
let admins = users.filter((u) => u.isAdmin);
let testers = users.filter((u) => u.isTester);
let inactive = users.filter((u) => !u.isActive);
```

**Correct (1 iteration):**

```typescript
let admins: User[] = [];
let testers: User[] = [];
let inactive: User[] = [];

for (let user of users) {
  if (user.isAdmin) admins.push(user);
  if (user.isTester) testers.push(user);
  if (!user.isActive) inactive.push(user);
}
```

**Alternative using `Array#reduce`:**

```typescript
type UserGroup = "admins" | "testers" | "inactive";

let { admins, testers, inactive } = users.reduce(
  (acc, user) => {
    if (user.isAdmin) acc.admins.push(user);
    if (user.isTester) acc.testers.push(user);
    if (!user.isActive) acc.inactive.push(user);
    return acc;
  },
  { admins: [], testers: [], inactive: [] } as Record<UserGroup, User[]>,
);
```
