---
title: Home
layout: default
nav_order: 1
---

# Open, Decentralized Artificial Intelligence

We believe that artificial intelligence should be a **shared public resource**, not a closed system controlled by a few.

[Read more about why we are doing this.](/why)

This initiative exists to build a **fully open-source AI ecosystem** — from training algorithms to infrastructure to pre-trained models — developed transparently and owned by the community.

Importantly, our distributed network is designed to operate a **single community-governed model and service**. It is not a general-purpose public computing platform. Governance for training data, ethics, safeguards, and bias will be handled by a community process when the model reaches maturity.

---

## Mission

Our mission is to create an AI stack where:

- Algorithms are **open and auditable**
- Pre-trained models are **freely accessible**
- Compute is **distributed, voluntary, and decentralized** — but narrowly scoped to serving and training a single open model/service
- Training is **efficient without massive memory requirements**

Every layer of the system is designed to be inspectable, reproducible, and community-owned.

---

## Core Projects

### 🧠 Algorithms
Research into memory-efficient, decentralized training algorithms that work on heterogeneous machines and compose well with the distributed architecture.

→ [Algorithms](/algorithms)

---

### 🌐 Distributed Architecture
The **Distributed Composable Neural Runtime (DCNR)** is a permissionless, fault-tolerant compute network. It uses a distributed agent-based architecture with **orchestrator-managed allocation** for composable neural networks. 

Explore the components:
- **Orchestrator**: Central coordination for network topology and node allocation.
- **Physical Nodes (PNodes)**: Compute worker processes that host local execution.
- **Virtual Nodes (VNodes)**: Stateful agents representing neural network components (layers, activations, cost functions).
- **Gradient Locality**: Distributed training without global coordination, using local parameters and gradients.

→ [Distributed Architecture](/distributed-architecture)

---

### 📦 Models
Fully open, reproducible pre-trained models built on top of the open infrastructure.

→ [Models](/models)

---

## Principles

- Open source is non-negotiable  
- Transparency over convenience  
- Decentralization over control  
- Community ownership over profit  
- Single-service scope over general-purpose compute

---

## Roadmap & Status

Our development follows a **compute-first roadmap**: we start by building a decentralized distributed computing network, then progressively use it for training and inference.

The runtime is no longer just a design: multi-node distributed training works today, on both **Apple Silicon (Metal) and NVIDIA (CUDA)** GPUs, with mixed precision (fp16/bf16), fused attention and loss kernels, a tokenized dataset service, and a live monitoring dashboard — all developed in the open in the [compute-all](https://github.com/Free-LLM/compute-all) repository.

→ [Roadmap](/roadmap) · [Project Status](/status)

## Blog

Development notes and milestones, straight from the commit history.

→ [Blog](/blog)

## Learn the Architecture

- Overview: [Architecture Overview](/architecture)
- Components: [Orchestrator](/orchestrator) · [Physical Node (PNode)](/pnode) · [Virtual Node (VNode) & Node Types](/vnode) · [Protocol](/protocol)
- Building networks: [Network Definition (YAML)](/network-definition) · [Production Training Networks](/training-topologies)
- Network layer: [Network Membership & Discovery](/network-membership)
- Security: [Threat Model](/threat-model)
- Reliability & trust: [Trust & Validation](/trust-and-validation)

---

---

## Join the Movement

This is an evolving project.  
The design is not finished — and that is intentional.

→ [Contributing](/contributing)
