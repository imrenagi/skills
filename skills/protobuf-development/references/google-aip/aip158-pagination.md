---
title: Pagination (AIP-158)
impact: HIGH
impactDescription: Mandatory for collections to ensure performance and scalability.
tags: protobuf, aip, pagination, list
---

## Pagination (AIP-158)

**Impact: HIGH**

All RPCs returning collections must provide pagination from the start. Adding it later is a breaking change because it changes the behavior of existing clients.

**Implementation Rules:**

- **Request Fields**:
  - `int32 page_size`: Maximum results to return. Must be optional. If `0`, use a default (doc required).
  - `string page_token`: Token to retrieve the next page. Must be optional.
- **Response Fields**:
  - `repeated <Resource> <resources>`: The first field in the response (number `1`).
  - `string next_page_token`: Opaque token for the next page. Empty if no more pages exist.
- **Opacity**: Page tokens must be opaque and URL-safe strings. Do not let users parse them.
- **Coercion**: If `page_size` exceeds the server's maximum, coerce it down to the maximum. Return `INVALID_ARGUMENT` for negative values.
- **Stability**: If other request parameters change during pagination, return `INVALID_ARGUMENT`.
- **Skip**: May include an `int32 skip` field to skip a number of individual resources.

**Correct (Standard Pagination):**

```proto
message ListBooksRequest {
  string parent = 1;
  // Default is 50, max is 1000.
  int32 page_size = 2;
  string page_token = 3;
}

message ListBooksResponse {
  repeated Book books = 1;
  string next_page_token = 2;
}
```

Reference: [AIP-158: Pagination](https://google.aip.dev/158)
