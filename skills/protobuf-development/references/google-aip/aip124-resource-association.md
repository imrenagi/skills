---
title: Resource Association (AIP-124)
impact: MEDIUM
impactDescription: Standardizes how many-to-one and many-to-many relationships are modeled.
tags: protobuf, aip, resource-association, design
---

## Resource Association (AIP-124)

**Impact: MEDIUM**

APIs often have resource hierarchies that cannot be expressed in a simple tree. AIP-124 provides guidance on handling complex relationships like many-to-one and many-to-many associations.

**Implementation Rules:**

- **Canonical Parent**: A resource must have at most one canonical parent.
- **Many-to-One**:
  - Choose one resource as the canonical parent in the resource name pattern.
  - Associate with other resources using additional fields (e.g., `author` field in a `Book` resource where `Publisher` is the parent).
  - Use `string filter` in `List` requests to filter by non-parent associations.
- **Many-to-Many**:
  - Prefer a repeated field of resource names (e.g., `repeated string authors`).
  - If more metadata is needed for the relationship, use a sub-resource with two one-to-many associations.
- **Consistency**: Resource reference fields must use the same resource name format as the target resource.

**Incorrect (Ambiguous Hierarchy):**

```proto
message Book {
  option (google.api.resource) = {
    type: "library.googleapis.com/Book"
    // Bad: Trying to have two parents in the pattern
    pattern: "publishers/{publisher}/authors/{author}/books/{book}"
  };
}
```

**Correct (Clear Parent and Associations):**

```proto
message Book {
  option (google.api.resource) = {
    type: "library.googleapis.com/Book"
    pattern: "publishers/{publisher}/books/{book}"
  };

  string name = 1;

  // Association with Author (not a parent)
  string author = 2 [(google.api.resource_reference) = {
    type: "library.googleapis.com/Author"
  }];

  // Many-to-many association using repeated field
  repeated string tags = 3;
}
```

Reference: [AIP-124: Resource association](https://google.aip.dev/124)
