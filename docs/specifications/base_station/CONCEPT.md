# Base Station — Concept Specification

## Abstract

A **Base Station** is a cloud-hosted, non-authoritative infrastructure service responsible for authenticated connectivity, persistence of opaque Resource Snapshots, and relay of opaque Snapshot and Delta frames for exactly one Resource.

A Base Station has **zero semantic awareness**, **no authority over state**, and **no application logic**. It verifies possession of Resource-bound cryptographic material, stores opaque snapshot blobs, and relays opaque blobs between verified peers.

All semantic interpretation, authorization, attribution, merge logic, revocation enforcement, and lifecycle semantics are performed exclusively by Actors.

Every Resource **MUST** be served by exactly one logically isolated Base Station instance.

---

## Status

This document is a **Draft Specification**, version **1.0.0**, edited **16 January 2026**.  
It is provided for review and experimentation and **MUST NOT** be considered stable or complete.

Normative requirements and deployment constraints may change without notice.

---

## Conformance

This specification defines requirements for **Base Station implementations**.

An implementation conforms **iff** it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

---

## Base Station Definition *(Normative)*

A Base Station:

- serves **exactly one Resource identifier**,  
- verifies **possession** of Resource-bound cryptographic material,  
- **persists opaque Snapshot frames**,  
- **relays opaque Delta frames**, and  
- **relays opaque Snapshot frames on verified request**.

A Base Station **MUST NOT**:

- interpret payload contents,  
- validate or merge state,  
- enforce authorization semantics,  
- attribute identity,  
- track acknowledgments,  
- implement revocation logic, or  
- act as an Actor.

---

## One-Resource-Per-Station *(Normative)*

Each Resource **MUST** have its own logically isolated Base Station.

- No cross-Resource persistence is permitted.  
- No cross-Resource relay is permitted.  
- No cross-Resource peer visibility is permitted.

Isolation **MUST** hold under normal operation, overload, and partial failure.

---

## Required Infrastructure Components *(Normative)*

A conforming Base Station deployment **MUST** consist of the following **architecturally distinct components**.

### 1. Stateless Edge Gateway

The Stateless Edge Gateway:

- **MUST** be highly available and stateless,  
- **MUST** accept inbound connection requests,  
- **MUST** route requests deterministically to the correct per-Resource Base Station instance,  
- **MUST NOT** persist application state,  
- **MUST NOT** upgrade connections to verified sessions,  
- **MUST NOT** interpret payloads.

The gateway exists solely for **routing, termination, and load distribution**.

---

### 2. Per-Resource Stateful Base Station Core

The per-Resource Base Station Core:

- **MUST** exist as a distinct execution unit per Resource,  
- **MUST** perform protocol-defined verification gating,  
- **MUST** upgrade connections to verified sessions,  
- **MUST** maintain the verified peer set,  
- **MUST** persist opaque Snapshots,  
- **MUST** relay opaque Snapshot and Delta frames.

The Base Station Core **MUST** conform to:

- [Base Station Core — Component Specification](components/STATION.md)

---

### 3. In-Memory Snapshot Cache

While active, the Base Station Core:

- **MUST** keep the latest persisted Snapshot resident in memory,  
- **MUST NOT** interpret the Snapshot,  
- **MUST** use the cached Snapshot to serve Snapshot relay.

---

### 4. Durable Snapshot Storage

On eviction, restart, or hibernation, the Base Station:

- **MUST** persist the latest Snapshot to durable storage,  
- **MUST** restore it into memory on reactivation,  
- **MUST** treat storage contents as opaque.

Storage **MUST NOT** be used for indexing, inference, or correlation.

---

## Connection Lifecycle *(Normative)*

### Pre-Upgrade Verification

Before upgrading to a verified session, the Base Station **MUST**:

- require proof of possession of Resource-bound cryptographic material,  
- verify the proof without semantic inspection,  
- reject the connection on failure.

Unverified connections **MUST NOT** send or receive Snapshot or Delta frames.

---

### Verified Session

Once verified, a connection:

- **MUST** be added to the verified peer set,  
- **MAY** submit Snapshot frames,  
- **MAY** submit Delta frames,  
- **MAY** request Snapshot relay,  
- **MAY** receive relayed frames.

Verification **MUST NOT** be treated as identity attribution or authorization.

---

### Connection Closure

On disconnect, the connection **MUST** be removed from the verified peer set.

No semantic inference or cleanup beyond removal is permitted.

---

## Frame Classes *(Normative)*

### Snapshot Frames

- Encrypted, opaque full Resource state.  
- **MUST** be persisted.  
- **MUST** be relayed on verified request.  
- **MAY** be relayed automatically after verification.

---

### Delta Frames

- Encrypted, opaque incremental updates.  
- **MUST NOT** be persisted.  
- **MUST** be relayed to verified peers.

---

## Relay Semantics *(Normative)*

### Delta Relay

For each Delta frame, the Base Station **MUST**:

- relay to all verified peers except the sender,  
- allow unordered, duplicated, or lossy delivery.

---

### Snapshot Relay

Upon Snapshot request from a verified peer, the Base Station **MUST**:

- relay the latest persisted Snapshot,  
- preserve payload bytes exactly,  
- perform no validation or synthesis.

Snapshot relay **DOES NOT** replace persistence.

---

## Host Environment Requirements *(Normative)*

A Base Station **MUST** be deployed in a compliant host environment that enforces baseline protections **before traffic reaches the Stateless Edge Gateway**.

These controls protect infrastructure availability and **MUST NOT** introduce application semantics.

### Pre-Gateway Traffic Filtering

The host environment **MUST**:

- enforce rate and burst limits,  
- enforce connection limits,  
- reject malformed or non-conforming protocol requests,  
- drop traffic failing basic transport integrity checks.

---

### Protocol Shape Validation

The host environment **MUST** validate only **wire-level protocol shape**, including:

- framing correctness,  
- declared frame-type validity,  
- size and bounds limits.

The host environment **MUST NOT** inspect encrypted payload contents.

---

### Abuse and Trash Traffic Mitigation

The host environment **MUST** mitigate:

- flooding and amplification attempts,  
- repeated failed verification attempts,  
- malformed or abusive handshake patterns.

Mitigation **MAY** include throttling or temporary blocking without semantic interpretation.

---

### Isolation and Blast Radius Control

The host environment **MUST** ensure:

- isolation between Resources,  
- isolation between verified and unverified traffic paths,  
- that overload or failure of one Resource **MUST NOT** cascade to others.

---

### Logging Constraints

The host environment **MAY** log connection metadata and security events but **MUST NOT** log Snapshot or Delta payloads or derived meaning.

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

## Cross-References

- [Base Station Core — Component Specification](components/STATION.md)
- [Wire Control Protocol](../wire_control/PROTOCOL.md)

---

## Closing Principle *(Non-Normative)*

The Base Station persists Snapshots.  
The Base Station relays Deltas.  
The Base Station understands none of it.

Infrastructure only. Authority elsewhere.
