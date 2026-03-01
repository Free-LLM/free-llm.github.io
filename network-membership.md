---
title: Network Membership & Discovery
layout: default
nav_order: 7
---

# Network Membership & Discovery

This document describes how **Physical Nodes (PNodes) join, leave, and discover each other** in the distributed computing network.

Network membership is a core architectural concern: it determines how the system grows, how resilient it is to failures, and how accessible participation remains over time.

The design explicitly avoids centralized registries or permanent authorities.

---

### See also

- [Architecture Overview](/architecture)
- [Orchestrator](/orchestrator)
- [Physical Node (PNode)](/pnode)
- [Virtual Node (VNode)](/vnode)

## Core Assumptions

The membership system is designed under the following assumptions:

- PNodes are **voluntary** and may leave at any time
- PNodes are **heterogeneous** in hardware and connectivity
- The network is **open**: anyone may attempt to join
- Failures, partitions, and churn are normal

No assumption is made about stable identities or long-lived availability.

---

## Design Goals

1. **Permissionless participation**  
   No approval or registration is required to join (untrusted mode).

2. **Decentralization**  
   No single node or service is required for membership.

3. **Fault tolerance**  
   The network continues to function despite churn.

4. **Low barrier to entry**  
   Joining the network should be operationally simple.

5. **Evolvability**  
   The mechanism must allow future extensions.

---

## Joining the Network

When a **Physical Node (PNode)** starts, it attempts to **join the network**.

The join process consists of:

1. Obtaining one or more initial peer or orchestrator addresses (bootstrap)
2. Establishing contact with an Orchestrator
3. Registering and advertising local capabilities (CPU, memory, etc.)
4. Sending periodic heartbeats to maintain active status
5. Entering the normal operation state (hosting VNodes)

Joining does not imply trust, commitment, or permanence.

---

## Bootstrap Mechanisms

Because the network is decentralized, new PNodes require an initial entry point.

Possible bootstrap mechanisms include:

- Static lists of well-known peers or orchestrators
- Community-maintained bootstrap endpoints
- Peer exchange from previously known nodes

Bootstrap mechanisms are **replaceable and non-authoritative**.

---

## Discovery Model

Once bootstrapped, PNodes discover additional orchestrators or peers dynamically.

Candidate discovery approaches include:

### Gossip-Based Discovery
- PNodes periodically exchange peer information
- Naturally resilient to churn
- Eventually consistent

### Distributed Hash Tables (DHTs)
- Structured peer discovery
- Scales to large networks
- Higher implementation complexity

### Hybrid Approaches
- Gossip for liveness
- DHT for structured lookup

No single approach is mandated in early versions.

---

## Node Information Propagation

PNodes exchange **descriptive information**, not authoritative state.

Propagated data may include:
- Network addresses
- Capability summaries
- Protocol versions
- Liveness hints

All information is considered **best-effort and potentially stale**.

---

## Handling Churn

Node churn is expected and normal.

The membership system must tolerate:
- Sudden disconnects
- Network partitions
- Intermittent connectivity

PNodes that disappear (stop sending heartbeats) are eventually deallocated by the orchestrator and forgotten without explicit removal.

---

## Identity Model

Early versions of the system use a **weak identity model**.

Characteristics:
- PNodes are identified by ephemeral identifiers (UUIDs)
- No strong cryptographic identity is required for untrusted participation
- No reputation or trust score is assumed initially

This avoids barriers to entry and premature centralization.

---

## Security Considerations

The open nature of the network implies inherent risks.

Potential threats include:
- Sybil attacks
- False capability advertisement
- Malicious peer lists

Mitigations are intentionally limited in early versions and rely on:
- Conservative scheduling (allocation)
- Resource isolation (sandboxing)
- Redundancy and retries

Strong identity and attestation may be introduced later for trusted nodes.

---

## Relationship to Orchestration

Membership and discovery provide **inputs** to orchestrators.

Orchestrators:
- Consume PNode information opportunistically
- Do not rely on global membership views
- Treat discovery data as advisory for VNode allocation

This loose coupling prevents cascading failures.

---

## What Network Membership Is Not

The membership system is **not**:
- An authentication service
- A consensus mechanism
- A reputation system
- A registry of trusted nodes

Its sole purpose is enabling connectivity and resource contribution.

---

## Trusted Registration (Extension)

While permissionless participation remains core, operators may optionally enroll as **trusted PNodes** to enable authentication and reduced validation overhead.

High-level flow:

1. Obtain operator credentials from the project’s registration process
2. Sign the PNode descriptor with the operator key
3. Advertise capabilities over an authenticated channel
4. Renew/rotate credentials on a schedule

Registration is an additive extension and can be ignored by untrusted PNodes.

---

## Summary

Network membership is intentionally simple and weakly consistent.

By prioritizing openness and resilience over strict guarantees, the system enables large-scale participation without central coordination.

Stronger guarantees can be layered on later — but openness cannot be retrofitted.