---
title: OpenAPI Message and Field Customization
impact: MEDIUM
impactDescription: Ensures generated documentation matches exact API requirements for types and descriptions.
tags: grpc-gateway, openapi, protobuf
---

## OpenAPI Message and Field Customization

**Impact: MEDIUM**

This rule defines how to customize the OpenAPI (Swagger) output for Protobuf messages and fields using `grpc-gateway`.

**Correct (Message and Field Customization):**

To customize the OpenAPI Schema Object for a message, use the `grpc.gateway.protoc_gen_openapiv2.options.openapiv2_schema` option.

```protobuf
import "protoc-gen-openapiv2/options/annotations.proto";
import "google/api/field_behavior.proto";

message UserProfile {
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_schema) = {
    json_schema: {
      title: "User Profile"
      description: "A message representing the user's public profile information."
      required: ["id", "username"]
    }
    example: "{\"id\": \"user_123\", \"username\": \"jdoe\"}"
  };

  string id = 1 [(google.api.field_behavior) = OUTPUT_ONLY];
  string username = 2 [
    (google.api.field_behavior) = REQUIRED,
    (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_field) = {
      description: "The user's unique username.",
      example: "\"jdoe\""
    }
  ];
}
```

**Correct (Using In-Proto Comments):**

By default, comments in `.proto` files are translated into OpenAPI descriptions.

```protobuf
// User represents a system user.
message User {
  // Unique identifier for the user.
  string id = 1;
}
```

**Correct (Hiding Fields):**

Use `google.api.VisibilityRule` to hide internal fields.

```protobuf
import "google/api/visibility.proto";

message SensitiveData {
  string internal_id = 1 [(google.api.field_visibility).restriction = "INTERNAL"];
  string public_name = 2;
}
```

_Note: This requires passing `visibility_restriction_selectors` (e.g., `INTERNAL`) to the `openapiv2` plugin to include them, or omitting them to hide them._

Reference: [Customizing OpenAPI Output](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_openapi_output/)
