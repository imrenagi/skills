---
title: Automatic Retry Configuration (AIP-194)
impact: MEDIUM
impactDescription: Defines which error codes are safe for automated retries by client libraries.
tags: protobuf, aip, retry, errors, resilience
---

## Automatic Retry Configuration (AIP-194)

**Impact: MEDIUM**

Clients should automatically retry requests that are safe (unary, non-transactional, idempotent-like) when transient failures occur.

**Implementation Rules:**

- **Retryable Codes**:
  - `UNAVAILABLE`: Always retry (transient network issue).
- **Non-Retryable Codes**:
  - `INVALID_ARGUMENT`, `NOT_FOUND`, `ALREADY_EXISTS`, `PERMISSION_DENIED`, `UNAUTHENTICATED`: Retrying won't change the outcome without user action.
  - `DEADLINE_EXCEEDED`, `CANCELLED`: Honor the application's boundaries.
  - `DATA_LOSS`: Surface immediately.
- **Generally Non-Retryable**:
  - `RESOURCE_EXHAUSTED`: May involve quota/billing; wait or seek user intent.
  - `INTERNAL`, `UNKNOWN`: Surface to user; likely bugs.
  - `ABORTED`: Retry at a higher level (e.g., the whole transaction).

**Best Practice**: Ensure your methods are idempotent if they are intended to be retried automatically.

Reference: [AIP-194: Automatic retry configuration](https://google.aip.dev/194)
