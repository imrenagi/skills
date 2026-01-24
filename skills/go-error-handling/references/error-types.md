---
title: Sentinel Errors vs. Typed Errors
impact: MEDIUM
impactDescription: Clarifies when to use simple error variables versus custom error structs for better maintainability and API clarity.
tags: go, error-handling
---

## Sentinel Errors vs. Typed Errors

**Impact: MEDIUM**

Choosing between a sentinel error (a package-level variable) and a typed error (a custom struct) depends on whether the caller needs to inspect additional metadata associated with the error.

### Sentinel Errors

Use **sentinel errors** when the error is a fixed condition that doesn't require dynamic data. These are typically defined at the package level using `errors.New` or `fmt.Errorf`.

**Correct (Simple Signal):**

```go
var ErrUserNotFound = errors.New("user not found")

func GetUser(id string) (*User, error) {
    if !exists(id) {
        return nil, ErrUserNotFound
    }
    // ...
}
```

### Typed Errors

Use **typed errors** when the error needs to carry extra context or metadata that the caller might need to handle the error programmatically (e.g., specific missing permissions, resource types, or retry durations).

**Incorrect (Using Simple Error for Detailed Data):**

```go
// Bad: Caller has to parse the string to find which permission or resource failed
return fmt.Errorf("missing %s permission for %s", permission, resource)
```

**Correct (Metadata/Context Required):**

```go
type AuthorizationError struct {
    Resource   string
    Permission string
}

func (e *AuthorizationError) Error() string {
    return fmt.Sprintf("missing permission %s for resource %s", e.Permission, e.Resource)
}

func CheckAccess(userID string, resource string) error {
    if !hasPermission(userID, resource, "read") {
        return &AuthorizationError{Resource: resource, Permission: "read"}
    }
    // ...
}
```

### GRPC Status Code

When using typed errors, it is a good practice to implement the `GRPCStatus` method to return the appropriate gRPC status code.

```go
import (
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

func (e AuthorizationError) GRPCStatus() *status.Status {
	return status.New(codes.PermissionDenied, e.Error())
}
```

Reference: [Go Blog: Working with Errors in Go 1.13](https://go.dev/blog/go1.13-errors)
