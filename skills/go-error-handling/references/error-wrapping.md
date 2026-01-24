---
title: Wrap Errors with %w
impact: HIGH
impactDescription: Enables error chain inspection using errors.Is and errors.As
tags: go, error-handling
---

## Wrap Errors with %w

**Impact: HIGH**

When returning an error from a function, use `fmt.Errorf` with the `%w` verb to wrap the original error. This creates a link in the error chain, allowing developers to use `errors.Is` and `errors.As` to check for specific error types or values further up the call stack.

**Incorrect (loses error type information):**

```go
if err != nil {
    return fmt.Errorf("failed to process: %v", err)
}

// Caller cannot do errors.Is(err, originalErr)
```

**Correct (wraps error for inspection):**

```go
if err != nil {
    return fmt.Errorf("failed to process: %w", err)
}
// Caller can use errors.Is(err, originalErr)
```

Reference: [Go Blog: Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)
