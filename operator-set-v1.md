---
title: Operator Set v1
layout: default
nav_order: 7
---

# Operator Set v1

This document defines **Operator Set v1**, the minimal and intentionally constrained set of computational primitives supported by the Free LLM Network compute agents.

The operator set is a **security boundary**, not just an API. It determines:

- What agents are allowed to execute
- What can be validated and recomposed
- What kinds of distributed algorithms are feasible

Operator Set v1 prioritizes **simplicity, determinism, and verifiability** over expressiveness.

---

### See also

- [Operator Test Vectors](/operator-test-vectors)

## Design Goals

Operator Set v1 is designed to:

- Be **small and auditable**
- Enable **useful linear-algebra workloads**
- Avoid Turing-completeness
- Allow **result validation and redundancy**
- Run within **bounded memory and time**

It explicitly avoids:

- Arbitrary code execution
- User-defined control flow
- Dynamic memory allocation beyond declared bounds
- File system or network access

---

## Execution Model

An operator invocation is defined by:

- Operator name
- Input tensors (or references)
- Output shape
- Resource bounds (memory, time)

All operators are:

- **Pure** (no side effects)
- **Deterministic**
- **Stateless**

Given the same inputs, an operator must always produce the same outputs.

---

## Data Model

### Tensors

All data is represented as tensors with:

- Fixed shape
- Fixed data type
- Explicit memory footprint

Supported data types (v1):

- `float32`
- `float16`
- `int32`

Tensors are immutable during execution.

---

## Core Operators

### 1. Matrix Multiplication (`matmul`)

Performs matrix multiplication:
```
C = A × B
```
Constraints:

- 2D tensors only
- Compatible shapes
- Output shape known in advance

Use cases:

- Linear layers
- Attention projections

Validation (cheap):

- Shape checks: verify `A.shape == (m,k)`, `B.shape == (k,n)`, `C.shape == (m,n)`.
- Freivalds' algorithm: pick random vector `r ∈ {0,1}^n`, check `A·(B·r) == C·r` within tolerance; repeat 2–3 times to reduce error probability. Cost: O(mk + kn + mn) per check.
- Spot rows/cols: sample a few indices `i` or `j` and recompute `C[i,*]` or `C[* ,j]` directly.
- Checksums: compare `sum(C)` with `sum(A)·mean(B)` only as a loose heuristic (not sufficient alone).

---

### 2. Element-wise Operations (`ewise_*`)

Supported operations:

- `add`
- `sub`
- `mul`
- `div`
- `max`
- `min`

Constraints:

- Identical input shapes

Use cases:

- Residual connections
- Element-wise transformations

Validation (cheap):

- Index sampling: sample K random indices and recompute locally; expect exact match (or within numeric tolerance for floats).
- Invariants by op:
  - `add`/`mul`: commutativity check on samples (`x ⊕ y == y ⊕ x`).
  - `div`: spot-check `b != 0` and `a == (a/b)*b` within tolerance.
  - `max`/`min`: sampled outputs must equal `max(a,b)`/`min(a,b)`.
- Range/tensor shape checks: output shape == input shape; dtype rules preserved.

---

### 3. Reduction Operations (`reduce_*`)

Supported reductions:

- `sum`
- `mean`
- `max`

Constraints:

- Reduction axis must be explicit
- Output shape deterministic

Use cases:

- Loss computation
- Aggregations

Validation (cheap):

- Partial recompute: for sampled slices along the reduced axis, recompute the reduction and compare.
- Invariants by reduction:
  - `sum`: total sum of partitions equals reported sum (recompute on a few random partitions).
  - `mean`: `mean == sum/count`; verify consistency on samples.
  - `max`: check that no sampled input exceeds the reported max and at least one element equals it in sampled blocks; optionally require/report argmax index for extra checks.
- Shape check: verify output shape matches input with reduced axis removed or set to 1 per spec.

---

### 4. Unary Functions (`unary_*`)

Supported functions:

- `relu`
- `tanh`
- `sigmoid`
- `exp`
- `log`

Constraints:

- Element-wise only
- No dynamic branching

Validation (cheap):

- Index sampling: recompute function on K random positions.
- Function-specific invariants:
  - `relu(x) ≥ 0` and `relu(x) == x` when `x ≥ 0`.
  - `tanh(x) ∈ [-1,1]`, odd function: `tanh(-x) == -tanh(x)` on samples.
  - `sigmoid(x) ∈ (0,1)`, monotonic increasing; `sigmoid(0) ≈ 0.5` sanity.
  - `exp(x) > 0`, monotonic increasing; guard overflow/inf.
  - `log(x)` domain: inputs must be `> 0`; output monotonic; check `exp(log(x)) ≈ x` on samples.

---

### 5. Transpose (`transpose`)

Reorders tensor axes.

Constraints:

- Axis permutation must be explicit
- Resulting shape known in advance

Validation (cheap):

- Shape and permutation checks: confirm output shape equals input shape permuted by provided axes.
- Index mapping spot-checks: sample a few indices and verify `out[idx_perm] == in[idx]`.
- Invariants: `transpose(transpose(X, p), inverse(p)) == X` on a small sampled subset.

---

## Explicitly Excluded Operations

The following are **intentionally not supported** in Operator Set v1:

- Control flow (`if`, `while`, `for`)
- Recursion
- Dynamic shape creation
- Random number generation
- File I/O
- Network I/O
- Custom kernels

These exclusions are deliberate and foundational.

---

## Validation & Redundancy

Because operators are pure and deterministic:

- Tasks can be executed redundantly
- Results can be compared byte-for-byte
- Discrepancies can be detected and penalized

This enables:

- Probabilistic Byzantine fault tolerance
- Reputation-based trust mechanisms

Numeric tolerance and determinism:

- Unless otherwise specified, comparisons for floating-point tensors use an absolute/relative tolerance suitable to the tensor dtype (e.g., `float32`: atol≈1e-5, rtol≈1e-4; `float16`: atol≈1e-3, rtol≈1e-2).
- Deterministic kernels should be preferred; where minor nondeterminism exists due to parallel reductions, validation should rely on tolerant comparisons or algebraic checks (e.g., Freivalds for `matmul`).

Redundancy strategies (cheap first):

- Spot-checks on random indices or slices (constant K per tensor size cap).
- Algebraic randomized checks (e.g., Freivalds' algorithm for `matmul`).
- Cross-agent redundant execution on a fraction of tasks; escalate to full recompute on mismatch.

---

## Relationship to Training & Inference

Operator Set v1 supports:

- Forward passes of simple models
- Portions of backpropagation
- Distributed linear-algebra workloads

It does **not** fully support:

- End-to-end backpropagation without orchestration
- Dynamic computation graphs

Training algorithms must be expressed as **graphs of operator invocations** managed by the orchestrator.

---

## Evolution Strategy

Future versions may introduce:

- Batched operators
- Sparse tensor support
- Quantized operations
- Cryptographic commitments to results

Backward compatibility is a strict requirement.

---

## Summary

Operator Set v1 defines a **minimal, safe, and verifiable execution surface**.

Its constraints are intentional and foundational, enabling:

- Decentralization
- Fault tolerance
- Trust minimization

This is not a limitation — it is the system’s core strength.