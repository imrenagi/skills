---
title: Resource-Oriented Design (AIP-121)
impact: MEDIUM
impactDescription: Standardizes API structure around resources and collections for predictability and consistency.
tags: protobuf, aip, resource-oriented, design
---

## Resource-Oriented Design (AIP-121)

**Impact: MEDIUM**

Following Resource-Oriented Design principles (AIP-121) ensures that APIs are modeled consistently around "Resources" and "Collections". This pattern makes APIs more predictable, easier to learn, and simplifies the implementation of common patterns like pagination, filtering, and standard CRUD operations.

**Implementation Rules:**

- **Standard Methods**: Use standard methods where possible: `Get`, `List`, `Create`, `Update`, `Delete`.
- **Naming Conventions**:
  - **Resource Message**: Use a singular noun in PascalCase (e.g., `Book`).
  - **Request Message**: Named `<Method><Resource>Request` (e.g., `GetBookRequest`, `CreateBookRequest`).
  - **Response Message**:
    - For `Get`, `Create`, `Update`: Use the Resource message itself.
    - For `List`: Named `List<Resource>sResponse` (e.g., `ListBooksResponse`).
    - For `Delete`: Use `google.protobuf.Empty`.
- **Resource Identification**: Use a `name` field as the primary identifier. In many implementations, this follows the pattern `collections/{id}/subcollections/{sub_id}`.
- **Request/Response Objects**: Prefer dedicated messages for every RPC to allow for future field expansion without breaking changes.

**Incorrect (Ad-hoc API Design):**

```proto
service LibraryService {
  // Bad: Non-standard naming and structure
  rpc FetchBookData(BookQuery) returns (BookList);
  rpc RemoveBook(id_struct) returns (bool);
}

message BookQuery {
  string id = 1;
}

message BookList {
  repeated Book items = 1;
}
```

**Correct (Resource-Oriented Design):**

```proto
import "google/protobuf/empty.proto";

service LibraryService {
  rpc GetBook(GetBookRequest) returns (Book);
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse);
  rpc CreateBook(CreateBookRequest) returns (Book);
  rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty);
}

message Book {
  // Use 'name' as the unique identifier for the resource.
  string name = 1;
  string title = 2;
  string author = 3;
}

message GetBookRequest {
  // The name of the book to retrieve.
  // Format: publishers/{publisher}/books/{book}
  string name = 1;
}

message ListBooksRequest {
  // The parent resource to list books from.
  string parent = 1;
  int32 page_size = 2;
  string page_token = 3;
}

message ListBooksResponse {
  repeated Book books = 1;
  string next_page_token = 2;
}
```

Reference: [AIP-121: Resource-oriented design](https://google.aip.dev/121)
