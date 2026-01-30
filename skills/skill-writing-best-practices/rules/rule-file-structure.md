---
title: Rule File Structure
impact: HIGH
tags: [structure, rules]
---

# Rule File Structure

Each rule file in rules/ follows a consistent structure: frontmatter, explanation, examples, and numbered takeaways.

## Why

- **Consistency**: Same structure across all rules makes them easy to read
- **Completeness**: Structure ensures you cover why, what, and how
- **Actionable**: Numbered rules at the end give clear takeaways

## Frontmatter

Every rule file starts with YAML frontmatter:

```yaml
---
title: Human-Readable Rule Title
impact: HIGH
tags: [relevant, tags, here]
---
```

**Impact levels**:
- `CRITICAL` or `HIGH` - Core patterns, always follow
- `MEDIUM` - Important but with flexibility
- `LOW` - Nice-to-haves, edge cases

**Tags** help with discovery and grouping.

## Content Sections

### Title and Introduction

```markdown
# Rule Title

One paragraph explaining what to do. This is the quick summary someone
reads to understand the rule at a glance.
```

### Why Section

Explain the benefits with bullet points:

```markdown
## Why

- **Benefit One**: Concrete reason this matters
- **Benefit Two**: Another tangible benefit
- **Benefit Three**: Third reason to follow this pattern
```

Bold the benefit name, then explain. Each point should be a real, specific reason.

### Pattern/Example Section

Show the main pattern with code. Lead with the bad pattern, then show good:

```markdown
## Pattern

\`\`\`ruby
# Bad: Explanation of what's wrong
class BadExample
  def scattered_logic
    # logic here, there, everywhere
  end
end

# Good: Explanation of improvement
class GoodExample
  def focused_logic
    delegate_to_model
  end
end
\`\`\`

\`\`\`typescript
// Bad: Explanation of what's wrong
function BadExample() {
  // logic scattered in component
  const data = fetch(...);
  const processed = data.map(...);
  return <div>{processed}</div>;
}

// Good: Explanation of improvement
function GoodExample() {
  const { data } = useProcessedData();
  return <div>{data}</div>;
}
\`\`\`
```

### Additional Sections

Add sections as needed:

```markdown
## When to Use This

Specific situations where this pattern applies.

## When NOT to Use This

Exceptions and alternatives.

## Real-World Example

Actual code from a codebase showing this pattern.

## Common Mistakes

Pitfalls to avoid.
```

### Rules Section

End with numbered takeaways:

```markdown
## Rules

1. First concrete action to take
2. Second thing to remember
3. Third key point
```

Keep to 3-6 rules. These should be scannable action items.

## Complete Examples

### Ruby Example

```markdown
---
title: Keep Jobs Thin
impact: HIGH
tags: [jobs, architecture]
---

# Keep Jobs Thin

Jobs should be thin wrappers that call model methods. All business logic belongs in the model layer.

## Why

- **Testability**: Model methods can be unit tested without job infrastructure
- **Reusability**: Same logic works sync or async
- **Debuggability**: Logic isn't buried in job classes

## Pattern

\`\`\`ruby
# Bad: Logic in job
class ProcessOrderJob < ApplicationJob
  def perform(order)
    order.items.each { |i| i.product.decrement!(:stock) }
    order.update!(status: :processing)
  end
end

# Good: Job delegates to model
class ProcessOrderJob < ApplicationJob
  def perform(order)
    order.process
  end
end
\`\`\`

## Rules

1. Jobs call one method on the received record
2. All business logic lives in models
3. Namespace jobs to mirror model structure
```

### TypeScript Example

```markdown
---
title: Use Named Exports
impact: MEDIUM
tags: [modules, imports, typescript]
---

# Use Named Exports

Use named exports instead of default exports for better tooling support and explicit imports.

## Why

- **Refactoring**: Renaming is easier when the name is explicit at export
- **Autocomplete**: IDEs can suggest imports automatically
- **Tree-shaking**: Bundlers can eliminate unused named exports

## Pattern

\`\`\`typescript
// Bad: Default export
export default function formatCurrency(amount: number) {
  return `$${amount.toFixed(2)}`;
}

// Importing - name can be anything, easy to mismatch
import format from "./format";

// Good: Named export
export function formatCurrency(amount: number) {
  return `$${amount.toFixed(2)}`;
}

// Importing - name must match, IDE autocompletes
import { formatCurrency } from "./format";
\`\`\`

## Exception

Remix route components use default exports by convention:

\`\`\`typescript
// app/routes/dashboard.tsx
export default function Dashboard() {
  return <div>...</div>;
}
\`\`\`

## Rules

1. Use named exports for utilities, hooks, and components
2. Default exports only for framework conventions (routes)
3. One export per file is fine, still use named
```

## Rules

1. Start with frontmatter (title, impact, tags)
2. One-paragraph intro explains the rule
3. "Why" section has bulleted benefits
4. Show bad/good code examples
5. End with numbered takeaways (3-6 items)
6. Add extra sections only when needed
