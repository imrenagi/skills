---
title: Errors (AIP-193)
impact: HIGH
impactDescription: Standardizes error responses using `google.rpc.Status`, canonical codes, and rich error details.
tags: protobuf, aip, errors, status, grpc
---

## Errors (AIP-193)

**Impact: HIGH**

Standardized errors allow clients to implement centralized, robust error handling logic.

**Implementation Rules:**

- **Structure**: Must return `google.rpc.Status` and use the canonical `google.rpc.Code`.
- **Message**: Developer-facing, human-readable English debug message. Avoid implementation details.
- **Details**:
  - Must include `ErrorInfo` with machine-readable `reason` and `domain`.
  - Use `LocalizedMessage` for user-facing localized strings.
  - Use `BadRequest` for field-specific validation errors.
  - Use `Help` for links to troubleshooting docs.
- **Tone**: Professional and helpful. Avoid jargon or blame.
- **Permission checks**: Check permissions _before_ existence. Return `PERMISSION_DENIED` even if the resource doesn't exist if the user lacks "exists" check permissions.
- **Partial Errors**: Strongly discouraged. Use LRO metadata if bulk operations require partial successes.

**Correct (Error Result Structure):**

```json
{
  "error": {
    "code": 404,
    "status": "NOT_FOUND",
    "message": "The book 'publishers/123/books/missing' does not exist.",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.ErrorInfo",
        "reason": "BOOK_NOT_FOUND",
        "domain": "library.googleapis.com"
      }
    ]
  }
}
```

Reference: [AIP-193: Errors](https://google.aip.dev/193)
