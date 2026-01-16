---
title: Declarative-friendly Interfaces (AIP-128)
impact: MEDIUM
impactDescription: Ensures resources can be managed by DevOps tools like Terraform or Kubernetes.
tags: protobuf, aip, declarative, design
---

## Declarative-friendly Interfaces (AIP-128)

**Impact: MEDIUM**

Declarative-friendly interfaces allow resources to be managed by tools that specify a desired "outcome" rather than a series of actions. This is essential for infrastructure-as-code and automated systems.

**Implementation Rules:**

- **Standard Methods**: Must use strongly-consistent standard methods for lifecycle management (Create, Get, List, Update, Delete).
- **Annotation**: Designate declarative-friendly resources using `style: DECLARATIVE_FRIENDLY` in the `google.api.resource` annotation.
- **Reconciliation**: If updates take time, include an `OUTPUT_ONLY` `bool reconciling` field. Set to `true` when the current state does not match the intended state.
- **GET Request**: Must return the resource's _current_ state, not the intended state.
- **No Custom Methods**: Declarative resources should generally avoid custom methods (AIP-136).
- **Concurrency**: Must have an `etag` field for optimistic concurrency (AIP-154).
- **Required Fields**: Must include standard fields like `uid`, `create_time`, `update_time`, etc. (AIP-148).

**Incorrect (Non-declarative Resource):**

```proto
message Instance {
  option (google.api.resource) = {
    type: "compute.googleapis.com/Instance"
    pattern: "projects/{project}/instances/{instance}"
  };

  string name = 1;
  // Bad: No way to tell if the resource is still being provisioned
}

service ComputeService {
  // Bad: Using a custom method for something that should be an update
  rpc StartInstance(StartInstanceRequest) returns (google.longrunning.Operation);
}
```

**Correct (Declarative-friendly Resource):**

```proto
message Instance {
  option (google.api.resource) = {
    type: "compute.googleapis.com/Instance"
    pattern: "projects/{project}/instances/{instance}"
    style: DECLARATIVE_FRIENDLY
  };

  string name = 1;

  // Indicates if the server is still working to reach the desired state.
  bool reconciling = 2 [(google.api.field_behavior) = OUTPUT_ONLY];

  // For optimistic concurrency.
  string etag = 3;
}
```

Reference: [AIP-128: Declarative-friendly interfaces](https://google.aip.dev/128)
