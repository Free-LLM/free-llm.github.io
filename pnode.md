---
title: Physical Node (PNode)
layout: default
nav_order: 4
---

# Physical Node (PNode)

The **Physical Node (PNode)** is the fundamental building block of the distributed computing network.

It is the primary interface through which individuals contribute computational resources. As such, it is designed to be **simple, safe, transparent, and easy to run**.

---

### See also

- [Architecture Overview](/architecture)
- [Orchestrator](/orchestrator)
- [Virtual Node (VNode)](/vnode)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)
 - [Data Sources & Data Service](/data-sources)

---

## Role in the System

The Physical Node is a **compute worker process** (typically a Docker container or Kubernetes pod) responsible for hosting **Virtual Nodes (VNodes)**.

It handles the physical execution, networking, and storage for the VNodes allocated to it by the orchestrator.

## Key Responsibilities

1. **Auto-registration**: Registers with the orchestrator on startup with its unique identifier and hardware capabilities.
2. **Heartbeats**: Sends periodic heartbeats (default every 30 seconds) to the orchestrator to signal availability and current load.
3. **VNode Hosting**: Lazily instantiates VNodes on-demand from the network topology provided by the orchestrator.
4. **Local Registry**: Maintains a directory of local VNode instances and handles dynamic resolution for remote VNodes.
5. **Resource Enforcement**: Enforces strict memory and CPU limits on hosted VNodes to ensure system stability.

## Hosted VNodes

The PNode does not execute arbitrary code. Instead, it hosts VNodes which implement a predefined set of deterministic operators.

VNodes are the entities that:
- Execute **forward passes**
- Perform **local backward passes** (Gradient Locality)
- Persist state to distributed storage (S3/MinIO)

PNodes are intentionally simple and replaceable. They do **not**:
- Coordinate other PNodes
- Decompose large jobs
- Make global scheduling decisions

Those responsibilities belong to the orchestrator.

---

## Design Goals

1. **Ease of participation**  
   Anyone should be able to join the network by running a PNode with minimal setup.

2. **Safety by default**  
   VNodes must be executed within strict resource and execution boundaries.

3. **Deterministic behavior**  
   Given the same inputs and parameters, VNodes on any PNode should produce the same outputs.

4. **Observability**  
   All PNode and VNode behavior should be inspectable and debuggable by the operator.

5. **Replaceability**  
   PNodes are disposable and interchangeable. If one fails, the orchestrator reallocates its VNodes to others.

---

## Implementation Language

The reference implementation is written in **Go**.

Reasons for this choice:
- Single static binary distribution
- Strong concurrency primitives
- Good cross-platform support
- Mature gRPC ecosystem

---

## PNode Lifecycle

1. **Startup**
   - Load configuration (UUID, Orchestrator address, Storage settings)
   - Detect local hardware capabilities
   - Initialize internal VNode registry

2. **Network Join**
   - Register with the orchestrator
   - Advertise hardware capabilities and version

3. **Operation State**
   - Await VNode allocation or resolution requests
   - Send periodic heartbeats
   - Instantiate VNodes as needed

4. **VNode Management**
   - Forward gRPC requests (Forward/Train) to local VNodes
   - Handle RemoteVNode proxies for distributed communication
   - Enforce resource limits

5. **Shutdown**
   - Unregister from orchestrator
   - Graceful termination of hosted VNodes

---

## Data Access & Storage

PNodes interact with distributed storage (e.g., S3 or MinIO) to allow VNodes to persist and restore their state (weights, biases, cached inputs).

Guidelines:
- Enforce strict size/time limits on fetches
- Use short-lived credentials or pre-signed URLs where possible
- Cache state locally to improve performance of iterative training

See: [Data Sources & Data Service](/data-sources)

---

## Resource Enforcement

The PNode is responsible for enforcing **local execution limits**.

### Enforced Constraints
- Maximum memory usage (across all VNodes)
- CPU time limits
- Task-level timeouts

If a VNode exceeds limits, it is aborted and the failure is reported to the orchestrator.

The PNode always prioritizes **local system stability** over task completion.

---

## Failure Semantics

Failures are expected and normal.

The PNode may:
- Reject a VNode allocation
- Fail during VNode execution
- Disconnect unexpectedly

The PNode makes **no attempt to recover VNodes** after a crash. Recovery and re-allocation are handled entirely by the orchestrator.

---

## Trust & Security Model

The PNode operates under a **zero-trust assumption**.

- VNodes are treated as sandboxed execution units
- The orchestrator is the source of truth for topology, but PNodes validate inputs
- Other PNodes are contacted only via the orchestrator-provided locations

### Trusted vs Untrusted PNodes

PNodes can participate in two modes:

- **Untrusted (default)** — Anyone can run a PNode permissionlessly. Results from untrusted PNodes are subject to validation (e.g., via redundancy or algebraic checks).
- **Trusted** — Operators complete a registration process. Trusted PNodes may have reduced validation overhead and priority for sensitive workloads.

See: [Trust & Validation](/trust-and-validation)

---

## Configuration

PNodes are configured via environment variables or configuration files.

Typical options:
- `PNODE_UUID`: Unique identifier
- `ORCHESTRATOR_ADDRESS`: Endpoint for registration
- `PNODE_PORT`: Local gRPC port
- `S3_BUCKET`: Storage for VNode state

---

## What the PNode Is Not

The Physical Node is **not**:
- A general-purpose container runtime (like Docker)
- A scheduler
- A blockchain node
- A global data store

---

## Relationship to the Orchestrator

The PNode is **reactive**, not proactive.

It executes VNodes assigned by the orchestrator but does not attempt to reason about the global network topology.

This asymmetry keeps the PNode simple and reduces the attack surface.

---

## Summary

The Physical Node is the "muscle" of the network.

Its value comes from **numbers**, not sophistication: many simple PNodes, run by many people, hosting composable VNodes to create a massive, decentralized neural runtime.

This simplicity is what enables decentralization.
