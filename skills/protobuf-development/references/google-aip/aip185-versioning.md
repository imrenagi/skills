---
title: API Versioning (AIP-185)
impact: HIGH
impactDescription: Standardizes how APIs are versioned using major versions and stability channels (Alpha, Beta).
tags: protobuf, aip, versioning, alpha, beta, stability
---

## API Versioning (AIP-185)

**Impact: HIGH**

Every API must have a major version number encoded in the protobuf package and URI path. Google APIs do not use minor or patch versions in the public interface (e.g., use `v1`, not `v1.1`).

**Implementation Rules:**

- **Major Versions**: Encoded at the end of the package (e.g., `google.example.v1`). Updated in-place for compatible changes.
- **Stability Channels**:
  - **Alpha**: `v1alpha`. For rapid iteration, no stability guarantees.
  - **Beta**: `v1beta`. Feature complete, subject to public testing, minimal breaking changes allowed with 180-day deprecation.
  - **Stable**: `v1`. No breaking changes.
- **Dependency**: A new major version must not depend on a previous major version of the same API.
- **Deprecation**: Deprecated elements must not graduate from alpha to beta, or beta to stable. Remove deprecated beta features after 180 days.

**Correct (Package Versioning):**

```proto
// Stable
package google.library.v1;

// Beta Channel
package google.library.v1beta;

// Release-based (Incremental) Beta
package google.library.v1beta1;
```

Reference: [AIP-185: API Versioning](https://google.aip.dev/185)
