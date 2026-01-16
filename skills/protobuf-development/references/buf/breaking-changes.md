---
title: Buf Breaking Change Detection
impact: HIGH
impactDescription: Prevents developers from introducing changes that would break existing clients.
tags: protobuf, buf, breaking-changes
---

## Buf Breaking Change Detection

**Impact: HIGH**

Breaking change detection ensures that the API remains backwards compatible (AIP-180). Buf can detect wire-format changes, package renames, and field removals.

### Configuration

Set the breaking change policy in `buf.yaml`. Use `FILE` or `PACKAGE` level detection.

```yaml
version: v2
breaking:
  use:
    - FILE # Strictest: catches removals of files, packages, and components.
```

### Running Checks

Always check changes against the state of the `main` branch or a released tag.

```bash
# Check against local git history
buf breaking --against ".git#branch=main"

# Check against a remote BSR module
buf breaking --against "buf.build/imrenagi/library:main"
```

### What Constitutes a Breaking Change?

- **Removal**: Removing a message, field, service, or RPC.
- **Type Change**: Changing the type of a field (e.g., `int32` to `string`).
- **Renames**: Changing a field name (breaks JSON/Text format compatibility).
- **Numbering**: Changing a field's tag number.

If a breaking change is **unavoidable**, it **must** be accompanied by a major version bump (`v1` -> `v2`) in a new package and directory.

Reference: [Buf Breaking Change Detection](https://buf.build/docs/breaking/overview/)
