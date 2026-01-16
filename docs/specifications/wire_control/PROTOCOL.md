## Status

This document defines the **Wire Control Protocol** for z-base.  
It is a **partially normative** specification describing **wire-level behavior only**: framing, handshake, routing intent, and delivery guarantees.  
It deliberately excludes all semantic interpretation, authorization meaning, merge logic, attribution, revocation, or state understanding.

This protocol is the **single shared contract** between:

- `base_station/components/ISOLATE.md`
- `actor/components/BASE_STATION_CLIENT.md`

Both components **MUST** implement this protocol exactly as specified and **MUST** cross-reference this document for all wire behavior.

RFC 2119 / 8174 keywords apply.

---

## Scope and Non-Scope

### In Scope

- Binary frame encoding
- Connection phases
- Verification gating
- Frame classes and routing intent
- Relay and persistence obligations
- Failure and tolerance properties

### Explicitly Out of Scope

- Cryptographic algorithm choice
- Snapshot or Delta structure
- Authorization semantics
- Actor identity meaning
- Merge, ordering, or acknowledgment logic
- Revocation handling

All payloads are **opaque** to the wire.

---

## Fundamental Invariants

- The wire moves **bytes only**.
- The wire has **no authority**.
- The wire **never interprets payloads**.
- The wire **cannot infer identity, roles, or meaning**.
- All correctness emerges **above** this protocol.

---

## Transport Assumptions

- The protocol is defined over a **reliable, ordered byte stream** (e.g. WebSocket).
- Message boundaries are preserved.
- TLS or equivalent transport security is assumed but **not specified here**.

---

## Frame Structure

Every message on the wire **MUST** conform to the following structure:

```

[ 1 byte FRAME_CODE ][ N bytes PAYLOAD ]

```

- `FRAME_CODE` defines **routing intent only**.
- `PAYLOAD` is an opaque binary blob.
- The wire **MUST NOT** inspect, transform, or validate payload contents.

---

## Frame Codes

| Code | Name                      | Direction             | Persistence | Description |
|-----:|---------------------------|-----------------------|-------------|-------------|
| 0x01 | AssertChallenge           | Station → Client      | No          | Verification challenge |
| 0x05 | ProvePossession           | Client → Station      | No          | Proof of cryptographic possession |
| 0x10 | SubmitSnapshot            | Client → Station      | Yes         | Full opaque snapshot |
| 0x11 | SubmitDelta               | Client → Station      | No          | Opaque delta |
| 0x12 | RequestSnapshot           | Client → Station      | No          | Request latest persisted snapshot |
| 0x13 | RelaySnapshot             | Station → Client(s)   | No          | Snapshot relay |
| 0x14 | RelayDelta                | Station → Client(s)   | No          | Delta relay |

Frame codes define **delivery behavior**, not meaning.

---

## Connection Phases

### Phase 0 — Unverified

Immediately after connection establishment:

- The Station **MUST** send `AssertChallenge (0x01)`.
- The Client **MUST NOT** send any frame except `ProvePossession (0x05)`.

Any other frame received in this phase **MUST** cause immediate termination.

---

### Phase 1 — Verification

- The Client responds with `ProvePossession (0x05)` containing proof of possession of Resource-bound cryptographic material.
- The Station **MUST** verify the proof without semantic inspection.
- On failure, the connection **MUST** be closed.
- On success, the connection transitions to **Verified**.

---

### Phase 2 — Verified Session

Once verified:

- The connection is added to the **verified peer set**.
- All frame types defined in this document become valid.
- Relay and persistence rules apply.

---

## Snapshot Semantics (Wire-Level)

### SubmitSnapshot (0x10)

- Payload is an opaque encrypted Snapshot.
- Station **MUST** persist the payload verbatim as the latest Snapshot.
- Station **MUST** replace any previously persisted Snapshot.
- Station **MUST NOT** modify, validate, or interpret it.

---

### RequestSnapshot (0x12)

- A verified Client may request the latest persisted Snapshot.
- Station **MUST** respond with `RelaySnapshot (0x13)` if a Snapshot exists.
- If no Snapshot exists, the Station **MAY** remain silent.

---

### RelaySnapshot (0x13)

- Station sends the latest persisted Snapshot payload.
- Delivery **MAY** be duplicated or reordered.
- Payload integrity is preserved byte-for-byte.

---

## Delta Semantics (Wire-Level)

### SubmitDelta (0x11)

- Payload is an opaque encrypted Delta.
- Station **MUST NOT** persist Delta payloads.
- Station **MUST** relay the Delta to all verified peers except the sender.

---

### RelayDelta (0x14)

- Station forwards the Delta payload verbatim.
- No ordering, uniqueness, or delivery guarantees are provided.

---

## Relay Rules

For all relay frames:

- Station **MUST** forward to **all verified peers except the sender**.
- Station **MUST NOT** inspect payloads.
- Station **MUST** preserve payload bytes exactly.
- Station **MAY** drop, duplicate, or reorder frames.

---

## Resource Isolation

- All connections governed by this protocol are scoped to **exactly one Resource**.
- Frames **MUST NOT** cross Resource boundaries.
- A Station instance **MUST NOT** multiplex Resources on the same verified session.

---

## Failure Model

This protocol **assumes**:

- Connection drops
- Duplicate frames
- Partial delivery
- Reordering
- Station restarts

The wire provides **no recovery logic**.  
All recovery occurs at the Actor layer via replay and merge.

---

## Security Properties (Wire-Level)

The protocol guarantees only:

- Possession-based access gating
- Opaque payload handling
- Isolation by Resource identifier

It does **not** guarantee:

- Authorization correctness
- Identity validity
- Replay prevention
- Revocation enforcement

Those properties **MUST** be enforced by Actors.

---

## Conformance

A component conforms to this protocol **iff** it:

- Implements all mandatory frame handling rules,
- Enforces verification gating,
- Preserves payload opacity,
- Obeys relay and persistence constraints,
- Does not introduce semantic interpretation.

---

## Required Cross-References

- Base Station behavior: `base_station/components/ISOLATE.md`
- Actor client behavior: `actor/components/BASE_STATION_CLIENT.md`

Those documents **MUST NOT** redefine wire behavior and **MUST** reference this file.

---

## Closing Statement

This protocol defines **how bytes move**.  
It intentionally defines **nothing else**.  

Meaning lives elsewhere.
Authority lives elsewhere.
