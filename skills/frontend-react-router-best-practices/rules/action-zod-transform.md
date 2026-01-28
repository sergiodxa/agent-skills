---
title: Use Zod Transform for Input Sanitization
impact: MEDIUM
tags: [action, validation, zod, transform]
---

# Use Zod Transform for Input Sanitization

Use Zod's `.transform()` to sanitize and normalize form input during validation.

## Why

- Sanitization happens at the same place as validation
- Input is normalized before it reaches business logic
- Type-safe: transformed type is reflected in TypeScript
- No separate sanitization step needed

## Common Transforms

### Trim Whitespace

```typescript
const schema = z.object({
  name: z.string().trim(),
  email: z.string().email().trim().toLowerCase(),
});

// Input: { name: "  John  ", email: "  JOHN@Example.COM  " }
// Output: { name: "John", email: "john@example.com" }
```

### Convert Empty Strings to Undefined

```typescript
const schema = z.object({
  nickname: z
    .string()
    .transform((val) => (val === "" ? undefined : val))
    .pipe(z.string().optional()),
});

// Input: { nickname: "" }
// Output: { nickname: undefined }
```

### Parse Numbers from Strings

```typescript
const schema = z.object({
  amount: z
    .string()
    .transform((val) => parseFloat(val.replace(/[,$]/g, "")))
    .pipe(z.number().positive()),
});

// Input: { amount: "$1,234.56" }
// Output: { amount: 1234.56 }
```

### Normalize Phone Numbers

```typescript
const schema = z.object({
  phone: z
    .string()
    .transform((val) => val.replace(/\D/g, ""))
    .pipe(z.string().length(10)),
});

// Input: { phone: "(555) 123-4567" }
// Output: { phone: "5551234567" }
```

### Convert Checkbox to Boolean

```typescript
const schema = z.object({
  subscribe: z
    .string()
    .optional()
    .transform((val) => val === "on"),
});

// Input: { subscribe: "on" } or {}
// Output: { subscribe: true } or { subscribe: false }
```

## Full Example: Action with Transforms

```typescript
// schemas.server.ts
import { z } from "zod";
import type { TFunction } from "i18next";

export function orderSchema(t: TFunction) {
  return z.object({
    amount: z
      .string()
      .transform((val) => parseFloat(val.replace(/[,$]/g, "")))
      .pipe(
        z
          .number({ message: t("Amount is required") })
          .positive({ message: t("Amount must be positive") })
          .max(1000000, { message: t("Amount exceeds maximum") }),
      ),

    itemId: z
      .string()
      .trim()
      .pipe(z.string().uuid({ message: t("Invalid item") })),

    note: z
      .string()
      .trim()
      .transform((val) => (val === "" ? undefined : val))
      .pipe(z.string().max(500).optional()),

    isPrivate: z
      .string()
      .optional()
      .transform((val) => val === "on"),

    email: z
      .string()
      .trim()
      .toLowerCase()
      .pipe(z.string().email({ message: t("Invalid email") })),
  });
}

// route.tsx
export async function action({ request }: Route.ActionArgs) {
  let t = await i18next.getFixedT(request);
  let formData = await request.formData();

  let result = orderSchema(t).safeParse(Object.fromEntries(formData));

  if (!result.success) {
    return data(
      { errors: result.error.flatten().fieldErrors },
      { status: 400 },
    );
  }

  // result.data is fully sanitized and typed
  const { amount, itemId, note, isPrivate, email } = result.data;

  await createOrder({ amount, itemId, note, isPrivate, email });
  return redirect("/orders");
}
```

## Chaining Transforms

Use `.pipe()` to validate after transforming:

```typescript
const schema = z.object({
  // Transform first, then validate the transformed value
  age: z
    .string()
    .transform((val) => parseInt(val, 10))
    .pipe(z.number().int().min(18).max(120)),
});
```

## Default Values with Transform

```typescript
const schema = z.object({
  page: z
    .string()
    .optional()
    .transform((val) => (val ? parseInt(val, 10) : 1))
    .pipe(z.number().int().positive()),

  sort: z
    .string()
    .optional()
    .transform((val) => val || "date")
    .pipe(z.enum(["date", "amount", "name"])),
});
```

## Rules

1. Use `.trim()` on all string inputs
2. Use `.toLowerCase()` on emails and case-insensitive fields
3. Use `.transform()` to convert form strings to proper types (numbers, booleans)
4. Use `.pipe()` to validate after transforming
5. Handle empty strings explicitly (transform to undefined or use .optional())
6. Put transforms in schema factory functions for i18n support
