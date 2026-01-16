---
title: Database Access with SQLX
impact: MEDIUM
impactDescription: SQLX client initialization
tags: sql, sqlx, postgres
---

## Database Access with SQLX

**Impact: MEDIUM**

- Use `github.com/jmoiron/sqlx` for database interactions when you need raw SQL control with struct mapping.
- Implement client initialization in `initernal/pg/db.go`.
- Always enable opentelemetry sql instrumentation.

**Client initialization:**

```go
package pg

import (
	"github.com/jmoiron/sqlx"
	"github.com/uptrace/opentelemetry-go-extra/otelsql"
	"github.com/uptrace/opentelemetry-go-extra/otelsqlx"
	semconv "go.opentelemetry.io/otel/semconv/v1.10.0"
)

func New(c config.SQL) *sqlx.DB {
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
