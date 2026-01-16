---
title: Strongly Typed Domain Enums
impact: MEDIUM
impactDescription: Ensures type safety and validation for enumerated values, preventing invalid data from entering the domain.
tags: go, domain-driven-design, enum
---

## Strongly Typed Domain Enums

**Impact: MEDIUM**

Go does not have a native `enum` type. To implement enums in the domain layer, use a custom defined type based on `string` or `int` combined with `const` and `iota`. This provides type safety and allows you to attach behavior (like validation) directly to the enum type.

### Implementation Pattern

Define the type and the valid constants. Always include an "Unknown" or "Unspecified" value as the zero-value (0) to handle uninitialized states.

```go
package domain

import (
    "fmt"
    "strings"
)

type OrderStatus int

const (
    OrderStatusUnknown OrderStatus = iota
    OrderStatusPending
    OrderStatusProcessing
    OrderStatusCompleted
    OrderStatusCancelled
)

// IsValid checks if the enum value is within the allowed set
func (s OrderStatus) IsValid() bool {
    return s > OrderStatusUnknown && s <= OrderStatusCancelled
}

// ToProto converts the domain enum to the generated gRPC status enum
func (s OrderStatus) ToProto() pb.OrderStatus {
    switch s {
    case OrderStatusPending:
        return pb.OrderStatus_ORDER_STATUS_PENDING
    case OrderStatusProcessing:
        return pb.OrderStatus_ORDER_STATUS_PROCESSING
    case OrderStatusCompleted:
        return pb.OrderStatus_ORDER_STATUS_COMPLETED
    case OrderStatusCancelled:
        return pb.OrderStatus_ORDER_STATUS_CANCELLED
    default:
        return pb.OrderStatus_ORDER_STATUS_UNSPECIFIED
    }
}
```

### Parsing from Protobuf

When receiving data from gRPC requests, provide a helper to map the gRPC enum back into your domain type.

```go
func FromProtoOrderStatus(p pb.OrderStatus) OrderStatus {
    switch p {
    case pb.OrderStatus_ORDER_STATUS_PENDING:
        return OrderStatusPending
    case pb.OrderStatus_ORDER_STATUS_PROCESSING:
        return OrderStatusProcessing
    case pb.OrderStatus_ORDER_STATUS_COMPLETED:
        return OrderStatusCompleted
    case pb.OrderStatus_ORDER_STATUS_CANCELLED:
        return OrderStatusCancelled
    default:
        return OrderStatusUnknown
    }
}
```

### Best Practices

1.  **Use `iota` for Integers**: Prefer integer-based enums with `iota` for internal logic as they are more efficient.
2.  **Zero-Value is Unknown**: Always define the `0` value as `Unknown` or `Unspecified`. This catches bugs where a struct is initialized without explicitly setting the enum.
3.  **Encapsulate Validation**: Use the `IsValid()` method before performing logic that depends on the enum value.
4.  **Avoid Naked Strings**: Never use raw strings (e.g., `"pending"`) for status or types in your business logic. Use the domain enum type.
5.  **Favor gRPC Conversion**: Always prefer converting to/from generated gRPC enum types over using raw strings. Implement conversion methods (e.g., `ToProto`, `FromProto`) to maintain type safety across the API boundary. Use string conversion only if strictly required for legacy JSON APIs or logging.

### References

- [Go by Example: Enums](https://gobyexample.com/enums)
- [Uber Go Style Guide: Enums](https://github.com/uber-go/guide/blob/master/style.md#enums)
