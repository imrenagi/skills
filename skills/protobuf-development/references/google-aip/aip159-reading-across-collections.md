---
title: Reading Across Collections (AIP-159)
impact: MEDIUM
impactDescription: Allows users to list or get resources without knowing the exact parent collection.
tags: protobuf, aip, list, get, wildcard
---

## Reading Across Collections (AIP-159)

**Impact: MEDIUM**

Reading across collections allows users to find resources when the specific parent is unknown or when they need a global view.

**Implementation Rules:**

- **Wildcard**: Support `-` as a wildcard for parent collection IDs in `List` or `Get` methods.
- **URI Pattern**: The HTTP annotation must use `*` for the collection, not hard-code `-`.
- **Response Names**: Resources returned must use their _canonical_ names (actual parent ID, not `-`).
- **Uniqueness**: Only support `Get` with `-` if the resource ID is guaranteed to be unique across all parents.
- **Ordering**: Cross-parent `List` should not support `order_by`, or it must be documented as "best effort".
- **Partial Failure**: If reading across collections could fail partially (e.g., one location is down), implement the `unreachable` field (AIP-217).

**Correct (List Across Collections):**

```proto
// In the service definition
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  option (google.api.http) = {
    get: "/v1/{parent=publishers/*}/books"
  };
}

// Example usage: GET /v1/publishers/-/books
// Response would contain: { "name": "publishers/123/books/abc", ... }
```

Reference: [AIP-159: Reading across collections](https://google.aip.dev/159)
