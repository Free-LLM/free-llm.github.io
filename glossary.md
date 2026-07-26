---
title: Glossary
layout: default
nav_order: 27
---

# Glossary

Short definitions to align terminology across the docs. (Terminology from
the earlier design phase — agents, tasks, operator sets — lives in the
[Archive](/archive).)

---

## DCNR (Distributed Composable Neural Runtime)
The system implemented in [`compute-all`](https://github.com/Free-LLM/compute-all): a runtime that executes neural networks as graphs of small agents distributed across many machines.

## Physical Node (PNode)
The worker process volunteers run. It registers with the orchestrator, sends heartbeats, and hosts VNodes. See: [PNode](/pnode)

## Virtual Node (VNode)
A stateful agent representing one neural-network component (layer, activation, cost function…). Allocated to a PNode by the orchestrator; persists its state to storage. See: [VNode & Node Types](/vnode)

## Orchestrator
The coordination component: PNode registry, network management, VNode allocation, training sessions, dataset and vocabulary APIs. See: [Orchestrator](/orchestrator)

## Network (Network Definition)
A neural network described as YAML — typed nodes plus edges — submitted to the orchestrator. See: [Network Definition](/network-definition)

## Node Type
The kind of component a VNode implements (`linear`, `attention_head`, `layernorm`, `swiglu`, `cost`, …), each with its own config parameters. See: [VNode & Node Types](/vnode)

## Gradient Locality
The core principle: each VNode's backward pass uses only local parameters, locally cached inputs, and the incoming gradient — no global computational graph.

## Training Session
A first-class, resumable training run managed by the orchestrator, with step metrics streamed to the monitor.

## Colocation Group
A config label (`colocate_group`) that makes the allocator place a set of VNodes on the same PNode; `colocate_policy: required` pins the group to one healthy PNode.

## Weight Tying / Shared Weight Alias
Mechanism (`shared_weight_alias` / `tie_weight_alias`) by which one VNode reuses another's parameters — e.g. the LM head tied to the token embedding.

## Mixed Precision
Running parameters/activations in `fp16` or `bf16` while optimizers keep fp32 master weights and the loss is computed in fp32.

## Attention-Head Fusion
An orchestrator optimization pass that rewrites `broadcast → N × attention_head → stack` into a single multi-head VNode to batch GPU dispatches, without breaking existing checkpoints.

## Monitor
The observability stack: a serverless collector receiving PNode/orchestrator updates, a GraphQL API, and a Vue dashboard with training charts and the live network graph.

## dcnr (CLI)
The command-line client for creating networks, running and monitoring training, and driving the tokenizer and datasets.

## BPE Vocabulary
A trained byte-pair-encoding vocabulary used by `tokenizer`/`detokenizer` nodes and the dataset pipeline; managed through the orchestrator's vocabulary API.
