---
title: Network Definition (YAML)
layout: default
nav_order: 5.5
---

# Network Definition (YAML)

Networks are defined as **YAML documents** and submitted to the
[orchestrator](/orchestrator) (`CreateNetwork` / `UpdateNetwork`). The
orchestrator parses the definition, validates the topology (including cycle
detection), applies optimization passes, instantiates one VNode per node,
and allocates them across the available PNodes.

The parser lives in
[`orchestrator/internal/network/parser.go`](https://github.com/Free-LLM/compute-all/blob/main/orchestrator/internal/network/parser.go);
real production examples are in the repository's
[`data/`](https://github.com/Free-LLM/compute-all/tree/main/data) directory.

---

## Top-level structure

```yaml
name: my-network            # human-readable network name

# Optional network-wide precision defaults, inherited by every node
# that doesn't override them:
precision: fp16             # sets both param and activation dtype
# param_precision: fp16     # or set them independently
# activation_precision: fp16

nodes:    [...]             # the VNodes (see below)
edges:    [...]             # directed connections between node slots
outputs:  [...]             # named network outputs
costs:    [...]             # named loss outputs used for training
metrics:  [...]             # named training metrics (e.g. learning rate)
```

## Nodes

Each node has an `id`, a `type` from the
[node type catalog](/vnode), a `config` map of string values, and optional
explicit shapes:

```yaml
nodes:
  - id: embedder
    type: embedder
    config:
      vocab_size: "10000"
      dim: "512"
      seq_len: "128"
      position_encoding: "none"     # RoPE is applied inside the heads
      shared_weight_alias: "token_embedding"
      colocate_group: "token_embedding"
      colocate_policy: "required"
      optimizer: "adam"
    input_shapes:
      - [128]                       # token IDs
    output_shapes:
      - [128, 512]                  # embeddings
```

Notes:

- **Config values are strings** (quoted numbers) — each node type parses its
  own keys.
- **`colocate_group` / `colocate_policy`** guide allocation: nodes in the
  same group are placed on the same PNode; `required` keeps later members
  pinned to the same healthy PNode once the first is allocated.
- **`shared_weight_alias` / `tie_weight_alias`** implement weight tying
  across nodes (e.g. LM head tied to the token embedding).

## Edges and slots

Edges connect node outputs to node inputs. Nodes with multiple inputs or
outputs address them with **slots**:

```yaml
edges:
  - from: broadcast_to_heads      # output slot 0 by default
    to: head_0
    from_slot: 0
  - from: attn_dropout
    to: residual_attn
    to_slot: 0                    # main branch
  - from: embedder_broadcast
    from_slot: 1
    to: residual_attn
    to_slot: 1                    # residual branch
```

## Outputs, costs, and metrics

```yaml
outputs:
  - name: logits
    node_id: lm_head
    slot_id: 0

costs:
  - name: ce_loss
    node_id: loss
    slot_id: 0

metrics:
  - name: head_learning_rate
    type: "learning_rate"
    node_id: lm_head
    slot_id: 0
```

`outputs` name the tensors clients can read back, `costs` name the loss
values the training session optimizes and reports, and `metrics` expose
per-node training metrics (such as the current learning rate) to the
[monitoring dashboard](/status#observability).

---

## What the orchestrator does with it

1. **Validation** — unknown node types, bad configs, dangling edges, and
   cycles are rejected at submission time.
2. **Optimization passes** — the operational node list may be rewritten
   while the submitted YAML is kept verbatim. The current pass fuses the
   `broadcast → N × attention_head → stack` pattern into a single
   multi-head attention VNode (batching GPU dispatches across heads) while
   remaining checkpoint-compatible with the unfused layout.
3. **Instantiation & allocation** — each node becomes a VNode with a unique
   UUID; allocation is locality-aware (adjacent VNodes and colocation
   groups on the same PNode, load-balanced across the fleet).
4. **Persistence** — the definition and VNode placement are persisted so
   the orchestrator can recover after a restart.

---

## See also

- [Virtual Node (VNode) & Node Types](/vnode) — all node types and their
  parameters
- [Production Training Networks](/training-topologies) — full real-world
  examples
- [Orchestrator](/orchestrator) · [Protocol](/protocol)
