---
title: Implement gRPC Interceptors for AuthN and AuthZ
impact: HIGH
impactDescription: Standardizes how security and cross-cutting concerns are applied to gRPC methods, including selective application.
tags: grpc, interceptor, auth, security
---

## Implement gRPC Interceptors for AuthN and AuthZ

**Impact: HIGH**

In gRPC services, cross-cutting concerns like logging, authentication (AuthN), and authorization (AuthZ) should be handled via interceptors. This ensures consistency and keeps business logic clean.

### Selective Interceptor Application

Not all interceptors should apply to every method (e.g., login or public methods shouldn't require AuthN). Use `grpc-ecosystem/go-grpc-middleware/v2/interceptors/selector` to selectively apply interceptors based on method names or prefixes.

#### Define a Match Function

```go
func (s *Server) isAdminOnly(ctx context.Context, c interceptors.CallMeta) bool {
    // Return true if the interceptor should be applied to this method
    return strings.HasPrefix(c.FullMethod(), telegramv1.WebhookService_HandleWebhook_FullMethodName)
}
```

#### Apply with Selector

```go
grpcSelector.UnaryServerInterceptor(
    grpcAuth.UnaryServerInterceptor(auth.AllowAdminOnly()),
    grpcSelector.MatchFunc(s.isAdminOnly),
)
```

### Authentication (AuthN) Interceptors

Authentication interceptors verify the identity of the caller. They should usually be applied globally or to a broad set of methods.

**Example:**

```go
func (s *Server) AuthenticateGRPCInterceptor() grpc.UnaryServerInterceptor {
    return func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
        // Extract credentials from metadata and verify
        // newCtx := ContextWithPrincipal(ctx, principal)
        return handler(ctx, req)
    }
}
```

### Authorization (AuthZ) Interceptors

Authorization interceptors check if the authenticated principal has the required permissions. These are often applied selectively.

**Example:**

```go
func AllowAdminOnly() grpcAuth.AuthFunc {
    return func(ctx context.Context) (context.Context, error) {
        p := PrincipalFromContext(ctx)
        if p == nil || !p.IsAdmin() {
            return nil, status.Error(codes.PermissionDenied, "admin role required")
        }
        return ctx, nil
    }
}
```

### Chaining Interceptors

Order matters when chaining interceptors. Standard logging and tracing should come first, followed by AuthN, then AuthZ.

```go
func (s *Server) newGRPCServer(ctx context.Context) *grpc.Server {
    opts := []grpc.ServerOption{
        grpc.StatsHandler(otelgrpc.NewServerHandler()),
        grpc.ChainUnaryInterceptor(
            // 1. Observability
            grpcutil.UnaryServerAppLoggerInterceptor(),
            grpcutil.UnaryServerGRPCLoggerInterceptor(),

            // 2. Authentication (Global or selective)
            s.AuthenticateGRPCInterceptor(),

            // 3. Authorization (Selective)
            grpcSelector.UnaryServerInterceptor(
                grpcAuth.UnaryServerInterceptor(auth.AllowAdminOnly()),
                grpcSelector.MatchFunc(s.isAdminOnly),
            ),
        ),
    }
    return grpc.NewServer(opts...)
}
```

### Best Practices

1.  **Fail Fast**: Return errors immediately in the interceptor if checks fail.
2.  **Order of Execution**: Ensure AuthN runs _before_ AuthZ so that identity is established before checking permissions.
3.  **Use `CallMeta`**: Use `interceptors.CallMeta` in `MatchFunc` to get details about the service and method being called.
4.  **Logging**: Log authentication failures at a relevant level (e.g., `Warn` or `Info`) to aid in debugging and security monitoring.
5.  **Context Propagation**: Use interceptors to inject information (like a `Principal`) into the `context.Context` so it's available to downstream handlers.

### References

- [gRPC Middleware V2](https://github.com/grpc-ecosystem/go-grpc-middleware)
- [Auth Interceptor Documentation](https://github.com/grpc-ecosystem/go-grpc-middleware/tree/v2/interceptors/auth)
- [Selector Interceptor Documentation](https://github.com/grpc-ecosystem/go-grpc-middleware/tree/v2/interceptors/selector)
