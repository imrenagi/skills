---
title: Redis Configuration
impact: MEDIUM
impactDescription: Standardizes Redis connection parameters for consistency across the application.
tags: go, redis, configuration
---

## Redis Configuration

**Impact: MEDIUM**

Defining a consistent Redis configuration structure ensures that timeouts, pool sizes, and connection addresses are handled uniformly across different services.

**Directory Structure:**

```
├── internal
    └── redis
        └── config.go
```

**Correct (Minimum Configuration):**

```go
type Config struct {
	Host               string   `yaml:"host"`
	Port               string   `yaml:"port"`
	DB                 int      `yaml:"db"`
	Password           string   `yaml:"password"`
	SentinelEnabled    bool     `yaml:"sentinelEnabled"`
	SentinelHosts      []string `yaml:"sentinelHost"`
	SentinelMasterName string   `yaml:"sentinelMasterName"`
	ConnDialTimeoutSec int      `yaml:"connDialTimeoutSec"`
	ReadTimeoutSec int          `yaml:"readTimeoutSec"`
	WriteTimeoutSec int         `yaml:"writeTimeoutSec"`
	ConnPoolSize int            `yaml:"connPoolSize"`
	ConnPoolTimeoutSec int      `yaml:"connPoolTimeoutSec"`
	MinIdleConn int             `yaml:"minIdleConn"`
	MaxIdleConn int             `yaml:"maxIdleConn"`
}

func (r Config) Addr() string {
	return r.Host + ":" + r.Port
}
```

Reference: [go-redis Documentation](https://redis.uptrace.dev/)
