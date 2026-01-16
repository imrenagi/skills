---
title: Policy Preview (AIP-236)
impact: MEDIUM
impactDescription: A safety mechanism for policy rollouts, allowing users to validate changes against production traffic before promotion.
tags: protobuf, aip, policies, security, preview, rollout
---

## Policy Preview (AIP-236)

**Impact: MEDIUM**

Preview is a safety mechanism for policy resources (Firewalls, IAM, etc.), allowing customers to see the effect of proposed changes before they go live.

**Implementation Rules:**

- **Experiments**: Store proposed configurations in an `experiments` sub-collection under the live policy.
- **Naming**: Experiment resource type should be `*Experiment` (e.g., `FirewallPolicyExperiment`).
- **Resource Structure**: The experiment proto must contain:
  - The policy message being tested (same type as the live policy).
  - A `preview_metadata` field (output only).
- **Metadata**: Tracks the state of the preview (`ACTIVE`, `SUSPENDED`), `log_prefix`, `start_time`, and `stop_time`.
- **Custom Methods**:
  - `startPreview`: Begins generating comparison logs.
  - `stopPreview`: Ceases log generation.
  - `commit`: Atomically promotes the experiment to the live policy (overwriting it) and deletes the experiment.
- **Logging**: Comparison logs must be preceded by the `log_prefix` and include etags of both the live and experimental policies for identification.

**Example (Experiment Resource):**

```proto
message FirewallPolicyExperiment {
  string name = 1;

  // The policy payload to test.
  FirewallPolicy firewall_policy = 2;

  // Output only. Metadata about the preview state.
  FirewallPolicyPreviewMetadata preview_metadata = 3
      [(google.api.field_behavior) = OUTPUT_ONLY];
}
```

Reference: [AIP-236: Policy preview](https://google.aip.dev/236)
