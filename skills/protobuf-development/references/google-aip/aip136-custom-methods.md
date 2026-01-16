---
title: Custom Methods (AIP-136)
impact: MEDIUM
impactDescription: Provides a standard way to express actions that don't fit into the standard CRUDL pattern.
tags: protobuf, aip, custom-methods, design
---

## Custom Methods (AIP-136)

**Impact: MEDIUM**

Custom methods are used for actions that are difficult to model using standard methods (Get, List, Create, Update, Delete). They should be used sparingly to maintain API consistency.

**Implementation Rules:**

- **When to Use**: Only if the action cannot be cleanly expressed via standard methods.
- **Naming**:
  - Should be a verb followed by a noun (e.g., `ArchiveBook`).
  - Must not contain prepositions ("for", "with").
  - Must not use standard method verbs (Get, List, etc.).
  - Must not include the term `Async`. Use `LongRunning` suffix if needed.
- **HTTP Mapping**:
  - Must use `GET` (for read-only actions) or `POST` (for everything else).
  - Path must end with `:` followed by the custom verb (e.g., `:archive`).
  - Body should be `"*"` for `POST` methods.
- **Request/Response**: Should be named `<MethodName>Request` and `<MethodName>Response`.
- **Resource Association**:
  - If operating on a resource, the URI variable must be `name`.
  - If operating on a collection, the variable must be `parent`.

**Incorrect (Non-standard Custom Method):**

```proto
service Library {
  // Bad: Uses a preposition and a non-standard URI
  rpc SearchBooksByAuthor(SearchReq) returns (SearchResp) {
    option (google.api.http) = {
      post: "/v1/{publisher=publishers/*}/books/search"
      body: "*"
    };
  }
}
```

**Correct (Standard Custom Method):**

```proto
import "google/api/annotations.proto";

service Library {
  rpc ArchiveBook(ArchiveBookRequest) returns (ArchiveBookResponse) {
    option (google.api.http) = {
      post: "/v1/{name=publishers/*/books/*}:archive"
      body: "*"
    };
  }
}

message ArchiveBookRequest {
  string name = 1;
}

message ArchiveBookResponse {
  // Can also return the Resource itself if appropriate.
}
```

Reference: [AIP-136: Custom methods](https://google.aip.dev/136)
