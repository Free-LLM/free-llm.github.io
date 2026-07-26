---
title: Getting Started
layout: default
nav_order: 20
---

# Getting Started

This quickstart shows how to run a minimal local setup from the
[`compute-all`](https://github.com/Free-LLM/compute-all) repository: one
orchestrator and one or two Physical Nodes (PNodes) on your machine, submit
a network, and see DCNR in action.

---

## Prerequisites

- [Bazel (Bazelisk recommended)](https://github.com/bazelbuild/bazelisk) —
  the primary build system (plain `go build` also works)
- Go 1.24+ (managed by Bazel)
- Docker (optional, for containerized runs; images are built with Bazel)
- An S3-compatible store for VNode state (MinIO works locally)

```bash
git clone https://github.com/Free-LLM/compute-all
cd compute-all
bazel build //...        # build everything
bazel test //...         # run the test suite
```

---

## 1) Start the Orchestrator

The orchestrator manages network topology, PNode registration, allocation,
and training sessions.

```bash
bazel run //orchestrator/cmd/orchestrator
```

Key environment variables: `ORCHESTRATOR_PORT` (default `50052`) and the
DynamoDB table names used for persistence.

See also: [Orchestrator](/orchestrator) · [Protocol](/protocol)

---

## 2) Start a Physical Node (PNode)

The PNode provides the compute resources and hosts the Virtual Nodes
(VNodes).

```bash
ORCHESTRATOR_ADDRESS=localhost:50052 PNODE_PORT=50051 \
  bazel run //pnode/cmd/pnode
```

Expected logs:
- `PNode server listening at :50051`
- `Successfully registered with orchestrator`

Start a second PNode on another port to see real distribution.

See also: [Physical Node (PNode)](/pnode)

---

## 3) Submit a Network

A network is a graph of typed VNodes defined in
[YAML](/network-definition). The repository ships working examples in
`data/` — from a tiny test network to the
[production transformer topologies](/training-topologies).

Use the `dcnr` CLI to create the network and drive it:

```bash
bazel run //cli/cmd/dcnr -- --help
```

The CLI covers network creation/inspection, training sessions, dataset and
tokenizer management, and live monitoring.

---

## 4) Observe Execution

- The orchestrator allocates VNodes to available PNodes, honoring
  colocation groups and locality.
- PNodes lazily instantiate VNodes and execute forward/training requests;
  training hops between PNodes are asynchronous.
- Training sessions stream step metrics; the
  [monitoring dashboard](/status#observability) shows charts, the live
  network graph, and per-PNode resources.

---

## 5) Observe Fault Tolerance

Stop one of your PNodes while a network is deployed. The orchestrator
detects the missing heartbeats and re-allocates its VNodes to the remaining
PNodes, which restore state from storage.

---

## 6) Next Steps

- Read the [Network Definition (YAML)](/network-definition) format and the
  [node type catalog](/vnode).
- Look at the [production training networks](/training-topologies) we train
  today.
- Check the current [Project Status](/status) and the [Roadmap](/roadmap).
