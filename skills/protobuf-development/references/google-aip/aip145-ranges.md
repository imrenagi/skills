---
title: Ranges (AIP-145)
impact: MEDIUM
impactDescription: Standardizes how to represent intervals of values, prioritizing half-closed intervals.
tags: protobuf, aip, ranges, interval
---

## Ranges (AIP-145)

**Impact: MEDIUM**

Consistent range representation prevents confusion about whether end values are included and makes it easier to reason about continuous data.

**Implementation Rules:**

- **Standard Naming**: Use two fields with `start_` and `end_` prefixes (e.g., `start_page`, `end_page`).
- **Half-Closed Intervals**: Use inclusive start and exclusive end values: `[start, end)`.
- **Timestamp Intervals**: Use `google.type.Interval` for ranges between two timestamps.
- **Exceptions (Closed Intervals)**: For concepts with strong colloquial precedent for inclusive ends (like dates or days of the week), use `first_` and `last_` prefixes (e.g., `first_page`, `last_page`).
- **Nesting**: Avoid `google.type.Interval` if the message itself is inherently an interval and extra nesting would be cumbersome.

**Incorrect (Ambiguous End):**

```proto
message Chapter {
  // Bad: Is end_page inclusive or exclusive?
  int32 start_page = 1;
  int32 end_page = 2;
}
```

**Correct (Standard Range):**

```proto
message Chapter {
  // Good: Inclusive start, exclusive end [start, end)
  int32 start_page = 1;
  int32 end_page = 2;
}

message Conference {
  // Good: Colloquial inclusive range [first, last]
  google.type.Date first_date = 1;
  google.type.Date last_date = 2;
}
```

Reference: [AIP-145: Ranges](https://google.aip.dev/145)
