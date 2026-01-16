# z-base/docs/specifications/actor/components/STATION_CLIENT.md

## Status

This document defines the **z-base Station Client** component.

It specifies the **normative client-side responsibilities** for an Actor interacting with a Base Station over the **z-base Wire Control Protocol (WCP)**.  
The Station Client is a **transport adapter**, not an authority.

This component **MUST** implement `wire_control/PROTOCOL.md` exactly and **MUST NOT** introduce semantic coupling to the Base Station.

RFC 2119 / 8174 keywords apply.

---

## Role Definition

The **Station Client** is the Actor-owned component responsible for:

- establishing and maintaining a connection to a Base Station,
- completing possession-based verification,
- sending opaque Snapshots and Deltas,
- receiving and injecting opaque Snapshots and Deltas into the Actor’s merge pipeline.

The Station Client:

- has **no authority over state correctness**,
- **does not block** Actor execution,
- **does not assume ordering or completeness** of received frames.

---

## Authority Boundary (Normative)

All of the following **MUST remain outside** the Station Client:

- state interpretation,
- authorization checks,
- merge logic,
- acknowledgment logic,
- revocation enforcement,
- identity attribution semantics.

The Station Client **MUST** treat the Base Station as untrusted infrastructure.

---

## Protocol Compliance (Normative)

The Station Client **MUST** implement:

- frame envelope and version handling,
- frame code space law,
- connection phases and verification rules,

exactly as defined in:

```

wire_control/PROTOCOL.md

```

Any deviation is **non-conformant**.

---

## Connection Lifecycle

### Connection Establishment

The Station Client **MAY** establish a connection at any time.  
Connection establishment **MUST NOT** gate Actor execution or state availability.

---

### Unverified Phase

Upon connection:

- The Client **MUST** await `AssertChallenge (0x01)`.
- The Client **MUST NOT** send any frame except `ProvePossession (0x21)`.

Any other behavior is **non-conformant**.

---

### Verification Phase

- The Client **MUST** respond to `AssertChallenge (0x01)` with `ProvePossession (0x21)`.
- The proof **MUST** demonstrate possession of Resource-bound cryptographic material.
- The Client **MUST NOT** assume identity attribution or authorization as a result of verification.

Verification is **only a gate to byte exchange**.

---

### Verified Session

Once verified:

- The Client **MUST** be prepared to receive `OfferSnapshot (0x41)` immediately.
- The Client **MUST** accept Snapshot and Delta frames concurrently and without ordering assumptions.
- The Client **MUST NOT** treat Snapshot receipt as a synchronization barrier.

---

## Snapshot Handling (Normative)

### Receiving Snapshots

Upon receiving `OfferSnapshot (0x41)`:

- The Client **MUST** treat the payload as an opaque Snapshot.
- The Client **MUST** inject it into the Actor’s merge pipeline asynchronously.
- The Client **MUST NOT** block UI, state access, or local operation production.

Snapshots **MAY** arrive:

- immediately after verification,
- after Deltas,
- multiple times,
- duplicated or reordered.

All such cases **MUST** be tolerated.

---

### Submitting Snapshots

The Client **MAY** submit a Snapshot via `SubmitSnapshot (0x22)` when:

- the Actor has produced a new authoritative Snapshot,
- or local policy determines persistence is desirable.

The Client **MUST NOT** assume acceptance, ordering, or exclusivity.

---

### Requesting Snapshots

The Client **MAY** send `RequestSnapshot (0x24)` at any time after verification.

This request **MUST NOT** block Actor execution.

---

## Delta Handling (Normative)

### Receiving Deltas

Upon receiving `RelayDelta (0x42)`:

- The Client **MUST** inject the payload into the Actor’s merge pipeline.
- The Client **MUST NOT** assume ordering, uniqueness, or completeness.

---

### Submitting Deltas

The Client **MAY** submit Deltas via `SubmitDelta (0x23)` as operations are produced locally.

Delta submission **MUST NOT** wait for Snapshot convergence or network acknowledgment.

---

## Asynchrony Guarantees

The Station Client **MUST** preserve the Actor’s asynchronous model:

- No operation **MAY** block on network I/O.
- State access **MUST** remain immediate.
- Merge occurs opportunistically as bytes arrive.

The Station Client exists to **feed bytes**, not to coordinate state.

---

## Failure Handling

The Station Client **MUST** tolerate:

- connection drops,
- duplicate frames,
- reordered delivery,
- partial relay,
- Station restarts.

Recovery **MUST** rely solely on Actor-side merge and replay.

---

## Security Properties

The Station Client **MUST**:

- validate protocol version and frame codes,
- reject frames violating the frame code space law,
- never trust ordering or delivery guarantees.

The Station Client **MUST NOT**:

- infer trust from delivery,
- treat persistence as authority,
- expose wire assumptions to application logic.

---

## Conformance

A Station Client implementation conforms **iff** it:

- fully implements `wire_control/PROTOCOL.md`,
- preserves Actor asynchrony and authority,
- injects all received frames into the merge pipeline,
- submits Snapshots and Deltas without blocking or assumptions,
- tolerates all defined failure modes.

---

## Cross-Reference

- Wire behavior: `wire_control/PROTOCOL.md`
- Base Station behavior: `base_station/components/STATION.md`

---

## Closing Principle

The Station Client is **not a synchronizer**.  
It is **not a coordinator**.  

It is a narrow conduit between an already-running Actor and an untrusted wire.
