# Actor — Concept Specification

## Abstract

An **Actor** is a user-controlled, authoritative execution unit responsible for creating, validating, mutating, attributing, synchronizing, revoking, erasing, and cryptographically protecting application state. All semantic meaning, trust decisions, authorization, conflict resolution, and lifecycle enforcement occur exclusively inside the Actor. External systems—storage, transport, peers, and infrastructure—are treated as untrusted, opaque, and non-authoritative.

This specification defines the **normative requirements** for an Actor implementation: its authority boundary, identity and key model, internal components, state lifecycle, merge semantics, synchronization behavior, revocation enforcement, and failure tolerance. The Actor is the sole locus of truth. Infrastructure merely moves bytes.

---

## Status of This Document

This document is a **Draft Specification** for the **Actor Concept**, version **1.0.0**, edited on **16 January 2026**.  
It is provided for review and experimentation and **must not be considered stable or complete**.  
Normative requirements and component boundaries may change without notice.

---

## Conformance

This specification defines requirements for **Actor Implementations**.

An implementation **conforms** if and only if it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

---

## Actor Definition *(Normative)*

An **Actor** is a software agent executing in a user-controlled environment that is **authoritative over state**.

An Actor **MUST**:

- create and mutate state locally,
- validate and authorize all operations it accepts,
- attribute operations to cryptographic identities,
- merge state deterministically,
- enforce revocation and mandatory erasure,
- encrypt, decrypt, and authenticate all persisted or transmitted state,
- operate correctly without network connectivity.

An Actor **MUST NOT** delegate semantic interpretation, authorization, attribution, merge logic, or revocation enforcement to any external system.

---

## Authority Boundary *(Normative)*

The Actor defines a strict authority boundary.

### Inside the Boundary (Trusted)

- state interpretation and validation  
- cryptographic verification  
- role and capability enforcement  
- causal ordering and merge semantics  
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

An Actor **MUST** treat all externally supplied data as hostile until fully verified.

---

## Actor Identity *(Normative)*

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

## Internal Component Model *(Normative)*

An Actor implementation **MUST** be decomposable into the following **distinguishable components**.  
Each component represents a stable responsibility boundary and **MUST** be specifiable independently.  
Separate specifications **SHALL** define normative requirements for each component listed below.

### Required Components

- **Identity & Key Manager**  
  Manages Actor Identity Keys, role keys, rotation, secure persistence, and in-use lifecycle boundaries.

- **Credential & Capability Manager (Discoverable Credentials)**  
  Resolves Resource access material, capabilities, and cryptographic possession proofs without semantic leakage.

- **Offline Storage Engine**  
  Persists encrypted Envelopes, operation logs, acknowledgments, and metadata locally; supports crash recovery and replay.

- **Resource Manager**  
  Tracks Resource identifiers, local replicas, authorization state, and lifecycle transitions (active, revoked, erased).

- **Operation Engine**  
  Constructs, validates, signs, verifies, and applies CRDT operations with full causal metadata.

- **Merge Engine**  
  Deterministically merges validated operations into authoritative state; enforces idempotence and convergence.

- **Revocation Monitor & Enforcer**  
  Detects role revocation and performs mandatory erasure and access shutdown without user intervention.

- **Acknowledgment Engine**  
  Emits, verifies, and tracks acknowledgments bound to causal state and Actor identity.

- **Garbage Collector / Compactor**  
  Performs local history compaction subject to acknowledgment and merge-safety constraints.

- **Base Station Client**  
  Handles authenticated interaction with non-authoritative infrastructure for opaque relay and persistence.

- **Peer & Local Broadcast Channel**  
  Enables offline and local-network synchronization (e.g. BroadcastChannel, device-local transports).

- **Synchronization Controller**  
  Orchestrates snapshot and delta exchange, retry, resynchronization, and failure recovery.

- **Cryptographic Envelope Engine**  
  Encrypts and decrypts Resource Snapshots; enforces confidentiality, integrity, and non-observability.

- **Policy & Validation Layer**  
  Applies application-defined invariants, authorization rules, and structural checks prior to merge.

Additional components **MAY** exist but **MUST NOT** weaken Actor authority, offline-first operation, revocation guarantees, or deterministic merge semantics.

---

## State Ownership *(Normative)*

An Actor **MUST** maintain its own local replica of all state it participates in.

Local state is **authoritative for that Actor**.

Remote state:

- **MUST NOT** override local validation rules,  
- **MUST NOT** be trusted for attribution or authorization,  
- **MAY** be used as merge input only after verification.

Actors **MUST** assume remote data may be delayed, duplicated, reordered, incomplete, or malicious.

---

## Operation Production *(Normative)*

An Actor **MAY** produce an operation only if:

- the Actor holds valid authorization at the causal point,  
- the operation is valid under local state rules,  
- causal dependencies are satisfied.

Every operation **MUST** be:

- authorized via role or capability keys,  
- attributable via the Actor Identity Key,  
- causally referenced,  
- replay-safe and idempotent.

---

## Operation Acceptance *(Normative)*

An Actor **MUST** validate every incoming operation prior to merge.

Validation **MUST** include:

- authorization at the causal point,  
- identity attribution verification,  
- structural and semantic validity,  
- CRDT correctness,  
- revocation checks.

Invalid operations **MUST** be ignored without side effects.

---

## Deterministic Merge *(Normative)*

An Actor **MUST** merge state deterministically.

Given identical sets of valid operations, all conforming Actors **MUST** converge to identical authoritative state, independent of:

- arrival order,  
- duplication,  
- transport medium,  
- timing.

Merge logic **MUST** be associative, commutative, and idempotent.

---

## Offline-First Operation *(Normative)*

An Actor **MUST** function correctly without network connectivity.

This includes:

- reading state,  
- producing operations,  
- validating history,  
- enforcing authorization,  
- detecting revocation,  
- making garbage collection decisions.

Network availability **MAY** improve convergence speed but **MUST NOT** be required for correctness.

---

## Synchronization Behavior *(Normative)*

Actors synchronize opportunistically.

An Actor **MAY** publish, request, relay, or receive encrypted state via any transport.

An Actor **MUST NOT** assume:

- reliable delivery,  
- ordered delivery,  
- unique delivery,  
- honest intermediaries.

All synchronization failures **MUST** be recoverable via replay or resynchronization.

---

## Revocation Enforcement *(Normative)*

An Actor **MUST** continuously evaluate its own authorization state.

Upon detecting revocation of its role or capability for a Resource, the Actor **MUST**, without user intervention:

- immediately cease all interaction with the Resource,  
- irreversibly erase decrypted state and derived secrets,  
- erase all keys and metadata enabling access,  
- refuse further operation or acknowledgment emission.

Failure to enforce revocation **MUST** be considered non-conformant.

---

## Acknowledgment Semantics *(Normative)*

An Actor **MAY** emit acknowledgments after successful local verification and merge.

An acknowledgment:

- **MUST** be non-mutating,  
- **MUST** be signed by the Actor Identity Key,  
- **MUST** reference specific causal state.

Actors **MUST NOT** acknowledge unverified or unmerged state.

---

## Garbage Collection *(Normative)*

An Actor **MAY** compact or garbage-collect local history only when:

- application-defined acknowledgment conditions are met,  
- future deterministic merge remains possible,  
- attribution and revocation guarantees are preserved.

Garbage collection **MUST** be local, unilateral, and never infrastructure-coordinated.

---

## Failure Model *(Normative)*

An Actor **MUST** tolerate:

- crashes and restarts,  
- partial or stale state,  
- duplicated inputs,  
- malicious peers,  
- hostile infrastructure.

Failures **MUST NOT** result in silent corruption, authority escalation, or unverifiable state.

---

## Security Posture *(Non-Normative)*

The Actor model assumes untrusted infrastructure and imperfect peers. Security arises from explicit cryptographic verification, deterministic behavior, and strict authority boundaries—not from trusted environments or central coordination.

---

## Closing Principle *(Non-Normative)*

An Actor is not a client.  
An Actor is not a node.  
An Actor is a sovereign executor of truth, bounded only by the keys it holds and the rules it enforces.
