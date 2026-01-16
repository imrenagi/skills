---
title: Repeated Fields (AIP-144)
impact: MEDIUM
impactDescription: Standardizes how to handle lists of data, including strategies for updates and avoiding payload bloat.
tags: protobuf, aip, repeated-fields, list
---

## Repeated Fields (AIP-144)

**Impact: MEDIUM**

Representing lists of data correctly is essential for maintaining API performance and ensuring that users can modify lists effectively.

**Implementation Rules:**

- **Naming**: Must use a plural field name (e.g., `authors`). If the word is already plural, use the dictionary form.
- **Payload Size**: Should have an enforced upper bound (e.g., 100 elements). If the list can grow indefinitely, use a sub-resource with pagination instead.
- **No Inline Resources**: Must not represent the full body of another resource inline. Use resource names (strings) instead.
- **Scalars vs. Messages**:
  - Use scalars (e.g., `string`) if no additional fields will be needed.
  - Use messages proactively if you expect to add metadata to each list item later.
- **Update Strategy**:
  - **Direct Update**: Use the standard `Update` method (AIP-134) to replace the entire list. This is required for declarative-friendly resources.
  - **Custom Add/Remove**: Use custom methods like `AddAuthor` and `RemoveAuthor` for atomic modifications.
- **Add/Remove RPCs**:
  - Names must start with `Add` or `Remove` followed by the singular field name.
  - HTTP verb must be `POST`.
  - URI must end with `:add*` or `:remove*` (e.g., `:addAuthor`).
  - Error with `ALREADY_EXISTS` on `Add` if present, or `NOT_FOUND` on `Remove` if missing.

**Incorrect (Payload Bloat and Bad Naming):**

```proto
message Book {
  // Bad: singular name, might grow too large
  repeated string tag = 1;
  // Bad: inlining another resource body
  repeated Publisher publishers = 2;
}
```

**Correct (Standard Repeated Fields):**

```proto
message Book {
  // Good: plural name, bounded usage
  repeated string tags = 1;
  // Good: reference by name, not body
  repeated string publisher_names = 2;
}

// Atomic modification
rpc AddTag(AddTagRequest) returns (Book) {
  option (google.api.http) = {
    post: "/v1/{book=publishers/*/books/*}:addTag"
    body: "*"
  };
}
```

Reference: [AIP-144: Repeated fields](https://google.aip.dev/144)
