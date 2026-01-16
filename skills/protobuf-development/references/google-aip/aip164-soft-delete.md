---
title: Soft Delete (AIP-164)
impact: MEDIUM
impactDescription: Standardizes recovery from accidental deletions via soft delete and undelete methods.
tags: protobuf, aip, soft-delete, undelete, lifecycle
---

## Soft Delete (AIP-164)

**Impact: MEDIUM**

Soft delete allows resources to be recovered after deletion. This is implemented by marking a resource as deleted rather than removing it immediately.

**Implementation Rules:**

- **Method**: The standard `Delete` method marks the resource as deleted and returns the updated resource (instead of `Empty`).
- **Undelete**:
  - Provide an `Undelete` custom method (e.g., `rpc UndeleteBook`).
  - Use `POST` with `body: "*"`.
  - Return the resource itself.
- **Fields (AIP-148)**:
  - `google.protobuf.Timestamp delete_time`: When it was soft-deleted.
  - `google.protobuf.Timestamp purge_time`: When it will be permanently removed.
- **Visibility**: Soft-deleted resources should be hidden from `List` by default but returned by `Get`.
- **States**: If using a `state` enum (AIP-216), include a `DELETED` value.
- **Errors**: `Undelete` for a non-deleted resource should return `ALREADY_EXISTS`.

**Correct (Undelete Pattern):**

```proto
service Library {
  rpc DeleteBook(DeleteBookRequest) returns (Book); // Returns book with delete_time set

  rpc UndeleteBook(UndeleteBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{name=publishers/*/books/*}:undelete"
      body: "*"
    };
  }
}
```

Reference: [AIP-164: Soft delete](https://google.aip.dev/164)
