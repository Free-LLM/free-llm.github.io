---
title: Observability
layout: default
parent: Archive
nav_order: 22
---

# Observability

> ⚠️ **Archived.** This page describes an early design phase that predates the actual DCNR implementation in [`compute-all`](https://github.com/Free-LLM/compute-all) and has been superseded by it. It is kept for historical context. See the [Architecture Overview](/architecture) and [Project Status](/status) for the current state.


Logging, metrics, and tracing for agents and orchestrators to make the demo auditable and debuggable.

---

## Logs

Recommended levels: debug, info, warn, error

Structured fields (examples):
- component: agent|orchestrator
- task_id, job_id
- operator: matmul|ewise_add|…
- duration_ms, bytes_in, bytes_out
- validation: started|passed|failed, rounds (for Freivalds)

Example agent log lines:
```
{"lvl":"info","component":"agent","event":"task_start","task_id":"...","operator":"matmul"}
{"lvl":"info","component":"agent","event":"task_done","task_id":"...","duration_ms":12}
```

---

## Metrics (Prometheus-style)

Counters/gauges (suggested):
- compute_tasks_dispatched_total
- compute_tasks_completed_total
- compute_validation_attempts_total
- compute_validation_failures_total
- compute_agents_quarantined_total
- compute_bytes_transferred_total
- compute_task_latency_ms (histogram)
- compute_freivalds_rounds_total

Data & caching metrics:
- data_bytes_in_total
- data_bytes_out_total
- data_cache_hits_total
- data_cache_misses_total
- data_fetch_latency_ms (histogram)

Labels:
- operator, dtype, trusted (true/false), result (ok/failed)
 - source_scheme (mem|file|http|s3|gs), cache (hit|miss)

---

## Tracing (optional)

Simple spans:
- orchestrator.dispatch
- agent.execute
- orchestrator.validate

Correlate spans by job_id/task_id.

---

## Dashboards

- Task throughput and latencies by operator
- Validation failure rate over time
- Quarantine events and affected agents

See also: [Demo (E2E) Guide](/demo-guide)
