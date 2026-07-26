---
title: Distributed Architecture
layout: default
nav_order: 3
---

# Distributed Architecture

## Purpose

To provide a **decentralized, permissionless execution layer** dedicated to a single, community-governed AI model/service.

The system is designed to survive partial failures and operate without trusted coordinators.

---

## Core Principles

- No single point of failure  
- No central authority  
- Graceful degradation  
- Horizontal scalability  
 - Single-service scope over general-purpose compute  

---

## Key Concepts

- Peer-to-peer coordination  
- Task and model sharding  
- Redundant computation  
- Bandwidth-aware protocols  
- Incentive-neutral participation  
 - Trust & validation for permissionless agents  

---

### See also

- [Architecture Overview](/architecture)
- [Physical Node (PNode)](/pnode)
- [Orchestrator](/orchestrator)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)
 - [Trust & Validation](/trust-and-validation)

## Repository

The implementation lives in
[`compute-all`](https://github.com/Free-LLM/compute-all): orchestrator,
PNode runtime, CLI, tokenizer, and monitoring stack in a single Bazel
workspace.

---

## Status

Actively evolving — see [Project Status](/status). Expect breaking changes.