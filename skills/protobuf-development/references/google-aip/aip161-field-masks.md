---
title: Field Masks (AIP-161)
impact: HIGH
impactDescription: Provides a deep dive into the syntax and behavior of field masks for partial updates and reads.
tags: protobuf, aip, field-mask, partial-update
---

## Field Masks (AIP-161)

**Impact: HIGH**

Field masks are the primary mechanism for supporting partial updates and ensuring that clients don't accidentally overwrite data they didn't intend to modify.

**Implementation Rules:**

- **Type**: Must use `google.protobuf.FieldMask`.
- **Relativity**: Masks must always be relative to the resource being updated (e.g., `title`, not `book.title`).
- **Traversal**: Supports the `.` character for reaching fields in nested messages (e.g., `author.given_name`).
- **Maps**: Supports keys via `.` (e.g., `reviews.smith`). If keys have special characters, use backticks (e.g., `reviews.`John Smith``).
- **Wildcards**: `*` can be used to indicate all subfields in a collection (e.g., `authors.*.name`).
- **Output Only Fields**: Services must ignore `OUTPUT_ONLY` fields included in a mask.
- **Errors**: Return `INVALID_ARGUMENT` if a user attempts to access a repeated field by index (e.g., `authors.0`).

**Correct (Update with Mask):**

```proto
import "google/protobuf/field_mask.proto";

message UpdateBookRequest {
  Book book = 1 [(google.api.field_behavior) = REQUIRED];

  // Fields in mask are relative to 'book'.
  // Example: paths: ["title", "author.family_name"]
  google.protobuf.FieldMask update_mask = 2;
}
```

Reference: [AIP-161: Field masks](https://google.aip.dev/161)
