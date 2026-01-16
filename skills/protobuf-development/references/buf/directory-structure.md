---
title: Buf Directory Structure
impact: MEDIUM
impactDescription: Ensures that proto files are organized in a way that Buf can easily process and generate code from.
tags: protobuf, buf, structure, modules
---

## Buf Directory Structure

**Impact: MEDIUM**

A consistent directory structure is essential for Buf modules to resolve imports correctly and for generated code to land in the right places.

### Root Module Layout

Each microservice or logical API group should be a **Buf Module**, characterized by a `buf.yaml` file at its root.

```text
.
├── buf.yaml                # Module configuration
├── buf.gen.yaml            # Code generation configuration
├── buf.lock                # Dependency lock file
└── proto/                  # Source directory for .proto files
    └── imrenagi/           # Organization/Domain
        └── <service>/      # Service name
            └── v1/         # Major version (AIP-185)
                ├── service.proto
                └── resources.proto
```

### Organizing Packages

- **Versioned Packages**: Always include the version in the directory path and package name (e.g., `proto/imrenagi/library/v1/library.proto` -> `package imrenagi.library.v1;`).
- **Domain Namespacing**: Use your application/domain context (e.g., `imrenagi.com`) as the base package to avoid collisions.
- **File Matching**: The directory structure **must** match the package name. `package a.b.v1;` must live in `a/b/v1/`.

### Import Resolution

Imports should be relative to the root specified in `buf.yaml`.

**Incorrect (Relative import):**
`import "../common/common.proto";`

**Correct (Absolute import from root):**
`import "imrenagi/common/v1/common.proto";`

Reference: [Buf Directory Structure](https://buf.build/docs/lint/rules/#directory_same_package)
