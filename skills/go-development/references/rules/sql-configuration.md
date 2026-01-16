---
title: SQL Database Configuration
impact: MEDIUM
impactDescription: Standardizes database connection parameters and string generation
tags: sql, configuration, postgres
---

## SQL Database Configuration

**Impact: MEDIUM**

Define a standard `Config` configuration struct to manage database connection parameters consistently across the application. This struct should include helper methods to generate the `DatabaseUrl` and `DataSourceName` required by various database drivers and libraries. This reduces code duplication and the risk of malformed connection strings.

**Directory Structure:**

```
.
├── internal
    └── pg
        └── config.go
    └── mysql
        └── config.go
```

**Incorrect (Ad-hoc string construction):**

```go
// Bad: Manually constructing connection strings leads to duplication and errors
connStr := fmt.Sprintf("postgres://%s:%s@localhost:5432/mydb", user, pass)
db, err := sqlx.Open("postgres", connStr)
```

**Correct (Standardized Config Struct):**

```go
package pg

import (
	"fmt"
)

type Config struct {
	Host        string `yaml:"host"`
	Port        int    `yaml:"port"`
	User        string `yaml:"user"`
	Password    string `yaml:"password"`
	Database    string `yaml:"database"`
	SSLMode     string `yaml:"sslmode"`
	MaxIdleConn int    `yaml:"maxIdleConn"`
	MaxOpenConn int    `yaml:"maxOpenConn"`
}

func (s Config) DatabaseUrl() string {
	return fmt.Sprintf("postgres://%s:%s@%s:%d/%s?sslmode=%s",
		s.User, s.Password, s.Host, s.Port, s.Database, s.SSLMode)
}

func (s Config) DataSourceName() string {
	return fmt.Sprintf("user=%s password=%s host=%s port=%d dbname=%s sslmode=%s",
		s.User, s.Password, s.Host, s.Port, s.Database, s.SSLMode)
}
```

Reference: [sqlx Documentation](https://github.com/jmoiron/sqlx)
