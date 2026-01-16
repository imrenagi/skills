---
title: Stability Levels (AIP-181)
impact: MEDIUM
impactDescription: Defines the expectations for API stability across Alpha, Beta, and Stable stages.
tags: protobuf, aip, stability, lifecycle, versioning
---

## Stability Levels (AIP-181)

**Impact: MEDIUM**

Stability levels communicate to users how much they can rely on an API component and what the process for change is.

**Implementation Rules:**

- **Alpha**:
  - Rapid iteration with a small, known set of users.
  - Breaking changes are expected and allowed without notice.
  - No stability guarantee.
- **Beta**:
  - Feature complete and ready for public testing.
  - Breaking changes should be minimal and require a deprecation period.
  - Intended to be time-boxed (rule of thumb: 90 days) before moving to Stable.
- **Stable**:
  - Fully supported for the lifetime of the major version.
  - No breaking changes permitted (except for extreme cases like security).
  - Turndown requires a formal process and a long deprecation period.

**Guidance**:

- Create a new major version (v1 -> v2) when breaking changes are unavoidable in a Stable API.

Reference: [AIP-181: Stability levels](https://google.aip.dev/181)
