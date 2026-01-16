---
title: Standard Fields (AIP-148)
impact: HIGH
impactDescription: Defines common fields used across all APIs to ensure structural consistency.
tags: protobuf, aip, standard-fields, timestamps, names
---

## Standard Fields (AIP-148)

**Impact: HIGH**

Standardized fields allow users to apply knowledge from one API to another. They also enable common infrastructure to handle features like soft-delete or auditing automatically.

**Implementation Rules:**

- **Identification**:
  - `name`: Every resource must have a `string name` as the first field.
  - `uid`: A system-assigned UUID4, `OUTPUT_ONLY`.
- **Relationship**:
  - `parent`: Used in `List` and `Create` requests to refer to the parent resource.
- **Human-Readable Names**:
  - `display_name`: Mutable, non-unique, <= 63 chars.
  - `title`: Formal variant (e.g., company name).
  - `given_name` / `family_name`: Use these instead of `first_name` / `last_name`.
- **Audit Timestamps**:
  - `create_time`: `OUTPUT_ONLY`, when creation started or finished.
  - `update_time`: `OUTPUT_ONLY`, when the resource was last modified by a user.
- **Lifecycle Timestamps**:
  - `delete_time`: When soft-deleted.
  - `expire_time`: When the resource becomes invalid.
  - `purge_time`: When a soft-deleted resource will be permanently removed.
- **Metadata**:
  - `annotations`: `map<string, string>`, for small amounts of arbitrary client data.
- **Network**:
  - `ip_address`: Use `string` and the name `ip_address` or suffix `_ip_address`.

**Incorrect (Non-standard Standard Fields):**

```proto
message User {
  // Bad: wrong name and type for ID
  int64 id = 1;
  // Bad: past tense, non-standard component
  string created_at = 2;
  // Bad: culturally biased naming
  string first_name = 3;
}
```

**Correct (Standard Fields):**

```proto
import "google/api/field_behavior.proto";
import "google/protobuf/timestamp.proto";

message User {
  string name = 1 [(google.api.field_behavior) = IDENTIFIER];

  string uid = 2 [(google.api.field_behavior) = OUTPUT_ONLY];

  google.protobuf.Timestamp create_time = 3 [(google.api.field_behavior) = OUTPUT_ONLY];

  string given_name = 4;
  string family_name = 5;
}
```

Reference: [AIP-148: Standard fields](https://google.aip.dev/148)
