---
title: Request Identification (AIP-155)
impact: MEDIUM
impactDescription: Standardizes idempotency guarantees using customer-provided request identifiers.
tags: protobuf, aip, idempotency, request-id
---

## Request Identification (AIP-155)

**Impact: MEDIUM**

Request IDs allow clients to safely retry requests after network failures. The server uses the ID to detect duplicate requests and ensure they are only processed once.

**Implementation Rules:**

- **Request Field**: Add `string request_id` to the request message.
- **Location**: Must be a field on the request message, _never_ on the resource itself.
- **Idempotency Guarantee**: Providing a `request_id` must guarantee that subsequent duplicate requests return the same result as the first successful one.
- **Format**: Should be optional. Should support UUID4.
- **Annotation**: If using UUID4, annotate with `(google.api.field_info).format = UUID4`.
- **Timeframe**: Services choose a reasonable timeframe to honor request IDs (e.g., 24 hours).
- **Stale Successes**: If the historical response is no longer available, returning the current state of the resource is permissible.

**Incorrect (Resource containing Request ID):**

```proto
message Book {
  string name = 1;
  // Bad: Request ID shouldn't be part of the resource state!
  string request_id = 10;
}
```

**Correct (Standard Request ID):**

```proto
import "google/api/field_info.proto";

message CreateBookRequest {
  string parent = 1;
  Book book = 2;

  // Good: used for safe retries
  string request_id = 3 [(google.api.field_info).format = UUID4];
}
```

Reference: [AIP-155: Request identification](https://google.aip.dev/155)
