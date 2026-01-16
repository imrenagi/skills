---
title: Forward Custom HTTP Headers to gRPC
impact: MEDIUM
impactDescription: Ensures security and application-specific headers are correctly translated into gRPC metadata.
tags: grpc, gateway, header, metadata
---

## Forward Custom HTTP Headers to gRPC

**Impact: MEDIUM**

By default, the gRPC-Gateway permanent header matcher only forwards standard HTTP headers to gRPC metadata. To forward custom headers (e.g., webhook signatures, API keys, or custom request IDs), you must implement a custom header matcher and pass it to the gRPC-Gateway mux using `runtime.WithIncomingHeaderMatcher`.

### Implementation

Define a matcher function that selectively returns the keys you want to keep. It's recommended to fall back to `runtime.DefaultHeaderMatcher` to ensure standard headers (like `Authorization` or `Content-Type`) are still handled correctly.

```go
import (
	"strings"
	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
)

// CustomHeaderMatcher allows specific headers to be passed to gRPC metadata.
func CustomHeaderMatcher(key string) (string, bool) {
	switch strings.ToLower(key) {
	case "x-hub-signature-256",            // common webhook signature
		"x-signature-ed25519",             // security signature
		"x-signature-timestamp",           // security timestamp
		"x-custom-api-key":                // custom application header
		return key, true
	default:
		// Fallback to default gRPC header matching logic
		return runtime.DefaultHeaderMatcher(key)
	}
}
```

### Usage

Inject the matcher when initializing the gRPC-Gateway Mux.

```go
gwmux := runtime.NewServeMux(
    runtime.WithIncomingHeaderMatcher(CustomHeaderMatcher),
)
```

### Best Practices

1.  **Lower-case comparison**: The `key` passed to the matcher is typically normalized by the HTTP server, so always use `strings.ToLower(key)` for comparisons.
2.  **Explicit allow-listing**: Only forward headers that are actually needed by your gRPC services to minimize context size and prevent accidental leakage of internal-only HTTP headers.
3.  **Namespace consistency**: Note that gRPC-Gateway prefixes most headers with `grpcgateway-` unless they are explicitly allowed by a custom matcher or defined in the standard set. Using a custom matcher allows you to retain the original header name in the gRPC metadata.

Reference: [gRPC-Gateway Customizing Headers](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_your_gateway/#customizing-the-header-matching)
