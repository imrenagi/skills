---
title: Log Errors Once at the Outer Layer
impact: HIGH
impactDescription: Prevents duplicate log entries and ensures errors are captured with full context at the boundary of the system.
tags: logging, error-handling, zerolog
---

## Log Errors Once at the Outer Layer

**Impact: HIGH**

In a layered architecture (Controller -> Service -> Repository), a common mistake is logging an error at every level it is encountered. This creates "log noise," making it difficult to trace a single request and obscuring the root cause.

### The "Log Once" Principal

Errors should be **logged exactly once**, ideally at the point where they are handled or at the outermost boundary of the system (e.g., gRPC interceptors or HTTP middleware).

- **Middle layers** (Service/Repository) should **wrap** errors with additional context using `%w` but should **not** log them.
- **Outer layers** (Handlers/Interceptors) should **log** the error chain and return an appropriate status code.

### Guidelines for different layers

#### 1. Repository/Client Layer (Implementation)

Do NOT log errors here. Wrap them to provide context about the operation that failed.

```go
func (r *OrderRepository) FindByID(ctx context.Context, id string) (*Order, error) {
    order, err := r.db.Get(id)
    if err != nil {
        // WRAP, don't log
        return nil, fmt.Errorf("failed to get order from db: %w", err)
    }
    return order, nil
}
```

#### 2. Service Layer (Business Logic)

Do NOT log errors here. Map to domain errors and wrap, or return custom typed errors with metadata.

**Option A: Wrap with Domain Error**

```go
func (s *OrderService) GetOrder(ctx context.Context, id string) (*Order, error) {
    order, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("%w: %v", domain.ErrNotFound, err)
    }
    return order, nil
}
```

**Option B: Return Custom Typed Error**

Use this when you need to pass specific metadata to the caller (refer to [Sentinel Errors vs. Typed Errors](error-types.md)).

```go
func (s *OrderService) GetOrder(ctx context.Context, id string) (*Order, error) {
    order, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, &domain.NotFoundError{
            Resource: "Order",
            ID:       id,
            Err:      err, // preserve for wrapping
        }
    }
    return order, nil
}
```

#### 3. Outermost Layer (Handler/Interceptor)

This is where the logging happens. The handler has access to the full request context and the complete error chain.

```go
func (s *Server) GetOrder(ctx context.Context, req *pb.GetOrderRequest) (*pb.Order, error) {
    order, err := s.orderService.GetOrder(ctx, req.Id)
    if err != nil {
        // LOG EXACTLY ONCE
        log.Ctx(ctx).Error().
            Err(err).
            Str("order_id", req.Id).
            Msg("failed to get order")

        // Convert to gRPC status and return
        return nil, status.Error(codes.NotFound, "order not found")
    }
    return order, nil
}
```

### Best Practices

1.  **Context is King**: Always use `log.Ctx(ctx)` (refer to [Pass Context to Loggers](logging-context.md)) to include request IDs and other trace metadata.
2.  **Avoid Log Duplication**: If you find yourself seeing the same error reported 3 times in your logs for one request, you are likely logging at every layer.
3.  **Wrap for Context**: Only wrap an error if you are adding new, useful information. Don't wrap for the sake of wrapping.
4.  **Sentry/Monitoring**: If using an error tracking tool like Sentry, capture the exception at the same point where you log it (the outer boundary).

### References

- [Wrap Errors with %w](error-wrapping.md)
- [Define and Return Domain Errors](error-domain-error.md)
- [Pass Context to Loggers](logging-context.md)
- [Standardize gRPC Error Handling](server-grpc-error.md)
