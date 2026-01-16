---
title: Quantities (AIP-141)
impact: MEDIUM
impactDescription: Standardizes how to represent counts and measurements with clear units and naming conventions.
tags: protobuf, aip, quantities, units
---

## Quantities (AIP-141)

**Impact: MEDIUM**

Representing quantities consistently ensures that users understand the magnitude and unit of the values they are providing or receiving.

**Implementation Rules:**

- **Units in Suffix**: Fields with a clear unit must include the unit as a suffix, using generally accepted abbreviations (e.g., `distance_km`, `file_size_bytes`).
- **Counts**: Use the `_count` suffix (not `num_` prefix) for a number of items (e.g., `node_count`).
- **Unsigned Integers**: Must not use `uint32`, `uint64`, etc. Use signed integers (`int32`, `int64`).
- **Compound Units**: Separate unabbreviated units with underscores (e.g., `energy_kw_fortnights`). Do not separate metric prefixes from their base (e.g., `kwh`).
- **Inverse Units**: Use the word "per" to indicate inverse units (e.g., `speed_miles_per_hour`, `event_count_per_hour`).
- **Specialized Messages**: Use `google.type.Money` for prices and `google.protobuf.Duration` for time spans.

**Incorrect (Ambiguous Quantities):**

```proto
message Cluster {
  // Bad: Missing unit, uses num_ prefix, and unsigned int
  uint32 num_nodes = 1;
  // Bad: Pluralized abbreviation
  int32 distance_kms = 2;
}
```

**Correct (Standard Quantities):**

```proto
message Cluster {
  // Good: _count suffix and signed int
  int32 node_count = 1;
  // Good: accepted abbreviation, singular
  int32 distance_km = 2;
}
```

Reference: [AIP-141: Quantities](https://google.aip.dev/141)
