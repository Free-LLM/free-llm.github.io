---
title: Compute Agent
layout: default
nav_order: 4
---

# Compute Agent

The **compute agent** is the fundamental building block of the distributed computing network.

It is the primary interface through which individuals contribute computational resources. As such, it is designed to be **simple, safe, transparent, and easy to run**.

---

### See also

- [Architecture Overview](/architecture)
- [Orchestrator](/orchestrator)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)

## Role in the System

The compute agent is responsible for **executing small, bounded computational tasks** on behalf of the network.

It does **not**:
- Coordinate other nodes
- Split large jobs
- Make global scheduling decisions

Those responsibilities belong to the orchestrator.

---

## Design Goals

1. **Ease of participation**  
   Anyone should be able to join the network with minimal setup.

2. **Safety by default**  
   Tasks must be executed within strict resource and execution boundaries.

3. **Deterministic behavior**  
   Given the same inputs, an agent should produce the same outputs.

4. **Observability**  
   All behavior should be inspectable and debuggable by the operator.

5. **Replaceability**  
   Agents are disposable and interchangeable.

---

## Implementation Language

The reference implementation is written in **Go**.

Reasons for this choice:
- Single static binary distribution
- Strong concurrency primitives
- Good cross-platform support
- Mature gRPC ecosystem

Alternative implementations are not forbidden, but Go is the baseline.

---

## Agent Lifecycle

1. **Startup**
   - Load configuration
   - Detect local hardware capabilities
   - Initialize operator registry

2. **Network Join**
   - Discover peers or orchestrators
   - Advertise capabilities

3. **Idle State**
   - Await task assignments
   - Send periodic heartbeats

4. **Task Execution**
   - Validate task request
   - Enforce resource limits
   - Execute compute operator

5. **Result Reporting**
   - Return result or error
   - Release resources

6. **Shutdown**
   - Graceful termination
   - No global cleanup required

---

## Capability Advertisement

Each agent advertises a **capability descriptor** that may include:

- CPU architecture and core count
- Available memory
- Operating system
- Supported operator set
- Optional accelerators (if supported)

Capabilities are **descriptive, not prescriptive**. The agent never commits to executing tasks beyond its local limits.

---

## Operator Model

Agents expose a **small, fixed set of compute operators**.

### Examples
- Matrix multiplication
- Vector addition
- Reductions (sum, max, min)
- Element-wise operations

Operators:
- Have explicit input and output sizes
- Must be side-effect free
- Must run within declared resource bounds

Complex workloads are built by **composing operators**, not by expanding agent complexity.

---

## Resource Enforcement

The agent is responsible for enforcing **local execution limits**.

### Enforced Constraints
- Maximum memory usage
- CPU time limits
- Task-level timeouts

If a task exceeds limits, it is aborted and reported as failed.

The agent always prioritizes **local system stability** over task completion.

---

## Failure Semantics

Failures are expected and normal.

The agent may:
- Reject a task before execution
- Fail during execution
- Disconnect unexpectedly

The agent makes **no attempt to recover tasks**. Recovery is handled entirely by the orchestrator.

---

## Trust & Security Model

The agent operates under a **zero-trust assumption**.

- Tasks are treated as untrusted input
- The orchestrator is not implicitly trusted
- Other agents are never trusted

Security measures include:
- Strict input validation
- Resource sandboxing
- Minimal exposed surface area

Cryptographic identity and attestation are considered future extensions.

---

## Configuration

Agents are configured explicitly by the operator.

Typical configuration options:
- Maximum CPU usage
- Maximum memory usage
- Network endpoints
- Enabled operators

There is no automatic escalation of privileges.

---

## What the Agent Is Not

The compute agent is **not**:
- A container runtime
- A general-purpose execution environment
- A scheduler
- A blockchain node

Keeping the agent minimal is a deliberate design choice.

---

## Relationship to the Orchestrator

The agent is **reactive**, not proactive.

It executes tasks assigned by orchestrators but does not attempt to reason about the global system.

This asymmetry keeps the agent simple and reduces the attack surface.

---

## Summary

The compute agent is intentionally boring.

Its value comes from **numbers**, not sophistication: many simple agents, run by many people, cooperating through a transparent and fault-tolerant system.

This simplicity is what enables decentralization.
