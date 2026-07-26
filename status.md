---
title: Project Status
layout: default
nav_order: 11
---

# Project Status

_Last updated: July 2026 — based on the current state of the
[`compute-all`](https://github.com/Free-LLM/compute-all) repository._

The **Distributed Composable Neural Runtime (DCNR)** has moved well past the
design phase: there is a working implementation that can define a transformer
network in YAML, distribute its components across multiple physical nodes,
train it with mixed precision and GPU acceleration, and watch the run live on
a web dashboard.

This page summarizes what exists today. For the longer-term plan, see the
[Roadmap](/roadmap). For the story of how we got here, see the [Blog](/blog).

---

## What is implemented today

### Tensor runtime

- A lightweight **Go tensor runtime** with the core operations needed for
  transformer models (matmul, softmax, layer norm, elementwise ops,
  reductions, reshapes/views/slices).
- **Mixed precision**: `float32`, `float16`, and `bfloat16` dtypes flow
  through construction, casts, views, persistence, and ops. Optimizers keep
  **fp32 master parameters** for reduced-precision weights, with dynamic loss
  scaling for stability.
- **GPU acceleration** (Metal backend) for the hot paths, including fp16
  typed kernels and a fused Adam update; the runtime interfaces are designed
  so additional backends (CUDA/ROCm) can be slotted in.

### Virtual Nodes (VNodes)

Each neural-network component is a stateful agent that can be placed on any
physical node. The registry currently includes:

- **Layers**: `linear`/`dense`, `layernorm`, `embedder`, attention —
  including **multi-head attention and a distributed per-head pattern**,
  where each attention head can run on a different machine.
- **Position encoding**: RoPE.
- **Activations & regularization**: `relu`, `gelu`, `dropout`, masking
  (including causal masks).
- **Structural nodes**: `broadcast`, `split`, `stack`, `add`, `reshape`,
  shared weights across nodes.
- **Text I/O**: `tokenizer` and `detokenizer` nodes backed by a trainable
  BPE tokenizer.
- **Cost functions** for training (e.g. cross-entropy, MSE) plus validation
  support.

All VNodes obey **gradient locality**: `Backward()` uses only local
parameters, locally cached inputs, and the incoming gradient — no global
computational graph — which is what makes extreme horizontal distribution
possible.

### Orchestrator

- PNode **registration, heartbeats, status tracking, and automatic
  failover** (VNodes on a dead node are reallocated).
- **YAML network definitions** with cycle detection and topology validation.
- **Locality-aware allocation**: adjacent VNodes are co-located on the same
  PNode where possible, with load balancing across the fleet.
- **Training sessions**: the trainer now lives inside the orchestrator, with
  dedicated protocol messages for session management, an optimized Adam
  path, and training-step metrics reporting.
- **Persistence and crash recovery** via DynamoDB.

### Protocol & communication

- gRPC APIs for orchestrator↔PNode and PNode↔PNode communication.
- Training hops between PNodes are **asynchronous**
  (`ReceiveTrainRequest`/`ReceiveTrainResponse` with request IDs), removing
  the head-of-line blocking of the earlier synchronous design.

### Data & tokenization

- A **trainable BPE tokenizer** with vocabulary training (vocabularies up to
  256K tokens are checked into the repo), state saving, and text
  preprocessing utilities.
- A **dataset API** that defines and serves datasets in tokenized format,
  with **dataset splitting** (train/validation) using multiple strategies.

### Observability

- A **monitoring backend** (serverless, DynamoDB-backed) that collects
  request-status updates from PNodes and training-step metrics from the
  orchestrator, exposed through a **GraphQL API**.
- A **Vue 3 dashboard** with per-session training charts, a live network
  topology graph (real stored topology, per-node memory details), and a
  PNode dashboard (hosted VNodes, process/GPU memory, pending requests).

### Tooling & engineering

- A **CLI (`dcnr`)** for creating/inspecting networks, running training,
  driving the tokenizer, and listening to monitoring streams.
- **Bazel** builds for everything (with plain `go build` kept working),
  Docker images, and a Kotlin-based **integration test** suite.

---

## Where this sits on the roadmap

The implementation validates the core bets of the
[compute-first roadmap](/roadmap): jobs really are split into many small
tasks, executed on heterogeneous nodes, and recombined — and real
transformer training runs across multiple physical nodes today.

| Area | Status |
|------|--------|
| Compute agent (PNode) | ✅ Implemented, GPU-accelerated, mixed precision |
| Orchestrator | ✅ Implemented, incl. training sessions and failover |
| Protocol (gRPC) | ✅ Implemented, async training hops |
| End-to-end distributed jobs | ✅ Multi-node transformer training works |
| Training workloads | 🔄 In progress — LLM-scale runs, validation, datasets |
| Open network membership & discovery | 🔜 Design phase (currently orchestrator-managed) |
| Trust & validation at scale | 🔜 Design phase |
| Inference | ⏳ Deliberately later |

## What's next

- Scaling training runs toward the model sizes described in the
  [training strategy](/algorithms) (124M+ parameter decoder-only models with
  RoPE and 2K context).
- Hardening validation, checkpointing, and dataset pipelines.
- Additional GPU backends beyond Metal.
- Moving from orchestrator-managed membership toward the open, decentralized
  [membership and discovery](/network-membership) design.
