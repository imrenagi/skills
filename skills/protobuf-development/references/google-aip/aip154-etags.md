---
title: Resource Freshness (ETags) (AIP-154)
impact: HIGH
impactDescription: Standardizes optimistic concurrency control to prevent race conditions during updates or deletions.
tags: protobuf, aip, etag, concurrency
---

## Resource Freshness (ETags) (AIP-154)

**Impact: HIGH**

ETags (Entity Tags) allow the server to ensure that a client is acting on the most recent version of a resource. This is critical for preventing "lost updates" in parallel environments.

**Implementation Rules:**

- **Resource Field**: Must include a `string etag` field.
- **Value Format**: Should follow RFC 7232 (e.g., `"foo"`, including quotes). Weak ETags must start with `W/`.
- **Match Behavior**: If the provided ETag matches, permit the request. If it does not match, return an `ABORTED` (HTTP 409) error.
- **Missing ETag**: Usually permit the request without validation, unless strong consistency is required (if so, return `INVALID_ARGUMENT`).
- **Declarative-Friendly**: Resources marked as declarative-friendly (AIP-128) _must_ implement ETags.
- **Request Usage**: Include `string etag` in `Update` (inline in resource) and `Delete` (as a request field) methods.

**Incorrect (Missing ETag):**

```proto
message Book {
  string name = 1;
  // Bad: No way to check if this book was modified since the user last read it.
}
```

**Correct (Standard ETags):**

```proto
message Book {
  string name = 1;
  // Good: used for optimistic concurrency
  string etag = 2;
}

message DeleteBookRequest {
  string name = 1;
  // Good: ensures the user is deleting the version they expect
  string etag = 2 [(google.api.field_behavior) = OPTIONAL];
}
```

Reference: [AIP-154: Resource freshness validation](https://google.aip.dev/154)
