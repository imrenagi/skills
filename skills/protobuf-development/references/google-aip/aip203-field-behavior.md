---
title: Field Behavior Documentation (AIP-203)
impact: HIGH
impactDescription: Mandatory annotations for every field in a request or resource to define its lifecycle and requirement status.
tags: protobuf, aip, field-behavior, required, optional, output-only
---

## Field Behavior Documentation (AIP-203)

**Impact: HIGH**

APIs must explicitly document how fields behave using the `google.api.field_behavior` annotation. This is critical for client library generation (SDKs, CLIs).

**Implementation Rules:**

- **Mandatory**: Every field on a message used in a request _must_ have an annotation.
- **Core Values**:
  - `REQUIRED`: Field must be present and set to a "truthy" value.
  - `OPTIONAL`: Field is not required. Default if omitted (but should be explicit).
  - `OUTPUT_ONLY`: Field is provided by the server. Input values are ignored.
- **Advanced Values**:
  - `IMMUTABLE`: Field cannot be changed after creation.
  - `IDENTIFIER`: Applied _only_ to the `name` field of a resource.
  - `INPUT_ONLY`: Provided in request, not included in response (rare).
  - `UNORDERED_LIST`: Repeated field where the server doesn't guarantee order.
- **Nesting**: Nested message behaviors are independent of their parents.
- **Compatibility**: Changing from `OPTIONAL` to `REQUIRED` is a breaking change.

**Correct (Annotated Resource):**

```proto
import "google/api/field_behavior.proto";

message Book {
  // Output only. The resource name of the book.
  string name = 1 [(google.api.field_behavior) = IDENTIFIER];

  // Required. The title of the book.
  string title = 2 [(google.api.field_behavior) = REQUIRED];

  // Optional. The description of the book.
  string description = 3 [(google.api.field_behavior) = OPTIONAL];

  // Output only. When the book was created.
  google.protobuf.Timestamp create_time = 4 [(google.api.field_behavior) = OUTPUT_ONLY];
}
```

Reference: [AIP-203: Field behavior documentation](https://google.aip.dev/203)
