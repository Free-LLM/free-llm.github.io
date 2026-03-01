---
title: Orchestrator
layout: default
nav_order: 5
---

# Orchestrator

The **orchestrator** is the coordinating component of the distributed computing system.

Its role is to manage the **network topology**, allocate **Virtual Nodes (VNodes)** across **Physical Nodes (PNodes)**, and provide location resolution for distributed communication.

Unlike traditional centralized schedulers, the orchestrator is designed to operate in an environment where **failure is normal and expected**.

---

### See also

- [Architecture Overview](/architecture)
- [Physical Node (PNode)](/pnode)
- [Virtual Node (VNode)](/vnode)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)
 - [Data Sources & Data Service](/data-sources)

---

## Role in the System

The orchestrator is responsible for **global coordination**, but not for execution.

It:
- Manages **PNode registration** and health monitoring via heartbeats
- Stores **network definitions** and topology (VNode parent-child relationships)
- Handles **VNode allocation** to PNodes based on availability and capability
- Provides **VNode location resolution** for distributed P2P communication between VNodes
- Exposes gRPC API for network management and training/inference operations

It does **not**:
- Execute computation itself
- Assume PNodes or VNodes are reliable
- Require global consensus

---

## Design Goals

1. **Fault tolerance by default**  
   PNode failures, disconnects, and slow responses are expected and handled via re-allocation.

2. **Dynamic allocation**  
   VNodes are allocated based on real-time PNode capabilities and availability.

3. **Eventual completion**  
   Progress is asynchronous; correctness and resilience matter more than peak speed.

4. **Statelessness where possible**  
   Orchestrators should be replaceable; critical state is persisted in durable storage (e.g., DynamoDB).

5. **Horizontal scalability**  
   Multiple orchestrators may coexist without tight coordination by sharing the same backend storage.

---

## Network Model

A **Network** represents a composable neural runtime configuration submitted by a user.

A Network:
- Is defined by a set of VNodes and their connections (topology)
- Can be trained or used for inference via the orchestrator's entry points
- May include inputs as explicit tensors or as references to external data

---

## VNode Allocation Model

The orchestrator manages the mapping of **VNodes** to **PNodes**.

Allocation properties:
- **Lazy instantiation**: PNodes only create VNodes when assigned or requested.
- **Location Transparency**: VNodes discover each other via the orchestrator's resolution service.
- **Gradient Locality**: Backpropagation is handled locally by VNodes, reducing the need for global graph coordination.

This enables safe retries and redundant execution.

---

## Scheduling Strategy

VNode assignment is **capability-aware**.

The orchestrator considers:
- Declared PNode hardware (CPU, memory, accelerators)
- Current PNode load (number of allocated VNodes)
- Historical reliability and trust status
- Data locality when VNodes use external data references

---

## Failure Handling

Failures are handled by re-allocating VNodes.

The orchestrator may:
- Detect PNode failure via missing heartbeats
- Mark a PNode as unavailable and deallocate its VNodes
- Reschedule VNodes to different PNodes
- Execute VNodes redundantly for validation

---

## Result Validation & PNode Health

To maintain reliability with permissionless participation, the orchestrator applies validation to results from untrusted PNodes:

- Perform lightweight checks (e.g., invariants, checksums, or redundant execution) before accepting a result.
- Mark PNodes as "bugged" if they fail validation beyond a threshold.
- Prefer trusted PNodes for critical paths.

Trust is a scheduling optimization, not a correctness requirement.

---

## State Management

The orchestrator maintains minimal durable state in a backend like DynamoDB:
- PNode registry and status
- VNode allocation map
- Network topology definitions

State is independent of specific PNodes or orchestrator instances.

---

## Relationship to Physical Nodes (PNodes)

The orchestrator treats all PNodes as:
- Ephemeral
- Untrusted
- Replaceable

This assumption simplifies orchestration logic and ensures the system survives massive churn.

---

## Relationship to Protocol

All interaction with PNodes and VNodes happens through the **gRPC protocol**.

The orchestrator:
- Never bypasses the protocol
- Uses versioned APIs for backward compatibility
- Provides a resolution service for VNode-to-VNode communication

---

## Relationship to Training & Inference

The orchestrator provides the execution substrate for **DCNR**.

Training systems express their needs by defining a network of VNodes. The orchestrator then ensures these VNodes are alive, allocated, and reachable, while the VNodes themselves handle the mathematical operations and gradient updates.

---

## Summary

The orchestrator is the system’s **coordination brain**, but not its executor.

By assuming unreliable nodes and embracing asynchrony, it enables large-scale computation to emerge from many small, voluntary contributions.

This design trades peak efficiency for **resilience, openness, and scalability through participation**.
