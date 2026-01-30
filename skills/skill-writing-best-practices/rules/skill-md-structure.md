---
title: SKILL.md Structure
impact: HIGH
tags: [structure, skill-file]
---

# SKILL.md Structure

The main SKILL.md file has four parts: frontmatter, overview, rules summary, and optional philosophy section.

## Why

- **Quick reference**: Agents can scan SKILL.md to find relevant rules fast
- **Context**: Frontmatter helps agents decide when to load this skill
- **Depth on demand**: Summary links to detailed rules when needed

## 1. Frontmatter

YAML frontmatter with name and description:

```yaml
---
name: topic-best-practices
description: Brief description of what this skill covers. Mention when to use it.
---
```

The description should help agents understand when to reference this skill. Include trigger conditions if relevant.

## 2. Overview

Title, intro, and application guidance:

```markdown
# Topic Best Practices

Brief intro about what's covered. Mention rule count and categories.

## When to Apply

Reference these guidelines when:

- Doing X
- Working with Y
- Reviewing Z code
```

Keep the intro to 1-2 sentences. The bullet list helps agents quickly assess relevance.

## 3. Rules Summary

Group rules by category with impact levels. Each rule gets:
- Header linking to full file
- One-sentence description
- Short code example showing the core pattern

```markdown
## Rules Summary

### Category Name (IMPACT)

#### rule-name - @rules/rule-name.md

One sentence explaining what to do.

\`\`\`ruby
# Bad
bad_example

# Good
good_example
\`\`\`

#### another-rule - @rules/another-rule.md

Another one-sentence explanation.
```

Impact levels:
- **CRITICAL/HIGH** - Core patterns, always follow
- **MEDIUM** - Important but flexible
- **LOW** - Nice-to-haves

## 4. Philosophy (Optional)

End with core principles if the skill embodies a specific approach:

```markdown
## Philosophy

These patterns embody X approach:

1. **Principle One** - Brief explanation
2. **Principle Two** - Brief explanation
```

## Complete Example

```markdown
---
name: example-best-practices
description: Example patterns. Use when working with examples.
---

# Example Best Practices

Patterns for examples. Contains 3 rules in 2 categories.

## When to Apply

- Writing examples
- Reviewing example code

## Rules Summary

### Structure (HIGH)

#### example-structure - @rules/example-structure.md

Examples should be self-contained.

\`\`\`ruby
# Good: Complete example
def complete_example
  setup
  action
  verify
end
\`\`\`

### Style (MEDIUM)

#### example-naming - @rules/example-naming.md

Use descriptive names.

## Philosophy

1. **Clarity** - Examples should be obvious
2. **Brevity** - Show only what matters
```

## Rules

1. Frontmatter has `name` and `description`
2. Overview includes "When to Apply" bullets
3. Rules are grouped by category with impact levels
4. Each rule gets one sentence + short code example
5. Link to full rules with `@rules/rule-name.md`
