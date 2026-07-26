---
title: Key Management & Trusted Agents
layout: default
parent: Archive
nav_order: 26
---

# Key Management & Trusted Agents

> ⚠️ **Archived.** This page describes an early design phase that predates the actual DCNR implementation in [`compute-all`](https://github.com/Free-LLM/compute-all) and has been superseded by it. It is kept for historical context. See the [Architecture Overview](/architecture) and [Project Status](/status) for the current state.


Guidance for operating trusted agents: signing descriptors, authenticating deployments, rotating credentials, and responding to compromise.

See also: [Trust & Validation](/trust-and-validation) · [Cloud Deployment](/cloud-deployment)

---

## Trusted Agent Overview

- Trusted = registered operators with authenticated agents
- Benefits: reduced validation overhead, priority scheduling (policy)
- Trust is revocable; untrusted mode remains always available

---

## Agent Descriptor Signing

- Obtain operator credentials from the project’s registration process
- Sign canonical agent descriptor (capabilities + build provenance)
- Advertise over an authenticated channel

Rotation:
- Short-lived credentials; rotate regularly (e.g., 24–72h)
- Automate rotation via CI/agents; audit key usage

---

## Secure Storage & Access

- Store keys in cloud KMS or local secure enclave
- Limit blast radius with scoped IAM
- Use separate credentials per deployment/environment

---

## Compromise Runbook

1. Revoke credentials immediately
2. Quarantine associated agents in orchestrator
3. Rotate all dependent secrets
4. Redeploy from known-good binaries with fresh credentials

See also: Threats 5.6/5.7 in [Threat Model](/threat-model)

---

## Attestation (Optional)

- Where supported, emit attestation references alongside signed descriptors
- Use for additional confidence; treat as advisory, not absolute

---

## Checklist

- [ ] Descriptor signing in CI
- [ ] Automated rotation
- [ ] Audit logs enabled
- [ ] Quarantine playbook tested
