---
title: Form Data Validation
impact: HIGH
tags: [action, validation, zod, forms]
---

# Form Data Validation

Validate form data with Standard Schema in actions, using a shared `validate` helper.

## Why

- Type-safe validation with automatic parsing
- Consistent result handling across actions
- Consistent error handling pattern
- Catches invalid data before mutations

## Pattern

```ts
import type { StandardSchemaV1 } from "@standard-schema/spec";
import { failure, success } from "~/lib/result";

export async function validate<Schema extends StandardSchemaV1>(
  input: FormData | Request,
  schema: Schema,
): Promise<
  ReturnType<typeof success<StandardSchemaV1.InferOutput<Schema>>> |
    ReturnType<typeof failure<ValidationError>>
> {
  if (input instanceof Request) input = await input.formData();
  let entries = Object.fromEntries(input.entries());
  let result = schema["~standard"].validate(entries);
  if (result instanceof Promise) result = await result;
  if (result.issues) return failure(new ValidationError(result.issues));
  return success(result.value);
}

export class ValidationError extends Error {
  issues: StandardSchemaV1.ValidationIssue[];

  constructor(issues: StandardSchemaV1.ValidationIssue[]) {
    super("Validation Error");
    this.issues = issues;
  }
}
```

```ts
import { data, redirect } from "react-router";
import { isFailure } from "~/lib/result";
import { validate } from "~/lib/validation";
import { currentUser } from "~/lib/authorize.server";

export async function action({ request }: Route.ActionArgs) {
  let user = currentUser();

  let result = await validate(request, schema);
  if (isFailure(result)) {
    return data(result.error.issues, { status: 422 });
  }

  await createRecord({ userId: user.id, ...result.data });
  throw redirect("/success");
}
```

## Common Validators

```ts
const schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  amount: z.coerce.number().positive(),
  quantity: z.coerce.number().int().min(1).max(100),
  page: z.coerce.number().default(1),
  status: z.enum(["draft", "published", "archived"]),
  agreed: z.coerce.boolean(),
  tags: z.array(z.string()).min(1),
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
