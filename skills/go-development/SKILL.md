---
name: go-development
description: Comprehensive rules for Go development, covering gRPC, error handling, domain-driven design, logging, and infrastructure configuration. Use these rules when writing, reviewing, or refactoring Go code.
---

# Go Development

This skill provides a comprehensive set of rules and best practices for building robust, scalable, and secure applications in Go. It focuses on gRPC services, domain-driven design, structured error handling, and observability.

## When to Apply

Reference these guidelines when:

- **Designing Protobuf/gRPC APIs**: Setting up servers, interceptors, and gateway configurations.
- **Implementing Business Logic**: Developing domain models, handling errors, and managing enums.
- **Configuring Infrastructure**: Initializing SQL, Redis, and various client libraries.
- **Optimizing Observability**: Implementing contextual logging and standardized error reporting.
- **Refactoring**: Improving code structure and consistency across the codebase.

## Quick Reference table

| Category           | Rule                                                                               | Impact     |
| :----------------- | :--------------------------------------------------------------------------------- | :--------- |
| **gRPC Server**    | [gRPC-Gateway Implementation](references/rules/server-grpc-gateway.md)             | **HIGH**   |
|                    | [gRPC Interceptors (AuthN/AuthZ)](references/rules/server-grpc-interceptor.md)     | **HIGH**   |
|                    | [AuthZ in gRPC Handlers](references/rules/server-grpc-handler-authz.md)            | **HIGH**   |
|                    | [Standardized gRPC Error Handling](references/rules/server-grpc-error.md)          | **HIGH**   |
|                    | [Header Forwarding to Metadata](references/rules/server-grpc-header-matcher.md)    | **MEDIUM** |
| **Domain Logic**   | [Favor Rich Domain Models](references/rules/domain-driven.md)                      | **HIGH**   |
|                    | [Define and Return Domain Errors](references/rules/error-domain-error.md)          | **HIGH**   |
|                    | [Proper Context Usage](references/rules/context-usage.md)                          | **HIGH**   |
|                    | [Strongly Typed Domain Enums](references/rules/domain-enum.md)                     | **MEDIUM** |
| **Error Handling** | [Wrap Errors with %w](references/rules/error-wrapping.md)                          | **HIGH**   |
|                    | [Sentinel vs. Typed Errors](references/rules/error-types.md)                       | **MEDIUM** |
| **Logging**        | [Pass Context to Loggers](references/rules/logging-context.md)                     | **HIGH**   |
|                    | [Log Errors Once at Outer Layer](references/rules/logging-error.md)                | **HIGH**   |
|                    | [gRPC Logging Interceptors](references/rules/grpc-logging-interceptor.md)          | **HIGH**   |
|                    | [HTTP Logging Middleware](references/rules/http-logging-middleware.md)             | **MEDIUM** |
| **Infrastructure** | [Slowloris Mitigation (HTTP)](references/rules/server-http-slowloris-attack.md)    | **HIGH**   |
|                    | [SQL Config & Optimization](references/rules/sql-configuration.md)                 | **MEDIUM** |
|                    | [sqlx Initialization](references/rules/sql-sqlx-initialization.md)                 | **MEDIUM** |
|                    | [Fluent SQL with Squirrel](references/rules/sql-squirrel.md)                       | **MEDIUM** |
|                    | [Redis Configuration](references/rules/redis-configuration.md)                     | **MEDIUM** |
|                    | [Redis Initialization](references/rules/redis-initialization.md)                   | **MEDIUM** |
|                    | [Viper Configuration Management](references/rules/config-viper.md)                 | **MEDIUM** |
| **Clients**        | [gRPC Client Initialization](references/rules/client-grpc-initialization.md)       | **HIGH**   |
|                    | [General Client Initialization](references/rules/client-general-initialization.md) | **MEDIUM** |
| **Testing**        | [Table-Driven Unit Tests](references/rules/testing-unit-test-table-driven.md)      | **MEDIUM** |
| **General & CLI**  | [CLI Command Structure](references/rules/cli-structure.md)                         | **MEDIUM** |

## How to Use

1.  **Read the Rules**: Before starting a new feature or refactor, browse the relevant rule files in `references/rules/`.
2.  **Follow the Examples**: Use the provided "Correct" code patterns as templates for your implementation.
3.  **Ensure Consistency**: Apply the same patterns (e.g., error wrapping, logging) across all layers of the application.
4.  **Automatic Enforcement**: These rules are meant to be referenced by AI assistants and used as a basis for code reviews.

For any specific questions on implementation, refer to the individual rule files which contain detailed explanations, code snippets, and additional context.
