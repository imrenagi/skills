---
title: Resource Revisions (AIP-162)
impact: MEDIUM
impactDescription: Standardizes how to handle historical versions of a resource, including rollback and aliasing.
tags: protobuf, aip, revisions, versioning, rollback
---

## Resource Revisions (AIP-162)

**Impact: MEDIUM**

Revisions allow users to track the history of a resource, roll back to previous states, or reference a snapshot in time.

**Implementation Rules:**

- **Structure**: Abstract revisions as a nested collection named `revisions`.
- **Resource Name**: `{resource_name}/revisions/{revision_id}`.
- **Message Name**: `{ResourceType}Revision`.
- **Fields**:
  - `string name`: The name of the revision.
  - `<Resource> snapshot`: The full state of the parent at that point in time, `OUTPUT_ONLY`.
  - `google.protobuf.Timestamp create_time`: When the revision was created, `OUTPUT_ONLY`.
- **Aliases**: Use `revisions/latest` for the most recent one. Support `Alias{Resource}Revision` for user-defined tags (e.g., `published`).
- **Rollback**: Use a custom `Rollback{Resource}` method that takes a revision name and reverts the parent resource to that state.
- **Default Order**: `List` of revisions must be in reverse chronological order by default.

**Correct (Revision Pattern):**

```proto
message BookRevision {
  option (google.api.resource) = {
    type: "library.googleapis.com/BookRevision"
    pattern: "publishers/{publisher}/books/{book}/revisions/{revision}"
  };

  string name = 1;
  Book snapshot = 2 [(google.api.field_behavior) = OUTPUT_ONLY];
  google.protobuf.Timestamp create_time = 3 [(google.api.field_behavior) = OUTPUT_ONLY];
}

service Library {
  rpc RollbackBook(RollbackBookRequest) returns (BookRevision) {
    option (google.api.http) = {
      post: "/v1/{name=publishers/*/books/*/revisions/*}:rollback"
      body: "*"
    };
  }
}
```

Reference: [AIP-162: Resource Revisions](https://google.aip.dev/162)
