---
title: Buf Code Generation
impact: HIGH
impactDescription: Automates the generation of consistent client and server code across multiple languages.
tags: protobuf, buf, generation
---

## Buf Code Generation

**Impact: HIGH**

Buf uses a `buf.gen.yaml` file to define how code should be generated from Protobuf definitions. This replaces complex `protoc` shell scripts and ensures consistency across the team.

### Configuration (`buf.gen.yaml`)

We use a version 2 configuration with **Managed Mode** enabled. This allows us to inject `go_package` options automatically during generation, keeping the `.proto` files focused on API definitions.

```yaml
version: v2
managed:
  enabled: true
  disable:
    # Disable for well-known types to avoid conflicts with upstream packages
    - file_option: go_package
      module: buf.build/googleapis/googleapis
    - file_option: go_package
      module: buf.build/grpc-ecosystem/grpc-gateway
  override:
    - file_option: go_package_prefix
      value: github.com/imrenagi/<application_name>/<api_dir>/internal/gen/proto
plugins:
  - local: protoc-gen-go
    out: <api_dir>/internal/gen/proto
    opt:
      - paths=source_relative
  - local: protoc-gen-go-grpc
    out: <api_dir>/internal/gen/proto
    opt:
      - paths=source_relative
  - local: protoc-gen-grpc-gateway
    out: <api_dir>/internal/gen/proto
    opt:
      - logtostderr=true
      - paths=source_relative
      - generate_unbound_methods=true
  - local: protoc-gen-openapiv2
    out: <api_dir>/internal/gen/openapi
    strategy: all
    opt:
      - allow_merge=true
      - merge_file_name=app
      - include_package_in_tags=true
      - proto3_optional_nullable=true
```

### Managed Mode

We use **Managed Mode** (`managed: enabled: true`) specifically to manage the `go_package` option. By setting `go_package_prefix`, Buf automatically constructs the correct Go package path based on the directory structure of the proto files.

**Note**: We disable management for `googleapis` and `grpc-gateway` to ensure we use the official imported Go packages for those dependencies.

### Tooling Setup

To use local plugins, ensure all necessary binaries are installed in your Go path. Add the following to your `Makefile`:

```makefile
install-tools:
	go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest
	go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest
	go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
	go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
	go install github.com/bufbuild/buf/cmd/buf@latest
	go install github.com/go-swagger/go-swagger/cmd/swagger@latest
```

### Generation Workflow

1.  **Install Tools**: Run `make install-tools`.
2.  **Lint**: Run `buf lint`.
3.  **Generate**: Run `buf generate`.

### Best Practices

- **Never** manually edit generated code.
- **Commit** generated code to version control to allow others to build the project without having the full Protobuf toolchain installed immediately.
- Use `paths=source_relative` to maintain alignment between proto paths and Go package paths.

Reference: [Buf Generation](https://buf.build/docs/generate/overview/)
