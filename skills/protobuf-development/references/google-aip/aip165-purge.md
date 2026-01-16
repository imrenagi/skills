---
title: Criteria-Based Delete (Purge) (AIP-165)
impact: MEDIUM
impactDescription: Standardizes deleting large numbers of resources based on a filter, with safety checks like `force`.
tags: protobuf, aip, purge, bulk-delete
---

## Criteria-Based Delete (Purge) (AIP-165)

**Impact: MEDIUM**

Purge methods are for deleting thousands or millions of resources at once using a filter. This is rare and should be used cautiously.

**Implementation Rules:**

- **Method Name**: Must start with `Purge` followed by plural resource (e.g., `PurgeBooks`).
- **LRO**: Must return a `google.longrunning.Operation`.
- **Request Fields**:
  - `string parent`: The collection parent (or `-` for all).
  - `string filter`: Mandatory filter (AIP-160).
  - `bool force`: Mandatory safety flag.
- **Safety**:
  - If `force = false`, the method must NOT delete anything. Instead, return a count and a sample of resources that _would_ be deleted.
  - If `force = true`, perform the deletion.
- **Response**: Should include `int32 purge_count` and `repeated string purge_sample` (names of deleted/to-be-deleted items).

**Correct (Purge Pattern):**

```proto
rpc PurgeBooks(PurgeBooksRequest) returns (google.longrunning.Operation) {
  option (google.api.http) = {
    post: "/v1/{parent=publishers/*}/books:purge"
    body: "*"
  };
}

message PurgeBooksRequest {
  string parent = 1;
  string filter = 2 [(google.api.field_behavior) = REQUIRED];
  bool force = 3;
}
```

Reference: [AIP-165: Criteria-based delete](https://google.aip.dev/165)
