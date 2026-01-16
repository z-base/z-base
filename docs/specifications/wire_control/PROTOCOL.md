## Status

This document defines the **z-base Wire Control Protocol (WCP)**.  
It specifies a **binary, versioned, role-aware wire protocol** governing how Actors and Base Stations exchange opaque bytes.

This protocol is **normative for wire behavior** and **authoritative only over delivery obligations**, never over meaning.

It is the **single shared contract** referenced by:

- `base_station/components/ISOLATE.md`
- `actor/components/BASE_STATION_CLIENT.md`

RFC 2119 / 8174 keywords apply.

---

## Design Goals

The protocol is designed to:

- Support **fully asynchronous Actors** already running before connectivity.
- Allow **baseline state injection** without blocking or coordination.
- Preserve **semantic blindness** at the Base Station.
- Scale across Resources without cross-contamination.
- Enable **mechanical validation** of protocol correctness without payload inspection.
- Remain extensible without breaking old implementations.

The wire defines **how bytes move**, never **what they mean**.

---

## Core Invariants

- Payloads are always opaque.
- Delivery may be unordered, duplicated, or lossy.
- Verification gates byte exchange; it does not define lifecycle.
- Authority, merge, and correctness exist strictly above the wire.
- Every connection is scoped to exactly one Resource.

---

## Transport Assumptions

- Reliable framed transport (e.g. WebSocket).
- Message boundaries preserved.
- Transport security (e.g. TLS) is assumed but not specified here.

---

## Frame Envelope

Every frame **MUST** use the following envelope:

```

[ 1 byte VERSION ][ 1 byte FRAME_CODE ][ N bytes PAYLOAD ]

```

### VERSION

- Protocol version.
- This specification defines **VERSION = 0x01**.
- Frames with unknown VERSION **MUST** be rejected.

### FRAME_CODE

- Defines **wire-level obligation only**.
- Determines who may emit the frame and how the Base Station must treat it.
- Payload meaning is undefined at the wire level.

---

## Frame Code Space Law (Normative)

Frame codes are allocated by **emitter role and obligation class**.

This law is mandatory and mechanically enforceable.

| Range       | Class                         | Emitted By | Obligations |
|-------------|------------------------------|------------|-------------|
| 0x00–0x1F   | Station Control               | Station    | Verification / gating |
| 0x20–0x3F   | Client Submission             | Client     | Persistence or relay |
| 0x40–0x5F   | Station Relay / Offer         | Station    | Forwarding |
| 0x60–0x7F   | Reserved (future control)     | —          | — |
| 0x80–0xFF   | Forbidden                     | —          | Must reject |

Any violation of emitter role or range **MUST** terminate the connection.

---

## Defined Frame Codes (v0x01)

### Station Control (0x00–0x1F)

| Code | Name            | Direction | Description |
|------|-----------------|-----------|-------------|
| 0x01 | AssertChallenge | Station → Client | Sends verification challenge |

---

### Client Submission (0x20–0x3F)

| Code | Name             | Direction | Persisted | Description |
|------|------------------|-----------|-----------|-------------|
| 0x21 | ProvePossession  | Client → Station | No | Proof of Resource-bound key possession |
| 0x22 | SubmitSnapshot   | Client → Station | Yes | Persist opaque Snapshot |
| 0x23 | SubmitDelta      | Client → Station | No | Submit opaque Delta |
| 0x24 | RequestSnapshot  | Client → Station | No | Explicit snapshot request |

---

### Station Relay / Offer (0x40–0x5F)

| Code | Name           | Direction | Description |
|------|----------------|-----------|-------------|
| 0x41 | OfferSnapshot  | Station → Client(s) | Send persisted Snapshot |
| 0x42 | RelayDelta     | Station → Client(s) | Relay Delta |

---

## Connection Phases

### Phase 0 — Unverified

Immediately after connection:

- Station **MUST** send `AssertChallenge (0x01)`.
- Client **MUST NOT** send any frame except `ProvePossession (0x21)`.

Any other frame **MUST** close the connection.

---

### Phase 1 — Verification

- Client sends `ProvePossession (0x21)` with proof of possession.
- Station verifies without semantic inspection.
- On failure: connection **MUST** close.
- On success: transition to **Verified**.

---

### Phase 2 — Verified Session

Upon verification:

1. Connection is added to the verified peer set.
2. **If a Snapshot is persisted**, Station **MUST immediately send** `OfferSnapshot (0x41)`.
3. Delta relay **MAY** begin concurrently.

No ordering guarantees exist between Snapshot offers and Delta relays.

This ensures baseline state is injected opportunistically into the Actor’s merge stream.

---

## Snapshot Rules (Wire-Level)

### SubmitSnapshot (0x22)

- Payload is an opaque Snapshot.
- Station **MUST** persist it verbatim as the latest Snapshot.
- Previous Snapshot **MUST** be replaced.
- No inspection or validation is permitted.

---

### OfferSnapshot (0x41)

- Sent automatically after verification if Snapshot exists.
- Sent in response to `RequestSnapshot (0x24)`.
- Payload is the latest persisted Snapshot.
- Station **MUST NOT** wait for explicit request on initial verification.

---

### RequestSnapshot (0x24)

- Client **MAY** request Snapshot at any time after verification.
- Station **MUST** respond with `OfferSnapshot (0x41)` if available.

---

## Delta Rules (Wire-Level)

### SubmitDelta (0x23)

- Payload is an opaque Delta.
- Station **MUST NOT** persist it.
- Station **MUST** relay to all verified peers except sender.

---

### RelayDelta (0x42)

- Delta forwarded verbatim.
- Delivery may be duplicated or reordered.
- No acknowledgment or ordering guarantees exist.

---

## Relay Semantics

For all relay frames:

- Forward to all verified peers except sender.
- Preserve bytes exactly.
- Never block on persistence.
- Never infer identity, meaning, or causality.

---

## Resource Isolation

- Every connection is bound to exactly one Resource.
- Frames **MUST NOT** cross Resource boundaries.
- No multiplexing of Resources per session.

---

## Failure Model

The protocol tolerates:

- Connection churn
- Station restarts
- Duplicate frames
- Reordering
- Partial delivery

Recovery is exclusively Actor-side.

---

## Security Properties (Wire-Level)

The protocol enforces only:

- Possession-based gating
- Resource isolation
- Structural protocol correctness

It enforces **no semantic correctness**.

---

## Conformance

An implementation conforms **iff** it:

- Enforces VERSION and frame code space law,
- Applies verification gating strictly,
- Offers persisted Snapshots immediately after verification,
- Preserves payload opacity and byte integrity,
- Obeys relay and persistence obligations exactly.

---

## Required Cross-References

- Base Station behavior: `base_station/components/ISOLATE.md`
- Actor behavior: `actor/components/BASE_STATION_CLIENT.md`

Those documents **MUST** reference this file and **MUST NOT** redefine wire semantics.

---

## Closing Principle

The wire is a **byte discipline**, not a state machine.  
Actors are live before, during, and after the wire.  

This protocol exists to **inject bytes**, nothing more.
