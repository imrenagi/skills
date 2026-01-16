---
title: General API Client Initialization
impact: MEDIUM
impactDescription: Standardizes third-party API client implementation and configuration.
tags: go, api, client, initialization, configuration
---

## API Client Initialization

**Impact: MEDIUM**

When implementing a client for a third-party API, standardize the implementation by creating a dedicated package and using a `Config` struct for initialization. This ensures consistency, simplifies configuration management, and facilitates testing through dependency injection and interfaces.

**Directory Structure:**

Each third-party API should have its own package under `internal`.

```
.
├── internal
    └── <api-name>
        └── client.go
        └── config.go
```

**Implementation Rules:**

- Define a `Config` struct in `config.go` with all necessary fields (BaseURL, API Key, Timeouts).
- Define a `Client` struct in `client.go` that wraps the HTTP client.
- Provide a `New(c Config)` function to initialize the client.
- When using an SDK, return the SDK's client object directly from New() function.
- Always include a `Timeout` in the `http.Client`.
- Always instrument HTTP client with OpenTelemetry `go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp`.
- Always instrument gRPC client with OpenTelemetry `go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc`

**Incorrect (Ad-hoc client usage):**

```go
// Bad: Hardcoded values, no configuration struct, and no timeout
func FetchData() {
    resp, _ := http.Get("https://api.example.com/v1/data?api_key=secret")
    // ...
}
```

**Correct (Standardized Client and Config):**

`internal/example/config.go`:

```go
package example

import "time"

type Config struct {
	BaseURL string        `yaml:"baseUrl"`
	APIKey  string        `yaml:"apiKey"`
	Timeout time.Duration `yaml:"timeout"`
}
```

`internal/example/client.go`:

```go
package example

import (
	"net/http"
	"time"
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
)

type Client struct {
	baseURL    string
	apiKey     string
	httpClient *http.Client // or Client from the SDK
}

func New(c Config) *Client {
	if c.Timeout == 0 {
		c.Timeout = 30 * time.Second
	}

	return &Client{
		baseURL: c.BaseURL,
		apiKey:  c.APIKey,
		httpClient: &http.Client{
            Transport: otelhttp.NewTransport(http.DefaultTransport),
			Timeout: c.Timeout,
		},
	}
}

func (c *Client) GetData() error {
    // Implementation using c.httpClient, c.baseURL, and c.apiKey
    return nil
}
```

`internal/firestore/client.go`:

```go
package firebase

import 

func New(c Config) *firestore.Client {
    // initialize client SDK
    // make sure to always enable the otelhttp or otelgrpc instrumentation
}
```

Reference: [Go HTTP Client Best Practices](https://medium.com/@nate_52383/best-practices-for-using-http-clients-in-go-8c88017c677)
`
