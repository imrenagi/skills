---
title: Jobs (AIP-152)
impact: MEDIUM
impactDescription: Standardizes long-running tasks that require setup, configuration, and potentially multiple executions.
tags: protobuf, aip, jobs, execution, automation
---

## Jobs (AIP-152)

**Impact: MEDIUM**

A `Job` resource represents a task that takes significant time to complete and where a simple LRO is not sufficient (e.g., if it needs to be configured once and run multiple times).

**Implementation Rules:**

- **Naming**: The resource name must end with "Job" (e.g., `WriteBookJob`). The prefix should be an RPC-style verb-noun.
- **Methods**: Use standard CRUD methods (`Get`, `List`, `Create`, `Update`, `Delete`) to manage the job configuration.
- **Run Method**:
  - Define a custom `Run` method (e.g., `rpc RunWriteBookJob`).
  - Use `POST` with `:run` custom verb.
  - Return a `google.longrunning.Operation`.
  - The LRO should resolve to a response message containing the result.
- **Executions**: If a job runs multiple times (e.g., on a schedule), store individual runs as a sub-collection named `executions`.
- **Errors**: Errors starting the job return immediate gRPC errors. Errors during execution are placed in the LRO metadata or result.

**Correct (Job and Run Method):**

```proto
message WriteBookJob {
  option (google.api.resource) = {
    type: "library.googleapis.com/WriteBookJob"
    pattern: "publishers/{publisher}/writeBookJobs/{write_book_job}"
  };
  string name = 1;
  // Configuration fields...
}

rpc RunWriteBookJob(RunWriteBookJobRequest) returns (google.longrunning.Operation) {
  option (google.api.http) = {
    post: "/v1/{name=publishers/*/writeBookJobs/*}:run"
    body: "*"
  };
}
```

Reference: [AIP-152: Jobs](https://google.aip.dev/152)
