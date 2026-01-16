---
title: Filtering (AIP-160)
impact: MEDIUM
impactDescription: Standardizes the syntax for searching and filtering collections.
tags: protobuf, aip, filtering, search
---

## Filtering (AIP-160)

**Impact: MEDIUM**

Using a standardized string-based filter syntax allows APIs to evolve their search capabilities without breaking clients or requiring UI changes for every new field.

**Implementation Rules:**

- **Field**: Use exactly one field: `string filter`.
- **Syntax**:
  - **Literals**: Bare values match globally (e.g., `Victor`).
  - **Logical**: `AND` and `OR` (higher precedence for `OR`).
  - **Negation**: `NOT` or `-`.
  - **Comparison**: `=`, `!=`, `<`, `>`, `<=`, `>=`. Field must be on the left.
  - **Traversal**: `.` operator for nested fields (e.g., `author.name`).
  - **Has Operator**: `:` operator for membership in maps or repeated fields.
- **Data Types**:
  - Booleans: `true`, `false`.
  - Durations: `20s`.
  - Timestamps: RFC-3339 strings.
- **Validation**: Return `INVALID_ARGUMENT` for non-compliant or schematically invalid filter strings.

**Correct (Filter Usage):**

```proto
message ListBooksRequest {
  string parent = 1;
  // Syntax: "authors:Hugo AND publish_time > 1900-01-01T00:00:00Z"
  string filter = 2;
}
```

Reference: [AIP-160: Filtering](https://google.aip.dev/160)
