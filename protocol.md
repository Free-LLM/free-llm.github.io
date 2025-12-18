---
title: Protocol
layout: default
nav_order: 6
---

# Protocol

The **protocol** defines the contract between **orchestrators** and **compute agents**.

It is the most stability-critical component of the system: once adopted, it must evolve carefully to avoid fragmenting the network.

The protocol is intentionally **minimal, explicit, and versioned**.

---

### See also

- [Architecture Overview](/architecture)
- [Compute Agent](/compute-agent)
- [Orchestrator](/orchestrator)

## Role in the System

The protocol is responsible for:
- Enabling communication between orchestrators and agents
- Making capabilities, tasks, and results explicit
- Defining failure and retry semantics

It does **not**:
- Encode business logic
- Assume trust between parties
- Impose execution strategies

---

## Transport

All communication uses **gRPC** over standard transport layers.

Reasons for this choice:
- Strong typing via protobuf
- Efficient binary serialization
- Streaming support
- Mature tooling across languages

gRPC is treated as an implementation detail, but **protobuf schemas are part of the public API**.

---

## Versioning Strategy

The protocol follows strict **semantic versioning**:

- **Major version**: breaking changes
- **Minor version**: backward-compatible extensions
- **Patch version**: clarifications and fixes

Agents and orchestrators:
- Must declare supported protocol versions
- Must reject incompatible versions explicitly

Silent downgrade or implicit compatibility is forbidden.

---

## Core Concepts

### Node Descriptor

A **node descriptor** advertises agent capabilities.

Typical fields include:
- Architecture and OS
- CPU cores and limits
- Memory limits
- Supported operators
- Protocol version
 - Optional trust signals (e.g., signed descriptor, attestation reference)

Descriptors are informational, not contractual.

---

### Task Descriptor

A **task descriptor** defines a unit of work to be executed by an agent.

Properties:
- Operator identifier
- Input references or payloads
- Expected resource usage
- Timeout constraints

Tasks are immutable once dispatched.

---

### Result Descriptor

A **result descriptor** represents the outcome of a task.

It includes:
- Task identifier
- Execution status
- Output references or payloads
- Optional diagnostics
 - Optional validation receipt (if validated pre-acceptance)

Duplicate or late results are valid and expected.

---

## Core RPCs

The protocol defines a small, fixed set of RPCs.

### Capability Advertisement
- Agent → Orchestrator
- Announces node descriptor

### Task Dispatch
- Orchestrator → Agent
- Sends task descriptor

### Task Acknowledgment
- Agent → Orchestrator
- Confirms task acceptance or rejection

### Result Submission
- Agent → Orchestrator
- Submits task result

### Heartbeat
- Bidirectional
- Signals liveness

---

## Failure Semantics

Failures are first-class protocol outcomes.

Possible failure modes include:
- Task rejection
- Execution failure
- Timeout
- Agent disappearance

The protocol:
- Never hides failures
- Never retries implicitly
- Always makes failure explicit

Recovery logic belongs entirely to the orchestrator.

---

## Idempotency & Retries

All protocol operations must be **idempotent** where applicable.

This allows:
- Safe retries
- Redundant execution
- Duplicate message handling

Identifiers are globally unique within a job scope.

---

## Security Model

The protocol assumes **zero trust**.

- All inputs are untrusted
- All peers are potentially malicious
- No implicit authentication or authorization is assumed

Initial versions rely on:
- Strict validation
- Minimal exposed surface

Authentication, encryption, and basic signature verification are introduced for trusted agents; stronger attestation remains an extension.

---

## Extensibility

The protocol is designed to be extended by:
- Adding optional fields
- Introducing new RPCs
- Defining new operator identifiers

Backward compatibility is mandatory.

---

## What the Protocol Is Not

The protocol is **not**:
- A consensus mechanism
- A scheduling policy
- A security framework
- A high-level workflow language

Its role is to communicate *facts*, not *intent*.

---

## Summary

The protocol is the **constitutional layer** of the system.

By keeping it small, explicit, and versioned, the project enables independent evolution of agents, orchestrators, and higher-level workloads without fragmenting the network.

Stability here enables experimentation everywhere else.