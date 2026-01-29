---
title: Use a Result Type for Error Flow
impact: MEDIUM
tags: [errors, types]
---

# Use a Result Type for Error Flow

Represent success/failure explicitly instead of throwing in normal control flow.

## Pattern

```ts
export interface Success<T> {
  status: "success";
  data: T;
}

export interface Failure<E extends Error> {
  status: "failure";
  error: E;
}

export type Result<T, E extends Error> = Success<T> | Failure<E>;

export function success<T>(data: T): Success<T> {
  return { status: "success", data };
}

export function failure<E extends Error>(error: E): Failure<E> {
  return { status: "failure", error };
}

export function isSuccess<T, E extends Error>(
  result: Result<T, E>,
): result is Success<T> {
  return result.status === "success";
}

export function isFailure<T, E extends Error>(
  result: Result<T, E>,
): result is Failure<E> {
  return result.status === "failure";
}

export function succeeded<T, E extends Error>(
  result: Result<T, E>,
  message = "Result is a failure",
): asserts result is Success<T> {
  if (isFailure(result)) throw new Error(message, { cause: result.error });
}

export function failed<T, E extends Error>(
  result: Result<T, E>,
  message = "Result is a success",
): asserts result is Failure<E> {
  if (isSuccess(result)) throw new Error(message, { cause: result.data });
}
```

## Rules

1. Use `Result` for expected failure cases
2. Prefer `isFailure`/`isSuccess` checks in normal flow
3. Use `succeeded`/`failed` as assertions at boundaries
