---
title: Backwards Compatibility (AIP-180)
impact: HIGH
impactDescription: Essential for maintaining long-term API contracts and preventing client breakages.
tags: protobuf, aip, compatibility, versioning
---

## Backwards Compatibility (AIP-180)

**Impact: HIGH**

APIs are contracts. Once released, they must not be changed in ways that break existing clients (same major version).

**Implementation Rules:**

- **Adding Components**: Generally safe to add fields, enums, or methods.
  - New fields must not be `REQUIRED` if clients didn't specify them before.
  - Adding pagination after the fact is breaking (see AIP-158).
- **Removing/Renaming**: Prohibited. Renaming is "remove and add" and is a breaking change.
- **Type Changes**: Prohibited, even if wire-compatible, as they break generated code.
- **Resource Names**: Must not change. v2.0 must be able to address resources created in v1.0 using the same name format.
- **Defaults**: Default values must not change once established.
- **Semantic Changes**: Behavioral changes (e.g., changing IP format from v4 to v6) are breaking and require a new major version.
- **Oneofs**: Moving fields into or out of a `oneof` is a breaking change.

**Breaking (Incompatible):**

```proto
// v1
message Book {
  string title = 1;
}

// v2 (Breaking: changed field type)
message Book {
  int32 title = 1; // BROKEN
}
```

Reference: [AIP-180: Backwards compatibility](https://google.aip.dev/180)
