---
title: Batch Get (AIP-231)
impact: MEDIUM
impactDescription: Standardizes efficient retrieval of multiple specific resources in a single atomic request.
tags: protobuf, aip, batch, get
---

## Batch Get (AIP-231)

**Impact: MEDIUM**

Batch Get allows users to retrieve multiple resource instances by their names in a single, atomic operation.

**Implementation Rules:**

- **RPC Name**: Must start with `BatchGet` followed by the plural resource name (e.g., `BatchGetBooks`).
- **HTTP**: Use `GET` method with `:batchGet` custom verb. No body.
- **Request Message**:
  - `string parent`: (Optional) The shared parent.
  - `repeated string names`: (Required) Names of resources to retrieve.
- **Response Message**:
  - `repeated <Resource> <resources>`: The retrieved resources.
- **Atomicity**: The method _must_ fail for all resources or succeed for all (no partial success). Use `List` for partial failure scenarios.
- **Ordering**: The response list order _must_ match the request `names` order.
- **Pagination**: Batch Get _should not_ support pagination.

**Correct (Batch Get Pattern):**

```proto
rpc BatchGetBooks(BatchGetBooksRequest) returns (BatchGetBooksResponse) {
  option (google.api.http) = {
    get: "/v1/{parent=publishers/*}/books:batchGet"
  };
}

message BatchGetBooksRequest {
  string parent = 1;
  repeated string names = 2 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference).type = "library.googleapis.com/Book"
  ];
}
```

Reference: [AIP-231: Batch methods: Get](https://google.aip.dev/231)
