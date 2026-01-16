---
title: Enumerations (AIP-126)
impact: MEDIUM
impactDescription: Ensures consistency and forward-compatibility for discrete sets of values.
tags: protobuf, aip, enum, design
---

## Enumerations (AIP-126)

**Impact: MEDIUM**

Enumerations are used for fields that accept a discrete and limited set of values. Proper enum design prevents naming collisions and ensures that APIs can evolve without breaking client code.

**Implementation Rules:**

- **Naming**: Use `UPPER_SNAKE_CASE` for all enum values.
- **Zero Value**: The first value (0) must be `<ENUM_NAME>_UNSPECIFIED`.
  - Exception: Use `UNKNOWN` if it's more appropriate and clearly useful.
- **Nesting**: Nest enums within the message that uses them if they are not shared.
- **Prefixing**:
  - For nested enums, do not prefix non-zero values with the enum name (to avoid verbosity like `State.STATE_ACTIVE`).
  - For top-level enums, prefix values with the enum name to avoid collisions in languages like C++.
- **Evolution**: Document whether new values will be added in the future.
- **Alternatives**: Use `string` for frequently changing values or when a standard representation (like ISO codes) exists. Use `bool` only if no future flexibility is needed.

**Incorrect (Non-standard Enum):**

```proto
enum Status {
  // Bad: Missing UNSPECIFIED as 0
  ACTIVE = 1;
  INACTIVE = 2;
}

message User {
  // Bad: Enum name is too generic for top-level
  Status status = 1;
}
```

**Correct (Standard Enum):**

```proto
message Book {
  enum Format {
    FORMAT_UNSPECIFIED = 0;
    HARDBACK = 1;
    PAPERBACK = 2;
    EBOOK = 3;
  }

  Format format = 1;
}

// Top-level enum with prefixing
enum LibraryState {
  LIBRARY_STATE_UNSPECIFIED = 0;
  LIBRARY_STATE_OPEN = 1;
  LIBRARY_STATE_CLOSED = 2;
}
```

Reference: [AIP-126: Enumerations](https://google.aip.dev/126)
