---
title: Standardize gRPC Error Handling
impact: HIGH
impactDescription: Ensures consistent error reporting across all gRPC services, enabling better client-side error handling and debugging.
tags: grpc, error, status, errdetails
---

## Standardize gRPC Error Handling

**Impact: HIGH**

In gRPC, errors should not be returned as raw Go errors. Instead, they must be converted into a `google.golang.org/grpc/status` object which includes a machine-readable status code and optionally, machine-readable error details.

### Mapping Internal Errors to gRPC Status

Internal functions should return domain‑specific errors. The gRPC handler is responsible for mapping these to the appropriate gRPC status code.

```go
import (
    "errors"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
    "github.com/imrenagi/app/internal/domain"
)

func (s *Server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    user, err := s.userService.FindByID(ctx, req.Id)
    if err != nil {
        if errors.Is(err, domain.ErrNotFound) {
            return nil, status.Error(codes.NotFound, "user not found")
        }
        if errors.Is(err, domain.ErrInvalidArgument) {
            return nil, status.Error(codes.InvalidArgument, err.Error())
        }
        // Fallback for unexpected errors
        return nil, status.Error(codes.Internal, "internal server error")
    }
    return user, nil
}
```

### Adding Rich Error Details

For complex errors (e.g., validation failures), use `status.WithDetails` to attach structured metadata from `google.golang.org/genproto/googleapis/rpc/errdetails`. This allows clients to provide better feedback (e.g., highlighting specific invalid fields).

```go
import (
    "google.golang.org/genproto/googleapis/rpc/errdetails"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

func (s *Server) RegisterUser(ctx context.Context, req *pb.RegisterRequest) (*pb.RegisterResponse, error) {
    // ... validation logic ...

    st := status.New(codes.InvalidArgument, "invalid request")
    v := &errdetails.BadRequest{
        FieldViolations: []*errdetails.BadRequest_FieldViolation{
            {
                Field:       "email",
                Description: "invalid email format",
            },
        },
    }

    st, err := st.WithDetails(v)
    if err != nil {
        // If we can't add details, return the status without them
        return nil, st.Err()
    }

    return nil, st.Err()
}
```

### Best Practices

1.  **Never leak internal details**: Avoid returning raw DB errors or stack traces in the error message. Use `codes.Internal` with a generic message for unexpected errors.
2.  **Use standard error details**: Prefer standard types from `google.golang.org/genproto/googleapis/rpc/errdetails` (e.g., `BadRequest`, `ResourceInfo`, `QuotaFailure`) instead of custom ones.
3.  **Client-safe messages**: The error message string in `status.New` should be safe for display to an end-user or at least safe to be logged by an external client.
4.  **Logging**: Log the original internal error at the handler level before converting it to a gRPC status.
5.  **Idempotency & Retries**: Use `errdetails.RetryInfo` for transient errors where the client should retry later.

### References

- [gRPC Status Codes](https://grpc.io/docs/guides/error/)
- [gRPC Error Handling Guide](https://github.com/grpc/grpc-go/blob/master/Documentation/grpc-errors.md)
- [Google AIP: Errors](https://google.aip.dev/193)
