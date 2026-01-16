# Station Client — Component Specification

## Status

This document defines the **Actor-side Station Client** component.

It specifies the **normative responsibilities, constraints, and prohibitions** for an Actor interacting with a Base Station over the **z-base Wire Control Protocol (WCP)**.

This component **MUST** implement the wire-level behavior defined in:

- [Wire Control Protocol](../../wire_control/PROTOCOL.md)

and **MUST** conform to:

- [Actor — Concept Specification](../CONCEPT.md)

RFC 2119 and RFC 8174 keywords apply.

---

## Role Definition

The **Station Client** is an Actor-owned transport component responsible for:

- establishing and maintaining a connection to a Base Station,
- completing possession-based verification,
- submitting opaque Snapshot and Delta frames,
- receiving opaque Snapshot and Delta frames,
- injecting received payloads into the Actor’s merge pipeline.

The Station Client:

- has **no authority over state correctness**,
- does **not coordinate synchronization**,
- does **not block Actor execution**,
- treats the Base Station as **untrusted infrastructure**.

---

## Authority Boundary *(Normative)*

The Station Client **MUST NOT** perform or assume responsibility for:

- state interpretation,
- authorization or role enforcement,
- merge logic,
- acknowledgment logic,
- revocation detection or enforcement,
- identity attribution semantics.

All such concerns remain strictly within the Actor.

---

## Protocol Compliance *(Normative)*

The Station Client **MUST** implement the Wire Control Protocol exactly as defined in:

- [Wire Control Protocol](../../wire_control/PROTOCOL.md)

Including, but not limited to:

- frame envelope and version handling,
- frame code space law,
- connection phases and verification rules,
- Snapshot and Delta submission semantics.

Any deviation is **non-conformant**.

---

## Connection Lifecycle *(Normative)*

### Connection Establishment

The Station Client **MAY** establish a connection at any time.

Connection establishment **MUST NOT** gate Actor execution, state access, or local operation production.

---

### Unverified Phase

Upon connection establishment, the Station Client **MUST**:

- await `AssertChallenge (0x01)` from the Base Station,
- **MUST NOT** send any frame other than `ProvePossession (0x21)`.

Sending any other frame during this phase is **non-conformant**.

---

### Verification Phase

The Station Client **MUST**:

- respond to `AssertChallenge (0x01)` with `ProvePossession (0x21)`,
- provide proof of possession of Resource-bound cryptographic material.

Verification **MUST** be treated strictly as a **gate to byte exchange**, not as identity attribution or authorization.

---

### Verified Session

Once verification succeeds, the Station Client **MUST**:

- be prepared to receive `OfferSnapshot (0x41)` immediately,
- accept Snapshot and Delta frames concurrently,
- tolerate unordered, duplicated, or interleaved delivery.

Receipt of a Snapshot **MUST NOT** be treated as a synchronization barrier.

---

## Snapshot Handling *(Normative)*

### Receiving Snapshots

Upon receiving `OfferSnapshot (0x41)`, the Station Client **MUST**:

- treat the payload as an opaque Snapshot,
- inject the payload into the Actor’s merge pipeline asynchronously,
- avoid blocking UI, state access, or local operation production.

Snapshots **MAY** arrive:

- immediately after verification,
- after Deltas,
- multiple times,
- duplicated or reordered.

All such cases **MUST** be tolerated.

---

### Submitting Snapshots

The Station Client **MAY** submit a Snapshot via `SubmitSnapshot (0x22)` when:

- the Actor has produced a new authoritative Snapshot, or
- local policy determines persistence is appropriate.

The Station Client **MUST NOT** assume ordering, exclusivity, or acceptance.

---

### Requesting Snapshots

The Station Client **MAY** send `RequestSnapshot (0x24)` at any time after verification.

Snapshot requests **MUST NOT** block Actor execution or state access.

---

## Delta Handling *(Normative)*

### Receiving Deltas

Upon receiving `RelayDelta (0x42)`, the Station Client **MUST**:

- inject the payload into the Actor’s merge pipeline,
- make no assumptions about ordering, uniqueness, or completeness.

---

### Submitting Deltas

The Station Client **MAY** submit Delta frames via `SubmitDelta (0x23)` as operations are produced locally.

Delta submission **MUST NOT** wait for Snapshot convergence, acknowledgment, or network guarantees.

---

## Asynchrony Guarantees *(Normative)*

The Station Client **MUST** preserve the Actor’s asynchronous execution model:

- no operation **MAY** block on network I/O,
- state access **MUST** remain immediate,
- merge occurs opportunistically as bytes arrive.

The Station Client exists to **transport bytes**, not to coordinate state.

---

## Failure Handling *(Normative)*

The Station Client **MUST** tolerate:

- connection drops,
- duplicate frames,
- reordered delivery,
- partial relay,
- Base Station restarts.

Recovery **MUST** rely exclusively on Actor-side merge and replay.

---

## Security Properties *(Normative)*

The Station Client **MUST**:

- validate protocol version and frame codes,
- reject frames violating the frame code space law,
- treat delivery and persistence as non-authoritative.

The Station Client **MUST NOT**:

- infer trust from delivery,
- infer correctness from persistence,
- expose wire assumptions to application logic.

---

## Conformance

A Station Client implementation conforms **iff** it:

- fully implements the Wire Control Protocol,
- preserves Actor authority and asynchrony,
- injects all received frames into the merge pipeline,
- submits Snapshots and Deltas without blocking or assumptions,
- tolerates all defined failure modes.

---

## Closing Statement *(Non-Normative)*

The Station Client is a transport adapter.

It connects an already-running Actor to an untrusted wire without introducing authority, ordering, or coordination.
