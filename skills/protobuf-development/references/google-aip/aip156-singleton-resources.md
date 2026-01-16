---
title: Singleton Resources (AIP-156)
impact: MEDIUM
impactDescription: Standardizes resources that have exactly one instance per parent, such as configuration objects.
tags: protobuf, aip, singleton, resource-design
---

## Singleton Resources (AIP-156)

**Impact: MEDIUM**

Singleton resources are useful for representing entities that always exist (exactly once) under a parent, such as a configuration or settings object.

**Implementation Rules:**

- **Implicit Existence**: A singleton must always exist if its parent exists.
- **No ID Field**: Must not have a user-provided ID; its name is the parent's name followed by a static segment (e.g., `users/123/config`).
- **Resource Definition**: Must provide both `singular` and `plural` fields in the `google.api.resource` annotation.
- **Prohibited Methods**: Must not define `Create` or `Delete`. These actions are tied to the parent's lifecycle.
- **Allowed Methods**: Should define `Get` and `Update` (unless read-only). May define `List` to show the singleton across multiple parents (AIP-159).
- **Update Exception**: Do not define `Update` if all fields are `OUTPUT_ONLY`.

**Incorrect (Singleton with Create method):**

```proto
service Library {
  // Bad: Singletons shouldn't be manually created or deleted
  rpc CreateConfig(CreateConfigRequest) returns (Config);
}
```

**Correct (Standard Singleton):**

```proto
message Config {
  option (google.api.resource) = {
    type: "library.googleapis.com/Config"
    pattern: "publishers/{publisher}/config"
    singular: "config"
    plural: "configs"
  };

  string name = 1; // publishers/123/config
  bool notifications_enabled = 2;
}

service Library {
  rpc GetConfig(GetConfigRequest) returns (Config) {
    option (google.api.http) = { get: "/v1/{name=publishers/*/config}" };
  }
}
```

Reference: [AIP-156: Singleton resources](https://google.aip.dev/156)
