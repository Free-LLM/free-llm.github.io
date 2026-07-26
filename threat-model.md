---
title: Threat Model (Draft)
layout: default
nav_order: 7.5
---

# Threat Model (Draft)

> 🚧 **Early, incomplete draft.** This page was started during the initial
> design phase and was never finished. It is kept because the threats it
> identifies are real and will matter as soon as the network opens up to
> permissionless participation. It will be completed alongside the
> [Network Membership & Discovery](/network-membership) and
> [Trust & Validation](/trust-and-validation) work — see the
> [Project Status](/status) page for where that sits today.

The notes below list identified threats and candidate mitigations. None of
the mitigations should be read as implemented unless the
[Project Status](/status) page says so.

---

## Identified threats

### Denial of Service (DoS)

**Threat:** Flooding peers or orchestrators with requests.

**Candidate mitigations:**
- Rate limiting
- Backpressure mechanisms
- Adaptive peer scoring
- Local resource caps

---

### Replay and Message Tampering

**Threat:** Reuse or modification of valid messages.

**Candidate mitigations:**
- Message signing
- Nonces and timestamps
- Deterministic message hashing

---

### Reputation Manipulation

**Threat:** Artificially inflating or deflating reputation.

**Candidate mitigations:**
- Multi-source reputation signals
- Decay over time
- On-chain or append-only logs (optional)
- Cross-validation by independent nodes

---

### Compromised Trusted Agent

**Threat:** A previously trusted agent begins returning bad results or is taken over by an attacker.

**Candidate mitigations:**
- Rapid trust revocation and credential rotation
- Continue spot validation of trusted agents
- Signed descriptor changes trigger re-verification
- Quarantine and require re-registration after incidents

---

### Registration Abuse

**Threat:** An adversary attempts to obtain many trusted credentials or forges signals.

**Candidate mitigations:**
- Manual or semi-automated vetting for initial issuance
- Rate limiting and abuse detection in issuance workflow
- Short-lived credentials with rotation and audit trails
- Require provenance for signed binaries/configuration

---

## Privacy Considerations

- Task payloads may contain sensitive data
- By default, nodes can inspect tasks they execute
- Optional mitigations:
  - Task encryption with trusted execution
  - Split computation (MPC-style)
  - Differential privacy (future work)

Privacy is treated as **configurable**, not guaranteed.

---

## Out-of-Scope Threats

The following are explicitly out of scope:

- Compromised operating systems
- Hardware backdoors
- Nation-state level global surveillance
- Legal or regulatory coercion of node operators

---

## Residual Risk

Even with mitigations, residual risks remain:

- Coordinated collusion by high-reputation nodes
- Software vulnerabilities
- Economic incentive misalignment

These risks are accepted in exchange for openness and decentralization.

---

## Future Work

Completing this draft is part of the membership & trust phase. Planned
improvements include:

- Enumerating the missing threat categories (Sybil attacks, result
  falsification, malicious orchestrators, storage tampering)
- Formal verification of protocol components
- Stronger economic modeling
- Optional confidential execution environments
- Integration with zero-knowledge proofs for result validation

---

## Summary

Security in the Free LLM Network is **defense-in-depth**, not absolute.  
The threat model favors transparency, explicit trade-offs, and incremental hardening over centralized control.
