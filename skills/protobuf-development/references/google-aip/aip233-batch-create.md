---
title: Batch Create (AIP-233)
impact: MEDIUM
impactDescription: Standardizes how to create multiple resources in a single RPC, either atomically or with partial success.
tags: protobuf, aip, batch, create
---

## Batch Create (AIP-233)

**Impact: MEDIUM**

A batch create method allows users to create multiple resources in a single transaction.

**Implementation Rules:**

- **Naming**: RPC name must start with `BatchCreate` followed by the plural resource name (e.g., `BatchCreateBooks`).
- **HTTP**: Use `POST`. The URI must end in `:batchCreate`. Body must be `"*"`.
- **Atomicity**:
  - Synchronous Batch Create **must** be atomic (all succeed or all fail).
  - Asynchronous (LRO) Batch Create **may** support partial success.
- **Request Message**:
  - `parent`: Shared parent of all resources.
  - `repeated CreateResourceRequest requests`: Individual requests.
- **Response Message**:
  - `repeated Resource resources`: The created resources.
- **Partial Success (LRO only)**:
  - If supporting partial success, the metadata must include `map<int32, google.rpc.Status> failed_requests` where the key is the index in the `requests` field.

**Example:**

```proto
rpc BatchCreateBooks(BatchCreateBooksRequest) returns (BatchCreateBooksResponse) {
  option (google.api.http) = {
    post: "/v1/{parent=publishers/*}/books:batchCreate"
    body: "*"
  };
}

message BatchCreateBooksRequest {
  string parent = 1 [(google.api.resource_reference).child_type = "Book"];
  repeated CreateBookRequest requests = 2 [(google.api.field_behavior) = REQUIRED];
}
```

Reference: [AIP-233: Batch methods: Create](https://google.aip.dev/233)
