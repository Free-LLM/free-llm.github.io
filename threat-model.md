---
title: Threat Model
layout: default
nav_order: 11
---

# Threat Model

This page describes the threat model for the Free LLM Network. The goal is to make security assumptions explicit, identify realistic adversaries, and document mitigations aligned with the project’s decentralization and openness principles.

---

### See also

- [Architecture Overview](/architecture)
- [Orchestrator](/orchestrator)
- [Compute Agent](/compute-agent)
- [Protocol](/protocol)

## 1. Security Goals

The system is designed to achieve the following high-level security goals:

- **Integrity**: Tasks, results, and reputation data must not be tampered with.
- **Authenticity**: Nodes, messages, and results must be verifiably attributable to a cryptographic identity.
- **Availability**: The network should continue functioning despite partial failures or malicious actors.
- **Fairness**: Resource contribution and task execution should not be trivially exploitable.
- **Privacy (Best-effort)**: Sensitive inputs and outputs should not be exposed beyond what is strictly required.

The network does **not** aim to provide:
- Full anonymity against a global adversary
- Protection against fully compromised local machines
- Strong confidentiality guarantees for plaintext tasks by default

---

## 2. Trust Assumptions

The threat model assumes:

- Nodes may be **honest, faulty, or malicious**
- There is **no central trusted authority**
- Cryptographic primitives (hashes, signatures) are secure
- Software implementations may contain bugs
- Network transport is hostile (MITM possible)

---

## 3. Adversary Model

### 3.1 Passive Adversary

Capabilities:
- Observe network traffic
- Collect metadata (timing, frequency, peer relationships)

Goals:
- Traffic analysis
- Network mapping
- Profiling nodes

### 3.2 Active Adversary

Capabilities:
- Inject, modify, replay messages
- Run malicious nodes
- Attempt Sybil attacks
- Attempt denial-of-service attacks

Goals:
- Corrupt results
- Degrade network performance
- Manipulate reputation
- Exclude honest participants

### 3.3 Economic Adversary

Capabilities:
- Optimize participation for profit
- Exploit incentives and pricing models

Goals:
- Free-riding
- Low-quality or fake computation
- Reputation farming

---

## 4. Attack Surfaces

### 4.1 Network Layer

- Message spoofing
- Replay attacks
- Peer flooding
- Eclipse attacks

### 4.2 Protocol Layer

- Invalid task announcements
- Malformed results
- State desynchronization
- Consensus manipulation (if applicable)

### 4.3 Compute Layer

- Returning incorrect or low-effort results
- Model substitution
- Hardware misreporting

### 4.4 Orchestrator Layer

- Biased task allocation
- Result censorship
- Reputation manipulation

---

## 5. Key Threats and Mitigations

### 5.1 Sybil Attacks

**Threat:** An attacker creates many identities to gain influence.

**Mitigations:**
- Costly identity creation (stake, proof-of-work, or proof-of-resource)
- Reputation accumulation over time
- Rate limiting per identity and per network segment

---

### 5.2 Malicious or Incorrect Results

**Threat:** Nodes return incorrect computation results.

**Mitigations:**
- Redundant execution
- Result comparison and quorum
- Challenge-response verification
- Reputation penalties

---

### 5.3 Denial of Service (DoS)

**Threat:** Flooding peers or orchestrators with requests.

**Mitigations:**
- Rate limiting
- Backpressure mechanisms
- Adaptive peer scoring
- Local resource caps

---

### 5.4 Replay and Message Tampering

**Threat:** Reuse or modification of valid messages.

**Mitigations:**
- Message signing
- Nonces and timestamps
- Deterministic message hashing

---

### 5.5 Reputation Manipulation

**Threat:** Artificially inflating or deflating reputation.

**Mitigations:**
- Multi-source reputation signals
- Decay over time
- On-chain or append-only logs (optional)
- Cross-validation by independent nodes

---

## 6. Privacy Considerations

- Task payloads may contain sensitive data
- By default, nodes can inspect tasks they execute
- Optional mitigations:
  - Task encryption with trusted execution
  - Split computation (MPC-style)
  - Differential privacy (future work)

Privacy is treated as **configurable**, not guaranteed.

---

## 7. Out-of-Scope Threats

The following are explicitly out of scope:

- Compromised operating systems
- Hardware backdoors
- Nation-state level global surveillance
- Legal or regulatory coercion of node operators

---

## 8. Residual Risk

Even with mitigations, residual risks remain:

- Coordinated collusion by high-reputation nodes
- Software vulnerabilities
- Economic incentive misalignment

These risks are accepted in exchange for openness and decentralization.

---

## 9. Future Work

Planned improvements to the threat model include:

- Formal verification of protocol components
- Stronger economic modeling
- Optional confidential execution environments
- Integration with zero-knowledge proofs for result validation

---

## 10. Summary

Security in the Free LLM Network is **defense-in-depth**, not absolute.  
The threat model favors transparency, explicit trade-offs, and incremental hardening over centralized control.