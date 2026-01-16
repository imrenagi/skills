---
title: Precedent (AIP-200)
impact: LOW
impactDescription: A mechanism for documenting intentional or historic violations of API standards to prevent them from becoming "precedent" for new designs.
tags: protobuf, aip, process, precedent
---

## Precedent (AIP-200)

**Impact: LOW**

API standards evolve over time. Existing APIs may violate newer guidance due to historic reasons, performance needs, or consistency with legacy systems. Such violations must be documented so they aren't copied into new APIs.

**Implementation Rules:**

- **Documentation**: If an API violates a standard, it **must** include an internal comment linking to `aip.dev/not-precedent`.
- **Justification**: The comment must explain what is being violated and why (e.g., local consistency with existing fields, adherence to an external spec like OAuth, or historic legacy).
- **Local Consistency**: If an entire service uses a non-standard pattern (e.g., `creation_time` instead of `create_time`), new resources in _that specific service_ should follow the local pattern but still document it as non-precedent-setting.

**Example:**

```proto
message DailyMaintenanceWindow {
  // Time within the maintenance window to start the maintenance operations.
  // (-- aip.dev/not-precedent: This was designed for consistency with crontab.
  //     Ordinarily, this type should be `google.type.TimeOfDay`. --)
  string start_time = 2;
}
```

Reference: [AIP-200: Precedent](https://google.aip.dev/200)
