---
title: Skill Directory Structure
impact: HIGH
tags: [structure, organization]
---

# Skill Directory Structure

Every skill lives in its own directory with a consistent structure: a main SKILL.md file and a rules/ subdirectory containing individual rule files.

## Why

- **Discoverability**: Consistent structure means agents know where to find things
- **Scalability**: Individual rule files keep content manageable
- **References**: SKILL.md can link to detailed rules with `@rules/rule-name.md`

## Structure

```
skills/
└── topic-best-practices/
    ├── SKILL.md              # Main summary file
    └── rules/
        ├── rule-one.md       # Detailed rule
        ├── rule-two.md       # Another rule
        └── rule-three.md     # And another
```

## Naming Conventions

**Skill directories**: `{topic}-best-practices`

```
ruby-on-rails-best-practices/
frontend-react-best-practices/
frontend-testing-best-practices/
skill-writing-best-practices/
```

**Rule files**: `kebab-case.md`

```
model-scoped-concerns.md
thin-controllers.md
current-in-other-contexts.md
```

## What Goes Where

**SKILL.md** contains:
- Frontmatter with name and description
- Overview and "When to Apply" section
- Condensed summaries of all rules with short examples
- Links to full rule files

**rules/*.md** contain:
- Detailed explanation of one specific rule
- Full code examples with bad/good patterns
- Edge cases and exceptions
- Numbered takeaways

## Rules

1. One skill = one directory with SKILL.md + rules/
2. Directory names use `{topic}-best-practices` pattern
3. Rule files use `kebab-case.md` naming
4. SKILL.md summarizes; rules/ files go deep
