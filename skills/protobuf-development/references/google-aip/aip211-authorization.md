---
title: Authorization Checks (AIP-211)
impact: HIGH
impactDescription: Ensures secure API design by checking permissions before existence and avoiding resource leaking.
tags: protobuf, aip, security, authorization, error-handling
---

## Authorization Checks (AIP-211)

**Impact: HIGH**

Services must check authorization before validating any other part of the request. This ensures security and prevents leaking information about resource existence to unauthorized users.

**Implementation Rules:**

- **Order**: Check authorization _first_, before any semantic validation or existence checks.
- **Error Code**: Return `PERMISSION_DENIED` if the user lacks the required permission.
- **Message**: Use a standard message: `"Permission '{p}' denied on resource '{r}' (or it might not exist)."`
- **Existence Leaking**: By returning `PERMISSION_DENIED` regardless of whether the resource exists, you avoid revealing that a resource persists to unauthorized users.
- **Parent Check**: If it's impossible to check authorization for a missing resource, check authorization on the parent collection instead. Return `NOT_FOUND` only if the user has permission to read/list that collection but the resource is missing.

Reference: [AIP-211: Authorization checks](https://google.aip.dev/211)
