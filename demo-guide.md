---
title: Demo (E2E) Guide
layout: default
nav_order: 21
---

# Demo (E2E) Guide

A scripted, end-to-end walkthrough that demonstrates dispatch → execution → validation → aggregation → quarantine.

---

## Scenario

- Operators: `matmul` → `ewise_add` → `unary_relu` → `reduce_sum`
- Sizes: 64×64 · 64×16 for fast laptop runs
- Topology: 1 orchestrator, 2 agents (1 trusted, 1 untrusted)

---

## Topology & Startup

1. Start orchestrator (docker-compose or binary)
2. Start a trusted agent (registered credentials)
3. Start an untrusted agent (default permissionless)

See also: [Getting Started](/getting-started) · [Cloud Deployment](/cloud-deployment)

---

## Job Submission

Submit a demo job with embedded inputs (or references) using CLI/curl. Capture job ID.

Expected orchestrator logs:
- Task graph creation → dispatch events → validation receipts → final result

---

## Validation & Quarantine Flow

1. `matmul` dispatched to untrusted agent → 2 rounds of Freivalds → accept on match
2. `ewise_add` on trusted agent → sampled index checks (still applied at low rate)
3. `unary_relu` on untrusted agent → sampled invariants
4. `reduce_sum` on trusted agent → partial recompute on slices

Failure injection:
- Toggle fault in the untrusted agent for `matmul` → Freivalds mismatch → quarantine → reschedule to trusted agent → success

---

## Expected Outputs

- Final scalar result and checksum
- Validation receipts per task (store/print JSON)

See also: [Operator Test Vectors](/operator-test-vectors) · [Observability](/observability)

---

## Troubleshooting

- Agent not visible: check capability advertisement and network settings
- Validation fails intermittently: verify numeric tolerances and deterministic kernels
- Quarantine loop: clear agent state or restart without fault injection

---

## Next Steps

- Automate this flow with a sample `docker-compose.yml`
- Add dashboards (Prometheus/Grafana) from [Observability](/observability)
