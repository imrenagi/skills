---
title: gRPC Client Initialization
impact: MEDIUM
impactDescription: Standardizes gRPC client initialization with OpenTelemetry and logging.
tags: go, grpc, initialization, otel, logging
---

## gRPC Client Initialization

**Impact: MEDIUM**

When initializing gRPC clients, especially for cloud providers like Google Cloud, always include OpenTelemetry instrumentation and standardized logging interceptors. This ensure consistency in observability across all gRPC-based services and clients.

**Implementation Rules:**

- Always use `go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc` for instrumentation.
- Use `otelgrpc.NewClientHandler()` for stats handling in `grpc.DialOption`.
- Include logging interceptors for both Unary and Stream calls to monitor request/response flows.
- If using Google cloud SDKs, pass these options via `option.WithGRPCDialOption`.

**Correct (gRPC Client with OTEL and Logging):**

```go
package speech

import (
	"context"

	stt "cloud.google.com/go/speech/apiv1"
	grpcutil "github.com/imrenagi/app/internal/grpc"
	"github.com/rs/zerolog/log"
	"go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc"
	"google.golang.org/api/option"
	"google.golang.org/grpc"
)

func NewGoogleSpeechToTextClient(ctx context.Context) *stt.Client {
    // Standard grpc dial options for all clients
	dialOptions := []grpc.DialOption{
		grpc.WithStatsHandler(otelgrpc.NewClientHandler()),
		grpc.WithChainUnaryInterceptor(
			grpcutil.UnaryClientGRPCLoggerInterceptor(),
			grpcutil.UnaryClientAppLoggerInterceptor(),
		),
		grpc.WithChainStreamInterceptor(
			grpcutil.StreamClientAppLoggerInterceptor(),
			grpcutil.StreamClientGRPCLoggerInterceptor(),
		),
	}

    // Convert dial options to Google API options
	var opts []option.ClientOption
	for _, dopts := range dialOptions {
		opts = append(opts, option.WithGRPCDialOption(dopts))
	}

	client, err := stt.NewClient(ctx, opts...)
	if err != nil {
		log.Fatal().Err(err).Msg("failed to create google speech to text client")
	}
	return client
}
```

Reference: [OpenTelemetry gRPC Instrumentation](https://github.com/open-telemetry/opentelemetry-go-contrib/tree/main/instrumentation/google.golang.org/grpc/otelgrpc)
