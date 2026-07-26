---
title: Trust & Validation
layout: default
nav_order: 8
---

# Trust & Validation

This document defines how the network maintains reliability while remaining open to permissionless participation.

The core idea: anyone can contribute compute as an **untrusted agent**, and the network maintains correctness through **validation**. Separately, vetted **trusted agents** can reduce validation overhead via authentication and attestation.

---

## Goals

- Preserve openness for new participants
- Maintain correctness and integrity of results
- Minimize attack surface by scoping the network to a single community model/service
- Provide a clear path to become a trusted operator

---

## Untrusted Agents

Untrusted is the default mode. No prior registration is needed.

Properties:
- Permissionless join
- Results are validated before acceptance
- Higher chance of redundant execution
- May be deprioritized under load

Validation techniques include:
- Deterministic recomputation on a smaller sample
- Algebraic or statistical invariants (operator-specific)
- Checksums/hashes of intermediate artifacts
- Redundant execution with quorum

Agents that fail validation repeatedly are marked as "bugged" and excluded for a cooling-off period or until operator intervention.

---

## Trusted Agents

Trusted agents are operated by entities that complete a lightweight registration process and can authenticate their deployments.

Signals for trust may include:
- Signed agent descriptors (build provenance and configuration)
- Authenticated control plane channel
- Environment attestation where available (e.g., TPM/TEE reports)

Benefits:
- Reduced validation overhead for routine operators
- Preferential scheduling for latency-sensitive tasks

Constraints:
- Trust is not permanent; it can be revoked rapidly
- Trusted agents are still subject to spot validation

---

## Registration (Initial Proposal)

1. Operator applies for a credential bound to a public key
2. Builds or downloads a vetted agent binary and signs the agent descriptor
3. Configures the agent with issued credentials
4. Orchestrators verify signatures during capability advertisement and on task assignment

Early versions focus on signed descriptors and authenticated channels; richer attestation can be added later.

---

## Orchestrator Behavior

- Treat trust as a scheduling hint, not a correctness guarantee
- Validate untrusted results using cheaper procedures than full recomputation whenever possible
- Track validation failures; quarantine agents that cross thresholds
- Demote trusted agents if their behavior degrades; promote untrusted agents only via the registration path

See: [Orchestrator](/orchestrator)

---

## Scope and Ethics

The network is limited to operating a single, community-governed AI model/service. This scoping reduces misuse risks and clarifies that compute cannot be repurposed for arbitrary workloads. Questions of training data, safeguards, and bias will be addressed by a community governance process when the model reaches maturity.

---

## Related

- [Physical Node (PNode)](/pnode)
- [Network Membership & Discovery](/network-membership)
- [Protocol](/protocol)
- [Threat Model](/threat-model)
