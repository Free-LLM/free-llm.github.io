---
title: Algorithms
layout: default
nav_order: 8
---

# Algorithms

## Purpose

This project focuses on **distributed training algorithms** designed to operate on the **Distributed Composable Neural Runtime (DCNR)**.

The aim is to enable training of large models using many **Virtual Nodes (VNodes)** distributed across heterogeneous **Physical Nodes (PNodes)**.

---

### See also

- [Architecture Overview](/architecture)
- [Distributed Architecture](/distributed-architecture)
- [Virtual Node (VNode)](/vnode)
- [Models](/models)
- [Protocol](/protocol)

## Design Goals

- **Gradient Locality**: Backpropagation is performed locally by each VNode using only local state and incoming gradients.
- **Asynchronous Updates**: VNodes update their parameters independently without global synchronization.
- **Fault Tolerance**: Training continues even if PNodes fail, as VNodes can be re-allocated and restored from storage.
- **Memory Efficiency**: Large models are split into many VNodes, each requiring only enough memory for its specific operation.

---

## Research Directions

- **Locality-based optimization**: Optimizing communication patterns between VNodes.
- **Asynchronous Gradient Descent**: Handling stale gradients in a distributed setting.
- **Sparsity and Modular Architectures**: Using VNodes to create dynamic, sparse networks.
- **Alternatives to Global Backpropagation**: Researching algorithms that natively support DCNR's decentralization.

---

## Repositories

- `algorithms-core`
- `algorithms-research`
- `algorithms-benchmarks`

---

## Status

This area is **research-driven and experimental**, focusing on the mathematical foundations of Gradient Locality.