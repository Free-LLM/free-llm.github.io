---
title: Protocol
layout: default
nav_order: 6
---

# Protocol

All communication in DCNR uses **gRPC** with protobuf schemas. The protobuf
definitions live in the
[`compute-all/api`](https://github.com/Free-LLM/compute-all/tree/main/api)
directory and are the public contract between the orchestrator, physical
nodes (PNodes), and clients such as the `dcnr` CLI.

There are two sides to the protocol: the **orchestrator services** (control
plane) and the **PNode/VNode services** (execution plane).

An earlier task/operator protocol design is preserved in the
[archive](/archive/legacy-protocol).

---

## Orchestrator services

### `OrchestratorService` — control plane

The main API used by clients and PNodes:

- **Networks**: `CreateNetwork`, `UpdateNetwork`, `DeleteNetwork`,
  `GetNetwork`, `ListNetworks` — networks are submitted as
  [YAML definitions](/network-definition), validated (topology, cycles) and
  optimized (e.g. attention-head fusion) on creation.
- **Execution**: `ForwardNetwork` (inference pass), `TrainNetwork` (single
  training step), `ValidateNetwork` (evaluation on a validation split),
  `ChatNetwork` (interactive generation).
- **Training sessions**: `CreateTrainSession`, `StartTrainSession`,
  `SuspendTrainSession`, `ListTrainSessions`, `DeleteTrainSession`, plus a
  bidirectional `TrainNetworkSession` stream and `ListenTrainSession` for
  live training events. The trainer runs inside the orchestrator; sessions
  are first-class, resumable objects.
- **Vocabularies**: `CreateVocabulary`, `GetVocabulary`, `ListVocabularies`,
  `DeleteVocabulary` — BPE vocabularies used by tokenizer/detokenizer nodes.
- **Datasets**: `CreatePreprocessedDataset` (streaming upload),
  `CreateDerivedDataset` (e.g. train/validation splits), `ExploreDataset`,
  `ListPreprocessedDatasets`, `DeletePreprocessedDataset` — datasets are
  served in tokenized form.
- **Network state**: `InspectNetwork`, `ExportNetwork`, `ImportNetwork`,
  `CloneNetwork`, `SyncNetworkState`, `DeleteNetworkState` — utilities to
  inspect, copy, and manage persisted parameters.
- **Health**: `Health` returns orchestrator statistics.

### `NodeManagementService` — PNode membership

- `RegisterPNode` / `UnregisterPNode` — PNodes register on startup with
  UUID, version, network address, and hardware capabilities.
- `HeartbeatPNode` — periodic liveness signal; missing heartbeats trigger
  automatic VNode reallocation.
- `GetVNodeLocation` — resolves which PNode currently hosts a VNode,
  allocating it on demand (with locality-aware placement).

### `StorageService`

- `GetDownloadURL` / `GetUploadURL` — short-lived pre-signed URLs for
  S3-compatible storage, so PNodes never hold long-lived credentials.

---

## PNode services

### `VNodeService` — execution

- `Forward` — run a forward pass through a hosted VNode.
- `ReceiveTrainRequest` / `ReceiveTrainResponse` — the **asynchronous
  training protocol**: training hops between PNodes are fire-and-forget
  messages carrying request IDs; the receiver derives the callback target
  from the incoming gRPC peer address. This is what lets slow nodes be slow
  without blocking the rest of the network.
- `Train` — unary wrapper kept for orchestrator/client compatibility.
- `Update` — apply optimizer updates.
- `Load` / `Save` — persist and restore VNode state (parameters, optimizer
  state) to distributed storage.
- `Health` — VNode-level health.

### `PNodeService` — local management

- `ListLocalVNodes`, `DropLocalVNode`, `InspectNetwork` — inspect and manage
  the VNodes hosted on a specific PNode.

---

## Design properties

- **Versioned, explicit schemas** — the protobuf files are part of the
  public API; changes must stay backward compatible.
- **Asynchrony where it matters** — the training path is asynchronous
  end-to-end; only control-plane calls are synchronous.
- **Failure is a first-class outcome** — requests carry IDs, duplicate and
  late responses are tolerated, and recovery (reallocation, retries) belongs
  to the orchestrator.
- **No long-lived credentials on workers** — storage access is mediated by
  pre-signed URLs.

---

## See also

- [Architecture Overview](/architecture)
- [Orchestrator](/orchestrator)
- [Physical Node (PNode)](/pnode)
- [Virtual Node (VNode) & Node Types](/vnode)
- [Network Definition (YAML)](/network-definition)
