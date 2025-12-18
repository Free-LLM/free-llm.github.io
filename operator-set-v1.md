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

---

### 5. Transpose (`transpose`)

Reorders tensor axes.

Constraints:

- Axis permutation must be explicit
- Resulting shape known in advance

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