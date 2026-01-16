---
title: Standard Methods: Get (AIP-131)
impact: HIGH
impactDescription: Standardizes the retrieval of a single resource.
tags: protobuf, aip, get, standard-methods
---

## Standard Methods: Get (AIP-131)

**Impact: HIGH**

The `Get` method is the standard way to retrieve a single resource by its name. Consistency in `Get` methods ensures that client libraries can be generated correctly and that the API is easy to use.

**Implementation Rules:**

- **Method Name**: Must begin with `Get` followed by the singular resource name (e.g., `GetBook`).
- **Request Message**: Must be named `<MethodName>Request` (e.g., `GetBookRequest`).
- **Response Message**: Must be the resource message itself (e.g., `returns (Book)`).
- **HTTP Mapping**:
  - Verb must be `GET`.
  - Path must include the resource name: `/v1/{name=publishers/*/books/*}`.
  - No `body` is allowed.
- **Request Fields**:
  - Must include a `string name` field, annotated as `REQUIRED`.
  - Should include a `google.api.resource_reference` to the resource type.
  - Must not contain other required fields.
- **Method Signature**: Should have `option (google.api.method_signature) = "name";`.

**Incorrect (Non-standard Get):**

```proto
service Library {
  // Bad: Non-standard name, returns a wrapper
  rpc FetchBook(FetchRequest) returns (FetchResponse);
}

message FetchRequest {
  string id = 1;
}

message FetchResponse {
  Book book = 1;
}
```

**Correct (Standard Get):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";
import "google/api/resource.proto";

service Library {
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=publishers/*/books/*}"
    };
    option (google.api.method_signature) = "name";
  }
}

message GetBookRequest {
  string name = 1 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference) = {
      type: "library.googleapis.com/Book"
    }];
}
```

Reference: [AIP-131: Standard methods: Get](https://google.aip.dev/131)
