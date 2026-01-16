---
title: External Software Dependencies (AIP-182)
impact: LOW
impactDescription: Guidelines for managing the lifecycle of third-party software exposed via an API.
tags: protobuf, aip, lifecycle, dependencies, versioning
---

## External Software Dependencies (AIP-182)

**Impact: LOW**

This rule covers how APIs should handle versions of external software (e.g., database engines, OS images, runtimes) that they expose to users.

**Implementation Rules:**

- **LTS Support**: Services should support current Long-Term Support (LTS) versions of the software.
- **End-of-Life (EOL)**:
  - Do not indefinitely allow creation of resources with EOL versions.
  - Removing support for EOL versions is _not_ a breaking change for the API service, but requires user notification.
- **Resources using EOL**: Avoid proactively deleting user resources running EOL software unless there's a critical security risk.
- **Maintenance**: If the service continues to support EOL software for business reasons, the service takes on maintenance/patching responsibility.

Reference: [AIP-182: External software dependencies](https://google.aip.dev/182)
