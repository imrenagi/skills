---
title: SQL Query Builder with Squirrel
impact: MEDIUM
impactDescription:
tags: postgresql, database, sql
---

## SQL Query Builder with Squirrel

Use `github.com/Masterminds/squirrel` to construct query sent to sql database.

**Store initialization**

```go
import (
	sq "github.com/Masterminds/squirrel"
	"github.com/jmoiron/sqlx"
)

func NewOrderRepository(db *sqlx.DB) *OrderRepository {
	return &OrderRepository{
		db: db,
	}
}

type OrderRepository struct {
	db *sqlx.DB
}

```

**Using it in query (insert sample):**

```go
// sample order definition might be defined on other files within the same or different package
type Order struct {
    ID   string
    Name string
}

func (s *OrderRepository) Create(ctx context.Context, order *Order, opts ...Option) error {
	options := &Options{}
	for _, o := range opts {
		o(options)
	}

	var runner sq.BaseRunner = s.db
	if options.tx != nil {
		runner = options.tx
	}

	sb := sq.StatementBuilder.RunWith(runner)
	qb := sb.Insert("orders").
		Columns("id", "name").
		Values(order.ID, order.Name).
		PlaceholderFormat(sq.Dollar)
	_, err := qb.ExecContext(ctx); if err != nil {
		return fmt.Errorf("failed to create order: %w", err)
	}

	return nil
}
```
