---
title: Unreachable Resources (AIP-217)
impact: LOW
impactDescription: Standardizes how to report partial failures when listing across collections and some data is missing.
tags: protobuf, aip, list, unreachable, partial-success
---

## Unreachable Resources (AIP-217)

**Impact: LOW**

When listing resources across multiple parents (AIP-159), some might be temporarily offline. The API should return reachable data and list what was missed.

**Implementation Rules:**

- **Field**: Include `repeated string unreachable` in the response message.
- **Content**: Must contain _service-relative_ resource names (e.g., `publishers/123/books/456`) of the unreachable resources or parent collections.
- **Annotation**: Must be marked with `(google.api.field_behavior) = UNORDERED_LIST`.
- **Behavior**:
  - The response must not provide error reasons for unreachable entries in the same response.
  - If the _entire_ requested scope is unreachable, the RPC must fail with an error.
  - For existing APIs (brownfield), use a `bool return_partial_success` flag in the request to opt-in to this behavior.

**Correct (Response Field):**

```proto
message ListBooksResponse {
  repeated Book books = 1;
  string next_page_token = 2;

  // Locations that could not be reached.
  repeated string unreachable = 3 [
    (google.api.field_behavior) = UNORDERED_LIST
  ];
}
```

Reference: [AIP-217: Unreachable resources](https://google.aip.dev/217)
