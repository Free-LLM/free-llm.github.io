---
title: Compatibility Matrix
layout: default
parent: Archive
nav_order: 24
---

# Compatibility Matrix

> ⚠️ **Archived.** This page describes an early design phase that predates the actual DCNR implementation in [`compute-all`](https://github.com/Free-LLM/compute-all) and has been superseded by it. It is kept for historical context. See the [Architecture Overview](/architecture) and [Project Status](/status) for the current state.


Track protocol, agent, orchestrator, and operator-set versions. Use this page to plan upgrades and ensure safe interoperability during demos.

---

## Version Policy

- Protocol: semantic versioning (major/minor/patch)
- Agent/Orchestrator: declare supported protocol and operator-set versions
- Operator Set: versioned (v1, v2, …); agents must refuse unsupported ops

---

## Matrix (to be filled as versions publish)

Columns:
- Protocol Version
- Agent Version → Supported Protocols, Operator Sets
- Orchestrator Version → Supported Protocols, Operator Sets
- Notes / Deprecations

Placeholder example:

```
Protocol | Agent     | Orchestrator | Operator Set | Notes
---------|-----------|--------------|--------------|------
1.0.0    | >=0.1.0   | >=0.1.0      | v1           | Initial demo baseline
```

---

## Deprecation & Upgrade Guidance

- Avoid silent downgrades; reject incompatible peers explicitly
- Phase upgrades: orchestrator first, then agents
- Keep at least one overlapping minor version window during rollouts

See also: [Protocol](/protocol) · [Operator Set v1](/operator-set-v1)
