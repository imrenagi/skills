---
title: Standard Methods: Delete (AIP-135)
impact: HIGH
impactDescription: Standardizes the removal of resources.
tags: protobuf, aip, delete, standard-methods
---

## Standard Methods: Delete (AIP-135)

**Impact: HIGH**

The `Delete` method is the standard way to remove a resource. It ensures that accidental deletions are minimized (via cascading checks) and that clients can predictably manage resource lifecycles.

**Implementation Rules:**

- **Method Name**: Must begin with `Delete` followed by the singular resource name (e.g., `DeleteBook`).
- **Request Message**: Must be named `Delete<Resource>Request`.
- **Response Message**: Should be `google.protobuf.Empty` (unless it's a soft delete).
- **HTTP Mapping**:
  - Verb must be `DELETE`.
  - Path: `/v1/{name=publishers/*/books/*}`.
  - No `body` is allowed.
- **Request Fields**:
  - `string name`: The resource name to delete, required.
  - `bool force`: If the resource has children, `force` must be `true` to perform a cascading delete.
  - `string etag`: Optional or required for protected deletion (AIP-128).
- **Consistency**: Completion of delete must mean the resource no longer exists (strong consistency).
- **Error Handling**: Fail with `FAILED_PRECONDITION` if children exist and `force` is false. Fail with `NOT_FOUND` if the resource doesn't exist (unless `allow_missing` is true).
- **Method Signature**: Should have `option (google.api.method_signature) = "name";`.

**Incorrect (Non-standard Delete):**

```proto
service Library {
  // Bad: Non-standard name, returns BOOL
  rpc RemoveBook(RemoveBookRequest) returns (bool);
}

message RemoveBookRequest {
  string book_id = 1;
}
```

**Correct (Standard Delete):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";
import "google/protobuf/empty.proto";

service Library {
  rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
    option (google.api.http) = {
      delete: "/v1/{name=publishers/*/books/*}"
    };
    option (google.api.method_signature) = "name";
  }
}

message DeleteBookRequest {
  string name = 1 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference).type = "library.googleapis.com/Book"
  ];

  // Opt-in to deleting child resources.
  bool force = 2;

  // For protected deletion.
  string etag = 3;
}
```

Reference: [AIP-135: Standard methods: Delete](https://google.aip.dev/135)
