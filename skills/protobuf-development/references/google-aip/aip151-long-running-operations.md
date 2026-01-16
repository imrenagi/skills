---
title: Long-Running Operations (AIP-151)
impact: HIGH
impactDescription: Standardizes how to handle tasks that take a significant amount of time, providing a promise-based pattern.
tags: protobuf, aip, lro, long-running-operations
---

## Long-Running Operations (AIP-151)

**Impact: HIGH**

For tasks that take longer than a few seconds (rule of thumb: 10s), APIs should return a "promise" rather than blocking. This is implemented using `google.longrunning.Operation`.

**Implementation Rules:**

- **Return Type**: Must be `google.longrunning.Operation`.
- **Annotation**: Must include `google.longrunning.operation_info` defining `response_type` and `metadata_type`.
- **Operations Service**: APIs using LROs must implement the standard `google.longrunning.Operations` service.
- **Guidance**:
  - `response_type` should not be `Empty` (except for `Delete`).
  - `metadata_type` should provide progress or partial failure info.
- **Method Behavior**:
  - `Create`/`Delete` LROs: The resource should appear in `List`/`Get` calls but indicate an "in-progress" state (e.g., via a `state` enum).
  - Parallelism: If parallel operations are not allowed on a resource, return `ABORTED`.
- **Errors**: Failures to _start_ return an immediate error. Failures _during_ execution are stored in `Operation.error`.

**Correct (Standard LRO):**

```proto
import "google/longrunning/operations.proto";

service Library {
  rpc CreateBook(CreateBookRequest) returns (google.longrunning.Operation) {
    option (google.api.http) = {
      post: "/v1/{parent=publishers/*}/books"
      body: "book"
    };
    option (google.longrunning.operation_info) = {
      response_type: "Book"
      metadata_type: "CreateBookMetadata"
    };
  }
}

message CreateBookMetadata {
  // Information about progress or partial successes.
}
```

Reference: [AIP-151: Long-running operations](https://google.aip.dev/151)
