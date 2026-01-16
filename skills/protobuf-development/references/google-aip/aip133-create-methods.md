---
title: Standard Methods: Create (AIP-133)
impact: HIGH
impactDescription: Standardizes the creation of new resources in a collection.
tags: protobuf, aip, create, standard-methods
---

## Standard Methods: Create (AIP-133)

**Impact: HIGH**

The `Create` method is the standard way to add a new resource to a collection. Consistency in `Create` methods ensures that clients can predictably create resources and handle user-specified IDs.

**Implementation Rules:**

- **Method Name**: Must begin with `Create` followed by the singular resource name (e.g., `CreateBook`).
- **Request Message**: Must be named `Create<Resource>Request`.
- **Response Message**: Must be the resource message itself (or `google.longrunning.Operation`).
- **HTTP Mapping**:
  - Verb must be `POST`.
  - Path: `/v1/{parent=publishers/*}/books`.
  - Must include a `body` mapping to the resource field (e.g., `body: "book"`).
- **Request Fields**:
  - `string parent`: The parent resource name, required.
  - `<Resource> <resource>`: The resource to create, required.
  - `string <resource>_id`: Optional or required field for the user to specify the ID component.
- **Resource ID**: User-specified IDs are mandatory for management plane resources.
- **Method Signature**: Should have `option (google.api.method_signature) = "parent,book,book_id";`.

**Incorrect (Non-standard Create):**

```proto
service Library {
  // Bad: Non-standard name, returns wrapper
  rpc NewBook(NewBookReq) returns (NewBookResp);
}

message NewBookReq {
  string publisher = 1;
  string title = 2;
  string author = 3;
}
```

**Correct (Standard Create):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";
import "google/api/resource.proto";

service Library {
  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=publishers/*}/books"
      body: "book"
    };
    option (google.api.method_signature) = "parent,book,book_id";
  }
}

message CreateBookRequest {
  string parent = 1 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference) = {
      child_type: "library.googleapis.com/Book"
    }];

  Book book = 2 [(google.api.field_behavior) = REQUIRED];

  string book_id = 3;
}
```

Reference: [AIP-133: Standard methods: Create](https://google.aip.dev/133)
