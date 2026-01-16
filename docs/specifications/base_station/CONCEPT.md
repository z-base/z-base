# Base Station — Concept Specification

## Abstract

A **Base Station** is a non-authoritative infrastructure service that persists, routes, and relays opaque state data between Actors without interpreting, validating, or enforcing semantic meaning. A Base Station **MUST treat all state as opaque bytes** and never participate in authorization, attribution, causal ordering, merge logic, or conflict resolution. It exists to provide *transport, persistence, and opportunistic delivery* across online and offline environments while preserving confidentiality, integrity, and relay fidelity.

Base Stations are protocol entities: they define how bytes move, not what they mean. All semantics reside in Actors. This specification defines the normative requirements for Base Station implementations that interoperate with Actor systems such as z-base.

---

## Status of This Document

This document is a **Draft Specification** for the **Base Station Concept**, version **1.0.0**, edited on **16 January 2026**.  
It is provided for review and experimentation and **must not be considered stable or complete**.

---

## Conformance

An implementation **conforms** if and only if it satisfies all **normative** requirements defined herein.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted per RFC 2119 and RFC 8174.

Sections marked *Non-Normative* are informational and **do not affect conformance**.

---

## Base Station Definition *(Normative)*

A **Base Station** is a service that:

- accepts and stores opaque binary data,
- relays that data between verified clients,
- enables opportunity delivery (store-and-forward),
- supports online (persistent network) and offline (local broadcast) modes,
- provides authenticated session boundaries but has no access to state semantics.

A Base Station **MUST** treat payloads as **opaque bytes** and **MUST NOT** decode, inspect, interpret, validate, or mutate them. It is strictly untrusted with respect to application meaning.

This constraint **MUST** be reflected in all components and behaviors.

---

## Authority Boundary *(Normative)*

### Inside Boundary (Base Station Authority)

The Base Station is authoritative only for:

- authenticated session state and routing,
- persistence of stored opaque bytes,
- delivery guarantees (best-effort store-and-forward, retry semantics),
- transport framing and session handshake.

### Outside Boundary (Untrusted Semantics)

The Base Station **MUST NOT** interpret:

- semantic structure of payload bytes,
- identity or role information within payloads,
- merge rules for state,
- causality, operation history, or validation effects.

All semantic authority resides in Actors.

---

## Security Guarantee *(Normative)*

A Base Station **MUST**:

- enforce authentication before accepting relay payloads,
- require proof of possession of cryptographic material for a channel,
- ensure transport encryption for all stateful interactions,
- never retain plaintext beyond authenticated session context.

It **MUST NOT** derive semantic meaning from payloads.  
It **MUST NOT** enforce or interpret authorization semantics.

---

## Base Station Sessions *(Normative)*

### Handshake

A Base Station **MUST** establish an authenticated session with a peer via a cryptographic handshake (e.g., HMAC challenge–response) before accepting relay traffic. Unauthenticated messages **MUST** be rejected.

### Relay Domain

The Base Station **MUST** partition traffic by Resource or namespace so that routing, storage, and forwarding occur only between verified peers for that domain.

---

## Opaque Payload Handling *(Normative)*

All stateful payloads **MUST** be treated as encrypted, opaque blobs.

- The Base Station **MUST** store and forward these blobs **without decoding**.  
- The Base Station **MUST** preserve all bits and ordering of the binary payload.  
- The Base Station **MUST** be able to route based on minimal framing metadata (e.g., a one-byte code) but **MUST NOT** require decoding. :contentReference[oaicite:1]{index=1}

---

## Transport & Relay Semantics *(Normative)*

A Base Station **MUST**:

- relay encrypted payloads between verified clients in both **online** and **offline** modes,
- support **fan-out** (forward to all relevant peers except origin),
- isolate channels per verified Resource or namespace,
- allow optionally storing replicas for opportunistic delivery.

Base Stations **MUST NOT** participate in conflict resolution, causal ordering, or merge semantics.

---

## Persistence & Store-and-Forward *(Normative)*

Persistence in a Base Station **MUST** be:

- opaque, unparsed binary,
- stored under authenticated session context,
- retrievable by clients upon verified request,
- subject to capacity and retention policies defined by the station.

Base Stations **MAY** delete or evict stored payloads per local retention policies but **MUST NOT** interpret contents.

---

## Offline & Local Relay *(Normative)*

A Base Station **MUST** provide relay semantics for offline contexts via local transports (e.g., BroadcastChannel) using identical wire formats. Offline relay **MUST NOT** require the handshake step but **MUST** preserve encryption invariants.

---

## Component Model *(Normative)*

A conforming Base Station **MUST** be decomposable into the following components:

- **Session & Authentication Manager**  
  Establishes and tracks verified peer sessions.

- **Opaque Storage Layer**  
  Stores binary state without decoding; supports persistence and eviction.

- **Transport Relay Engine**  
  Routes and forwards opaque bytes between peers; supports both online and local offline transports.

- **Resource Namespace Manager**  
  Segregates relay domains by Resource or logical namespace.

- **Framing & Routing Logic**  
  Routes based on minimal framing metadata without interpreting payloads.

- **Security & TLS Layer**  
  Enforces encryption, authentication, and transport integrity.

Additional components **MAY** exist but **MUST NOT** derive semantic meaning.

---

## Failure & Delivery Guarantees *(Normative)*

A Base Station **MUST**:

- tolerate partial network failures,
- retry best-effort delivery,
- not buffer content in a way that induces semantic interpretation,
- provide last-write-lossy or store-and-forward semantics as configured.

It **MUST** not make semantic guarantees about convergence or ordering.

---

## Non-Goals *(Normative)*

A Base Station **MUST NOT**:

- decode encrypted payloads,
- validate or enforce application semantics,
- merge or apply CRDT logic,
- act as an authority for state decisions.

All such authority resides in Actors.

---

## Security Posture *(Non-Normative)*

The Base Station is designed to minimize trust in infrastructure: state is opaque, semantics are local, and transport security is orthogonal to application logic.

---

## Closing Principle *(Non-Normative)*

A Base Station moves bytes, not meaning.  
It enables delivery, persistence, and opportunistic relay—nothing more.

---
