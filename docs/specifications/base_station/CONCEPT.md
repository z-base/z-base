# Base Station — Concept Specification

## Abstract

A **Base Station** is a cloud-hosted, non-authoritative infrastructure service responsible for authenticated connectivity, **persistence of opaque Resource Snapshots**, and **relay of opaque Delta and Snapshot frames** for exactly one Resource. A Base Station has **zero semantic awareness**, **no local authority**, and **no application logic**. It verifies possession of Resource-bound cryptographic material, stores meaningless snapshot blobs, and relays meaningless blobs between verified peers.

Every Resource **MUST** be served by exactly one logically isolated Base Station instance.

---

## Conformance

An implementation **conforms** iff it satisfies all **normative** requirements herein.

RFC 2119 / 8174 keywords apply.

---

## Base Station Definition *(Normative)*

A **Base Station**:

- serves **exactly one Resource identifier**,
- verifies possession of Resource-related cryptographic material,
- **persists Snapshot frames**,
- **relays Delta frames**, and
- **relays Snapshot frames on verified request**.

A Base Station **MUST NOT**:

- interpret payloads,
- validate or merge state,
- enforce authorization semantics,
- attribute identity,
- act as an Actor.

---

## One-Resource-Per-Station *(Normative)*

Each Resource **MUST** have its own logically isolated Base Station.

No cross-Resource persistence, relay, or peer sets are permitted.

---

## Required Infrastructure Components *(Normative)*

### 1. Stateless Edge Gateway

The gateway:

- **MUST** be highly available and stateless,
- **MUST** accept inbound connection requests,
- **MUST** route requests deterministically to the correct per-Resource Base Station,
- **MUST NOT** persist state,
- **MUST NOT** upgrade connections to verified sessions.

---

### 2. Per-Resource Stateful Base Station Core

The Base Station core:

- **MUST** exist per Resource,
- **MUST** perform WebSocket upgrade,
- **MUST** verify possession of Resource-bound cryptographic material **before** verification,
- **MUST** maintain the verified connection set,
- **MUST** persist Snapshots,
- **MUST** relay Deltas and Snapshots.

---

### 3. In-Memory Snapshot Cache

The Base Station:

- **MUST** keep the **latest persisted Snapshot** resident in memory while active,
- **MUST NOT** interpret it,
- **MUST** use it to serve Snapshot relay and retrieval.

---

### 4. Durable Snapshot Storage

On hibernation, eviction, or restart the Base Station:

- **MUST** persist the latest Snapshot to durable filesystem / object / blob storage,
- **MUST** restore it into memory on reactivation,
- **MUST** treat storage as opaque.

---

## Connection Lifecycle *(Normative)*

### Pre-Upgrade Verification

Before upgrading to a verified WebSocket session, the Base Station **MUST**:

- require proof of possession of Resource-bound cryptographic material,
- verify the proof without semantic inspection,
- reject the connection on failure.

Unverified connections **MUST NOT** send or receive frames.

---

### Verified Session

Once verified, a connection:

- **MUST** be added to the verified peer set,
- **MAY** submit Snapshot frames,
- **MAY** submit Delta frames,
- **MAY** request Snapshot relay,
- **MAY** receive relayed frames.

---

### Connection Closure

On disconnect, the connection **MUST** be removed from the verified set.  
No semantic inference is permitted.

---

## Frame Classes *(Normative)*

### Snapshot Frames

- Encrypted, opaque full Resource state.
- **MUST** be persisted.
- **MUST** be relayed **on explicit request by a verified peer**.
- **MAY** be relayed for initial synchronization.

---

### Delta Frames

- Encrypted, opaque incremental updates.
- **MUST NOT** be persisted.
- **MUST** be relayed.

---

## Relay Semantics *(Normative)*

### Delta Relay

For each Delta frame, the Base Station **MUST**:

- relay to **all verified peers except the sender**,
- allow unordered, duplicated, or lossy delivery.

---

### Snapshot Relay *(Mandatory on Request)*

Upon receiving a Snapshot relay request from a verified peer, the Base Station **MUST**:

- relay the **latest persisted Snapshot**,
- treat the Snapshot as opaque,
- **MUST NOT** synthesize, modify, or validate it.

Snapshot relay **DOES NOT** replace persistence.

---

## Host Environment Requirements *(Normative)*

A Base Station **MUST** be deployed in a **compliant host environment** that enforces baseline web-application and network-level protections *before traffic reaches the Stateless Edge Gateway*. These controls exist to protect infrastructure availability and **MUST NOT** introduce application semantics.

### Pre-Gateway Traffic Filtering

The host environment **MUST**:

- enforce rate limits and burst controls per source,
- enforce connection limits,
- reject malformed or non-conforming protocol requests,
- drop traffic failing basic transport integrity checks (e.g. TLS).

### Protocol Shape Validation

The host environment **MUST** validate only **wire-level protocol shape**, including:

- framing correctness,
- declared frame-type validity,
- size and bounds limits,
- syntactic structure required for routing.

The host environment **MUST NOT** inspect encrypted payload contents or infer semantic meaning.

### Abuse and Trash Traffic Mitigation

The host environment **MUST** mitigate:

- flooding and amplification attempts,
- repeated failed verification attempts,
- protocol misuse and malformed handshakes.

Mitigation **MAY** include throttling, temporary blocking, or challenge escalation, without semantic interpretation.

### Isolation and Blast Radius Control

The host environment **MUST** ensure:

- isolation between Base Stations of different Resources,
- isolation between verified and unverified traffic paths,
- that overload or failure of one Resource **MUST NOT** cascade to others.

### Logging Constraints

The host environment **MAY** log connection metadata and security events but **MUST NOT** log Snapshot or Delta payloads or derived meaning.

A deployment **MUST NOT** claim Base Station compliance unless these host requirements are met.

---

## Failure Model *(Normative)*

The Base Station **MUST** tolerate:

- restarts and hibernation,
- duplicate frames,
- partial relay,
- connection churn.

Correctness is preserved because the Base Station is non-authoritative.

---

## Explicit Non-Goals *(Normative)*

A Base Station **MUST NOT**:

- merge CRDTs,
- enforce ACLs or roles,
- track acknowledgments,
- implement revocation logic,
- infer meaning.

Any such behavior is **non-conformant**.

---

## Closing Principle *(Non-Normative)*

A Base Station:

- persists **Snapshots**,
- relays **Deltas**,
- relays **Snapshots on verified request**,
- understands **none of it**.

Infrastructure only. Authority elsewhere.
