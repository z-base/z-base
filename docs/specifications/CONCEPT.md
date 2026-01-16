# z-base — Concept Specification

## Abstract

z-base specifies a zero-knowledge state relay and persistence API for browser-based JavaScript Actors. It defines how encrypted, opaque state blobs are created, stored, relayed, and synchronized by a non-authoritative service, while all state validity, authorization, conflict resolution, attribution, and cryptographic verification are performed exclusively by Actors. The model guarantees offline-first operation, eventual consistency, unlinkability of stored data, and cryptographic impostor resistance, such that infrastructure cannot interpret, correlate, decrypt, or falsely attribute application state. This document defines the normative requirements for interoperable Actor and Base Station implementations.

---

## Status of This Document

This document is a **Draft Specification** for **z-base**, version **1.0.0**, edited on **16 January 2026**.  
It is provided for review and experimentation and **must not be considered stable or complete**.  
Normative requirements and conformance criteria may change without notice.

This specification is **AI-assisted** and authored with contributions from **Jori Lehtinen** and **GPT-5.2**.  
Implementations should expect breaking changes until this document reaches a finalized status.

---

## Conformance

This specification defines conformance requirements for two roles:

### Actor Implementation

Software executing in a user-controlled environment that creates, verifies, merges, encrypts, decrypts, attributes, and authoritatively interprets application state.

### Base Station Implementation

A non-authoritative service that stores, forwards, and relays opaque binary data according to this specification.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

An implementation **conforms** if and only if it satisfies all normative requirements applicable to its declared role.  
Implementations may conform to one or both roles independently.

Sections marked *Non-Normative*, along with examples and walkthroughs, are informational and **do not affect conformance**.

---

## Terminology

**Actor**  
A user-controlled software agent that is authoritative over state creation, validation, authorization, attribution, conflict resolution, encryption, and decryption.

**Base Station**  
A non-authoritative service that stores, forwards, and relays opaque binary data without the ability to interpret, validate, correlate, decrypt, or attribute it.

**Resource**  
The sole unit of addressable state in z-base, identified by a high-entropy identifier and represented only in encrypted form.

**Resource Snapshot**  
An Actor-defined, pre-encryption representation of Resource state consisting of verifiable operations suitable for deterministic merge.

**Envelope**  
The encrypted form of a Resource Snapshot, safe for storage, transport, and forwarding by an untrusted Base Station.

**Opaque Blob**  
A serialized binary representation of an Envelope with no semantic meaning outside an Actor.

**Authoritative State**  
State whose validity, ordering, authorization, attribution, and merge semantics are determined exclusively by Actors.

**Offline-First Operation**  
A mode in which Actors can read and mutate state without network connectivity, synchronizing opportunistically.

**Eventual Consistency**  
A property whereby independent Actor replicas converge to equivalent authoritative state given sufficient time and communication.

**Actor Identity Key**  
A persistent cryptographic signing key pair uniquely identifying an Actor instance for attribution and impostor resistance, distinct from role keys.

---

## Architecture Overview *(Non-Normative)*

z-base follows an **actor-authoritative architecture**. All meaningful interpretation, validation, authorization, attribution, and merge logic occurs within Actors. Base Stations act only as opaque persistence and relay nodes and never participate in state semantics.

Actors maintain local replicas of Resource state and interact with Base Stations using high-entropy identifiers and authenticated sessions to publish, retrieve, and relay encrypted Envelopes. All Base Station–handled data remains opaque and unlinkable.

Synchronization proceeds incrementally via encrypted snapshots and deltas. Each step reveals only what is necessary to locate subsequent state, preserving unlinkability while enabling offline-first operation, eventual consistency, and post-hoc verification of authorship.

---

## Resource Model *(Normative)*

A **Resource** is the sole unit of addressable state in z-base.

Each Resource **MUST** be identified by a **256-bit high-entropy identifier**, encoded as a **43-character Base64URL string**. The identifier **MUST** be unguessable and **MUST NOT** convey semantic meaning to a Base Station.

A Resource **MUST** be represented exclusively by an **Envelope**, produced by encrypting a Resource Snapshot.  
Base Stations **MUST** treat Envelopes as uninterpreted binary data and **MUST NOT** derive semantics, linkage, ordering, validity, or authorship from identifiers or payloads.

---

## Resource Snapshot *(Normative)*

A **Resource Snapshot** is the canonical Actor-defined representation of Resource state prior to encryption.

A Snapshot **MUST** consist of an ordered collection of **verifiable operations**. Each operation **MUST**:

- be cryptographically authenticated for authorization by a role key,  
- be cryptographically attributable to an Actor identity key,  
- represent a CRDT operation suitable for deterministic merge, and  
- be authorized according to the Resource’s role model.

Interpretation, authorization, attribution, and merge semantics **MUST** be performed exclusively by Actors.  
Base Stations **MUST NOT** interpret operations, roles, signatures, or identity bindings.

A Snapshot **MUST** be encrypted by an Actor to produce an Envelope.

---

## Authority, Attribution, and Access *(Normative)*

Actors **MUST** be authoritative over Resource state. All validation, merge semantics, conflict resolution, attribution, encryption, and decryption **MUST** occur at the Actor layer.

Base Stations **MUST NOT** enforce state rules, resolve conflicts, attribute authorship, or reject data based on content.

Access **MUST** be capability-based. Possession of a valid Resource identifier and corresponding cryptographic material is sufficient to attempt access.  
Base Stations **MUST** require protocol-defined authentication but **MUST NOT** gain semantic or attributional insight as a result.

Resource Snapshots **MAY** contain application-defined references to other Resources. Such semantics **MUST** remain invisible to Base Stations.

---

## Actor Roles and Authorization Model *(Normative)*

z-base defines **role-based authorization** enforced exclusively by Actors.

### Roles

For each Resource, Actors **MUST** hold exactly one role:

- **Owner** — Full control, including assigning and revoking all roles and identity bindings.  
- **Manager** — May assign and revoke *Editor* and *Viewer* roles; **MUST NOT** modify Owner.  
- **Editor** — May produce state-modifying CRDT operations.  
- **Viewer** — Read-only access.  
- **Revoked** — No access.

### Role Keys and Verification

Each role **MUST** be associated with verifiable cryptographic key material.  
Operations **MUST** be signed using the key for the Actor’s current role.  
Actors **MUST NOT** accept operations whose role signatures do not verify against the role valid at the operation’s causal point.

### Role Transitions

Role assignments and revocations **MUST** be expressed as signed operations within the Snapshot.  
After merge, authoritative state reflects the most recent valid assignment.

---

## Actor Identity and Impostor Resistance *(Normative)*

### Actor Identity Keys

Each Actor **MUST** maintain a persistent **Actor Identity Key** pair distinct from all role keys.

The identity key:

- **MUST** uniquely identify a single Actor instance,  
- **MUST NOT** be shared between Actors,  
- **MAY** be rotated only via explicit signed operations recorded in the Resource Snapshot.

### Identity Binding

An Actor’s identity public key **MUST** be introduced and pinned into Resource state on first valid operation attributed to that Actor.

Once pinned, identity bindings **MUST** be used to verify future operations for attribution and impostor detection.

### Impostor Resistance Guarantees

An Actor **MUST** reject operations whose claimed Actor identity cannot be verified against the pinned identity key, even if the role signature is otherwise valid, for purposes of attribution and audit.

Failure of identity verification **MUST NOT** affect CRDT merge correctness but **MUST** be observable to higher layers as an impostor signal.

---

## Operation Format and Signing Semantics *(Normative)*

An **operation** is a self-contained, verifiable CRDT mutation and **MUST** include:

- Resource Identifier  
- Actor Identifier  
- Role Signature  
- Actor Identity Signature  
- Operation Payload  
- Causal Metadata

### Dual-Signature Requirement

- The **Role Signature** authorizes *what* the operation is allowed to do.  
- The **Actor Identity Signature** proves *who* produced the operation.

Signatures **MUST** be detached and cover all fields except other signatures. Any modification **MUST** invalidate the affected signature.

Actors **MUST** verify role authorization, identity attribution, and causal consistency before merge.  
Invalid operations **MUST** be excluded without affecting valid ones.

Acknowledgment operations **MAY** be emitted and **MUST NOT** mutate state.

---

## CRDT Merge Semantics *(Normative)*

Resource state **MUST** be derived from the validated operation set.

Given identical valid operation sets, all Actors **MUST** converge to identical authoritative state, independent of arrival order or transport behavior.

Operations **MUST** be idempotent and replay-safe.  
Garbage collection and compaction **MAY** be performed locally if equivalence and attribution verifiability are preserved.

Base Stations **MUST NOT** participate in merge, validation, or attribution.

---

## Envelope Cryptographic Properties *(Normative)*

- **Confidentiality:** Only authorized Actors can decrypt.  
- **Integrity:** Modification or replay **MUST** be detectable.  
- **Determinism:** Decryption yields exactly the encrypted Snapshot.  
- **Non-Observability:** No semantic or attributional leakage beyond routing necessity.  
- **Algorithm Agnosticism:** No specific algorithms are mandated.

---

## Synchronization and Wire Semantics *(Normative)*

This section defines how bytes move, not what they mean.

### Invariants

- Payloads are opaque to Base Stations.  
- Delivery may be lossy, duplicated, or reordered.  
- Authority and attribution remain with Actors.

### Framing

```

[ Frame Type ][ Payload Bytes ]

```

Frame Type indicates intent; Payload Bytes are forwarded losslessly.

### Access Gating

Before forwarding stateful frames, Base Stations **MUST** require proof of possession of cryptographic material associated with the Resource. The mechanism is implementation-defined and **MUST NOT** imply authority or identity.

### Snapshot and Delta Exchange

Actors **MAY** exchange full Snapshots or delta Envelopes.  
Ordering **MUST NOT** be assumed. Merge **MUST** be idempotent.

### Peer Relay and Offline Sync

Base Stations **MUST** act purely as relays.  
Identical semantics **MAY** be used offline (e.g. BroadcastChannel).

### Failure Handling

All failures **MUST** be recoverable via re-synchronization.  
Base Stations **MUST NOT** attempt repair.

---

## Non-Normative Walkthroughs

*(Cold start, offline usage, lazy resolution, concurrent sync, multi-source asynchrony, and failure recovery are unchanged in meaning and emphasize the invariant: state evolves locally; synchronization is opportunistic.)*

---

## Security Considerations *(Non-Normative)*

z-base assumes untrusted infrastructure, hostile networks, and potentially malicious peers. Confidentiality, integrity, authorization, and attribution are enforced end-to-end by Actors. Base Stations cannot forge, interpret, or falsely attribute state.

Metadata leakage is minimized but not eliminated. Compromised user agents are out of scope; recovery is delegated to higher layers.

---

## Future Extensions *(Non-Normative)*

Potential extensions include standardized sharing and delegation, automation Actors, public Resources, interoperable schemas, identity interop, advanced key lifecycle management, transport profiles, and formal verification.

All extensions **MUST NOT** weaken actor authority, attribution guarantees, zero-knowledge properties, or offline-first operation.

---

## Closing Note

z-base is intentionally small, strict, and composable at its core.  
Innovation is expected to occur at the edges, not by expanding the trusted center.
