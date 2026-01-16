---
title: Prevent Slowloris Attacks on HTTP Server
impact: HIGH
impactDescription: Protects the application from connection-exhaustion attacks by configuring appropriate timeouts.
tags: security, http, networking
---

## Prevent Slowloris Attacks on HTTP Server

**Impact: HIGH**

A Slowloris attack works by opening many connections to the server and keeping them open as long as possible by sending partial requests. This can exhaust the server's connection pool, making it unavailable to legitimate users.

To mitigate this, always configure specific timeouts on your `http.Server`.

### Configure Timeouts using `TCPServerConfig`

Use the unified `TCPServerConfig` to manage these settings consistently.

```go
type TCPServerConfig struct {
    Host              string        `yaml:"host"`
    Port              int           `yaml:"port"`
    ReadHeaderTimeout time.Duration `yaml:"read_header_timeout"`
    ReadTimeout       time.Duration `yaml:"read_timeout"`
    WriteTimeout      time.Duration `yaml:"write_timeout"`
    IdleTimeout       time.Duration `yaml:"idle_timeout"`
    MaxHeaderBytes    int           `yaml:"max_header_bytes"`
}
```

### Apply to the Server Instance

```go
func NewSecureHTTPServer(cfg config.TCPServerConfig, handler http.Handler) *http.Server {
    return &http.Server{
        Addr:              fmt.Sprintf("%s:%d", cfg.Host, cfg.Port),
        Handler:           handler,
        // Crucial timeouts to prevent connection exhaustion
        ReadHeaderTimeout: cfg.ReadHeaderTimeout,
        ReadTimeout:       cfg.ReadTimeout,
        WriteTimeout:      cfg.WriteTimeout,
        IdleTimeout:       cfg.IdleTimeout,
        MaxHeaderBytes:    cfg.MaxHeaderBytes,
    }
}
```

### Best Practices

1.  **ReadHeaderTimeout**: Set this to a small value (e.g., `2s` to `5s`). This is the most effective defense against Slowloris.
2.  **ReadTimeout**: Covers the time from when the connection is accepted until the request body is fully read.
3.  **WriteTimeout**: Covers the time from the end of the request header read until the end of the response write.
4.  **IdleTimeout**: The maximum amount of time to wait for the next request when keep-alives are enabled.
5.  **MaxHeaderBytes**: Limit the maximum size of request headers to prevent memory exhaustion. Default is `1MB` if not set, but consider setting it explicitly (e.g., `1 << 20`).

### References

- [Go net/http Server Documentation](https://pkg.go.dev/net/http#Server)
- [Cloudflare: What is a Slowloris attack?](https://www.cloudflare.com/learning/ddos/ddos-attack-methods/slowloris/)
- [gRPC-Gateway Implementation Rule](server-grpc-gateway.md)
