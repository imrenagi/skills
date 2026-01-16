---
title: Sensitive Fields (AIP-147)
impact: MEDIUM
impactDescription: Standardizes how to handle sensitive data like private keys or secrets that should not be read back.
tags: protobuf, aip, security, sensitive-data, input-only
---

## Sensitive Fields (AIP-147)

**Impact: MEDIUM**

Sensitive data requires special handling to ensure it can be provided by the user but is not accidentally exposed in results or logs.

**Implementation Rules:**

- **Input-Only for Required Data**: If the sensitive data is required for the resource, use `(google.api.field_behavior) = INPUT_ONLY` and provide no output field.
- **Presence Check for Optional Data**: For optional secrets, use an `INPUT_ONLY` field for the secret and an `OUTPUT_ONLY bool {field}_set` field to indicate if it's populated.
- **Obfuscation**: Use an `obfuscated_` prefix (e.g., `obfuscated_email`) if you need to provide some context without exposing the full value.
- **Bytes for Keys**: Prefer `bytes` for raw key material.

**Incorrect (Exposing Secrets):**

```proto
message Webhook {
  // Bad: secret will be returned in Get/List calls!
  string shared_secret = 1;
}
```

**Correct (Standard Sensitive Fields):**

```proto
import "google/api/field_behavior.proto";

message Webhook {
  string name = 1;

  // Good: user can set it, but never read it back
  string shared_secret = 2 [(google.api.field_behavior) = INPUT_ONLY];

  // Good: informs the user that a secret exists
  bool shared_secret_set = 3 [(google.api.field_behavior) = OUTPUT_ONLY];
}
```

Reference: [AIP-147: Sensitive fields](https://google.aip.dev/147)
