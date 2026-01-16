---
title: Batch Update (AIP-234)
impact: MEDIUM
impactDescription: Standardizes how to update multiple resources in a single RPC.
tags: protobuf, aip, batch, update
---

## Batch Update (AIP-234)

**Impact: MEDIUM**

A batch update method allows users to modify a set of resources in a single transaction.

**Implementation Rules:**

- **Naming**: RPC name must start with `BatchUpdate`.
- **HTTP**: Use `POST`. The URI must end in `:batchUpdate`. Body must be `"*"`.
- **Atomicity**:
  - Synchronous Batch Update **must** be atomic.
  - Asynchronous (LRO) Batch Update **may** support partial success.
- **Request Message**:
  - `parent`: Shared parent of all resources.
  - `repeated UpdateResourceRequest requests`: Individual update requests.
  - Fields like `update_mask` can be "hoisted" to the top level if they apply to all requests.
- **Response Message**:
  - `repeated Resource resources`: The updated resources.
- **Partial Success**: Similar to Batch Create, use a map of index to status in operation metadata.

**Example:**

```proto
rpc BatchUpdateBooks(BatchUpdateBooksRequest) returns (BatchUpdateBooksResponse) {
  option (google.api.http) = {
    post: "/v1/{parent=publishers/*}/books:batchUpdate"
    body: "*"
  };
}

message BatchUpdateBooksRequest {
  string parent = 1 [(google.api.resource_reference).child_type = "Book"];
  repeated UpdateBookRequest requests = 2 [(google.api.field_behavior) = REQUIRED];
}
```

Reference: [AIP-234: Batch methods: Update](https://google.aip.dev/234)
