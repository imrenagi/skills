---
title: Naming Conventions (AIP-190)
impact: HIGH
impactDescription: Standardizes terminology and casing for all API elements to ensure a consistent developer experience.
tags: protobuf, aip, naming, terminology, casing
---

## Naming Conventions (AIP-190)

**Impact: HIGH**

All names in an API should be straightforward, intuitive, and consistent, using correct American English and standard casing.

**Implementation Rules:**

- **Language**: Use American English (e.g., `color`, not `colour`; `license`, not `licence`).
- **Casing**:
  - `UpperCamelCase`: Interfaces (Services), Resources, Messages, Enums.
  - `snake_case`: Fields (AIP-140).
- **Terminology**:
  - Prefer common, simple words (e.g., `delete` over `erase`, `stop` over `terminate`).
  - Avoid overly general names like `info`, `data`, `service`, `instance` without context.
- **Interfaces**: Name after a noun (e.g., `service Library`). Only use `Api` or `Service` suffixes if there's a conflict.
- **Methods**: Use `VerbNoun` (e.g., `ListBooks`).
- **Messages**:
  - Keep names short and concise.
  - Avoid prepositions like "With" or "For" in message names (use fields instead).
  - Request/Response naming (AIP-131-AIP-136).

**Correct (Naming Example):**

```proto
service Library {
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse);
}

message Book {
  string title = 1;
}
```

Reference: [AIP-190: Naming conventions](https://google.aip.dev/190)
