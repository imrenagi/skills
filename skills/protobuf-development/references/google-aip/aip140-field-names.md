---
title: Field Names (AIP-140)
impact: HIGH
impactDescription: Ensures intuitive and consistent naming of fields across APIs and generated client code.
tags: protobuf, aip, field-names, naming
---

## Field Names (AIP-140)

**Impact: HIGH**

Simple, intuitive, and consistent field names make APIs easier to understand and use. Since field names appear directly in generated client surfaces, their quality significantly impacts the developer experience.

**Implementation Rules:**

- **Language**: Use correct American English.
- **Case**: Must use `lower_snake_case` in Protobuf files.
- **Pluralization**:
  - Repeated fields must be plural (e.g., `books`).
  - Singular fields must be singular (e.g., `book`).
- **No Prepositions**: Avoid "with", "for", "at", "by", etc. Use `error_reason` instead of `reason_for_error`.
- **Abbreviations**: Use well-known abbreviations like `config`, `id`, `info`, `spec`, and `stats`.
- **Adjectives**: Place the adjective before the noun (e.g., `collected_items`, not `items_collected`).
- **Nouns only**: Field names must be nouns, not verbs. Use `disabled` instead of `disable`.
- **Booleans**: Omit the "is" prefix (e.g., `required` instead of `is_required`), unless it's a reserved word.
- **Binary Data**: Use `bytes` for binary data and let the JSON mapping handle base64 encoding.
- **URIs**: Use `uri` for arbitrary URIs and `url` only if it's strictly a URL.
- **Display Names**: Use `display_name` for human-readable labels and `title` for formal names (like book titles). Neither should be unique.

**Incorrect (Non-standard Naming):**

```proto
message User {
  // Bad: starts with number, uses 'is' prefix, and is a verb
  bool is_1st_login = 1;
  // Bad: unnecessary preposition
  string reason_for_ban = 2;
  // Bad: plural message but singular field
  repeated string tag = 3;
}
```

**Correct (Standard Naming):**

```proto
message User {
  bool first_login = 1;
  string ban_reason = 2;
  repeated string tags = 3;
}
```

Reference: [AIP-140: Field names](https://google.aip.dev/140)
