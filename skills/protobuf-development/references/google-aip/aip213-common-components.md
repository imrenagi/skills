---
title: Common Components (AIP-213)
impact: HIGH
impactDescription: Defines the standard library of shared Protobuf types to avoid redundant definitions and ensure interoperability.
tags: protobuf, aip, components, types
---

## Common Components (AIP-213)

**Impact: HIGH**

To maintain a consistent developer experience, APIs must use the standard "common component" packages for generic concepts instead of defining their own.

**Global Common Components (Approved Imports):**

You may safely import and use types from these packages:

- `google.protobuf.*`: (`Duration`, `Timestamp`, `Struct`, `Empty`, `FieldMask`, etc.)
- `google.type.*`: (`Date`, `Money`, `PostalAddress`, `LatLng`, `Color`, `TimeOfDay`, etc.)
- `google.rpc.*`: (`Status`, `ErrorInfo`, etc.)
- `google.api.*`: (Annotations like `http`, `resource`, `field_behavior`)
- `google.longrunning.Operation`: For long-running tasks.

**Implementation Rules:**

- **No Redefinition**: Do not create your own `Date` or `Money` messages if a global common component exists.
- **Organization-Specific Types**: If shared concepts are needed across your organization but aren't global, they should be in a `.type` package (e.g., `imrenagi.geo.type`).
- **Stability**: Fields must not be removed from common components. New fields should be added with caution.

**Example:**

```proto
import "google/protobuf/timestamp.proto";
import "google/type/money.proto";

message Order {
  string name = 1;
  google.type.Money total_amount = 2;
  google.protobuf.Timestamp create_time = 3;
}
```

Reference: [AIP-213: Common components](https://google.aip.dev/213)
