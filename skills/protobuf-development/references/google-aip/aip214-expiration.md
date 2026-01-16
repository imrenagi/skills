---
title: Resource Expiration (AIP-214)
impact: MEDIUM
impactDescription: Standardizes how to handle absolute and relative expiration (TTL) for resources.
tags: protobuf, aip, ttl, expiration, time
---

## Resource Expiration (AIP-214)

**Impact: MEDIUM**

APIs allowing resources to expire should support both absolute timestamps and relative durations (TTL).

**Implementation Rules:**

- **Absolute**: Use a `google.protobuf.Timestamp` field called `expire_time`.
- **Relative (TTL)**: Support a `google.protobuf.Duration` field called `ttl`.
- **Nesting**: Place both fields inside a `oneof expiration` (or `{something}_expiration`).
- **Input/Output**:
  - `ttl` must be `INPUT_ONLY`. It is used to calculate `expire_time`.
  - `expire_time` is always provided on output.
- **Output Only**: APIs should always return the calculated `expire_time` and leave `ttl` blank on retrieval.
- **Rationale**: Storing `expire_time` is better than storing `ttl` because `expire_time` is static, whereas `ttl` would need to be updated constantly like a countdown clock.

**Correct (Expiration Pattern):**

```proto
message ExpiringResource {
  oneof expiration {
    // The time when this resource expires.
    google.protobuf.Timestamp expire_time = 1;

    // Input only. The time-to-live for this resource.
    google.protobuf.Duration ttl = 2 [(google.api.field_behavior) = INPUT_ONLY];
  }
}
```

Reference: [AIP-214: Resource expiration](https://google.aip.dev/214)
