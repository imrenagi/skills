---
title: HTTP Client Logging Middleware
impact: MEDIUM
impactDescription: Standardizes HTTP client request/response logging, including payloads and headers.
tags: go, http, logging, middleware, roundtripper
---

## HTTP Client Logging Middleware

**Impact: MEDIUM**

Standardize HTTP client logging by implementing a custom `http.RoundTripper`. This allows for capturing detailed information about every outgoing request and incoming response, including JSON payloads and custom headers, which is essential for debugging third-party integrations.

**Directory Structure:**

Middleware should be implemented in an appropriate internal package, such as `internal/http/middleware`.

```
.
├── internal
    └── http
        └── middleware
            └── logging.go
```

**Implementation Rules:**

- Implement the `http.RoundTripper` interface to intercept requests and responses.
- **Request Logging**: Log the URL, method, and headers (especially `X-` custom headers).
- **Response Logging**: Log the status code and duration.
- **Payload Logging**: Debug the JSON body of both requests and responses. Use `RawJSON` from `zerolog` to ensure the payload is logged as a JSON object rather than an escaped string. Restore the body using `io.ReadAll` and `io.NopCloser`.
- **Context Awareness**: Ensure logs are linked to the existing request context (e.g., propagating `request_id`).

**Correct (HTTP RoundTripper Implementation):**

```go
package middleware

import (
	"bytes"
	"io"
	"net/http"
	"time"

	"github.com/rs/zerolog/log"
)

type loggingTransport struct {
	CapturedTransport http.RoundTripper
}

func (t *loggingTransport) RoundTrip(req *http.Request) (*http.Response, error) {
	start := time.Now()

	// Log Request
	reqBody, _ := io.ReadAll(req.Body)
	req.Body = io.NopCloser(bytes.NewBuffer(reqBody))

	log.Debug().
		Str("method", req.Method).
		Str("url", req.URL.String()).
		Interface("headers", req.Header).
		RawJSON("body", reqBody).
		Msg("Sending HTTP Request")

	resp, err := t.CapturedTransport.RoundTrip(req)
	if err != nil {
		return nil, err
	}

	// Log Response
	respBody, _ := io.ReadAll(resp.Body)
	resp.Body = io.NopCloser(bytes.NewBuffer(respBody))

	log.Debug().
		Int("status", resp.StatusCode).
		Duration("duration", time.Since(start)).
		Interface("headers", resp.Header).
		RawJSON("body", respBody).
		Msg("Received HTTP Response")

	return resp, nil
}

func NewLoggingTransport(transport http.RoundTripper) http.RoundTripper {
	if transport == nil {
		transport = http.DefaultTransport
	}
	return &loggingTransport{CapturedTransport: transport}
}
```

Reference: [Go Net/HTTP RoundTripper](https://pkg.go.dev/net/http#RoundTripper)
