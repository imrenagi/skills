---
title: Standardized Codes (AIP-143)
impact: MEDIUM
impactDescription: Promotes the use of international standards for common concepts like languages, regions, and currency.
tags: protobuf, aip, standard-codes, localization
---

## Standardized Codes (AIP-143)

**Impact: MEDIUM**

Using international standards (ISO, IETF, etc.) for common concepts prevents ambiguity and eliminates the need for custom lookup tables.

**Implementation Rules:**

- **Prefer Codes over Enums**: Use `string` fields for standardized codes, even if the allowed set is small.
- **Documentation**: Must link to the standard being followed (e.g., Wikipedia or the official ISO page).
- **Naming**: Field names should end in `_code` or `_type`.
- **Case Sensitivity**: Accept input case-insensitively but always provide canonical case (e.g., `en-GB`).
- **Standard Mappings**:
  - **Content Types**: Use IANA media types. Name the field `mime_type`.
  - **Countries/Regions**: Use Unicode CLDR/ISO-3166 region codes. Name the field `region_code` (not `country_code`).
  - **Currency**: Use ISO-4217 currency codes. Name the field `currency_code` (or use `google.type.Money`).
  - **Language**: Use IETF BCP-47 language codes. Name the field `language_code`.
  - **Time Zones**: Use IANA TZ codes. Name the field `time_zone`. Use `utc_offset` for ISO-8601 offsets.

**Incorrect (Custom Codes or Enums):**

```proto
message User {
  // Bad: Enum for a standard that might expand
  enum Language {
    ENGLISH = 0;
    SPANISH = 1;
  }
  Language language = 1;

  // Bad: Ambiguous name
  string country = 2;
}
```

**Correct (Standard Codes):**

```proto
message User {
  // The IETF BCP-47 language code.
  // https://en.wikipedia.org/wiki/IETF_language_tag
  string language_code = 1;

  // The Unicode CLDR region code.
  // https://cldr.unicode.org/
  string region_code = 2;
}
```

Reference: [AIP-143: Standardized codes](https://google.aip.dev/143)
