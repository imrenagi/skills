---
title: Standard Methods: List (AIP-132)
impact: HIGH
impactDescription: Standardizes the retrieval of a collection of resources with pagination, filtering, and ordering.
tags: protobuf, aip, list, standard-methods, pagination
---

## Standard Methods: List (AIP-132)

**Impact: HIGH**

The `List` method is used to retrieve a collection of resources. Standardizing `List` methods is critical for implementing consistent pagination, filtering, and ordering across an entire API.

**Implementation Rules:**

- **Method Name**: Must begin with `List` followed by the plural resource name (e.g., `ListBooks`).
- **Request/Response Messages**: Must be named `List<Resources>Request` and `List<Resources>Response`.
- **HTTP Mapping**:
  - Verb must be `GET`.
  - Path: `/v1/{parent=publishers/*}/books`.
  - No `body` is allowed.
- **Request Fields**:
  - `string parent`: The parent resource name, required unless it's a top-level collection.
  - `int32 page_size`: Maximum items to return.
  - `string page_token`: Token for the next page.
  - Should include `string filter` and `string order_by` if applicable.
- **Response Fields**:
  - `repeated <Resource> <resources>`: The list of resources. The field name should be the plural of the resource name.
  - `string next_page_token`: Token to retrieve the next page. Omitted if there are no more pages.
- **Pagination**: Must follow AIP-158.
- **Method Signature**: Should have `option (google.api.method_signature) = "parent";`.

**Incorrect (Non-standard List):**

```proto
message SearchBooksRequest {
  string publisher_id = 1;
  int32 limit = 2;
  int32 offset = 3;
}

message SearchBooksResponse {
  repeated Book items = 1;
}
```

**Correct (Standard List):**

```proto
import "google/api/annotations.proto";
import "google/api/field_behavior.proto";
import "google/api/resource.proto";

service Library {
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
    option (google.api.http) = {
      get: "/v1/{parent=publishers/*}/books"
    };
    option (google.api.method_signature) = "parent";
  }
}

message ListBooksRequest {
  string parent = 1 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference) = {
      child_type: "library.googleapis.com/Book"
    }];
  int32 page_size = 2;
  string page_token = 3;
  string filter = 4;
  string order_by = 5;
}

message ListBooksResponse {
  repeated Book books = 1;
  string next_page_token = 2;
}
```

Reference: [AIP-132: Standard methods: List](https://google.aip.dev/132)
