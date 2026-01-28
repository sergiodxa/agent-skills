---
title: Form Data Validation
impact: HIGH
tags: [action, validation, zod, forms]
---

# Form Data Validation

Validate form data with zod schemas in actions, using i18n for error messages.

## Why

- Type-safe validation with automatic parsing
- User-friendly localized error messages
- Consistent error handling pattern
- Catches invalid data before mutations

## Pattern

```tsx
import { data } from "react-router";
import { z } from "zod";
import { i18n } from "~/lib/i18n.server";

export async function action({ request }: Route.ActionArgs) {
  let client = await authenticate(request);
  await authorize(client, { accountStatus: "active" });

  let t = await i18n.getFixedT(request);
  let formData = await request.formData();

  try {
    const { amount, description } = z
      .object({
        amount: z.coerce.number().min(10, t("Amount must be at least $10")),
        description: z
          .string()
          .min(1, t("Description is required"))
          .max(500, t("Description must be under 500 characters")),
      })
      .parse({
        amount: formData.get("amount"),
        description: formData.get("description"),
      });

    await createRecord(client, { amount, description });
    throw redirect("/success");
  } catch (error) {
    if (error instanceof z.ZodError) {
      return data(
        { errors: error.issues.map(({ message }) => message) },
        { status: 400 },
      );
    }
    throw error;
  }
}
```

## Common Validators

```tsx
const schema = z.object({
  // String fields
  name: z.string().min(1, t("Name is required")),
  email: z.string().email(t("Invalid email address")),

  // Numbers (from form data, need coerce)
  amount: z.coerce.number().positive(t("Amount must be positive")),
  quantity: z.coerce.number().int().min(1).max(100),

  // Optional with default
  page: z.coerce.number().default(1),

  // Enums
  status: z.enum(["draft", "published", "archived"]),

  // Boolean (checkboxes)
  agreed: z.coerce.boolean().refine((v) => v, t("You must agree")),

  // Arrays (multiple select/checkboxes)
  tags: z.array(z.string()).min(1, t("Select at least one tag")),

  // Nullable
  notes: z.string().nullable(),
});
```

## Displaying Errors in Component

```tsx
export default function Component() {
  let fetcher = useFetcher<typeof action>();

  return (
    <fetcher.Form method="post">
      {/* Form fields */}

      {fetcher.data?.errors && (
        <ul className="text-failure-700 text-sm">
          {fetcher.data.errors.map((error, i) => (
            <li key={i}>{error}</li>
          ))}
        </ul>
      )}

      <Button type="submit">Submit</Button>
    </fetcher.Form>
  );
}
```
