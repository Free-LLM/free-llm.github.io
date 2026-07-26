---
title: Roadmap
layout: default
nav_order: 10
---

# Roadmap

This roadmap follows a **compute-first strategy**.

Our primary objective is to build a **decentralized, volunteer-powered distributed computing network** as early as possible, allowing the community to participate immediately.  
Once the network is operational, it will be progressively used for **model training** and, eventually, **inference**.

The guiding assumption is simple:

> If we cannot rely on a few machines with massive memory,  
> we can rely on **many small machines with normal resources**.

The system is therefore designed to take **large computational jobs**, split them into **many small units**, distribute them across heterogeneous nodes, and reliably recombine the results.

Each phase below carries its **current status**. Where the implementation
diverged from the original plan, the divergence is noted — the original
design documents are preserved in the [Archive](/archive).

| Legend | |
|---|---|
| ✅ | Done — implemented and working in [`compute-all`](https://github.com/Free-LLM/compute-all) |
| 🔄 | In progress |
| 🔜 | Ahead — design phase |
| ⏳ | Deliberately later |

---

### See also

- [Architecture Overview](/architecture)
- [Physical Node (PNode)](/pnode) · [Virtual Node (VNode) & Node Types](/vnode)
- [Orchestrator](/orchestrator) · [Protocol](/protocol)
- [Network Definition (YAML)](/network-definition) · [Production Training Networks](/training-topologies)
- [Network Membership & Discovery](/network-membership) · [Trust & Validation](/trust-and-validation)
- [Project Status](/status)

---

## Phase 0 — Foundations ✅

**Goal:** Establish shared assumptions, boundaries, and architectural clarity.

Delivered: the high-level architecture (orchestrator / worker / protocol /
membership separation), explicit non-goals (no permissioned access, no
mandatory tokenomics, single-service scope), terminology, and the initial
design sketches now kept in the [Archive](/archive).

---

## Phase 1 — Compute Worker (PNode) ✅

**Goal:** Allow anyone to join the network by running a lightweight worker.

**Delivered** as the [Physical Node (PNode)](/pnode): a Go worker process
that registers with the orchestrator, advertises its hardware, hosts
[Virtual Nodes](/vnode), enforces local limits, and persists VNode state to
S3-compatible storage.

**How the design evolved:** the original plan was a generic agent exposing a
small [operator set](/operator-set-v1) (matmul, element-wise ops,
reductions). The implementation went a level higher: workers host **typed
neural VNodes** (`linear`, `attention_head`, `layernorm`, `swiglu`, …) that
carry their own parameters, optimizer state, and backward logic under
**gradient locality**. This is what made distributed training practical.

---

## Phase 2 — Orchestrator ✅

**Goal:** Reliably execute large jobs across unreliable, heterogeneous nodes.

**Delivered** and exceeded: the [orchestrator](/orchestrator) manages PNode
registration/heartbeats, [YAML network definitions](/network-definition)
with validation and optimization passes (attention-head fusion),
locality-aware VNode allocation with colocation groups, automatic failover,
and DynamoDB persistence with crash recovery. The **trainer itself now runs
inside the orchestrator** as resumable training sessions — a responsibility
the original plan hadn't assigned to it.

---

## Phase 3 — Protocol ✅

**Goal:** Define a clear, efficient, and evolvable communication layer.

**Delivered**: gRPC services with versioned protobuf schemas in
[`compute-all/api`](https://github.com/Free-LLM/compute-all/tree/main/api) —
control plane (networks, sessions, datasets, vocabularies), node management
(registration, heartbeats, location resolution), storage (pre-signed URLs),
and the **asynchronous PNode-to-PNode training protocol**
(`ReceiveTrainRequest`/`ReceiveTrainResponse`). See [Protocol](/protocol);
the earlier task/operator protocol design is
[archived](/archive/legacy-protocol).

---

## Phase 3a — Clients & Tooling ✅

**Goal:** Give humans and programs a practical way to drive the network.

**Delivered** as the **`dcnr` CLI** (Go): network creation and inspection,
training sessions, dataset and tokenizer management, live monitoring.
The originally planned Kotlin DSL client was not pursued; Kotlin survives
as the integration-test harness. A client library API for third-party
programs remains future work.

---

## Phase 4 — Network Membership & Discovery 🔜

**Goal:** Enable nodes to join and leave the network without central authority.

Today membership is **orchestrator-managed** (register + heartbeat), which
is fine for trusted fleets but not yet permissionless. The decentralized
design — gossip, DHTs, or bootstrap lists — is still ahead, together with
the [trust & validation](/trust-and-validation) mechanisms and a completed
[threat model](/threat-model) that permissionless participation requires.

---

## Phase 4a — Data & Tokenization ✅

**Goal:** Make training data available to the network in a usable form.

**Delivered**, folded into the orchestrator rather than as the originally
planned standalone [Data Service](/data-sources): a trainable **BPE
tokenizer** with vocabulary management APIs, **preprocessed datasets served
in tokenized form**, dataset splitting (train/validation), and pre-signed
URL access to S3-compatible storage so workers never hold long-lived
credentials.

---

## Phase 5 — End-to-End Distributed Jobs ✅

**Goal:** Run real workloads across the network.

**Delivered**: transformer networks defined in YAML are distributed across
multiple PNodes and trained end-to-end, observable live on the monitoring
dashboard (training charts, network topology graph, per-PNode resources).

---

## Phase 6 — Training Workloads 🔄

**Goal:** Use the compute network for real model training.

**In progress — this is the current focus.** Working today: the two
[production topologies](/training-topologies) (up to 8 layers, 64
distributed attention heads, 32K vocab, 512 context, ≈45M parameters)
training with fp16 mixed precision, GPU acceleration, and validation
splits. The in-flight [performance push](/status#in-review-the-performance-push-pr-83)
(FlashAttention-2-style tiled attention, fused LM-head cross-entropy,
attention-head fusion) has production steps at roughly 175 seconds.

Next: scaling toward the 124M+ parameter decoder-only models described in
the [training strategy](/algorithms), hardening checkpointing and
validation, and GPU backends beyond Metal.

---

## Phase 7 — Inference ⏳

**Goal:** Support inference workloads using the same network.

Basic forward/chat RPCs exist for exercising trained networks, but
inference at scale is **deliberately deferred** until training is stable
and well understood.

---

## Guiding Principle

> **Participation first. Performance later.**

The primary success metric is not raw FLOPS, but the number of people who can **meaningfully participate** in building and running the system.

---

## Status

This roadmap is **living documentation** — phases may overlap, evolve, or
be re-ordered as the project grows. Statuses above reflect **July 2026**;
the detailed, regularly updated picture (including work still in review)
is on the [Project Status](/status) page.
