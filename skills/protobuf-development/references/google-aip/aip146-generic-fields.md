---
title: Generic Fields (AIP-146)
impact: MEDIUM
impactDescription: Outlines the best approaches for polymorphic or free-form data, prioritizing type safety.
tags: protobuf, aip, polymophism, oneof, struct, any
---

## Generic Fields (AIP-146)

**Impact: MEDIUM**

While specific schemas are preferred, generic fields are sometimes necessary. Choosing the "least generic" approach maintains as much type safety as possible.

**Implementation Rules:**

- **Prefer Oneof**: Use `oneof` for a fixed set of types. It provides the best type safety and semantic clarity.
- **Maps**: Use `map<string, T>` for multiple values of the _same_ type with user-determined keys.
- **google.protobuf.Struct**: Use for arbitrary nested JSON where the schema is unknown or user-provided.
- **google.protobuf.Any**: Avoid unless other options are infeasible. It requires manual deserialization and type registration.
- **Evolution**: Adding fields to a `oneof` is safe; moving fields in or out of a `oneof` is a breaking change.

**Incorrect (Overly Generic):**

```proto
message Configuration {
  // Bad: Forces every consumer to handle manual deserialization
  google.protobuf.Any settings = 1;
}
```

**Correct (Standard Generic Fields):**

```proto
import "google/protobuf/struct.proto";

message Configuration {
  oneof settings {
    // Good: specific types preferred
    DatabaseSettings database = 1;
    CacheSettings cache = 2;
    // Good: fallback for arbitrary user data
    google.protobuf.Struct custom_config = 3;
  }
}
```

Reference: [AIP-146: Generic fields](https://google.aip.dev/146)
