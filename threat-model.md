
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
### 5.6 Compromised Trusted Agent

**Threat:** A previously trusted agent begins returning bad results or is taken over by an attacker.

**Mitigations:**
- Rapid trust revocation and credential rotation
- Continue spot validation of trusted agents
- Signed descriptor changes trigger re-verification
- Quarantine and require re-registration after incidents

---

### 5.7 Registration Abuse

**Threat:** An adversary attempts to obtain many trusted credentials or forges signals.

**Mitigations:**
- Manual or semi-automated vetting for initial issuance
- Rate limiting and abuse detection in issuance workflow
- Short-lived credentials with rotation and audit trails
- Require provenance for signed binaries/configuration