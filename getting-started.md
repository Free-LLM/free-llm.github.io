---
title: Getting Started
layout: default
nav_order: 20
---

# Getting Started

This quickstart shows how to run a minimal local demo: one orchestrator and one or two compute agents on your machine, submit a tiny job, and see validation in action.

---

## Prerequisites

- Docker (recommended for the orchestrator)
- Go 1.22+ (if building the agent from source)
- curl or a minimal CLI to submit the demo job

---

## 1) Run an Untrusted Agent Locally

Steps outline (exact commands will be added when repos are public):

1. Download the agent binary or build from source
2. Create a minimal config with resource limits and enabled operators (Operator Set v1)
3. Start the agent and verify it advertises capabilities

Expected logs:
- Startup → capability descriptor → heartbeat

See also: [Compute Agent](/compute-agent) · [Operator Set v1](/operator-set-v1)

---

## 2) Start a Local Orchestrator

Option A: docker-compose (recommended for quickstart)

Option B: single binary (if available)

The orchestrator should expose gRPC for agents and a simple submission endpoint (gRPC/HTTP) for jobs.

See also: [Orchestrator](/orchestrator) · [Protocol](/protocol)

---

## 3) Connect Agent(s) to the Orchestrator

- Point the agent to the orchestrator endpoint
- Confirm the orchestrator logs the capability advertisement

Optional: start a second agent to test redundancy and validation.

---

## 4) Submit a Tiny Job

Demo graph (single forward slice):
- `matmul` → `ewise_add` → `unary_relu` → `reduce_sum`

Submit via CLI or curl with a small input (e.g., 64×64 × 64×16). Capture the job ID.

See also: [Operator Test Vectors](/operator-test-vectors)

---

## 5) Observe Results and Validation

- Orchestrator logs dispatch, result, and validation receipts
- Agent logs task execution and completion
- Final output is a scalar plus a checksum

If you launched two agents, try enabling a failure injection on one to see quarantine behavior.

See also: [Trust & Validation](/trust-and-validation) · [Observability](/observability)

---

## 6) Next Steps

- Try the [Demo (E2E) Guide](/demo-guide) for a fully scripted walkthrough
- Explore trusted mode and cloud hardening: [Cloud Deployment](/cloud-deployment) and [Key Management](/key-management)
