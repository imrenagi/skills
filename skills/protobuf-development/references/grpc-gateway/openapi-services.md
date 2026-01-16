---
title: OpenAPI Service and Operation Customization
impact: MEDIUM
impactDescription: Improves API discoverability and provides support for non-default success status codes (e.g. 201, 204).
tags: grpc-gateway, openapi, protobuf, services, operations
---

## OpenAPI Service and Operation Customization

**Impact: MEDIUM**

This rule defines how to customize the OpenAPI (Swagger) output for gRPC services and methods (operations) using `grpc-gateway`.

**Correct (Service Tag Customization):**

To customize the OpenAPI tag or other service-level metadata, use `grpc.gateway.protoc_gen_openapiv2.options.openapiv2_tag`.

```protobuf
import "protoc-gen-openapiv2/options/annotations.proto";

service UserService {
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_tag) = {
    description: "Operations related to user management."
    external_docs: {
      url: "https://example.com/docs/users"
      description: "Detailed User Guide"
    }
  };
}
```

**Correct (Operation Detail and Response Codes):**

Use `openapiv2_operation` for method-specific summaries and to override default response codes (e.g., `201 Created` or `204 No Content`).

```protobuf
rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty) {
  option (google.api.http) = {
    delete: "/v1/users/{id}"
  };
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
    summary: "Delete a user"
    description: "Permanently removes a user record from the system."
    responses: {
      key: "204"
      value: {
        description: "User deleted successfully."
        schema: {}
      }
    }
  };
}
```

**Correct (Security Requirements):**

Define security requirements per operation.

```protobuf
rpc UpdateUser(UpdateUserRequest) returns (User) {
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
    security: {
      security_requirement: {
        key: "BearerAuth";
        value: {
          scope: "user:write";
        }
      }
    }
  };
}
```

**Correct (File-level Global Settings):**

To customize the top-level OpenAPI document (info, host, base path, security definitions), use file-level options.

```protobuf
option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_swagger) = {
  info: {
    title: "My Awesome API";
    version: "1.0";
  };
  schemes: HTTPS;
  consumes: "application/json";
  produces: "application/json";
};
```

Reference: [Customizing OpenAPI Output](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_openapi_output/)
