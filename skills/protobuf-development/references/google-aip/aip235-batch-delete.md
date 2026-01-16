---
title: Batch Delete (AIP-235)
impact: MEDIUM
impactDescription: Standardizes how to delete multiple resources in a single RPC.
tags: protobuf, aip, batch, delete
---

## Batch Delete (AIP-235)

**Impact: MEDIUM**

A batch delete method allows users to delete a set of resources in a single transaction.

**Implementation Rules:**

- **Naming**: RPC name must start with `BatchDelete`.
- **HTTP**: Use `POST` (not `DELETE`). The URI must end in `:batchDelete`. Body must be `"*"`.
- **Atomicity**:
  - Synchronous Batch Delete **must** be atomic.
  - Asynchronous Batch Delete **may** support partial success.
- **Request Message**:
  - `parent`: Shared parent.
  - `repeated string names`: The resource names to delete.
  - Alternatively, use `repeated DeleteResourceRequest requests` if additional per-resource fields (like `etag`) are needed.
- **Response Message**:
  - Should be `google.protobuf.Empty` (unless soft-deleting and returning updated resources).
- **Filtering**: Do NOT support filter-based matching for batch deletion (too risky). Use [AIP-165 (Purge)](./aip165-purge.md) for that.

**Example:**

```proto
rpc BatchDeleteBooks(BatchDeleteBooksRequest) returns (google.protobuf.Empty) {
  option (google.api.http) = {
    post: "/v1/{parent=publishers/*}/books:batchDelete"
    body: "*"
  };
}

message BatchDeleteBooksRequest {
  string parent = 1 [(google.api.resource_reference).child_type = "Book"];
  repeated string names = 2 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference).type = "Book"
  ];
}
```

Reference: [AIP-235: Batch methods: Delete](https://google.aip.dev/235)
