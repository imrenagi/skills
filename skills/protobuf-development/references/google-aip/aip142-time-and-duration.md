---
title: Time and Duration (AIP-142)
impact: HIGH
impactDescription: Standardizes the representation of timestamps and time spans using common components.
tags: protobuf, aip, time, duration, timestamp
---

## Time and Duration (AIP-142)

**Impact: HIGH**

Time representation is complex. Using standard components like `google.protobuf.Timestamp` and `google.protobuf.Duration` ensures that tools and libraries can handle time correctly across different languages and platforms.

**Implementation Rules:**

- **Absolute Time**: Use `google.protobuf.Timestamp`. Names must end in `_time` (singular) or `_times` (repeated).
- **Activity Timestamps**: Use the imperative form of the verb (e.g., `create_time`, `publish_time`). Do not use past tense (`created_time`).
- **Durations**: Use `google.protobuf.Duration`.
- **Time Offsets**: For relative segments (e.g., in a video stream), use `google.protobuf.Duration` and the suffix `_offset` (e.g., `start_offset`).
- **Civil Dates/Times**:
  - Date: `google.type.Date` (ending in `_date`).
  - Time of Day: `google.type.TimeOfDay` (ending in `_time`).
  - Civil Timestamp (Date + Time): `google.type.DateTime` (ending in `_time`).
- **Legacy Compatibility**: If you must use integers, include the unit as a final suffix (e.g., `send_time_millis`).

**Incorrect (Non-standard Time):**

```proto
message Flight {
  // Bad: past tense
  google.protobuf.Timestamp created_time = 1;
  // Bad: ambiguous unit and type
  int64 flight_length = 2;
}
```

**Correct (Standard Time):**

```proto
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";

message Flight {
  // Good: imperative form
  google.protobuf.Timestamp create_time = 1;
  // Good: distinct type for span
  google.protobuf.Duration flight_duration = 2;
}
```

Reference: [AIP-142: Time and duration](https://google.aip.dev/142)
