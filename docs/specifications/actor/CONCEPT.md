# Actor — Concept Specification

## Abstract

An **Actor** is a user-controlled, authoritative execution unit responsible for creating, validating, mutating, attributing, synchronizing, revoking, erasing, and cryptographically protecting application state. All semantic meaning, trust decisions, authorization, conflict resolution, revocation enforcement, and lifecycle guarantees occur exclusively within the Actor.

External systems—including storage, transport, peers, and infrastructure—are treated as untrusted, opaque, and non-authoritative.

Actors are **fully asynchronous by design**. An Actor exposes state immediately using a developer-defined schema with default values and incrementally converges as operations are discovered, received, validated, and merged. No Actor operation blocks on network I/O or remote data.

This document defines the **normative requirements** for Actor implementations.

---

## Status

This document is a **Draft Specification**, version **1.0.0**, edited **16 January 2026**.  
It is provided for review and experimentation and **MUST NOT** be considered stable or complete.

Normative requirements and component boundaries may change without notice.

---

## Conformance

This specification defines requirements for **Actor implementations**.

An implementation conforms **iff** it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

Sections marked _Non-Normative_ are informational and **do not affect conformance**.

---

## Actor Definition _(Normative)_

An Actor is a software agent executing in a user-controlled environment that is **authoritative over state**.

An Actor **MUST**:

- create and mutate state locally,
- expose state immediately using schema-defined defaults,
- validate and authorize all operations it accepts,
- attribute operations to cryptographic identities,
- merge state deterministically and asynchronously,
- emit events as state advances,
- enforce revocation and mandatory erasure,
- encrypt, decrypt, and authenticate all persisted or transmitted state,
- operate correctly without network connectivity.

An Actor **MUST NOT** delegate semantic interpretation, authorization, attribution, merge logic, revocation enforcement, or state realization timing to any external system.

---

## Authority Boundary _(Normative)_

The Actor defines a strict authority boundary.

### Inside the Boundary (Trusted)

- schema interpretation and default state construction
- state validation and authorization
- cryptographic verification
- role and capability enforcement
- causal ordering and merge semantics
- asynchronous event emission
- acknowledgment logic
- revocation detection and enforcement
- garbage collection decisions

### Outside the Boundary (Untrusted)

- storage
- transport
- ordering
- availability
- durability
- peers

All externally supplied data **MUST** be treated as hostile until fully verified.

---

## Actor Identity _(Normative)_

### Actor Identity Key

Each Actor **MUST** possess a persistent **Actor Identity Key** pair.

The identity key:

- **MUST** uniquely identify a single Actor instance,
- **MUST NOT** be shared across Actors,
- **MUST** be used for attribution and acknowledgment signing,
- **MUST** be independent from authorization or role keys.

Loss or compromise of the identity key **MUST** be treated as loss of Actor continuity.

### Identity Persistence

An Actor **MUST** persist its identity key securely across restarts.  
Identity persistence **MUST NOT** depend on network availability.

---

## Asynchronous State Model _(Normative)_

### Schema-Defined Default State

For each Resource, an Actor **MUST** be initialized with a developer-defined schema that declares:

- the shape of state,
- default values for all fields,
- invariants enforced by the Actor.

Upon access, the Actor **MUST** expose an immediate default state view derived solely from the schema, without waiting for storage, synchronization, or merge completion.

### Incremental State Realization

Authoritative state **MUST** advance only through validated merge of operations.  
State **MUST** become progressively more complete as operations are discovered, received, validated, and merged.

No consumer **MUST** be required to wait for full state availability.

---

## Asynchronous Event Engine _(Normative)_

An Actor **MUST** include an asynchronous event engine governing how state changes are surfaced.

### Event Semantics

- State changes **MUST** be communicated exclusively via asynchronous events.
- Events **MUST** be emitted only after local validation and merge.
- Events **MUST NOT** be emitted speculatively for unmerged operations.

### Event Categories

An Actor **MUST** be able to emit at least:

- **State Advancement Events** — authoritative state changes after merge
- **Operation Rejection Events** — invalid or unauthorized operations
- **Conflict or Correction Events** — causal reconciliation signals
- **Revocation Events** — role loss and enforced erasure
- **Error Events** — structural or cryptographic failures

### Non-Blocking Guarantees

- Event emission **MUST** be non-blocking.
- State access **MUST** never block on I/O, synchronization, or merge.
- Application layers **MAY** subscribe to events and update incrementally.

---

## Internal Component Model _(Normative)_

An Actor implementation **MUST** be decomposable into the following distinguishable components:

- discoverable-credentials
- Identity & Key Manager
- Credential & Capability Manager
- Offline Storage Engine
- Resource Manager
- Operation Engine
- Merge Engine
- Asynchronous Event Engine
- Revocation Monitor & Enforcer
- Acknowledgment Engine
- Garbage Collector / Compactor
- **Station Client**
- Peer & Local Broadcast Channel
- Synchronization Controller
- Cryptographic Envelope Engine
- Policy & Validation Layer

The Station Client component **MUST** conform to:

- [Station Client — Component Specification](components/STATION_CLIENT.md)

Additional components **MAY** exist but **MUST NOT** weaken Actor authority, asynchrony, offline-first behavior, revocation guarantees, or deterministic merge semantics.

---

## State Ownership _(Normative)_

An Actor **MUST** maintain its own local replica of all state it participates in.

Local merged state is authoritative for that Actor.  
Schema-defined default state is authoritative until superseded by merged operations.

Remote data **MAY** influence state only after validation and merge.

---

## Synchronization Behavior _(Normative)_

Synchronization is opportunistic, unordered, lossy, and untrusted.

Actors **MUST** recover solely through replay and deterministic merge.

---

## Revocation Enforcement _(Normative)_

Upon detecting revocation, an Actor **MUST** immediately and irreversibly erase all access and state for the affected Resource and emit a revocation event.

---

## Failure Model _(Normative)_

Actors **MUST** tolerate crashes, restarts, partial state, duplication, malicious peers, and hostile infrastructure without authority escalation or silent corruption.

---

## Closing Principle _(Non-Normative)_

An Actor is authoritative by construction.  
Infrastructure provides transport, not truth.
