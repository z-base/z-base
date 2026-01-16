
```markdown
# base_station/components/STATION.md

## Status

This document defines the **z-base Base Station (Per-Resource Stateful Core)**.

It specifies the **normative responsibilities and prohibitions** of a Base Station instance serving **exactly one Resource**.  
It is authoritative **only over wire enforcement, persistence of opaque Snapshots, and relay obligations**.

This component **MUST** implement `wire_control/PROTOCOL.md` exactly and **MUST NOT** introduce additional semantics.

RFC 2119 / 8174 keywords apply.

---

## Role Definition

A **Base Station** is a non-authoritative, per-Resource execution unit that:

- verifies **possession** of Resource-bound cryptographic material,
- maintains a verified peer set,
- persists the **latest opaque Snapshot**,
- relays **opaque Delta and Snapshot frames**.

A Base Station:

- has **zero semantic awareness**,
- has **no authority over state correctness**,
- is **not an Actor**.

---

## Resource Scope (Normative)

- A Base Station **MUST** serve **exactly one Resource identifier**.
- All connections, frames, and persistence handled by the Station **MUST** be scoped to that Resource.
- Cross-Resource routing, persistence, or peer awareness is **non-conformant**.

---

## Protocol Compliance (Normative)

The Base Station **MUST** implement:

- Frame envelope, versioning, and frame code law,
- Connection phases and verification gating,
- Relay and persistence obligations,

exactly as defined in:

```

wire_control/PROTOCOL.md

```

Any deviation is **non-conformant**.

---

## Connection Handling

### Unverified Phase

Upon connection establishment, the Station **MUST**:

1. Immediately emit `AssertChallenge (0x01)`.
2. Reject or close the connection if any frame other than `ProvePossession (0x21)` is received.

No frames **MAY** be relayed or persisted during this phase.

---

### Verification Phase

The Station **MUST**:

- verify proof of possession without inspecting payload meaning,
- treat verification strictly as a **gating mechanism**, not identity attribution.

On failure, the connection **MUST** be terminated.

---

### Verified Session

Upon successful verification, the Station **MUST**:

1. Add the connection to the verified peer set.
2. **Immediately offer the persisted Snapshot**, if one exists, via `OfferSnapshot (0x41)`.
3. Begin concurrent relay of Deltas without ordering guarantees.

Verification **MUST NOT** be treated as a synchronization barrier.

---

## Snapshot Responsibilities (Normative)

### Persistence

- The Station **MUST** persist only frames received via `SubmitSnapshot (0x22)`.
- Persistence **MUST** be opaque and byte-for-byte.
- Only the **latest Snapshot** is retained.
- Persistence **MUST** survive restarts, eviction, or hibernation.

The Station **MUST NOT**:

- merge Snapshots,
- validate contents,
- infer causality or authorship.

---

### Snapshot Offering

The Station **MUST**:

- automatically send `OfferSnapshot (0x41)` after verification if a Snapshot exists,
- respond to `RequestSnapshot (0x24)` with `OfferSnapshot (0x41)` when possible.

The Station **MUST NOT** require explicit coordination or readiness signals.

---

## Delta Responsibilities (Normative)

- The Station **MUST NOT** persist Deltas.
- Upon receiving `SubmitDelta (0x23)`, the Station **MUST** relay the payload to all verified peers except the sender.
- Delivery **MAY** be duplicated, reordered, or dropped.

The Station **MUST NOT** attempt replay protection, acknowledgment tracking, or ordering.

---

## Relay Semantics

For all relayed frames, the Station **MUST**:

- preserve payload bytes exactly,
- avoid reflection to the sender,
- avoid semantic inspection,
- avoid blocking on persistence or peer state.

---

## Failure and Restart Model

The Station **MUST** tolerate:

- process restarts,
- connection churn,
- duplicate frames,
- partial relay.

On restart, the Station **MUST**:

- restore the latest persisted Snapshot into memory,
- resume normal protocol behavior.

Correctness is preserved because the Station is non-authoritative.

---

## Security and Isolation

The Station **MUST** ensure:

- isolation between Resources,
- isolation between verified and unverified connections,
- that failure or overload of one Resource **MUST NOT** cascade to others.

The Station **MUST NOT**:

- log Snapshot or Delta payloads,
- derive meaning from identifiers or payloads,
- attribute identity beyond possession gating.

---

## Explicit Non-Goals (Normative)

The Station **MUST NOT**:

- validate or merge state,
- enforce authorization or roles,
- track acknowledgments,
- detect or enforce revocation,
- infer Actor identity,
- act as a source of truth.

Any such behavior is **non-conformant**.

---

## Conformance

A Base Station implementation conforms **iff** it:

- fully implements `wire_control/PROTOCOL.md`,
- enforces strict per-Resource isolation,
- persists only opaque Snapshots,
- relays frames without semantic interpretation,
- injects persisted Snapshots immediately after verification.

---

## Cross-Reference

- Wire behavior: `wire_control/PROTOCOL.md`
- Client behavior: `actor/components/STATION_CLIENT.md`

---

## Closing Principle

The Base Station is **infrastructure**, not logic.  
It remembers bytes, forwards bytes, and forgets meaning.

Authority lives elsewhere.
```
