---
title: Unit Testing with Table-Driven Tests
impact: MEDIUM
impactDescription: improves test coverage and maintainability
tags: testing, table-driven
---

## Unit Testing with Table-Driven Tests

**Impact: MEDIUM**

Use table-driven tests for unit testing. This is the idiomatic Go way to test multiple scenarios (happy path, error cases) without duplicating setup code.

**Incorrect (Duplicated Logic):**

```go
func TestAdd_Positive(t *testing.T) {
    if Add(1, 2) != 3 { t.Fail() }
}

func TestAdd_Negative(t *testing.T) {
    if Add(-1, -1) != -2 { t.Fail() }
}
```

**Correct (Table Driven):**

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -5, -6},
        {"zero", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.want {
                t.Errorf("Add() = %v, want %v", got, tt.want)
            }
        })
    }
}
```
