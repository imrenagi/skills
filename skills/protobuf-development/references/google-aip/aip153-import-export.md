---
title: Import and Export (AIP-153)
impact: MEDIUM
impactDescription: Standardizes how to bulk load or extract data, addressing vendor lock-in and migration needs.
tags: protobuf, aip, import, export, migration
---

## Import and Export (AIP-153)

**Impact: MEDIUM**

Import and Export operations allow users to move data into or out of an API, typically involving multiple resources or external storage.

**Implementation Rules:**

- **Methods**: Use `:import` and `:export` custom verbs.
- **LRO**: Must return `google.longrunning.Operation` if the operation isn't guaranteed to be fast (seconds).
- **HTTP**: Use `POST` with `body: "*"`.
- **Parent**:
  - Include `parent` in the URI.
  - Support `"-"` to import into or export from multiple parents (AIP-159).
  - validate that imported resources match the provided parent.
- **Request Structure**:
  - Group source/destination config in a `oneof source` or `oneof destination`.
  - Place common data-specific config at the top level.
- **Inline**: Support `InlineSource` or `InlineDestination` where data is in the request/response body.
- **Errors**: Include partial failure information in the LRO metadata using `google.rpc.Status`.

**Correct (Import Pattern):**

```proto
rpc ImportBooks(ImportBooksRequest) returns (google.longrunning.Operation) {
  option (google.api.http) = {
    post: "/v1/{parent=publishers/*}/books:import"
    body: "*"
  };
}

message ImportBooksRequest {
  string parent = 1;
  oneof source {
    GcsSource gcs_source = 2;
    InlineSource inline_source = 3;
  }
}
```

Reference: [AIP-153: Import and export](https://google.aip.dev/153)
