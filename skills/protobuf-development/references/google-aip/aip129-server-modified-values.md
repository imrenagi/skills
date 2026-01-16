---
title: Server-Modified Values and Defaults (AIP-129)
impact: MEDIUM
impactDescription: Standardizes how to handle and document fields that the server might change or default.
tags: protobuf, aip, server-modified, design
---

## Server-Modified Values and Defaults (AIP-129)

**Impact: MEDIUM**

When servers modify user input or provide defaults, it can confuse declarative clients. AIP-129 provides a standard way to handle effective values and normalizations.

**Implementation Rules:**

- **Single Owner**: Fields must have a single owner (either client or server). Server-owned fields must be `OUTPUT_ONLY`.
- **Effective Values**: If a field can be either user-specified or system-generated (like an IP address or UUID):
  - Provide a mutable field for the user to set (e.g., `ip_address`).
  - Provide an `OUTPUT_ONLY` field for the system-decided value, prefixed with `effective_` (e.g., `effective_ip_address`).
- **Input Persistence**: The value returned by the service for a user-specified field must match what was provided in the request, with the exception of allowed normalizations.
- **Normalizations**: Allowed normalizations (e.g., lowercase email, UUID format) must be documented using the `google.api.field_info` annotation.
- **Naming**: Always use the `effective_` prefix for system-resolved values of user-provided fields.

**Incorrect (Ambiguous Ownership):**

```proto
message VM {
  // Bad: Server might change this value to a normalized version or a default.
  // The client doesn't know if their change was accepted or if '127.0.0.1'
  // was changed to '127.0.0.1/32' by the server.
  string ip_address = 1;
}
```

**Correct (Clear Ownership and Effective Values):**

```proto
import "google/api/field_behavior.proto";

message VM {
  // User-specified choice.
  string ip_address = 1;

  // The actual IP address assigned by the system.
  string effective_ip_address = 2 [(google.api.field_behavior) = OUTPUT_ONLY];
}
```

Reference: [AIP-129: Server-modified values and defaults](https://google.aip.dev/129)
