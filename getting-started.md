---
title: Getting Started
layout: default
nav_order: 20
---

# Getting Started

This quickstart shows how to run a minimal local demo: one Orchestrator and one or two Physical Nodes (PNodes) on your machine, submit a tiny network, and see DCNR in action.

---

## Prerequisites

- Docker (recommended for the Orchestrator)
- Go 1.22+ (to build the PNode from source)
- curl or a minimal CLI to submit the demo network

---

## 1) Start the Orchestrator

The orchestrator manages the network topology and PNode registrations.

```bash
# Run via Docker (if available)
docker run -p 50052:50052 dcnr-orchestrator
```

The orchestrator should expose gRPC for PNodes and an API for network submission.

See also: [Orchestrator](/orchestrator) · [Protocol](/protocol)

---

## 2) Start a Physical Node (PNode)

The PNode provides the compute resources and hosts the Virtual Nodes (VNodes).

```bash
# Register with the local orchestrator
ORCHESTRATOR_ADDRESS=localhost:50052 PNODE_PORT=50051 ./pnode
```

Expected logs:
- `PNode server listening at :50051`
- `Successfully registered with orchestrator`

See also: [Physical Node (PNode)](/pnode)

---

## 3) Submit a Simple Network

A network is a graph of VNodes.

Example network:
- `Input` -> `Linear` -> `ReLU` -> `Output`

Submit via CLI or curl. Capture the network ID.

See also: [API Examples](/api-examples)

---

## 4) Observe Execution

- The Orchestrator allocates VNodes to available PNodes.
- PNodes instantiate VNodes and execute the requested operations (Forward or Train).
- VNodes communicate with each other regardless of whether they are on the same PNode or different PNodes.

---

## 5) Observe Fault Tolerance

Try stopping one of your PNodes while a job is running. The Orchestrator will detect the missing heartbeats and re-allocate the VNodes to any remaining PNodes.

See also: [Architecture Overview](/architecture) · [Trust & Validation](/trust-and-validation)

---

## 6) Next Steps

- Explore the [Demo (E2E) Guide](/demo-guide) for a fully scripted walkthrough.
- Learn more about [Virtual Nodes (VNodes)](/vnode) and [Gradient Locality](/algorithms).
- Explore cloud deployment: [Cloud Deployment](/cloud-deployment).
