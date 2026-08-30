---
title: "NVIDIA GPUs join the network — and steps get 2.4× faster"
layout: default
parent: Blog
nav_order: 0
permalink: /blog/2026-08-cuda-parity
---

# NVIDIA GPUs join the network — and steps get 2.4× faster

_August 2026_

A decentralized network that only runs well on Apple Silicon is not
decentralized enough. Until this month, DCNR's fast paths — typed fp16
kernels, fused Adam, tiled attention — existed only for **Metal**. The CUDA
backend was a fp32 stub that couldn't even link into the deployed image.
That was the single largest barrier between this project and the hardware
most volunteers actually own.

August closed that gap. The [`compute-all`](https://github.com/Free-LLM/compute-all)
history from this month is two intertwined stories: **CUDA reaching
capability parity with Metal**, and the performance work from
[July](/blog/2026-07-gpu-mixed-precision-monitoring) landing and compounding
into a 2.4× faster training step.

## Getting CUDA to build at all

Before any kernel could be written, the build had to work. That took ten
live iterations against real hardware, and the obstacles were unglamorous
in a way anyone who has fought a toolchain will recognize: scoping `nvcc` to
a single-file `genrule` instead of hijacking the global C compiler, a Go
build tag that silently excluded `cuda_device.go`, and a C++ `new`/`delete`
call that wouldn't link without `libstdc++` — fixed at the source rather
than papered over.

Two pieces of infrastructure made the rest of the month possible:

- **Ephemeral AWS GPU validation.** A Terraform module spins up a
  `g4dn.xlarge` (Tesla T4), builds, runs the test suite, and tears itself
  down — with a persistent Bazel cache volume so runs don't start cold, and
  automatic retries when capacity stalls.
- **A genuinely CUDA-enabled deploy image.** The production PNode image was
  previously a pure-Go `CGO_ENABLED=0` build that could never link CUDA at
  all. It now reports `CUDA available Tesla T4` on GPU hardware — and still
  starts cleanly and reports no GPU on machines without one.

## Nine phases to parity

With the build working, parity was pursued in numbered phases, **each one
confirmed on real T4 hardware** rather than assumed from a local Mac:

| Phase | What landed |
|---|---|
| 1 | `Device` interface conformance (all 29 methods); a real `cudaMemGetInfo` bug fixed |
| 2 | Hardware-aware bf16 policy — fp16 below compute capability 8.0, native bf16 at 8.0+ |
| 3 | Typed fp16/bf16 matmul and elementwise ops via cuBLAS with fp32 accumulation |
| 4 | RoPE and layer norm |
| 5 | Embeddings and zero-device-memory |
| 6 | The linear parameter-gradient family, mixed-reduced matmul/broadcast |
| 7 | Attention — QKV split, RoPE, dense **and FA2-style tiled causal** attention, head split/merge |
| 8 | Fused LM-head cross-entropy and the remaining Adam variants |
| — | GeLU and SwiGLU device paths, closing the last audited gap |

The T4 is a Turing card at compute capability 7.5, which is exactly why
phase 2 matters: it has no native bf16, so the policy routes it to fp16
automatically instead of failing or silently degrading. Volunteer hardware
is heterogeneous by definition, and the runtime has to know that.

## The bug that only production could find

Then a lesson worth recording honestly. After all eight phases passed their
own verification and the checklist read "full capability parity", a real
end-to-end training run hit an illegal memory access and a **~41× slowdown**.

The cause: the plain-fp32 GPU dispatch path for `MatMulBatched`, `Add`, and
`Mul` was gated on the *output* dtype instead of checking that both *input*
dtypes matched. An fp32 gradient meeting an fp16 weight — completely routine
in a backward pass — took a path that could not handle it. Every phase's
tests passed because none of them exercised that particular mixed-dtype
combination at production shapes.

"Full capability parity" turned out not to mean "every dispatch path is
correct at production shapes and dtypes." The checklist now says so
explicitly. It is the kind of distinction you only learn by running the real
workload.

## The step time story

Meanwhile the Metal-side optimization work from July landed and kept
compounding. Measured on long-running production sessions on a single M3 Max
PNode — not synthetic benchmarks — training the 8-layer / 32K-vocab /
512-context network:

| Milestone | Avg step time |
|---|---|
| Fused LM-head cross-entropy kept on GPU | ~175 s |
| Attention-head fusion (72 h uninterrupted run) | ~152 s |
| FA2-style tiled attention default-on | **~100 s** |

From ~240 s to ~100 s: a **2.4× speedup**, with every stage confirmed by a
real training session rather than a microbenchmark.

## Fusion as a first-class idea

A pattern emerged from this work and is now a proper part of the
orchestrator. You author a network in the [simple, explicit
form](/network-definition) — one VNode per attention head, a linear LM head
followed by a cost node — and the orchestrator **rewrites it into fewer,
larger VNodes before allocation**:

- `broadcast → N × attention_head → stack` becomes one multi-head VNode,
  batching GPU dispatches across all heads.
- A weight-tied `linear` feeding a `ce_logits` cost becomes one
  `tied_lm_head_ce` VNode that computes the loss and all gradients
  **without ever materializing the `[batch, seq, vocab]` logits tensor** —
  the thing that dominates memory at a 32K vocabulary.

Both passes are conservative: anything they don't fully recognize is left
alone rather than guessed at. Both preserve checkpoints, by different
routes. And crucially, both keep the *authoring* model simple while the
*execution* model gets fast — you don't hand-write the fused form, and your
existing YAML keeps working.

## Also this month

- **A per-PNode TrainScheduler** that tracks each request's local work with
  Dijkstra–Scholten-style work tokens, so it knows exactly when a request is
  idle locally and waiting only on remote branches — the precise point at
  which its cached state could be offloaded. Groundwork for overlapping
  training steps safely.
- **A much better dashboard for long runs**: cursor-paginated GraphQL,
  progressive loading of full metric history, per-metric summary statistics
  instead of raw per-batch values, per-request histograms, chart zoom, and
  CSV export.

## Where that leaves us

The network can now be joined by machines with NVIDIA GPUs running the same
fast paths as Apple Silicon, on a training step 2.4× quicker than it was in
July. For a project whose entire premise is *many small machines with normal
resources*, opening up the most common GPU family is not an optimization —
it's the difference between a demo and an invitation.

Still ahead: scaling toward larger models, overlapping training steps,
multi-GPU NVIDIA deployments, and the [open membership](/network-membership)
and [trust](/trust-and-validation) work that permissionless participation
requires. The current state always lives on the
[Project Status](/status) page — and if you want to help, the door is open:
[Contributing](/contributing).
