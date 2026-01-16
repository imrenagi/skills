---
title: HTTP and gRPC Transcoding (AIP-127)
impact: HIGH
impactDescription: Enables RPCs to be accessed via REST/JSON, improving accessibility for web developers.
tags: protobuf, aip, gRPC, HTTP, transcoding
---

## HTTP and gRPC Transcoding (AIP-127)

**Impact: HIGH**

Transcoding allows gRPC services to be exposed as RESTful APIs. This is crucial for web and mobile clients that may not support gRPC natively. Following these rules ensures that the resulting REST API is consistent and predictable.

**Implementation Rules:**

- **Mandatory Annotations**: All RPCs (except bi-directional streaming) must define the HTTP method and path using the `google.api.http` annotation.
- **HTTP Verbs**:
  - Use `get`, `post`, `patch`, or `delete` as prescribed by the standard method (AIP-131-135).
  - Do not use `put` or `custom`.
- **URI Paths**:
  - Use the `{foo=bar/*}` syntax to map URL segments to request fields.
  - The variable must include the entire resource name, not just the ID.
  - Use `*` for ID segments and `**` for the final segment if it contains slashes.
- **Request Body**:
  - Use the `body` key to specify which field contains the HTTP request body.
  - Use `body: "*"` if the entire request message is the body.
  - Do not define a `body` for `GET` or `DELETE` methods.
  - The `body` must not be a nested field, a repeated field, or a URI parameter.
- **Additional Bindings**: Use `additional_bindings` if an RPC corresponds to multiple URIs. The `body` must be identical across all bindings.
- **Streaming**: Bi-directional streaming RPCs should not have `google.api.http` annotations. Provide non-streaming alternatives if possible.

**Incorrect (Missing Transcoding):**

```proto
service LibraryService {
  // Bad: No HTTP annotation
  rpc GetBook(GetBookRequest) returns (Book);
}
```

**Correct (Standard Transcoding):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";

service LibraryService {
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=publishers/*/books/*}"
    };
  }

  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=publishers/*}/books"
      body: "book"
    };
  }
}

message CreateBookRequest {
  string parent = 1 [(google.api.field_behavior) = REQUIRED];
  Book book = 2 [(google.api.field_behavior) = REQUIRED];
}
```

Reference: [AIP-127: HTTP and gRPC transcoding](https://google.aip.dev/127)
