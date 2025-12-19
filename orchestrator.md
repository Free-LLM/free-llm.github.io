---
title: Orchestrator
layout: default
nav_order: 5
---

# Orchestrator

The **orchestrator** is the coordinating component of the distributed computing system.

Its role is to take **large computational jobs**, decompose them into **many small, independent tasks**, distribute those tasks across unreliable and heterogeneous agents, and reliably reassemble the results.

Unlike traditional centralized schedulers, the orchestrator is designed to operate in an environment where **failure is normal and expected**.

---

### See also

- [Architecture Overview](/architecture)
- [Compute Agent](/compute-agent)
- [Protocol](/protocol)
- [Network Membership & Discovery](/network-membership)
 - [Data Sources & Data Service](/data-sources)

## Role in the System

The orchestrator is responsible for **global coordination**, but not for execution.

It:
- Understands the structure of large jobs
- Splits jobs into tasks
- Assigns tasks to suitable agents
- Tracks task progress
- Handles failures and retries

It does **not**:
- Execute computation itself
- Assume agents are reliable
- Require global consensus

---

## Design Goals

1. **Fault tolerance by default**  
   Node failures, disconnects, and slow responses are expected.

2. **Dynamic scheduling**  
   Tasks are scheduled based on real-time node capabilities and availability.

3. **Eventual completion**  
   Progress is asynchronous; correctness matters more than speed.

4. **Statelessness where possible**  
   Orchestrators should be replaceable and restartable.

5. **Horizontal scalability**  
   Multiple orchestrators may coexist without tight coordination.

---

## Job Model

A **job** represents a large unit of work submitted by a user or system component.

A job:
- Is immutable once accepted
- Can be decomposed into smaller tasks
- Has clear completion criteria
 - May include inputs as explicit tensors or as references to external data (see [Data Sources](/data-sources))

Jobs may represent:
- Large numerical computations
- Training steps or epochs
- Evaluation or benchmarking workloads

---

## Task Model

A **task** is the smallest schedulable unit of work.

Task properties:
- Bounded in time and memory
- Executable independently
- Retryable without side effects
 - Inputs/outputs may be provided as URIs/handles (DataRefs) to minimize payload sizes

Tasks are designed to be:
- Idempotent
- Deterministic
- Stateless

This enables safe retries and redundant execution.

---

## Job Decomposition

The orchestrator is responsible for transforming a job into a **task graph**.

Characteristics of decomposition:
- Tasks are as small as practical
- Dependencies are explicit
- Partial results are allowed

The decomposition strategy is job-specific and may evolve over time.

---

## Scheduling Strategy

Task assignment is **capability-aware**.

The orchestrator considers:
- Declared agent capabilities
- Current load and availability
- Historical reliability (optional)
 - Data locality and access costs when tasks use [DataRefs](/data-sources)

There is no assumption of fairness or long-term assignment stability.

---

## Failure Handling

Failures are handled at the task level.

The orchestrator may:
- Retry tasks on the same agent
- Reschedule tasks to different agents
- Execute tasks redundantly

The orchestrator never assumes a failure is permanent.

---

## Result Collection

Results are collected asynchronously.

The orchestrator:
- Validates result structure
- Associates results with tasks
- Aggregates partial results
 - Accepts large outputs as references and may stage or materialize selectively

Out-of-order and duplicate results are expected and handled gracefully.

### Result Validation & Agent Health

To maintain reliability with permissionless participation, the orchestrator applies validation to results from untrusted agents:

- Perform lightweight checks (e.g., invariants, checksums, recompute on a smaller sample, or redundant execution) before accepting a result.
- Mark agents as "bugged" if they fail validation beyond a threshold; quarantine or remove them from scheduling.
- Prefer trusted agents for critical paths or when resources are scarce.
 - When inputs are remote, prefer validation methods that fetch only sampled slices rather than full datasets.

Trust is a scheduling optimization, not a correctness assumption; untrusted agents remain usable under validation.

---

## State Management

The orchestrator maintains **minimal durable state**:
- Job definitions
- Task status
- Partial results

State should be:
- Serializable
- Recoverable after restart
- Independent of specific agents
 - Independent of specific storage providers; keep only handles/metadata for data references

---

## Relationship to Compute Agents

The orchestrator treats all agents as:
- Ephemeral
- Untrusted
- Replaceable

Agents are never assumed to be:
- Always available
- Correct
- Unique

This assumption simplifies orchestration logic.

---

## Relationship to Protocol

All interaction with agents happens through the **gRPC protocol**.

The orchestrator:
- Never bypasses the protocol
- Never relies on out-of-band coordination
- Uses explicit versioned APIs

This keeps coupling low and evolution manageable.

---

## Relationship to Training & Inference

The orchestrator is **workload-agnostic**.

Training and inference systems express their needs by defining:
- Job structure
- Task decomposition
- Result aggregation logic

The orchestrator provides execution guarantees, not ML semantics.

---

## What the Orchestrator Is Not

The orchestrator is **not**:
- A global consensus system
- A blockchain
- A cluster manager in the traditional sense
- A real-time scheduler

It prioritizes robustness and simplicity over optimal utilization.

---

## Summary

The orchestrator is the system’s **coordination brain**, but not its executor.

By assuming unreliable nodes and embracing asynchrony, it enables large-scale computation to emerge from many small, voluntary contributions.

This design trades peak efficiency for **resilience, openness, and scalability through participation**.
