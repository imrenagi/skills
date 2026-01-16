---
title: Documentation (AIP-192)
impact: MEDIUM
impactDescription: Ensures API comments are clear, complete, and formatted correctly for documentation generation.
tags: protobuf, aip, documentation, comments
---

## Documentation (AIP-192)

**Impact: MEDIUM**

Documentation is often the only resource users have. Protobuf comments must be comprehensive and follow specific formatting rules.

**Implementation Rules:**

- **Requirement**: Public comments _must_ be included for every component (service, method, message, field, enum, enum value).
- **Style**:
  - Use grammatically correct American English.
  - The first sentence should omit the subject and use third-person present tense (e.g., "Creates a book...").
- **Formatting**:
  - Use CommonMark (Markdown). No raw HTML, headings, or tables inside proto comments.
  - Use `code font` for field/method names and literals like `true`.
- **Content**: Describe what it is, how to use it, units, ranges, default values, and side effects.
- **Cross-references**: Link to other types using `[Book][google.example.v1.Book]`.
- **Deprecations**: Use `deprecated = true` and start the comment with "Deprecated: " followed by the reason or alternative.

**Correct (Standard Comment):**

```proto
// Creates a book under the given publisher.
//
// The parent publisher's name is required. If `validate_only` is true,
// the book is validated but not actually created.
rpc CreateBook(CreateBookRequest) returns (Book) { ... }
```

Reference: [AIP-192: Documentation](https://google.aip.dev/192)
