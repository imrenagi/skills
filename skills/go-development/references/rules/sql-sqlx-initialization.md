---
title: Database Access with SQLX
impact: MEDIUM
impactDescription: SQLX client initialization
tags: sql, sqlx, postgres
---

## Database Access with SQLX

**Impact: MEDIUM**

- Use `github.com/jmoiron/sqlx` for SQL database interactions when you need raw SQL control with struct mapping.
- Implement client initialization for each database in its own package in `internal` package.

**Directory Structure:**

```
.
├── internal
    └── pg
        └── db.go
		└── config.go
    └── sql
        └── db.go
		└── config.go
```

**Client initialization:**

- Always enable OpenTelemetry instrumentation for the client initialization.

```go
package pg

import (
	"github.com/jmoiron/sqlx"
	"github.com/uptrace/opentelemetry-go-extra/otelsql"
	"github.com/uptrace/opentelemetry-go-extra/otelsqlx"
	semconv "go.opentelemetry.io/otel/semconv/v1.10.0"
)

func New(c Config) *sqlx.DB {
	db, err := otelsqlx.Open("postgres", c.DataSourceName(),
		otelsql.WithAttributes(semconv.DBSystemPostgreSQL))
	if err != nil {
		panic(err)
	}
	db.SetMaxOpenConns(c.MaxOpenConn)
	db.SetMaxIdleConns(c.MaxIdleConn)
	return db
}

```
