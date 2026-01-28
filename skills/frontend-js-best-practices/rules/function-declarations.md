---
title: Function Declarations vs Arrow Functions
impact: MEDIUM
impactDescription: readability and debugging
tags: javascript, functions, conventions
---

## Function Declarations vs Arrow Functions

Prefer function declarations for named functions. Use arrow functions for inline callbacks or when needed for type inference.

### Prefer Function Declarations

```typescript
// Good: function declaration
function calculateTotal(items: Item[]): number {
  let total = 0;
  for (let item of items) {
    total += item.price * item.quantity;
  }
  return total;
}

function formatUser(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}

async function fetchUsers(): Promise<User[]> {
  let response = await fetch("/api/users");
  return response.json();
}

// Avoid: arrow function for named functions
const calculateTotal = (items: Item[]): number => {
  let total = 0;
  for (let item of items) {
    total += item.price * item.quantity;
  }
  return total;
};
```

### Use Arrow Functions For

#### 1. Inline Callbacks

```typescript
// Good: arrow for inline callbacks
let activeUsers = users.filter((user) => user.isActive);
let names = users.map((user) => user.name);
let sorted = items.toSorted((a, b) => a.price - b.price);

// Avoid: function expression for simple callbacks
let activeUsers = users.filter(function (user) {
  return user.isActive;
});
```

#### 2. When Types Require Arrow Syntax

```typescript
// Good: arrow needed for proper typing
const createHandler: ActionFunction = async ({ request }) => {
  let formData = await request.formData();
  // ...
};

// Good: arrow for typed event handlers
const handleClick: MouseEventHandler<HTMLButtonElement> = (event) => {
  event.preventDefault();
  // ...
};

// Good: arrow for React component props
const renderItem: ListRenderItem<User> = ({ item }) => (
  <UserCard user={item} />
);
```

### Why Function Declarations

1. **Hoisting** - Can be called before definition (useful for organizing code)
2. **Better stack traces** - Named functions show up clearly in error stacks
3. **Clearer intent** - `function` keyword signals "this is a function"
4. **Consistency** - One style for all named functions

### Summary

| Use Case                                    | Syntax                           |
| ------------------------------------------- | -------------------------------- |
| Named functions (including short utilities) | `function name() {}`             |
| Inline callbacks                            | `(x) => x.value`                 |
| Typed handlers/callbacks                    | `const handler: Type = () => {}` |
