---
title: API Methods Hierarchy (AIP-130)
impact: MEDIUM
impactDescription: Provides a decision framework for choosing the right category of RPC methods for consistency and client compatibility.
tags: protobuf, aip, methods, resource-oriented, rpc
---

## API Methods Hierarchy (AIP-130)

**Impact: MEDIUM**

When designing an API, authors should prioritize method categories that offer the most uniformity and client compatibility.

**Guidance on Category Priority:**

1.  **Standard Methods**: (List, Create, Get, Update, Delete). These provide the highest level of automation for CLI, UI, and declarative clients.
2.  **Standard Batch/Aggregate Methods**: (BatchGet, BatchCreate, etc., and cross-collection List). Useful for optimization while maintaining predictable patterns.
3.  **Custom Methods**: (e.g., `:run`, `:restart`). Used for imperative actions that don't fit CRUD. These are less automatable for declarative clients but still recognizable by CLIs/SDKs.
4.  **Misc Custom/Stateless**: (e.g., `:translate`). Methods with no permanent effect on API data.
5.  **Streaming**: Last resort, as they require hand-written client logic and are the least uniform.

**Decision Checklist:**

- Can this be expressed as a standard CRUD operation? (If yes, do it).
- Is it a batch operation of multiple resources? (Use Batch\*).
- Is it a state transition or imperative action on a resource? (Use a Custom method).
- Is it a long-running task? (Use Jobs or LROs).

Reference: [AIP-130: Methods](https://google.aip.dev/130)
