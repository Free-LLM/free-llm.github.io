---
title: Project Status
layout: default
nav_order: 11
---

# Project Status

_Last updated: August 2026 — based on the current state of the
[`compute-all`](https://github.com/Free-LLM/compute-all) repository. All
previously in-flight optimization work has merged; there are no open pull
requests._

The **Distributed Composable Neural Runtime (DCNR)** has moved well past the
design phase: there is a working implementation that can define a transformer
network in YAML, distribute its components across multiple physical nodes,
train it with mixed precision on **both Apple Silicon (Metal) and NVIDIA
(CUDA) GPUs**, and watch the run live on a web dashboard.

This page summarizes what exists today. For the longer-term plan, see the
[Roadmap](/roadmap). For the story of how we got here, see the [Blog](/blog).
The networks currently being trained are described in
[Production Training Networks](/training-topologies).

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
- **Fused, memory-efficient kernels**:
  - **FlashAttention-2-style tiled causal attention** (forward and backward)
    which never materializes the `[batch, seq, seq]` attention matrix,
    keeping only a compact per-row logsumexp statistic. Default-on.
  - **Fused tied LM head + cross-entropy**, which computes the loss and all
    three gradients without ever materializing the full
    `[batch, seq, vocab]` logits tensor. Auto-enabled above a vocabulary
    threshold (default 16,384).

### GPU backends

Both major consumer/datacenter GPU families are supported, dispatching
through a common `Device` interface with a CPU fallback:

- **Metal** (Apple Silicon) — the original backend: typed fp16 kernels,
  fused Adam, tiled attention, fused LM-head cross-entropy.
- **CUDA** (NVIDIA) — brought to **full capability parity with Metal** in a
  phased effort validated on real Tesla T4 hardware: typed fp16/bf16 matmul
  and elementwise ops (cuBLAS with fp32 accumulation), RoPE, layer norm,
  embeddings, the linear parameter-gradient family, attention (including
  FA2-style tiled causal attention and head split/merge), fused LM-head
  cross-entropy, the Adam variants, and GeLU/SwiGLU device paths. A
  hardware-aware precision policy picks fp16 below compute capability 8.0
  and native bf16 at 8.0+.
- The production PNode deploy image is genuinely **CUDA-enabled** (verified
  on live hardware reporting `CUDA available Tesla T4`), and still starts
  cleanly and reports no GPU on non-GPU machines.
- Validation runs on **ephemeral AWS GPU instances** provisioned by
  Terraform, with a persistent Bazel cache volume so runs don't start cold.

### Virtual Nodes (VNodes)

Each neural-network component is a stateful agent that can be placed on any
physical node — see the full [node type catalog](/vnode). The registry
covers:

- **Layers**: `linear`/`dense`, `layernorm`, `embedder`, `attention_head`
  (the distributed per-head unit), `attention` (fused multi-head), and
  `tied_lm_head_ce` (fused tied LM head + cross-entropy).
- **Position encoding**: RoPE.
- **Activations & regularization**: `relu`, `gelu`, `silu`, `swiglu`,
  `dropout`, masking (including causal masks).
- **Structural nodes**: `broadcast`, `stack`, `add`, `reshape`, plus shared
  and tied weights across nodes.
- **Text I/O**: `tokenizer` and `detokenizer` nodes backed by a trainable
  BPE tokenizer.
- **Cost functions** for training (`mse`, `ce`, `ce_logits`) plus validation
  support.

All VNodes obey **gradient locality**: `Backward()` uses only local
parameters, locally cached inputs, and the incoming gradient — no global
computational graph — which is what makes extreme horizontal distribution
possible.

### Orchestrator

- PNode **registration, heartbeats, status tracking, and automatic
  failover** (VNodes on a dead node are reallocated).
- **YAML network definitions** with cycle detection and topology validation.
- **Network optimization passes** that rewrite a submitted topology into
  fewer, larger VNodes before allocation, without changing how you author
  the YAML — currently attention-head fusion and tied-LM-head/cross-entropy
  fusion. Both are conservative (anything not fully recognized is left
  untouched) and checkpoint-compatible.
- **Locality-aware allocation**: adjacent VNodes are co-located on the same
  PNode where possible, with colocation groups and load balancing.
- **Training sessions**: the trainer lives inside the orchestrator, with
  dedicated protocol messages for session management, an optimized Adam
  path, and training-step metrics reporting.
- **Persistence and crash recovery** via DynamoDB, plus maintenance
  utilities (network sync, VNode age repair).

### Protocol & communication

- gRPC APIs for orchestrator↔PNode and PNode↔PNode communication — see
  [Protocol](/protocol).
- Training hops between PNodes are **asynchronous**
  (`ReceiveTrainRequest`/`ReceiveTrainResponse` with request IDs), removing
  the head-of-line blocking of the earlier synchronous design.
- A **TrainScheduler** on each PNode tracks each request's local work with
  Dijkstra–Scholten-style work tokens, so it knows precisely when a request
  is idle locally (waiting only on remote branches) and its cached state
  could be offloaded. Compute is gated at the VNode join points rather than
  at gRPC arrival, with slot limits and backward-priority queues.

### Data & tokenization

- A **trainable BPE tokenizer** with vocabulary training (vocabularies up to
  256K tokens are checked into the repo), state saving, and text
  preprocessing utilities.
- A **dataset API** that defines and serves datasets in tokenized format,
  with **dataset splitting** (train/validation) using multiple strategies.

### Observability

- A **monitoring backend** (serverless, DynamoDB-backed) that collects
  request-status updates from PNodes and training-step metrics from the
  orchestrator, exposed through a **cursor-paginated GraphQL API**.
- A **Vue 3 dashboard** with per-session training charts, a live network
  topology graph (real stored topology, per-node memory details), and a
  PNode dashboard (hosted VNodes, process/GPU memory, pending requests).
- Long-session ergonomics: full metric history paged in progressively,
  per-metric summary statistics (avg/min/max/stdev) instead of raw
  per-batch values, per-request histograms, chart zoom, and CSV export.

### Tooling & engineering

- A **CLI (`dcnr`)** for creating/inspecting networks, running training,
  driving the tokenizer and datasets, analyzing topologies
  (`network analyze` reports shapes and parameter counts), and listening to
  monitoring streams.
- **Bazel** builds for everything (with plain `go build` kept working),
  Docker images, and a Kotlin-based **integration test** suite.

---

## Training performance

The production workload is the 8-layer / 32K-vocab / 512-context network
(see [Production Training Networks](/training-topologies)), trained on a
single Apple M3 Max PNode. Average step time over successive optimization
rounds, each confirmed on long-running production sessions rather than
synthetic benchmarks alone:

| Milestone | Avg step time |
|---|---|
| Before the fused LM-head work | ~20 min (regressed path) / ~240 s |
| Fused LM-head cross-entropy kept on GPU | ~175 s |
| Attention-head fusion (72 h uninterrupted run) | ~152 s |
| FA2-style tiled attention default-on | **~100 s** |

A mixed-dtype CUDA dispatch bug — an fp32 gradient meeting an fp16 weight
took an illegal-memory-access path — was found only by an end-to-end
training run and cost roughly a 41× slowdown before being fixed. The
recommended maximum batch size for a single `g4dn.xlarge` (T4) on this
fixture is 32.

---

## Where this sits on the roadmap

The implementation validates the core bets of the
[compute-first roadmap](/roadmap): jobs really are split into many small
tasks, executed on heterogeneous nodes, and recombined — and real
transformer training runs across multiple physical nodes today.

| Area | Status |
|------|--------|
| Compute agent (PNode) | ✅ Implemented, GPU-accelerated, mixed precision |
| Orchestrator | ✅ Implemented, incl. training sessions, fusion passes, failover |
| Protocol (gRPC) | ✅ Implemented, async training hops, per-PNode scheduler |
| GPU backends | ✅ Metal and CUDA at capability parity |
| End-to-end distributed jobs | ✅ Multi-node transformer training works |
| Training workloads | 🔄 In progress — scaling up model size and throughput |
| Open network membership & discovery | 🔜 Design phase (currently orchestrator-managed) |
| Trust & validation at scale | 🔜 Design phase |
| Inference | ⏳ Deliberately later |

## What's next

- Scaling training runs toward the model sizes described in the
  [training strategy](/algorithms) (124M+ parameter decoder-only models with
  RoPE and 2K context).
- Completing the train-concurrency work so multiple training steps overlap
  safely on one PNode, with state offload for idle requests.
- Multi-GPU and multi-node NVIDIA deployments now that CUDA is at parity.
- Hardening validation, checkpointing, and dataset pipelines.
- Moving from orchestrator-managed membership toward the open, decentralized
  [membership and discovery](/network-membership) design, with the
  [trust and validation](/trust-and-validation) mechanisms and a completed
  [threat model](/threat-model) it requires.
