---
title: API Examples
layout: default
nav_order: 23
---

# API / Schema Examples

This page provides example protobuf snippets and message flows for the core protocol interactions between orchestrators and agents.

See also: [Protocol](/protocol)

---

## Protobuf Snippets (illustrative)

```proto
message NodeDescriptor {
  string node_id = 1;
  string arch = 2;
  string os = 3;
  uint32 cpu_cores = 4;
  uint64 mem_bytes = 5;
  repeated string operator_sets = 6; // e.g., ["v1"]
  string protocol_version = 7;
  // Optional trust signals
  bytes signed_descriptor = 8; // signature over canonical form
  string attestation_ref = 9;  // optional reference/URL
}

message TaskDescriptor {
  string task_id = 1;
  string job_id = 2;
  string operator = 3; // e.g., "matmul"
  bytes input_a = 4;   // encoding TBD; could be ref/URI in practice
  bytes input_b = 5;   // optional second input
  repeated int64 shape_out = 6;
  uint64 timeout_ms = 7;
}

message ValidationReceipt {
  string task_id = 1;
  string method = 2;    // e.g., "freivalds", "sample_checks"
  uint32 rounds = 3;    // if applicable
  bool   passed = 4;
  string notes = 5;
}

message ResultDescriptor {
  string task_id = 1;
  string job_id = 2;
  bool   ok = 3;
  bytes  output = 4;
  string error = 5;
  ValidationReceipt validation = 6; // optional
}
```

---

## Example RPC Flows

### Capability Advertisement
1. Agent → Orchestrator: `NodeDescriptor`
2. Orchestrator stores/updates capability view

### Task Dispatch
1. Orchestrator → Agent: `TaskDescriptor`
2. Agent → Orchestrator: Ack (accept/reject)

### Result Submission
1. Agent → Orchestrator: `ResultDescriptor`
2. Orchestrator validates (cheap checks) before accept

---

## Example Payloads (JSON-like)

```json
{
  "task_id": "t-001",
  "job_id": "j-001",
  "operator": "matmul",
  "shape_out": [64, 16],
  "timeout_ms": 5000
}
```

Validation receipt example:

```json
{
  "task_id": "t-001",
  "method": "freivalds",
  "rounds": 2,
  "passed": true,
  "notes": "n=16"
}
```

---

## Notes

- These examples are illustrative and will evolve with the reference `.proto` files
- Prefer references/URIs for large tensors to keep payload sizes small
