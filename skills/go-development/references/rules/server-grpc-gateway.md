---
title: Implement gRPC-Gateway effectively
impact: HIGH
impactDescription: Standardizes the way gRPC and HTTP/REST interfaces are exposed and managed within a single server process.
tags: grpc, gateway, http, server
---

## Implement gRPC-Gateway effectively

**Impact: HIGH**

When building microservices, it is often necessary to provide both a gRPC interface for internal communication and a RESTful JSON API for external clients. gRPC-Gateway provides a way to generate a reverse-proxy server that translates RESTful JSON API calls into gRPC.

### Unified TCP Server Configuration

To keep HTTP and gRPC server settings consistent, we define a shared `TCPServerConfig` struct. It lives in the project configuration (`config.Config`).

```go
type TCPServerConfig struct {
    Host               string        // optional custom host, defaults to "0.0.0.0"
    Port               int           // port number to listen on
    Scheme             string        // "http" or "https"
    ReadHeaderTimeout  time.Duration // max time to read request headers (HTTP only)
    ReadTimeout        time.Duration // max time to read the request body (HTTP only)
    WriteTimeout       time.Duration // max time to write the response (HTTP only)
    IdleTimeout        time.Duration // keep‑alive idle timeout (HTTP only)
    MaxHeaderBytes     int           // maximum size of request headers (HTTP only)
}
```

The main configuration now contains two sections, one for each protocol:

```go
type Config struct {
    HTTPServer TCPServerConfig
    GRPCServer TCPServerConfig
    // … other fields …
}
```

### Server Structure

The `Server` struct holds references to both the HTTP and gRPC servers and the shared configuration.

```go
type Server struct {
    httpServer *http.Server
    grpcServer *grpc.Server
    cfg        *config.Config
}
```

### Server Construction

```go
func NewServer(ctx context.Context, cfg *config.Config) *Server {
    s := &Server{cfg: cfg}
    s.grpcServer = s.newGRPCServer(ctx)
    // HTTP server will be created lazily in runHTTPGateway using cfg.HTTPServer
    return s
}
```

### gRPC Server Initialization

```go
func (s *Server) newGRPCServer(ctx context.Context) *grpc.Server {
    opts := []grpc.ServerOption{
        grpc.StatsHandler(otelgrpc.NewServerHandler()),
        grpc.ChainUnaryInterceptor(
            s.UnaryServerAppLoggerInterceptor(),
            s.UnaryServerGRPCLoggerInterceptor(),
        ),
    }
    grpcServer := grpc.NewServer(opts...)
    // Register your gRPC services here, e.g.:
    // examplepb.RegisterExampleServiceServer(grpcServer, s.service)
    return grpcServer
}
```

### HTTP Gateway Setup (using the shared TCPServerConfig)

```go
func (s *Server) runHTTPGateway(ctx context.Context) error {
    // Build the address from the shared config
    c := s.cfg.HTTPServer
    addr := fmt.Sprintf("%s:%d", c.Host, c.Port)

    // Dial the gRPC server (usually insecure for internal proxying)
    dialOpts := []grpc.DialOption{
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithStatsHandler(otelgrpc.NewClientHandler()),
    }
    conn, err := grpc.NewClient(fmt.Sprintf("%s:%d", s.cfg.GRPCServer.Host, s.cfg.GRPCServer.Port), dialOpts...)
    if err != nil {
        return fmt.Errorf("failed to dial grpc server: %w", err)
    }

    // Configure the gRPC‑Gateway mux
    gwmux := runtime.NewServeMux(
        runtime.WithIncomingHeaderMatcher(CustomHeaderMatcher),
        runtime.WithMetadata(func(ctx context.Context, r *http.Request) metadata.MD {
            md := make(map[string]string)
            if requestID := r.Header.Get("X-Request-ID"); requestID != "" {
                md["x-request-id"] = requestID
            }
            return metadata.New(md)
        }),
    )

    // Register your generated gateway handlers here, e.g.:
    // examplepb.RegisterExampleServiceHandler(ctx, gwmux, conn)

    // Router for extra endpoints (Swagger, metrics, etc.)
    r := mux.NewRouter()
    r.Use(otelmux.Middleware("server-name"))
    // Swagger UI
    swagger := http.StripPrefix("/api/swagger/", http.FileServer(http.Dir("./gen/openapi/")))
    r.PathPrefix("/api/swagger/").Handler(swagger)
    // Fallback to the gRPC‑Gateway mux
    r.PathPrefix("/").Handler(gwmux)

    // Build the HTTP server using the shared timeout configuration
    s.httpServer = &http.Server{
        Addr:              addr,
        Handler:           r,
        ReadHeaderTimeout: c.ReadHeaderTimeout,
        ReadTimeout:       c.ReadTimeout,
        WriteTimeout:      c.WriteTimeout,
        IdleTimeout:       c.IdleTimeout,
        MaxHeaderBytes:    c.MaxHeaderBytes,
    }
    return s.httpServer.ListenAndServe()
}
```

### Running the Server

```go
func (s *Server) Run(ctx context.Context) error {
    var wg sync.WaitGroup
    errChan := make(chan error, 2)

    // gRPC server
    wg.Add(1)
    go func() {
        defer wg.Done()
        lis, err := net.Listen("tcp", fmt.Sprintf("%s:%d", s.cfg.GRPCServer.Host, s.cfg.GRPCServer.Port))
        if err != nil {
            errChan <- err
            return
        }
        if err := s.grpcServer.Serve(lis); err != nil {
            errChan <- err
        }
    }()

    // HTTP gateway
    wg.Add(1)
    go func() {
        defer wg.Done()
        if err := s.runHTTPGateway(ctx); err != nil && err != http.ErrServerClosed {
            errChan <- err
        }
    }()

    select {
    case <-ctx.Done():
        return s.Shutdown(context.Background())
    case err := <-errChan:
        return err
    }
}
```

### Graceful Shutdown

The shutdown sequence mirrors the previous implementation, first closing the HTTP server then gracefully stopping the gRPC server.

```go
func (s *Server) Shutdown(ctx context.Context) error {
    if s.httpServer != nil {
        shutdownCtx, cancel := context.WithTimeout(ctx, 5*time.Second)
        defer cancel()
        s.httpServer.Shutdown(shutdownCtx)
    }
    if s.grpcServer != nil {
        stopped := make(chan struct{})
        go func() { s.grpcServer.GracefulStop(); close(stopped) }()
        t := time.NewTimer(5 * time.Second)
        defer t.Stop()
        select {
        case <-t.C:
            s.grpcServer.Stop()
        case <-stopped:
        }
    }
    return nil
}
```

### Helper Functions

```go
type RegisterFunc func(ctx context.Context, mux *runtime.ServeMux, conn *grpc.ClientConn) error

func mustRegisterGWHandler(ctx context.Context, register RegisterFunc, mux *runtime.ServeMux, conn *grpc.ClientConn) {
    if err := register(ctx, mux, conn); err != nil {
        panic(err)
    }
}
```

### References

- gRPC‑Gateway Documentation – https://github.com/grpc-ecosystem/grpc-gateway
- OpenTelemetry Go – https://github.com/open-telemetry/opentelemetry-go
- Gorilla Mux – https://github.com/gorilla/mux
