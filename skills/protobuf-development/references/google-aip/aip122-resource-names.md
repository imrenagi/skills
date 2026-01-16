---
title: Resource Names (AIP-122)
impact: HIGH
impactDescription: Standardizes how resources are identified and referenced across the API.
tags: protobuf, aip, resource-names, design
---

## Resource Names (AIP-122)

**Impact: HIGH**

Resource names are the unique identifiers for resources in an API. Standardizing their format ensures consistency, predictability, and ease of use across different services and client libraries.

**Implementation Rules:**

- **Uniqueness**: All resource names must be unique within the API.
- **Format**: Follow the URI path schema without a leading slash (e.g., `publishers/123/books/les-miserables`).
- **Separators**: Use the `/` character to separate individual segments.
- **Characters**: Use only characters available in DNS names (RFC-1123). Resource IDs should not use uppercase letters.
- **`name` Field**: Every resource must expose a `name` field of type `string` containing its resource name.
- **Collection Identifiers**: Must be the plural form of the resource noun (e.g., `publishers` for `Book` resources owned by a publisher).
- **Nested Collections**: Shorten child collection names if they are redundant with the parent (e.g., `users/123/events/456` instead of `users/123/userEvents/456`).
- **Resource ID Segments**: Document allowed formats for user-specified IDs. Conforming to RFC-1034 (lowercase, numbers, hyphens) is recommended.

**Incorrect (Inconsistent Naming):**

```proto
message Book {
  // Bad: Using 'id' instead of 'name'
  string id = 1;
  // Bad: Using a field named 'book_name' for display name
  string book_name = 2;
}

service LibraryService {
  // Bad: Non-standard path
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/book/{id}"
    };
  }
}
```

**Correct (Standard Resource Names):**

```proto
message Book {
  option (google.api.resource) = {
    type: "library.googleapis.com/Book"
    pattern: "publishers/{publisher}/books/{book}"
  };

  // The resource name of the book.
  // Format: publishers/{publisher}/books/{book}
  string name = 1 [(google.api.field_behavior) = IDENTIFIER];

  // Use display_name for the human-readable title if 'title' isn't used.
  string display_name = 2;
}

message GetBookRequest {
  // The name of the book to retrieve.
  // Format: publishers/{publisher}/books/{book}
  string name = 1 [
    (google.api.field_behavior) = REQUIRED,
    (google.api.resource_reference) = {
      type: "library.googleapis.com/Book"
    }];
}
```

Reference: [AIP-122: Resource names](https://google.aip.dev/122)
