---
title: Data Sources & Data Service
layout: default
parent: Archive
nav_order: 8
---

# Data Sources & Data Service

> ⚠️ **Archived.** This page describes an early design phase that predates the actual DCNR implementation in [`compute-all`](https://github.com/Free-LLM/compute-all) and has been superseded by it. It is kept for historical context. See the [Architecture Overview](/architecture) and [Project Status](/status) for the current state.


The distributed network must handle large tensors and datasets efficiently. Shipping big payloads with every task is wasteful and slow. Instead, we separate data into two categories:

---

## Data Model

### Explicit Data
Inputs/outputs are embedded directly in the task/result payload. This is suitable for small tensors (e.g., demo vectors, small matrices) and control data.

### Implicit Data (References)
Inputs/outputs are passed as references to external locations. Agents and orchestrators fetch only the needed slices when validating or computing. References are expressed as URIs.

Supported URI schemes (initial):
- `mem://` — in‑memory buffers or ephemeral process store
- `file://` — local filesystem paths (sandboxed per policy)
- `http(s)://` — generic object fetch via HTTP(S)
- `s3://` — Amazon S3 objects (usually via pre‑signed URLs)
- `gs://` — Google Cloud Storage (future)

Notes:
- For cloud backends use pre‑signed URLs or scoped tokens; never ship long‑lived credentials to untrusted agents.
- Large outputs can also be written by agents to a URI provided by the orchestrator, returning only a reference in results.

---

## Views vs Materialization

Many operations can be represented as logical VIEWS over a remote dataset:
- Transpose, slice/window, reshape, type cast (where safe), and some reductions produce derivable views without copying data.

Materialization creates a concrete in‑memory tensor from a reference or a view:
- Use when subsequent operators require random access or when performance outweighs bandwidth cost.
- Orchestrator may stage frequently used data locally to reduce egress.

Validation with references:
- Cheap checks should fetch only sampled rows/cols/slices needed for invariants (e.g., Freivalds vectors, sampled indices for ewise/unary, partition sums for reduce).

---

## Data Service

The Data Service is an optional component that manages registration and discovery of implicit data sources.

### Responsibilities
- Register datasets and return handles (stable IDs) that resolve to URIs
- Store metadata: dtype, shape, layout, partitioning/chunking, checksums
- Provide pre‑signed/short‑lived access URLs for trusted agents upon request
- Track basic access logs and lifecycles (TTL, ownership)

### Non‑Responsibilities
- It does not execute compute
- It is not a global source of truth; multiple data services may exist

### API sketch (illustrative)
- `POST /data/register` → `{handle, uri, meta}`
- `GET /data/{handle}` → `{uris[], meta}` (may include pre‑signed links for the requester)
- `POST /data/sign` → `{signed_uri, expires_at}` (for backends like S3)

---

## Access & Trust Policies

- Untrusted agents: fetch via anonymous/limited links; tight size/time limits; read‑only; no credential propagation.
- Trusted agents: may receive short‑lived signed URLs bound to identity and constrained by IP/time.
- Orchestrators should prefer locality and minimize cross‑region egress; schedule tasks near the data.

Privacy:
- Treat task payloads and referenced data as potentially sensitive.
- Prefer references + sampling for validation to avoid copying entire datasets.

---

## Failure & Consistency Semantics

- References may go stale; agents must handle 404/403/timeouts and report failures cleanly.
- Data is immutable within a job; changing inputs under the same handle during a job is forbidden.
- Checksums in metadata enable integrity verification for materialized tensors.

---

## Caching Strategy

- Agent local cache: bounded by size and time; keyed by `{uri, range, checksum}`.
- Orchestrator staging: optional; cache popular tensors near schedulers.
- Metrics: cache hit/miss, bytes_in/bytes_out, avg fetch latency.

---

## See also

- [Protocol](/protocol)
- [Orchestrator](/orchestrator)
- [Compute Agent](/pnode)
- [Observability](/observability)
- [Operator Set v1](/operator-set-v1)
