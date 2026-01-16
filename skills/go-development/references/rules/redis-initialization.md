---
title: Initialize Redis Client
impact: MEDIUM
impactDescription: Standardizes Redis client initialization with built-in observability.
tags: go, redis, initialization
---

## Initialize Redis Client

**Impact: MEDIUM**

Use `go-redis/redis/v9` to initialize redis client. This allows for a consistent interface for interacting with redis and provides a way to mock the client for testing.

**Directory Structure:**

By default, always implement standalone client implementation until sentinel implementation is required.

```
├── internal
    └── redis
        └── standalone
            └── client.go
        └── sentinel
            └── client.go
        └── config.go
```

**Correct (Client initialization):**

```go
import (
	"time"

	"github.com/imrenagi/imrenagicom-agent/internal/config"
	"github.com/redis/go-redis/extra/redisotel/v9"
	"github.com/redis/go-redis/v9"
	"github.com/rs/zerolog/log"
)

func New(c config.RedisConfig) *redis.Client {
	log.Debug().
		Str("master_hosts", c.Host).
		Msgf("initiating redis master/slave client")

	rdb := redis.NewClient(&redis.Options{
		Addr:         c.Addr(),
		ReadTimeout:  time.Duration(c.ReadTimeoutSec) * time.Second,
		WriteTimeout: time.Duration(c.WriteTimeoutSec) * time.Second,
		DialTimeout:  time.Duration(c.ConnDialTimeoutSec) * time.Second,
		PoolTimeout:  time.Duration(c.ConnPoolTimeoutSec) * time.Second,
		PoolSize:     c.ConnPoolSize,
		MinIdleConns: c.MinIdleConn,
		MaxIdleConns: c.MaxIdleConn,
		Password:     c.Password,
	})

	// Enable tracing instrumentation.
	if err := redisotel.InstrumentTracing(rdb); err != nil {
		panic(err)
	}
	// Enable metrics instrumentation.
	if err := redisotel.InstrumentMetrics(rdb); err != nil {
		panic(err)
	}
	return rdb
}
```

Reference: [go-redis Tracing & Metrics](https://redis.uptrace.dev/guide/go-redis-opentelemetry.html)
