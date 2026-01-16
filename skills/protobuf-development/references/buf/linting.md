---
title: Buf Linting rules
impact: HIGH
impactDescription: Standardizes API style and ensures consistency with Google AIPs.
tags: protobuf, buf, linting
---

## Buf Linting Rules

**Impact: HIGH**

Linting ensures that all developers follow the same style and best practices. While Buf provides powerful defaults, we occasionally disable specific rules to prioritize adherence to **Google AIP** standards or specific project needs.

### Configuration (`buf.yaml`)

Use the `STANDARD` category as a base. If a Buf linter rule conflicts with an AIP requirement or project context, disable the linter instead of compromising the API design.

```yaml
version: v2
lint:
  use:
    - STANDARD
  except:
    # Disable if package names don't end in v1, v2, etc. (AIP-185)
    # but we should follow AIP-185. Only disable if legacy.
    - PACKAGE_VERSION_SUFFIX

    # Disable if you want to allow Service suffix other than "Service"
    # though AIP recommends Service suffix for some cases.
    - SERVICE_SUFFIX

    # AIP-131, 132 etc. require specific naming. If Buf's naming
    # rules differ, prefer AIP and disable the corresponding Buf rule.
    - RPC_REQUEST_RESPONSE_UNIQUE
```

### Key Linting Rules to Observe

- **`ENUM_ZERO_VALUE_SUFFIX`**: By default, Buf enforces `_UNSPECIFIED`. This is consistent with AIP-126.
- **`SERVICE_PASCAL_CASE`**: Interfaces/Services must be `PascalCase`.
- **`FIELD_LOWER_SNAKE_CASE`**: Fields must be `snake_case`.
- **`RPC_REQUEST_STANDARD_NAME`**: RPC inputs should be `MethodNameRequest`.

### Handling Conflicts

When a linter error occurs:

1.  Check if the implementation follows **[Google AIP Guidelines](../google-aip/aip121-resource-oriented-design.md)**.
2.  If it does, and Buf still flags an error, add the rule to the `except` list in `buf.yaml`.
3.  **Do not** change a correct AIP implementation just to satisfy the linter.

**Example: Excluding a directory**

```yaml
build:
  excludes:
    - third_party # Ignore external protos
    - vendor
```

Reference: [Buf Linting Quickstart](https://buf.build/docs/lint/quickstart/)
