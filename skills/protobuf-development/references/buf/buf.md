---
title: Protobuf Management with Buf
impact: MEDIUM
impactDescription: High-level overview of using Buf for linting, breaking change detection, and generation.
tags: protobuf, buf, overview
---

## Protobuf Management with Buf

**Impact: MEDIUM**

Use [Buf](https://buf.build/) as the primary tool for managing Protocol Buffer files. It replaces complex `protoc` setups with a simplified, configuration-driven workflow.

### Core Components

Buf is managed through three primary configuration files:

1.  **`buf.yaml`**: Defines the module, dependencies, and lint/breaking change rules.
2.  **`buf.lock`**: (Generated) Tracks specific versions of dependencies in the BSR.
3.  **`buf.gen.yaml`**: Configures code generation plugins (Go, Java, etc.).

### Key Capabilities

Refer to the detailed rules for each area:

- **[Linting](./linting.md)**: Enforce style and consistency.
- **[Breaking Change Detection](./breaking-changes.md)**: Prevent wire-incompatible updates.
- **[Directory Structure](./directory-structure.md)**: Organize modules and packages.
- **[Code Generation](./generation.md)**: Automate the creation of client/server code.

### Basic Configuration Example

```yaml
version: v2
name: buf.build/imrenagi/<app-name>
deps:
  - buf.build/googleapis/googleapis
  - buf.build/grpc-ecosystem/grpc-gateway
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
```

### Common Commands

- `buf lint`: Check files for style violations.
- `buf breaking --against ".git#branch=main"`: Check for wire-breaking changes.
- `buf generate`: Generate source code.

Reference: [Buf Documentation](https://buf.build/docs/introduction)
