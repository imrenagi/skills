---
title: Unicode (AIP-210)
impact: MEDIUM
impactDescription: Standardizes character definitions, length limits, and normalization for string fields.
tags: protobuf, aip, unicode, strings, internalization
---

## Unicode (AIP-210)

**Impact: MEDIUM**

APIs must be consistent in how they handle string values, encodings, and length limits to avoid ambiguity across different languages and platforms.

**Implementation Rules:**

- **Character Definition**: "Characters" must always be defined as **Unicode code points**.
- **Length Units**: All string field length limits in documentation and enforcement **must** be measured in Unicode code points (not bytes).
- **Unique Identifiers**:
  - Should be limited to ASCII: `[a-zA-Z][a-zA-Z0-9_-]*`.
  - Should not start with a number (reserved for system-generated IDs).
  - Maximum length should generally be 64 characters.
- **Normalization**: All Unicode values **should** be stored in **Normalization Form C (NFC)**.
- **Uniqueness**: Uniqueness checks for identifiers **must** be performed after normalizing to NFC.

**Example (Documentation):**

```proto
message User {
  // The person's display name.
  // Maximum length is 120 characters (Unicode code points).
  string display_name = 1;

  // The unique identifier for the user.
  // Must match [a-z]([a-z0-9-]{0,61}[a-z0-9])?.
  string user_id = 2;
}
```

Reference: [AIP-210: Unicode](https://google.aip.dev/210)
