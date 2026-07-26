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

The system is therefore designed to take **large computational jobs**, split them into **many small tasks**, distribute them across heterogeneous nodes, and reliably recombine the results.

---

### See also

- [Architecture Overview](/architecture)
- [Physical Node (PNode)](/pnode)
- [Orchestrator](/orchestrator)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)
- [Algorithms](/algorithms)
- [Models](/models)
- [Threat Model](/threat-model)

## Phase 0 — Foundations

**Goal:** Establish shared assumptions, boundaries, and architectural clarity.

### Key Outcomes
- Clear separation between **agent**, **orchestrator**, **protocol**, and **network membership**
- Explicit non-goals (e.g. no permissioned access, no mandatory tokenomics)
- Definition of minimal compute primitives

### Deliverables
- High-level architecture overview
- Terminology and glossary
- Initial protocol sketches

---

## Phase 1 — Compute Agent

**Goal:** Allow anyone to join the network by running a lightweight agent.

### Description
The **compute agent** is a small, cross-platform program written in **Go**.  
It is designed for easy installation, low overhead, and safe execution of bounded compute tasks.

The agent exposes a minimal set of **efficient low-level operators** (e.g. matrix multiplication, vector operations, reductions) that can be composed into higher-level workloads.

### Design Principles
- Cross-platform (Linux, macOS, Windows)
- Explicit resource limits (CPU, memory)
- Stateless by default
- Fully observable and inspectable

### Deliverables
- `compute-agent` reference implementation
- Operator API (v1)
- Local execution and remote invocation support

---

## Phase 2 — Orchestrator

**Goal:** Reliably execute large jobs across unreliable, heterogeneous nodes.

### Description
The **orchestrator** is responsible for:

- Splitting large jobs into small tasks
- Matching tasks to nodes based on capabilities
- Dispatching tasks to agents
- Collecting and validating results
- Handling failures, retries, and node churn

The orchestrator must assume that:
- Nodes can disappear at any time
- Tasks may fail or be delayed
- Execution order is non-deterministic

### Deliverables
- `compute-orchestrator` reference implementation
- Task lifecycle model
- Failure detection and retry strategy

---

## Phase 2.5 — Shared Protos & Data Schemas

**Goal:** Stabilize shared protobuf definitions for agent↔orchestrator and data references.

### Deliverables
- `compute-protos` repository (Node/Task/Result/ValidationReceipt)
- Tensor/Content and DataRef messages (URI + headers/creds + byte ranges)
- Operator identifiers and shapes; Operator Set versioning

---

## Phase 3 — Protocol (Agent ↔ Orchestrator)

**Goal:** Define a clear, efficient, and evolvable communication layer.

### Description
Communication between agents and orchestrators is implemented using **gRPC**.

The protocol defines:
- Node capability advertisement
- Task dispatch and acknowledgment
- Result submission
- Heartbeats and liveness signals

### Design Principles
- Versioned APIs
- Backward compatibility
- Explicit resource declarations

### Deliverables
- `compute-protocol` (protobuf definitions)
- Reference gRPC implementations
- Protocol documentation

---

## Phase 3a — Kotlin Client Library & CLI

**Goal:** Provide a minimalistic frontend for Kotlin programs.

### Deliverables
- DSL for core ops (e.g., `A.matmul(B).add(C).relu().reduceSum()`)
- Automatic use of DataRefs for large tensors
- Simple CLI/submitter tool for demos

---

## Phase 4 — Network Membership & Discovery

**Goal:** Enable nodes to join and leave the network without central authority.

### Description
Nodes must be able to:
- Join the network autonomously
- Advertise their capabilities (CPU, memory, architecture)
- Discover orchestrators or peers

The mechanism must be:
- Decentralized
- Fault-tolerant
- Free of single points of failure

### Possible Approaches
- Gossip-based discovery
- Distributed hash tables (DHTs)
- Peer bootstrap lists

(No final approach is enforced at this stage.)

### Deliverables
- Membership and discovery design
- Initial implementation
- Threat model and failure analysis

---

## Phase 4a — Data Service (MVP)

**Goal:** Register datasets, return handles, and issue short‑lived access links.

### Deliverables
- Registry API (`/data/register`, `/data/{handle}`, `/data/sign`)
- Metadata store (dtype, shape, partitioning, checksums)
- Optional link signing for S3/HTTP backends

---

## Phase 5 — End-to-End Distributed Jobs

**Goal:** Run real workloads across the network.

### Description
At this stage, the system can:
- Accept a large computational job
- Split it into many small tasks
- Execute tasks across heterogeneous nodes
- Recompose a final result

Initial workloads may include:
- Large matrix operations
- Synthetic benchmarks
- Toy machine-learning workloads

### Deliverables
- End-to-end demo
- Benchmarks and metrics
- Operational documentation
 - Demo assets: `docker-compose`, submitter script, dashboards, test vectors wired

---

## Phase 6 — Training Workloads

**Goal:** Use the compute network for real model training.

### Description
Training algorithms developed in parallel can now target this infrastructure.

Key challenges include:
- Parameter sharding
- Synchronization vs asynchrony
- Fault-tolerant optimization

### Deliverables
- First distributed training runs
- Training-specific orchestration extensions
- Lessons learned and architectural revisions

---

## Phase 7 — Inference (Later)

**Goal:** Support inference workloads using the same network.

Inference is **not a priority** and will be addressed only after training is stable and well understood.

---

## Guiding Principle

> **Participation first. Performance later.**

The primary success metric is not raw FLOPS, but the number of people who can **meaningfully participate** in building and running the system.

---

## Status

This roadmap is **living documentation**.  
Phases may overlap, evolve, or be re-ordered as the project grows.

**As of July 2026**, the [compute-all](https://github.com/Free-LLM/compute-all)
implementation has working equivalents of the compute agent (PNode), the
orchestrator, and the gRPC protocol, and runs **end-to-end distributed
training of transformer networks across multiple nodes** — with GPU
acceleration, fp16/bf16 mixed precision, a tokenized dataset service, and a
live monitoring dashboard. Open network membership, trust & validation at
scale, and inference remain ahead.

→ See the detailed [Project Status](/status) page.