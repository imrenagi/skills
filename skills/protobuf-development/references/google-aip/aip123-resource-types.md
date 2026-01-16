---
title: Resource Types (AIP-123)
impact: MEDIUM
impactDescription: Provides a globally unique identifier for resource types, facilitating discovery and cross-API references.
tags: protobuf, aip, resource-types, design
---

## Resource Types (AIP-123)

**Impact: MEDIUM**

Resource types provide a way to uniquely identify the "kind" of a resource across different APIs. This is essential for tools that interact with multiple services and for defining resource references.

**Implementation Rules:**

- **Uniqueness**: Define a resource type for each resource using the pattern `{Service Name}/{Type}` (e.g., `pubsub.googleapis.com/Topic`).
- **Naming**:
  - Match the Protobuf message name.
  - Use PascalCase (e.g., `Topic`).
  - Use the singular form.
- **Annotations**: Use `google.api.resource` to define the type, pattern, singular, and plural names.
- **Patterns**:
  - Variables must use `snake_case` and not have an `_id` suffix (e.g., `{topic}`, not `{topic_id}`).
  - Variables must match the singular form of the resource type.
  - New patterns must be added at the end to maintain backward compatibility.
- **Singular/Plural**: Explicitly define `singular` and `plural` in the resource annotation.

**Incorrect (Missing or Inconsistent Resource Types):**

```proto
message Topic {
  // Missing resource annotation
  string name = 1;
}
```

**Correct (Standard Resource Types):**

```proto
import "google/api/resource.proto";

message Topic {
  option (google.api.resource) = {
    type: "pubsub.googleapis.com/Topic"
    pattern: "projects/{project}/topics/{topic}"
    singular: "topic"
    plural: "topics"
  };

  // The resource name of the topic.
  string name = 1;
}
```

Reference: [AIP-123: Resource types](https://google.aip.dev/123)
