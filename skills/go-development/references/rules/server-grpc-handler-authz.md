---
title: Authorization in gRPC Handlers using Principal
impact: HIGH
impactDescription: Standardizes how user and client identity is managed via a Principal object and how authorization checks are performed within gRPC handlers.
tags: grpc, authz, principal, context
---

## Authorization in gRPC Handlers using Principal

**Impact: HIGH**

In a multi-tenant or role-based system, every gRPC handler must be able to identify the caller and determine if they are authorized to perform the requested action. This is achieved using a `Principal` object, which is injected into the request context by an interceptor.

### Defining the Principal

A `Principal` represents the security context of the current request. It should contain information about the user or client and provide helper methods to check permissions.

> **Note**: The following code demonstrates a recommended **pattern**. The specific `User` struct and its fields can be adapted to fit the requirements of your application (e.g., using a `ClientID` for machine‑to‑machine auth).

```go
package auth

import (
    "context"
    "github.com/oklog/ulid/v2"
)

type User struct {
    ID    ulid.ULID
    Roles []string
}

type Principal struct {
    Anonymous bool
    User      *User
}

// IsAdmin checks if the principal has administrative privileges.
func (p Principal) IsAdmin() bool {
    if p.Anonymous || p.User == nil {
        return false
    }
    for _, role := range p.User.Roles {
        if role == "admin" {
            return true
        }
    }
    return false
}

// CanAccessResource is a generic method to check if the user can access a specific entity.
func (p Principal) CanAccessResource(resourceOwnerID ulid.ULID) bool {
    if p.IsAdmin() {
        return true
    }
    return !p.Anonymous && p.User != nil && p.User.ID == resourceOwnerID
}
```

### Context Propagation

To prevent collisions and ensure type safety, use a private context key for storing and retrieving the `Principal`.

```go
type contextKey struct {
    name string
}

var principalCtxKey = &contextKey{"principal"}

// WithPrincipalContext returns a new Context that carries the principal.
func WithPrincipalContext(ctx context.Context, principal Principal) context.Context {
    return context.WithValue(ctx, principalCtxKey, principal)
}

// PrincipalFromContext extracts the principal from the context.
func PrincipalFromContext(ctx context.Context) (Principal, bool) {
    p, ok := ctx.Value(principalCtxKey).(Principal)
    return p, ok
}
```

### Authorization in gRPC Handlers

Inside a gRPC handler, extract the `Principal` and use its methods to validate the request. If authorization fails, return a `codes.PermissionDenied` error.

```go
import (
    "context"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    "github.com/imrenagi/app/auth"
)

func (s *Server) GetSecretResource(ctx context.Context, req *pb.GetResourceRequest) (*pb.Resource, error) {
    // 1. Extract the principal
    p, ok := auth.PrincipalFromContext(ctx)
    if !ok || p.Anonymous {
        return nil, status.Error(codes.Unauthenticated, "authentication required")
    }

    // 2. Load the resource (e.g., from a database)
    resource, err := s.store.GetResource(ctx, req.Id)
    if err != nil {
        return nil, status.Error(codes.Internal, "failed to load resource")
    }

    // 3. Perform authorization check
    if !p.CanAccessResource(resource.OwnerID) {
        return nil, status.Error(codes.PermissionDenied, "you do not have permission to access this resource")
    }

    return resource, nil
}
```

### Best Practices

1.  **Fail-Close**: Always assume a request is forbidden unless it explicitly meets the authorization criteria.
2.  **Private Context Keys**: Never use raw strings as context keys to avoids collisions with other packages.
3.  **Encapsulate Logic**: Keep authorization logic (like role checks) within the `Principal` struct to keep handlers clean and testable.
4.  **Audit Logs**: Log authorization failures for security monitoring.
5.  **Return Specific Errors**: Use `codes.PermissionDenied` for authorization failures and `codes.Unauthenticated` for missing or invalid credentials.

### References

- [gRPC Error Codes](https://grpc.io/docs/guides/error/)
- [Effective Go: Context](https://golang.org/doc/effective_go#contexts)
- [gRPC Interceptor Auth Rule](server-grpc-interceptor.md)
