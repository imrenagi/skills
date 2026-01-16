---
title: OpenAPI Enum Customization
impact: MEDIUM
impactDescription: Standardizes how enums appear in API clients, preventing confusion between numeric and string representations.
tags: grpc-gateway, openapi, protobuf, enums
---

## OpenAPI Enum Customization

**Impact: MEDIUM**

This rule defines how to customize the OpenAPI (Swagger) output for Protobuf enums using `grpc-gateway`.

**Correct (Enum Customization):**

Enums can be customized similarly to messages using `grpc.gateway.protoc_gen_openapiv2.options.openapiv2_enum`.

```protobuf
import "protoc-gen-openapiv2/options/annotations.proto";

enum UserStatus {
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_enum) = {
    title: "User Status"
    description: "The current lifecycle state of a user account."
    example: "\"ACTIVE\""
  };

  USER_STATUS_UNSPECIFIED = 0;
  ACTIVE = 1;
  SUSPENDED = 2;
  DELETED = 3;
}
```

**Correct (Hiding Enum Values):**

To hide specific enum values from documentation, use `google.api.VisibilityRule`.

```protobuf
import "google/api/visibility.proto";

enum AccessLevel {
  ACCESS_LEVEL_UNSPECIFIED = 0;
  USER = 1;
  ADMIN = 2;
  SUPER_ADMIN = 3 [(google.api.value_visibility).restriction = "INTERNAL"];
}
```

Reference: [Customizing OpenAPI Output](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_openapi_output/)
