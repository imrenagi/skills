---
title: File and Directory Structure (AIP-191)
impact: MEDIUM
impactDescription: Standardizes how proto files are organized, named, and packaged.
tags: protobuf, aip, structure, packaging
---

## File and Directory Structure (AIP-191)

**Impact: MEDIUM**

A consistent structure makes it easier for developers to navigate service definitions and ensures smooth generation of client libraries.

**Implementation Rules:**

- **Syntax**: Must use `proto3`.
- **Package**: Each API must be in a single package ending in a version (e.g., `google.example.v1`).
- **Directory**: Must match the package (e.g., `google/example/v1/`).
- **File Names**: Use `snake_case`. Do not use the version as a filename (e.g., `v1.proto` is prohibited).
- **File Layout**:
  - Copyright/License.
  - `syntax`, then `package`.
  - `import`s (alphabetical).
  - File-level `option`s.
  - `service` definitions (methods grouped by resource).
  - Resource `message` definitions (parent before child).
  - Request/Response `message` definitions (in method order).
  - Enums and other messages.
- **Java Options**: `java_multiple_files = true` and `java_outer_classname` must be set.

**Correct (Standard Header):**

```proto
syntax = "proto3";

package google.library.v1;

import "google/api/annotations.proto";
import "google/api/client.proto";

option java_multiple_files = true;
option java_outer_classname = "LibraryProto";
option java_package = "com.google.library.v1";
option go_package = "google.golang.org/genproto/googleapis/library/v1;library";

service Library { ... }
```

Reference: [AIP-191: File and directory structure](https://google.aip.dev/191)
