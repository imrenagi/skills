---
title: Resource States (AIP-216)
impact: MEDIUM
impactDescription: Standardizes lifecycle state documentation using enums and custom transition methods.
tags: protobuf, aip, state, lifecycle, enum
---

## Resource States (AIP-216)

**Impact: MEDIUM**

Resources representing long-lived objects (VMs, jobs) should communicate their lifecycle position using a `State` enum.

**Implementation Rules:**

- **Naming**: Use an enum named `State` (nested in the resource) and a field named `state`.
- **Terminology**:
  - `ACTIVE`: Preferred over "ready" or "available".
  - Past Participles (`-ED`): For terminal/resting states (e.g., `SUCCEEDED`, `FAILED`, `DELETED`).
  - Present Participles (`-ING`): For temporary/transient states (e.g., `CREATING`, `RUNNING`).
- **Behavior**:
  - The `state` field must be `OUTPUT_ONLY`.
  - State changes should be initiated via custom methods (e.g., `rpc StartBook`), not direct updates to the `state` field.
- **Unspecified**: The 0 value must be `STATE_UNSPECIFIED = 0`. This value should not be used in practice.
- **Custom Methods**:
  - Name: `Verb + Resource` (e.g., `rpc PublishBook`).
  - HTTP: `POST` with `:` custom verb (e.g., `:publish`).
  - Return: The resource itself (or LRO resolving to the resource).

**Correct (State Enum):**

```proto
message Book {
  enum State {
    STATE_UNSPECIFIED = 0;
    DRAFT = 1;
    ACTIVE = 2;
    DELETED = 3;
  }
  State state = 1 [(google.api.field_behavior) = OUTPUT_ONLY];
}
```

Reference: [AIP-216: States](https://google.aip.dev/216)
