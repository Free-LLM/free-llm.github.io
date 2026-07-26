---
title: "Milestone: training across multiple nodes"
layout: default
parent: Blog
nav_order: 2
permalink: /blog/2026-04-first-multi-node-training
---

# Milestone: training a neural network across multiple nodes

_April 2026_

When we published the [roadmap](/roadmap), the core bet was written in one
line: if we cannot rely on a few machines with massive memory, we can rely on
**many small machines with normal resources**. Over the spring, that bet
stopped being a slide and became running code. This post retraces the work
from the [`compute-all`](https://github.com/Free-LLM/compute-all) commit
history — from a single-process prototype to a network that genuinely trains
across machines.

## Laying the foundations (March)

The early March commits were about turning a prototype into a project:

- **Repository reorganization** into clear modules — `pnode`, `orchestrator`,
  `cli`, `api` — with a Bazel build (plus plain `go build`) and an
  integration-test setup from day one.
- **A storage service** so VNode parameters and optimizer state persist to
  S3-compatible backends, decoupling model state from any single machine.
- **A minimal CLI client** (`dcnr`), because a distributed system you can't
  poke at from a terminal doesn't get debugged.
- **Registry keyed by network ID**, so the orchestrator can track every
  virtual node of every network independently, and **tokenizer state
  saving**, an early piece of the text pipeline.

## Better network definitions

Mid-March brought an improved **YAML network definition** format: a network
is a list of typed nodes (`linear`, `relu`, `attention`, …) and the edges
between them. The orchestrator validates the topology (including cycle
detection), instantiates a VNode per node, and decides which physical node
each one lives on — preferring to co-locate adjacent VNodes to keep traffic
off the wire.

This is the heart of the "composable" in Distributed Composable Neural
Runtime: the model is a graph of small agents, not a monolith.

## The milestone: more than one node

Late March delivered the commit the whole phase was building toward:
**fixing training and running on more than one node**. A network defined in
YAML could now be spread across several PNodes, run forward passes, and —
thanks to **gradient locality**, where each VNode computes its backward step
from purely local state — propagate gradients back through the distributed
graph without any global coordination.

April consolidated the milestone:

- A **train session method** in the protocol, giving training runs an
  explicit lifecycle instead of ad-hoc calls.
- CLI fixes for long training runs (the client used to hang — distributed
  systems keep you honest).
- **Network inspection and management utilities**, to see and clean up
  what's actually deployed on the network.

## Rethinking the wire protocol

The most consequential April change was architectural: PNode-to-PNode
training communication moved **from synchronous calls to asynchronous
request/response messages** with request IDs. Instead of a chain of blocking
RPCs holding connections open across the whole network — fragile and
unscalable — each training hop is now a fire-and-forget message, with
responses routed back via the sender's peer address.

This matters beyond performance. A volunteer compute network will be built
from unreliable, heterogeneous machines; an asynchronous protocol is what
lets slow nodes be slow without stalling everyone else, and it's the
foundation the later monitoring and failover work builds on.

## Where that left us

By the end of April we had the shape of the system the
[architecture pages](/architecture) describe: an
[orchestrator](/orchestrator) allocating virtual nodes across physical ones,
a gRPC [protocol](/protocol) with asynchronous training hops, persistent
state, and a CLI to drive it. Small models, few nodes — but genuinely
distributed, and genuinely training.

The next stretch of work would be about making it fast and observable:
GPU acceleration, mixed precision, and a live dashboard. That's the subject
of [the next post](/blog/2026-07-gpu-mixed-precision-monitoring).
