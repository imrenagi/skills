---
title: Unset Field Values (AIP-149)
impact: MEDIUM
impactDescription: Clarifies when to use Protobuf's `optional` keyword to track field presence.
tags: protobuf, aip, optional, field-presence
---

## Unset Field Values (AIP-149)

**Impact: MEDIUM**

Distinguishing between a default value (like `0`) and an unset value is sometimes necessary for business logic. AIP-149 provides guidance on when to use `optional` for this purpose.

**Implementation Rules:**

- **Use Sparingly**: Only use `optional` if you _must_ distinguish between a default value and an unset value.
- **Prefer Alternatives**: Avoid the need for this distinction if possible (e.g., using a different default or a oneof).
- **Primitives Only**: `optional` is most commonly useful for integers and floats.
- **Backwards Compatibility**: Do _not_ add or remove `optional` from an existing field; it changes the generated code (e.g., to pointers in Go) and is a breaking change.
- **Independence**: `optional` (wire format presence) is independent of `google.api.field_behavior = REQUIRED` (semantic requirement).

**Correct (Tracking Presence):**

```proto
message Book {
  string name = 1;

  // Good: allows distinguishing "rated 0 stars" from "not yet rated".
  optional int32 rating = 2;
}
```

Reference: [AIP-149: Unset field values](https://google.aip.dev/149)
