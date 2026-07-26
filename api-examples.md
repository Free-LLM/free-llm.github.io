---
title: API Examples
layout: default
parent: Archive
nav_order: 23
---

# API / Schema Examples

> ⚠️ **Archived.** This page describes an early design phase that predates the actual DCNR implementation in [`compute-all`](https://github.com/Free-LLM/compute-all) and has been superseded by it. It is kept for historical context. See the [Architecture Overview](/architecture) and [Project Status](/status) for the current state.


This page provides example protobuf snippets and message flows for the core protocol interactions between orchestrators and PNodes.

See also: [Protocol](/protocol)

---

## Protobuf Snippets (illustrative)

```proto
message PNodeDescriptor {
  string pnode_uuid = 1;
  string version = 2;
  string network_address = 3;
  HardwareInfo hardware = 4;
  int32 max_vnodes = 5;
}

message HardwareInfo {
  uint64 memory_mb = 1;
  uint32 cpu_cores = 2;
  string cpu_model = 3;
  bool has_gpu = 4;
}

message VNodeInfo {
  string vnode_id = 1;
  string type = 2;
  repeated string parent_vnode_ids = 3;
  map<string, string> config = 4;
}

message NetworkDefinition {
  string network_id = 1;
  string name = 2;
  repeated VNodeInfo vnodes = 3;
}
```

---

## Example RPC Flows

### PNode Registration
1. PNode → Orchestrator: `RegisterPNode(PNodeDescriptor)`
2. Orchestrator stores PNode in registry and returns success

### Network Creation
1. Client → Orchestrator: `CreateNetwork(NetworkDefinition)`
2. Orchestrator decomposes network into VNodes

### VNode Resolution
1. PNode → Orchestrator: `GetVNodeLocation(vnode_id)`
2. Orchestrator returns the address of the PNode hosting that VNode

---

## Example Payloads (YAML-like Network)

```yaml
network:
  name: "simple-relu-net"
  vnodes:
    - id: "input-0"
      type: "input"
      config:
        shape: [64, 64]
    - id: "layer-1"
      type: "linear"
      parents: ["input-0"]
      config:
        out_features: 128
    - id: "relu-1"
      type: "relu"
      parents: ["layer-1"]
```

---

## Notes

- These examples are illustrative and will evolve with the reference `.proto` files
- Prefer references/URIs for large tensors to keep payload sizes small
