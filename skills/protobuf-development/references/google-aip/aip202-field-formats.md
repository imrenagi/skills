---
title: Field Formats (AIP-202)
impact: LOW
impactDescription: Enriches primitive field schema with specific format markers like UUID4 or IPv4.
tags: protobuf, aip, fields, format, uuid, ip
---

## Field Formats (AIP-202)

**Impact: LOW**

The `google.api.field_info` extension allows developers to specify that a primitive field (usually `string`) follows a specific format, helping client libraries and tools.

**Implementation Rules:**

- **UUID4**: Use `(google.api.field_info).format = UUID4` for RFC 4122 compliance. Service may normalize to lowercase.
- **IPv4**: Use `(google.api.field_info).format = IPV4`. Service may condense by stripping leading zeros.
- **IPv6**: Use `(google.api.field_info).format = IPV6`. Service may normalize per RFC 5952 (lowercase, compressed zeros).
- **Comparison**: Servers and clients should use format-compliant libraries for comparison rather than primitive string comparison.
- **Compatibility**: Adding a format to an existing field is NOT backwards compatible unless the field always followed that format.

**Correct (UUID Format):**

```proto
import "google/api/field_info.proto";

message Request {
  // A unique ID identifying this request.
  string request_id = 1 [(google.api.field_info).format = UUID4];
}
```

Reference: [AIP-202: Fields](https://google.aip.dev/202)
