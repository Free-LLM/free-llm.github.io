

---
title: Network Membership & Discovery
layout: default
nav_order: 7
---

# Network Membership & Discovery

This document describes how **nodes join, leave, and discover each other** in the distributed computing network.

Network membership is a core architectural concern: it determines how the system grows, how resilient it is to failures, and how accessible participation remains over time.

The design explicitly avoids centralized registries or permanent authorities.

---

## Core Assumptions

The membership system is designed under the following assumptions:

- Nodes are **voluntary** and may leave at any time
- Nodes are **heterogeneous** in hardware and connectivity
- The network is **open**: anyone may attempt to join
- Failures, partitions, and churn are normal

No assumption is made about stable identities or long-lived availability.

---

## Design Goals

1. **Permissionless participation**  
   No approval or registration is required to join.

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

When a compute agent starts, it attempts to **join the network**.

The join process consists of:

1. Obtaining one or more initial peer addresses (bootstrap)
2. Establishing contact with existing nodes
3. Advertising local capabilities
4. Entering the normal operation state

Joining does not imply trust, commitment, or permanence.

---

## Bootstrap Mechanisms

Because the network is decentralized, new nodes require an initial entry point.

Possible bootstrap mechanisms include:

- Static lists of well-known peers
- Community-maintained bootstrap endpoints
- Peer exchange from previously known nodes

Bootstrap mechanisms are **replaceable and non-authoritative**.

---

## Discovery Model

Once bootstrapped, nodes discover additional peers dynamically.

Candidate discovery approaches include:

### Gossip-Based Discovery
- Nodes periodically exchange peer information
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

Nodes exchange **descriptive information**, not authoritative state.

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

Nodes that disappear are eventually forgotten without explicit removal.

---

## Identity Model

Early versions of the system use a **weak identity model**.

Characteristics:
- Nodes are identified by ephemeral identifiers
- No strong cryptographic identity is required
- No reputation or trust score is assumed

This avoids barriers to entry and premature centralization.

---

## Security Considerations

The open nature of the network implies inherent risks.

Potential threats include:
- Sybil attacks
- False capability advertisement
- Malicious peer lists

Mitigations are intentionally limited in early versions and rely on:
- Conservative scheduling
- Resource isolation
- Redundancy and retries

Strong identity and attestation may be introduced later.

---

## Relationship to Orchestration

Membership and discovery provide **inputs** to orchestrators.

Orchestrators:
- Consume peer information opportunistically
- Do not rely on global membership views
- Treat discovery data as advisory

This loose coupling prevents cascading failures.

---

## What Network Membership Is Not

The membership system is **not**:
- An authentication service
- A consensus mechanism
- A reputation system
- A registry of trusted nodes

Its sole purpose is enabling connectivity.

---

## Summary

Network membership is intentionally simple and weakly consistent.

By prioritizing openness and resilience over strict guarantees, the system enables large-scale participation without central coordination.

Stronger guarantees can be layered on later — but openness cannot be retrofitted.