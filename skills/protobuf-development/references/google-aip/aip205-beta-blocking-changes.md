---
title: Beta-blocking Changes (AIP-205)
impact: LOW
impactDescription: A process for documenting intentional departures from API standards during Alpha stages to ensure they are corrected before Beta.
tags: protobuf, aip, process, alpha, beta
---

## Beta-blocking Changes (AIP-205)

**Impact: LOW**

During the **Alpha** phase, APIs may deviate from standards to experiment or gather feedback. However, these issues must be addressed before promoting the API to **Beta**.

**Implementation Rules:**

- **Documentation**: Any field or method that intentionally violates a standard during Alpha **must** include an internal comment pointing to the Beta-blocking requirement.
- **Link**: Use the descriptive link `aip.dev/beta-blocker` in the comment.
- **Actionable Info**: The comment must explicitly state what change is required to reach compliance for the Beta launch.

**Example:**

```proto
message InputConfig {
  // Parameters for input.
  // (-- aip.dev/beta-blocker: Convert well-known parameters into explicit
  //     fields before the Beta launch. --)
  map<string, string> parameters = 1;
}
```

Reference: [AIP-205: Beta-blocking changes](https://google.aip.dev/205)
