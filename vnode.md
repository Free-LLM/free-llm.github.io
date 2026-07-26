---
title: Virtual Node (VNode) & Node Types
layout: default
nav_order: 4.5
---

# Virtual Node (VNode) & Node Types

A **Virtual Node (VNode)** is a stateful agent representing a single neural
network component — a layer, an activation, a structural operation, or a
cost function. Networks are graphs of VNodes, defined in
[YAML](/network-definition), and each VNode can be placed on any
[Physical Node (PNode)](/pnode) by the [orchestrator](/orchestrator).

Every VNode obeys **gradient locality**: its backward pass uses only local
parameters, locally cached inputs, and the incoming gradient. No global
computational graph exists anywhere — that is what makes the runtime
horizontally distributable.

VNodes with parameters persist their state (weights and optimizer state) to
S3-compatible storage and can be reloaded on any PNode after reallocation.

---

## Common configuration

Every node in a network definition has an `id`, a `type`, and a `config`
map of string keys. Some keys are understood by all (or most) node types:

| Key | Meaning |
|-----|---------|
| `optimizer` | Per-node optimizer for trainable nodes (e.g. `adam`, `sgd`). |
| `precision` | Sets both parameter and activation dtype (`fp16`, `bf16`, `float32`). Can also be set network-wide. |
| `param_precision` / `activation_precision` | Override parameter and activation dtypes independently. |
| `colocate_group` | Nodes sharing a group name are placed on the same PNode. |
| `colocate_policy` | `required` keeps later members of a group on the same healthy PNode after the first member is allocated. |
| `shared_weight_alias` | Publishes this node's weights under an alias other nodes can reference. |
| `tie_weight_alias` | Ties this node's weights to a published alias (e.g. tying the LM head to the token embedding). |
| `input_shape` / `output_shape` | Explicit shape overrides where the type doesn't infer them. |

Node entries can also declare `input_shapes` and `output_shapes` lists at
the node level (outside `config`).

---

## Node type catalog

The registry in
[`pnode/internal/vnode/nodetype_registry.go`](https://github.com/Free-LLM/compute-all/blob/main/pnode/internal/vnode/nodetype_registry.go)
currently defines the following types.

### Layers (trainable)

| Type | Aliases | Parameters | Description |
|------|---------|------------|-------------|
| `linear` | `dense` | `in_dim`, `out_dim`, `tie_weight_alias` | Fully connected layer. With `tie_weight_alias` it reuses another node's weights (e.g. a tied LM head). |
| `embedder` | — | `vocab_size`, `dim`, `seq_len`, `position_encoding` (`learned`, `rope`, `none`), `rope_theta`, `shared_weight_alias` | Token IDs → embedding vectors, optionally adding position encoding. |
| `layernorm` | — | `dim` | Layer normalization with learned scale/shift. |
| `attention_head` | `singleheadattention` | `head_dim`, `input_dim`, `seq_len`, `causal` (default `true`), `position_encoding` (`rope`/`none`), `rope_theta` | A single attention head: `[seq, input_dim] → [seq, head_dim]`. The unit of **distributed attention** — each head can live on a different PNode. |
| `attention` | `multiheadattention` | `dim`, `seq_len`, `position_encoding`, `rope_theta` (+ `num_heads`, `head_dim`, `causal`, `output_projection` when synthesized by the optimizer) | Multi-head attention in a single VNode. The orchestrator's fusion pass rewrites groups of `attention_head` nodes into this type to batch GPU dispatches. |

### Activations

| Type | Parameters | Description |
|------|------------|-------------|
| `relu` | — | Rectified linear unit. |
| `gelu` | — | Gaussian error linear unit (used in the production FFN blocks). |
| `silu` | — | Sigmoid-weighted linear unit. |
| `swiglu` | — | Swish-gated linear unit; splits its input into value/gate halves, so `[.., 2n] → [.., n]` (used in the production LM head). |
| `dropout` | `rate` (default `0.1`) | Random dropout during training. |

### Structural nodes

| Type | Parameters | Description |
|------|------------|-------------|
| `broadcast` | `num_outputs` (default 2) | Duplicates one input to N output slots — e.g. fan-out to attention heads, or feeding a residual connection. |
| `stack` | `axis` | Stacks multiple inputs along an axis — e.g. concatenating head outputs back into the model dimension. |
| `add` | — | Adds two inputs (residual connections). |
| `reshape` | `target_shape` | Reshapes the input to an explicit shape. |
| `mask` | `causal`, `mask_val` (default `-1e9`), `random_prob`, `seed` | Applies causal and/or random masking. |

### Text I/O

| Type | Parameters | Description |
|------|------------|-------------|
| `tokenizer` | `vocab_size` (default 50000), `pad_length`, `vocab_path`, `use_word_boundary_tokens` | Text → token IDs using a trained BPE vocabulary. |
| `detokenizer` | `vocab_size`, `vocab_path` | Token IDs → text. |

### Training

| Type | Parameters | Description |
|------|------------|-------------|
| `cost` | `cost_function` (`mse`, `ce`, `ce_logits`), `vocab_size`, `in_dim`, `cost_precision` | Loss node. `ce_logits` computes cross-entropy from raw logits; `cost_precision: float32` keeps the loss numerically stable in mixed-precision networks. A fused fast path folds a tied LM head and its `ce_logits` cost into a single GPU computation. |

---

## See also

- [Network Definition (YAML)](/network-definition) — how node types are
  assembled into a network
- [Production Training Networks](/training-topologies) — the topologies we
  train today
- [Physical Node (PNode)](/pnode) · [Orchestrator](/orchestrator) ·
  [Protocol](/protocol)
