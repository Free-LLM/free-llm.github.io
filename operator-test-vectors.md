---
title: Operator Test Vectors
layout: default
nav_order: 25
---

# Operator Test Vectors

Small, deterministic inputs/outputs to validate VNode implementations and demo runs for Operator Set v1.

See also: [Operator Set v1](/operator-set-v1)

---

## Conventions

- DType defaults: float32 unless specified
- Tolerances: float32 (atol≈1e-5, rtol≈1e-4)
- Shapes are explicit; tensors serialized row-major
- Used by VNodes to verify local execution correctness

---

## matmul

Inputs:
- A (2×3): [[1,2,3],[4,5,6]]
- B (3×2): [[7,8],[9,10],[11,12]]

Expected C (2×2):
- [[58,64],[139,154]]

Freivalds sample: n=2, rounds=2 (example r=[1,0])

---

## ewise_add

Inputs:
- X (3): [1,2,3]
- Y (3): [4,5,6]

Expected Z (3): [5,7,9]

---

## reduce_sum (axis=0)

Input:
- X (2×3): [[1,2,3],[4,5,6]]

Expected (3): [5,7,9]

---

## unary_relu

Input:
- X (5): [-2,-0.5,0,0.5,2]

Expected (5): [0,0,0,0.5,2]

---

## transpose

Input:
- X (2×3): [[1,2,3],[4,5,6]]

Permute axes: (1,0)

Expected (3×2): [[1,4],[2,5],[3,6]]

---

## Notes

- Extend with additional ops as Operator Set evolves
- Include binary fixtures when repositories are wired up
- These vectors are used for PNode-level unit testing of VNode operators
