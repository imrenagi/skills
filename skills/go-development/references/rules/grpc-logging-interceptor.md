---
title: gRPC Logging Interceptors
impact: MEDIUM
impactDescription: Standardizes gRPC logging and request tracking with request_id.
tags: go, grpc, logging, interceptor, observability
---

## gRPC Logging Interceptors

**Impact: MEDIUM**

Standardize gRPC logging by using `github.com/grpc-ecosystem/go-grpc-middleware/v2/interceptors/logging` and custom `AppLogger` interceptors to inject `request_id` into the context. This ensures consistency in logs and enables request tracing through unique identifiers.

**Directory Structure:**

Interceptors should be implemented in the `internal/grpc` package.

```
.
├── internal
    └── grpc
        └── interceptor.go
```

**Implementation Rules:**

- Use `github.com/grpc-ecosystem/go-grpc-middleware/v2/interceptors/logging` for standard gRPC lifecycle logging.
- Define a `Logger()` function that adapts `zerolog` to the middleware's expectation.
- Implement both **GRPCLogger** (library logs) and **AppLogger** (custom context/request_id management) interceptors.
- **Server Interceptors**: Must generate a new `request_id` (e.g., using UUID) and inject a logger with that ID into the context.
- **Client Interceptors**: Must propagate the logger from the context to the gRPC call.
- For **Unary** calls, log `StartCall`, `PayloadReceived`, `PayloadSent`, and `FinishCall`.
- For **Stream** calls, log `StartCall` and `FinishCall`.

**Correct (Interceptors implementation):**

```go
package grpc

import (
	"context"
	"fmt"

	"github.com/google/uuid"
	"github.com/grpc-ecosystem/go-grpc-middleware/v2/interceptors/logging"
	"github.com/rs/zerolog/log"
	"google.golang.org/grpc"
)

func Logger() logging.Logger {
	return logging.LoggerFunc(func(ctx context.Context, lvl logging.Level, msg string, fields ...any) {
		l := log.Ctx(ctx).With().Fields(fields).Logger()
		switch lvl {
		case logging.LevelDebug:
			l.Debug().Msg(msg)
		case logging.LevelInfo:
			l.Info().Msg(msg)
		case logging.LevelWarn:
			l.Warn().Msg(msg)
		case logging.LevelError:
			l.Error().Msg(msg)
		default:
			panic(fmt.Sprintf("unknown level %v", lvl))
		}
	})
}

var loggingOpts = []logging.Option{
	logging.WithLogOnEvents(
		logging.StartCall,
		logging.PayloadReceived,
		logging.PayloadSent,
		logging.FinishCall,
	),
}

var streamLoggingOpts = []logging.Option{
	logging.WithLogOnEvents(
		logging.StartCall,
		logging.FinishCall,
	),
}

func StreamServerGRPCLoggerInterceptor(opts ...logging.Option) grpc.StreamServerInterceptor {
	options := loggingOpts
	if len(opts) > 0 {
		options = opts
	}
	return logging.StreamServerInterceptor(Logger(), options...)
}

func UnaryServerGRPCLoggerInterceptor(opts ...logging.Option) grpc.UnaryServerInterceptor {
	options := loggingOpts
	if len(opts) > 0 {
		options = opts
	}
	return logging.UnaryServerInterceptor(Logger(), options...)
}

func UnaryClientGRPCLoggerInterceptor(opts ...logging.Option) grpc.UnaryClientInterceptor {
	options := loggingOpts
	if len(opts) > 0 {
		options = opts
	}
	return logging.UnaryClientInterceptor(Logger(), options...)
}

func StreamClientGRPCLoggerInterceptor(opts ...logging.Option) grpc.StreamClientInterceptor {
	options := streamLoggingOpts
	if len(opts) > 0 {
		options = opts
	}
	return logging.StreamClientInterceptor(Logger(), options...)
}

func UnaryClientAppLoggerInterceptor() grpc.UnaryClientInterceptor {
	return func(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
		log := log.Ctx(ctx)
		return invoker(log.WithContext(ctx), method, req, reply, cc, opts...)
	}
}

func StreamClientAppLoggerInterceptor() grpc.StreamClientInterceptor {
	return func(ctx context.Context, desc *grpc.StreamDesc, cc *grpc.ClientConn, method string, streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
		log := log.Ctx(ctx)
		return streamer(log.WithContext(ctx), desc, cc, method, opts...)
	}
}

func UnaryServerAppLoggerInterceptor() grpc.UnaryServerInterceptor {
	return func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
		log := log.With().Str("request_id", uuid.New().String()).Logger()
		return handler(log.WithContext(ctx), req)
	}
}

func StreamServerAppLoggerInterceptor() grpc.StreamServerInterceptor {
	return func(srv interface{}, ss grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
		err := handler(srv, newWrappedStream(ss))
		if err != nil {
			log.Error().Err(err).Msgf("Error: %v", err)
			return err
		}
		return nil
	}
}

type wrappedStream struct {
	grpc.ServerStream
}

func (w *wrappedStream) Context() context.Context {
	log := log.With().Str("request_id", uuid.New().String()).
		Logger()
	return log.WithContext(context.Background())
}

func newWrappedStream(s grpc.ServerStream) grpc.ServerStream {
	return &wrappedStream{s}
}
```

Reference: [gRPC Middleware Logging](https://github.com/grpc-ecosystem/go-grpc-middleware/tree/main/interceptors/logging)
