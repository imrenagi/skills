---
name: protobuf-developer
description: Protobuf schema development guidelines. This skill should be used when writing, reviewing, or refactoring Protocol Buffer definitions to ensure consistency, backwards compatibility, and adherence to style guides.
---

# Protobuf Developer

This skill provides best practices and rules for developing Protocol Buffer schemas (proto3).

## When to Apply

Reference these guidelines when:

- Defining new gRPC services and message types.
- Modifying existing `.proto` files.
- Refactoring message structures.
- Ensuring backwards compatibility for evolving APIs.
- Configuring Protobuf management, linting, and code generation using [Buf](./references/buf/buf.md).
- Customizing HTTP mapping and OpenAPI output using [gRPC-Gateway](./references/grpc-gateway/).

## Rule Categories

| Category             | Description                       | Key Rules                                   |
| :------------------- | :-------------------------------- | :------------------------------------------ |
| **Resource Design**  | Modeling entities and identifiers | AIP-122, 123, 124, 126                      |
| **Standard Methods** | CRUD patterns and pagination      | AIP-131 to 135, 158                         |
| **Operations**       | LROs and custom methods           | AIP-151, 136                                |
| **Fields**           | Behavior, naming, and types       | AIP-140 to 148                              |
| **Design Patterns**  | Complex API features              | AIP-159 to 165, 231 to 235                  |
| **Compatibility**    | Backward-safe evolution           | AIP-180, 181                                |
| **Polish & Style**   | Errors, docs, and formatting      | AIP-191, 192, 210 to 213                    |
| **Buf Management**   | Tooling, linting, and generation  | [Buf Rules](./references/buf/buf.md)        |
| **gRPC-Gateway**     | HTTP mapping & OpenAPI output     | [Gateway Rules](./references/grpc-gateway/) |

## Quick Reference

- **Resource Names**: Use `name` field with `collection/id` pattern.
- **Methods**: Stick to the 5 standard methods (`Get`, `List`, `Create`, `Update`, `Delete`) when possible.
- **Update**: Always use `update_mask` for partial updates.
- **Lists**: Always implement pagination (`page_size`, `page_token`).
- **LROs**: Use `google.longrunning.Operation` for tasks > 2s.
- **Style**: PascalCase for Messages/Services, snake_case for fields, UPPER_CASE for enums.
- **Defaults**: Value 0 in Enums must be `_UNSPECIFIED`.
- **Gateway**: Use `google.api.field_behavior` for standard OpenAPI markers (`REQUIRED`, `OUTPUT_ONLY`).
- **Binary**: Use manual handlers on the Gateway mux for `multipart/form-data` uploads.

## How to Use

Read individual rule files in `references/rules/`, `references/google-aip/`, `references/buf/`, and `references/grpc-gateway/` for detailed explanations and code examples.

**Important**: When implementing Google AIP patterns from `references/google-aip/`, you **must** modify the protobuf package name and resource type domains to match the current application context (e.g., use `package imrenagi.com.v1;` instead of `package google.example.v1;`).

Each rule file contains:

- Brief explanation of the rule
- "Incorrect" examples (Bad patterns)
- "Correct" examples (Recommended patterns)
- Additional context and references
