---
title: Define and Return Domain Errors
impact: HIGH
impactDescription: Decouples business logic from implementation details (e.g., DB drivers) and provides clear, actionable errors for caller.
tags: go, domain-driven-design, error-handling
---

## Define and Return Domain Errors

**Impact: HIGH**

Internal services and business logic should define and return **domain-specific errors** instead of leaking implementation details (like SQL errors, network failures, or Redis nil errors) to the caller. This allows the calling code to make decisions based on business logic rather than technology-specific failure modes.

### Define Domain Errors in the Service/Domain Package

Define sentinel errors (see [Sentinel Errors vs. Typed Errors](error-types.md)) that represent common business failure cases.

```go
package domain

import "errors"

var (
    ErrResourceNotFound = errors.New("resource not found")
    ErrDuplicateEntry   = errors.New("resource already exists")
    ErrValidationFailed = errors.New("invalid data provided")
)
```

### Map Implementation Details to Domain Errors

When an error occurs in a repository or external client (e.g., a database query fails), the implementation should map that error to one of the defined domain errors. Use `%w` to wrap the original error to preserve the stack for logging (see [Wrap Errors with %w](error-wrapping.md)).

**Correct (Mapping and Wrapping):**

```go
func (r *UserRepository) Find(ctx context.Context, id string) (*User, error) {
    user, err := r.db.QueryContext(ctx, "SELECT ... WHERE id = ?", id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            // Map the DB-specific error to a Domain error
            return nil, fmt.Errorf("%w: user %s", domain.ErrResourceNotFound, id)
        }
        // Return a generic internal error or wrap unexpected errors
        return nil, fmt.Errorf("failed to query user: %w", err)
    }
    return user, nil
}
```

### Handle Domain Errors in the Handler

The outermost layer (gRPC handler, HTTP handler, or CLI command) should interpret these domain errors and convert them to protocol-specific status codes (see [Standardize gRPC Error Handling](server-grpc-error.md)).

```go
func (s *Server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    user, err := s.userService.Find(ctx, req.Id)
    if err != nil {
        if errors.Is(err, domain.ErrResourceNotFound) {
            return nil, status.Error(codes.NotFound, err.Error())
        }
        return nil, status.Error(codes.Internal, "internal server error")
    }
    return user, nil
}
```

### Best Practices

1.  **Don't Leak Implementations**: Never return `sql.ErrNoRows`, `mongo.ErrNoDocuments`, or raw HTTP status errors from your core business logic.
2.  **Explicit Domain Logic**: If a business rule is violated (e.g., "insufficient funds"), create a typed error or sentinel error specifically for that case.
3.  **Preserve Context**: Always wrap the underlying error with `%w` so that DevOps and SREs can still see the root cause (e.g., database connection timeout) in the logs, even if the API client only sees "Internal Server Error".
4.  **Consistency**: Use the same set of domain errors across different repository implementations to ensure the service layer remains implementation-agnostic.

### References

- [Sentinel Errors vs. Typed Errors](error-types.md)
- [Wrap Errors with %w](error-wrapping.md)
- [Standardize gRPC Error Handling](server-grpc-error.md)
- [Domain-Driven Design Error Handling Pattern](https://martinfowler.com/articles/replace-throw-with-notification.html)
