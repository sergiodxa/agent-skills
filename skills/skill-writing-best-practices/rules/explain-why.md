---
title: Explain Why, Not Just What
impact: HIGH
tags: [content, reasoning, clarity]
---

# Explain Why, Not Just What

Don't just show what to do. Explain why it matters. Rules without reasoning feel arbitrary and are easy to ignore.

## Why

- **Buy-in**: People follow rules they understand
- **Judgment**: Understanding "why" helps apply rules to edge cases
- **Memory**: Reasoning makes patterns memorable
- **Trust**: Explained rules feel like advice, not commands

## Bad: Rules Without Reasoning

```markdown
## Pattern

# Ruby example
Use `after_create_commit` instead of `after_create` for jobs.

\`\`\`ruby
after_create_commit :notify_later
\`\`\`

# TypeScript example
Use `useCallback` for event handlers passed to children.

\`\`\`typescript
const handleClick = useCallback(() => {
  doSomething();
}, []);
\`\`\`
```

These tell you what to do but not why. Someone might wonder: "What's wrong with `after_create`?" or "Why do I need `useCallback`?"

## Good: Rules With Why

```markdown
## Why

- **Transaction safety**: `after_create` runs inside the transaction; if it fails, the record isn't saved. `after_commit` runs after the transaction succeeds.
- **Job reliability**: Jobs enqueued in `after_create` might run before the transaction commits, causing "record not found" errors.

## Pattern

\`\`\`ruby
# Bad: Job might run before transaction commits
after_create :notify_later

# Good: Job runs after transaction is committed
after_create_commit :notify_later
\`\`\`
```

```markdown
## Why

- **Referential stability**: Without `useCallback`, the function is recreated every render, causing child components to re-render unnecessarily.
- **Dependency safety**: React hooks that depend on this function won't trigger infinite loops.

## Pattern

\`\`\`typescript
// Bad: New function every render, children re-render
function Parent() {
  const handleClick = () => doSomething();
  return <Child onClick={handleClick} />;
}

// Good: Stable reference, children don't re-render
function Parent() {
  const handleClick = useCallback(() => doSomething(), []);
  return <Child onClick={handleClick} />;
}
\`\`\`
```

Now the reader understands the actual problem being solved.

## Structure for Why Sections

Use bulleted list with bold benefit names:

```markdown
## Why

- **Testability**: The sync method can be unit tested without job infrastructure
- **Flexibility**: Callers choose sync or async based on context
- **Clarity**: The `_later` suffix makes async behavior explicit
```

Each bullet should be:
1. A bolded benefit name (one or two words)
2. A concrete explanation (one sentence)

## Good Why Statements

**Concrete and specific**:
- "Jobs enqueued in `after_create` might run before the transaction commits"
- "Model methods can be tested without spinning up job infrastructure"
- "Without `useCallback`, the function reference changes every render"
- "Named exports enable tree-shaking and better IDE autocomplete"

**Bad Why Statements**

**Too vague**:
- "It's better practice"
- "It's more maintainable"
- "It follows the principle of X"

**Too theoretical**:
- "This adheres to SOLID principles"
- "It reduces coupling"
- "It improves separation of concerns"

These aren't wrong, but they don't help someone understand the practical impact.

## When to Skip Why

Some patterns are genuinely conventions without deep reasoning:

```markdown
# Naming convention - just explain it
Ruby: Use `-able` suffix for behavior concerns: `Closeable`, `Searchable`.
TypeScript: Use PascalCase for components: `UserCard`, `OrderList`.
```

But even here, you can add light reasoning:

```markdown
# Better
Use `-able` suffix for behavior concerns. This communicates that the concern
adds a capability: something that "can be closed" or "can be searched."

Use PascalCase for components. This distinguishes components from regular
functions and matches React's convention for JSX tag detection.
```

## Rules

1. Every non-trivial rule needs a "Why" section
2. Use bulleted list with bold benefit names
3. Each bullet = one concrete, specific reason
4. Avoid vague phrases like "more maintainable"
5. Practical impact beats theoretical principles
