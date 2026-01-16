---
title: Standard Methods: Update (AIP-134)
impact: HIGH
impactDescription: Standardizes how resources are modified, emphasizing partial updates and field masks.
tags: protobuf, aip, update, standard-methods, field-mask
---

## Standard Methods: Update (AIP-134)

**Impact: HIGH**

The `Update` method is the standard way to modify an existing resource. AIP-134 prioritizes partial updates using field masks to ensure that APIs can evolve without breaking client code.

**Implementation Rules:**

- **Method Name**: Must begin with `Update` followed by the singular resource name (e.g., `UpdateBook`).
- **Request Message**: Must be named `Update<Resource>Request`.
- **Response Message**: Must be the resource message itself (or `google.longrunning.Operation`).
- **HTTP Mapping**:
  - Verb should be `PATCH`. `PUT` is strongly discouraged as it prevents non-breaking field additions.
  - Path: `/v1/{book.name=publishers/*/books/*}`. Note the nested field reference.
  - Must include a `body` mapping to the resource field (e.g., `body: "book"`).
- **Request Fields**:
  - `<Resource> <resource>`: The resource to update, required.
  - `google.protobuf.FieldMask update_mask`: The list of fields to update. Required for partial updates.
- **Field Mask Behavior**:
  - An omitted mask must be treated as all populated fields.
  - Support `*` for full replacement.
  - Field names in the mask correspond to the resource, not the request.
- **Concurrency**: Use an `etag` field if optimistic concurrency is needed. Return `ABORTED` error if the etag doesn't match.
- **Method Signature**: Should have `option (google.api.method_signature) = "book,update_mask";`.

**Incorrect (Full Replacement Update):**

```proto
service Library {
  // Bad: Using PUT and no update mask
  rpc UpdateBook(UpdateBookRequest) returns (Book) {
    option (google.api.http) = {
      put: "/v1/{book.name=publishers/*/books/*}"
      body: "book"
    };
  }
}
```

**Correct (Standard Partial Update):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";
import "google/protobuf/field_mask.proto";

service Library {
  rpc UpdateBook(UpdateBookRequest) returns (Book) {
    option (google.api.http) = {
      patch: "/v1/{book.name=publishers/*/books/*}"
      body: "book"
    };
    option (google.api.method_signature) = "book,update_mask";
  }
}

message UpdateBookRequest {
  Book book = 1 [(google.api.field_behavior) = REQUIRED];

  google.protobuf.FieldMask update_mask = 2;

  // Optional: create if not found
  bool allow_missing = 3;
}
```

Reference: [AIP-134: Standard methods: Update](https://google.aip.dev/134)
