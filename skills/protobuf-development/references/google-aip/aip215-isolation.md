---
title: API Isolation and Dependencies (AIP-215)
impact: HIGH
impactDescription: Prevents complex dependency chains and versioning problems by ensuring APIs are self-contained.
tags: protobuf, aip, dependencies, versioning, isolation
---

## API Isolation and Dependencies (AIP-215)

**Impact: HIGH**

APIs must be self-contained to avoid versioning and dependency problems. This means an API should not depend on protos defined by other APIs (except for a specific set of common components).

**Implementation Rules:**

- **Self-Contained**: All protos specific to an API must be within a package with a major version (e.g., `google.library.v1`).
- **Resource References**: References to resources in _other_ APIs must use resource names (strings) and `(google.api.resource_reference)` annotations, NOT the resource messages themselves.
- **Duplication**: If two versions of an API (v1 and v2) use the same proto, that proto must be duplicated in each version. Do not create "API-specific common packages" shared across versions.
- **Common Components**: APIs may only import from generic, globally available common component packages:
  - `google.api.*`
  - `google.protobuf.*` (Duration, Timestamp, Struct, etc.)
  - `google.rpc.*` (Status, Code)
  - `google.type.*` (Money, Date, LatLng, etc.)
  - `google.longrunning.Operation`

**Incorrect (Cross-API Message Dependency):**

```proto
import "google/cloud/movies/v1/movie.proto";

message Recommendation {
  // INCORRECT: Depends on a message from another API
  google.cloud.movies.v1.Movie movie = 1;
}
```

**Correct (Decoupled Reference):**

```proto
message Recommendation {
  // CORRECT: Depends only on the resource name string
  string movie = 1 [(google.api.resource_reference).type = "movies.googleapis.com/Movie"];
}
```

Reference: [AIP-215: API-specific protos](https://google.aip.dev/215)
