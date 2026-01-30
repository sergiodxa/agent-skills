---
title: Writing Style
impact: MEDIUM
tags: [style, tone, formatting]
---

# Writing Style

Write naturally like documentation. Avoid AI-isms, excessive formatting, and filler phrases.

## Why

- **Readability**: Natural prose is easier to scan
- **Trust**: AI-sounding text feels generated, not authored
- **Density**: Removing filler packs more value per line
- **Professionalism**: Clean writing reflects clear thinking

## Avoid Horizontal Rules

Don't use `---` as section separators:

```markdown
# Bad
## Section One

Content here.

---

## Section Two

More content.

---

# Good
## Section One

Content here.

## Section Two

More content.
```

Headings already create visual separation.

## Avoid Filler Phrases

Cut phrases that add no information:

```markdown
# Bad
"In this section, we will explore the various ways in which..."
"It's important to note that..."
"As mentioned previously..."
"Let's take a look at..."

# Good
Just say the thing directly.
```

## Avoid Over-Formatting

Don't bold or bullet everything:

```markdown
# Bad: Everything is emphasized
**Always** use `after_commit` for jobs because:
- **It ensures** transaction safety
- **It prevents** race conditions
- **It guarantees** data consistency

# Good: Emphasis is meaningful
Use `after_commit` for jobs. This ensures the transaction has committed
before the job runs, preventing race conditions where the job can't find
the record.
```

Reserve bold for terms being defined or key concepts in lists.

## Use Direct Language

```markdown
# Bad: Passive and hedging
"It is recommended that consideration be given to..."
"One approach that could potentially be utilized..."
"It should be noted that in some cases..."

# Good: Direct
"Use X when Y."
"Consider X for Y situations."
"X doesn't apply when Y."
```

## Keep Paragraphs Short

Long paragraphs are hard to scan:

```markdown
# Bad: Wall of text
When implementing the pattern you should consider that there are multiple
approaches and each has tradeoffs. The first approach involves X which has
the benefit of Y but the downside of Z. The second approach...

# Good: Broken up
Consider two approaches:

**First approach**: X. Benefits from Y but has Z downside.

**Second approach**: A. Better for B situations.
```

## Code Comments Should Be Minimal

Let code speak for itself:

```ruby
# Bad: Over-commented
# This method closes the card by creating a closure record
# and then touching the updated_at timestamp
def close
  # Create the closure record for this card
  create_closure!(user: Current.user)
  # Update the timestamp
  touch
end

# Good: Comments add context code can't express
def close
  create_closure!(user: Current.user)
  touch  # Triggers cache invalidation
end
```

## Rules

1. No horizontal rules (`---`) as separators
2. Cut filler phrases that add no information
3. Reserve formatting (bold, bullets) for emphasis, not decoration
4. Use direct, active language
5. Keep paragraphs short and scannable
6. Code comments explain why, not what
