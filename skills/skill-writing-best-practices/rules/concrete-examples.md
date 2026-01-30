---
title: Use Concrete Examples
impact: HIGH
tags: [content, examples, clarity]
---

# Use Concrete Examples

Every rule needs code examples. Abstract advice without examples is hard to apply.

## Why

- **Actionable**: Code shows exactly what to do
- **Unambiguous**: Examples eliminate interpretation guesswork
- **Memorable**: Concrete patterns stick better than abstract principles
- **Verifiable**: Readers can compare their code to the example

## Bad: Abstract Advice

```markdown
# Bad: Too vague
"Keep your code organized and maintainable."

"Use appropriate design patterns."

"Structure your files logically."
```

These don't help because they don't show what "organized" or "appropriate" means.

## Good: Concrete Patterns

Show directory structures:

```markdown
# Ruby example
"Place model-specific concerns in `app/models/model_name/`."

\`\`\`
app/models/
├── card.rb
├── card/
│   ├── closeable.rb     # Card::Closeable
│   └── searchable.rb    # Card::Searchable
└── concerns/            # Only shared concerns
    └── mentionable.rb
\`\`\`

# TypeScript example
"Co-locate components with their tests and styles."

\`\`\`
app/components/
├── Button/
│   ├── Button.tsx
│   ├── Button.test.tsx
│   └── index.ts
└── Card/
    ├── Card.tsx
    └── index.ts
\`\`\`
```

The reader knows exactly where to put files.

## Show the Transformation

When showing a pattern, include before and after:

```markdown
# Ruby example
\`\`\`ruby
# Bad: Custom controller action
resources :cards do
  post :close
end

# Good: Resource controller
resources :cards do
  resource :closure, only: [:create, :destroy]
end
\`\`\`

# TypeScript example
\`\`\`typescript
// Bad: Inline conditional classes
<button className={`btn ${isActive ? 'btn-active' : ''} ${isDisabled ? 'btn-disabled' : ''}`}>

// Good: Using cn() utility
<button className={cn("btn", { "btn-active": isActive, "btn-disabled": isDisabled })}>
\`\`\`
```

The contrast makes the improvement obvious.

## Use Real Code

Patterns from real codebases are more convincing:

```markdown
# Ruby example from Fizzy:
\`\`\`ruby
module Card::Closeable
  extend ActiveSupport::Concern

  included do
    has_one :closure, dependent: :destroy
    scope :closed, -> { joins(:closure) }
  end

  def close
    create_closure!(user: Current.user)
  end
end
\`\`\`

# TypeScript example from a React codebase:
\`\`\`typescript
function Button({ className, variant, children }: ButtonProps) {
  return (
    <button
      className={cn(
        "inline-flex items-center rounded-lg font-medium",
        {
          "bg-teal-500 text-white": variant === "primary",
          "bg-neutral-100 text-neutral-900": variant === "secondary",
        },
        className
      )}
    >
      {children}
    </button>
  );
}
\`\`\`
```

Real code shows that the pattern actually works in production.

## Match Example Complexity to Rule Complexity

Simple rules get simple examples:

```markdown
# Ruby: Simple rule, simple example
Use `_later` suffix for async methods.

\`\`\`ruby
def notify
  # sync
end

def notify_later
  NotifyJob.perform_later(self)
end
\`\`\`

# TypeScript: Simple rule, simple example
Use named exports, not default exports.

\`\`\`typescript
// Bad
export default function formatCurrency() {}

// Good
export function formatCurrency() {}
\`\`\`
```

Complex rules may need longer examples with comments:

```markdown
# Ruby: Complex rule, annotated example
\`\`\`ruby
module CurrentAttributesJobExtensions
  def initialize(...)
    super
    @account = Current.account  # Capture at enqueue time
  end

  def serialize
    super.merge("account" => @account&.to_gid)  # Store in job payload
  end

  def perform_now
    Current.with_account(account) { super }  # Set before perform
  end
end
\`\`\`

# TypeScript: Complex rule, annotated example
\`\`\`typescript
function useDebounce<T>(value: T, delay: number): T {
  let [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // Set up timer to update debounced value
    let timer = setTimeout(() => setDebouncedValue(value), delay);

    // Clean up timer on value change or unmount
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
\`\`\`
```

## Rules

1. Every rule must have at least one code example
2. Show bad/good contrast when applicable
3. Use real code from actual codebases when possible
4. Match example complexity to rule complexity
5. Abstract advice alone is not a rule
