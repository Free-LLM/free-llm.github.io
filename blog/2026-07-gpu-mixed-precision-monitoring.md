---
title: "GPU acceleration, mixed precision, and a live dashboard"
layout: default
parent: Blog
nav_order: 1
permalink: /blog/2026-07-gpu-mixed-precision-monitoring
---

# GPU acceleration, mixed precision, and a live training dashboard

_July 2026_

The [previous post](/blog/2026-04-first-multi-node-training) ended with a
system that could train a network spread across several machines. The work
from May and June, visible in the
[`compute-all`](https://github.com/Free-LLM/compute-all) history, tackled the
obvious next questions: can it be **fast enough to matter**, and can you
**see what it's doing** while it runs? The answer to both turned out to be
yes.

## Watching the network train

Debugging a distributed training run over log files is miserable, so
observability came first:

- A **monitoring backend** — a serverless collector that receives
  request-status updates from every PNode and training-step metrics from the
  orchestrator, merging them into a per-session view in DynamoDB.
- A **GraphQL API** on top, with user-scoped sessions and API keys.
- A **Vue 3 dashboard** that renders live training charts, the real network
  topology as an interactive graph (with per-node memory details), and a
  PNode view showing hosted VNodes, process and GPU memory, and pending
  requests per node.

For a project whose principles include _transparency over convenience_, this
is more than tooling: anyone running or joining a training session can see
exactly what the network is doing, in real time.

## Making the runtime fast

With eyes on the system, the bottlenecks were visible — and May went after
them:

- A refactor of **internal tensor access** in the runtime, followed by
  **GPU acceleration** (a Metal backend) for the hot operations. The
  runtime's interfaces were kept backend-agnostic, so CUDA/ROCm ports slot
  into the same seams.
- **Mixed precision**: full `fp16` and `bf16` support through the tensor
  runtime — construction, casts, views, persistence, and ops — with
  per-layer precision configurable in the network YAML. Optimizers keep
  **fp32 master parameters** (with fp32 Adam state) so reduced-precision
  training stays numerically stable, helped by dynamic loss scaling. On the
  GPU side this includes fp16 typed kernels and a **fused Adam update** that
  writes back to fp16 weights.

Halving the memory per parameter isn't a micro-optimization for this
project — it's mission-critical. The whole premise is _many small machines
with normal resources_, and mixed precision roughly doubles what each
volunteer machine can host.

## A real data pipeline

Training is only as good as the data you feed it, and early summer built out
that layer:

- A **dataset API** that defines and serves datasets in tokenized format, so
  PNodes consume ready-to-train token streams instead of raw text.
- **Dataset splitting** with multiple strategies, giving every training run
  a proper held-out validation set.
- A round of hardening on the **BPE tokenizer and vocabulary training** —
  the repo now carries trained vocabularies up to 256K tokens.

## Smarter training sessions

The orchestrator grew from an allocator into a **trainer**:

- Trainer logic moved **into the orchestrator**, with new protocol messages
  for session management — training runs are first-class, resumable
  sessions rather than CLI-driven loops.
- An **optimized Adam path** in the orchestrator's training loop.
- **Validation support** end to end: cost-function nodes, orchestrator
  validation logic, and CLI commands, so a session tracks generalization,
  not just training loss.

## What this adds up to

Take stock of the stack as it stands: YAML-defined transformer networks
(attention with RoPE, layer norm, SwiGLU-style blocks, tied weights),
distributed across physical nodes with locality-aware allocation and
automatic failover, training with GPU-accelerated mixed precision, fed by a
tokenized dataset service with validation splits, observable on a live
dashboard. Every piece open source.

There is plenty left — scaling to the model sizes in our
[training strategy](/algorithms), opening up
[network membership](/network-membership) beyond orchestrator-managed
fleets, and the [trust and validation](/trust-and-validation) work that a
permissionless network demands. The current state is tracked on the
[Project Status](/status) page, and if any of this sounds like something you
want to build, the door is open: [Contributing](/contributing).
