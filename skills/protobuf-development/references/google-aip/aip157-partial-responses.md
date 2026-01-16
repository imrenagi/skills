---
title: Partial Responses (AIP-157)
impact: MEDIUM
impactDescription: Provides users control over which fields are returned, reducing payload size and computation cost.
tags: protobuf, aip, partial-response, field-mask, view
---

## Partial Responses (AIP-157)

**Impact: MEDIUM**

When resources are large or expensive to compute, APIs should allow clients to request only the fields they need. This is achieved through field masks or view enumerations.

**Implementation Rules:**

- **Field Masks**:
  - Should use a side channel (query parameter or header) for `google.protobuf.FieldMask`.
  - Must be optional; if omitted, default to `*` (all fields).
- **View Enumeration**:
  - Recommended for simplifying common use cases (e.g., `BASIC` vs `FULL`).
  - Should be a `view` field on the request message (e.g., `BookView view`).
  - Enum values should at minimum include `BASIC` and `FULL`.
  - `BOX_VIEW_UNSPECIFIED` must be valid and documented.
  - Default for `List` should be `BASIC`; default for `Get`/`Create`/`Update` should be `BASIC` or `FULL`.
- **Consistency**: Adding fields to a view is safe; removing fields is a breaking change.
- **Declarative Warning**: Partial responses as a default can hinder declarative clients; ensure a `FULL` view is always available.

**Correct (View Enum Pattern):**

```proto
enum BookView {
  BOOK_VIEW_UNSPECIFIED = 0;
  BOOK_VIEW_BASIC = 1;
  BOOK_VIEW_FULL = 2;
}

message GetBookRequest {
  string name = 1;
  BookView view = 2;
}
```

Reference: [AIP-157: Partial responses](https://google.aip.dev/157)
