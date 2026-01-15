
## Abstract

z-base specifies a zero-knowledge state relay and persistence API for browser-based JavaScript actors. It defines how encrypted, opaque state blobs are stored, relayed, and synchronized by a non-authoritative service, while all state validity, conflict resolution, and cryptographic verification are performed by actors. The specification guarantees offline-first operation, eventual consistency, and unlinkability of stored data, such that the service cannot interpret, correlate, or decrypt application state. This document defines the normative requirements for actors and base stations to interoperate within this model.

## Status of This Document

This document is a **Draft Specification** for **z-base**, version **1.0.0**, edited on **15 January 2026**.
It is provided for review and experimentation and **must not be considered stable or complete**.
The contents may change at any time without notice, including normative requirements and conformance criteria.

This specification is **AI-assisted** and authored with contributions from **Jori Lehtinen** and **GPT-5.2**.
Feedback and discussion are encouraged, but implementations should expect breaking changes until this document reaches a finalized status.

## Conformance

This specification defines conformance requirements for the following roles:

**Actor Implementation**
Software executing in a user-controlled environment that creates, verifies, merges, encrypts, decrypts, and authoritatively interprets application state.

**Base Station Implementation**
A non-authoritative service that stores, relays, and forwards opaque data according to this specification.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in RFC 2119 and RFC 8174.

An implementation **conforms** to this specification if and only if it satisfies all normative requirements applicable to its declared role.
An implementation may conform to one or more roles independently.

Sections explicitly marked as *non-normative*, as well as examples and notes, are provided for informational purposes only and **do not affect conformance**.

## Terminology

**Actor**
A user-controlled software agent that is authoritative over state creation, validation, conflict resolution, encryption, and decryption.

**Base Station**
A non-authoritative service that stores, forwards, and relays opaque binary data without the ability to interpret, validate, correlate, or decrypt its contents.

**Resource**
An addressable unit of state identified by a high-entropy identifier, whose contents are always transported and stored as opaque data.

**Opaque Blob**
A binary payload with no semantic meaning to the Base Station and no stable structure visible outside the Actor.

**Envelope**
An opaque binary container that encapsulates an encrypted payload together with minimal protocol metadata required for routing or delivery.

**Authoritative State**
State whose validity, ordering, and merge semantics are determined exclusively by one or more Actors, never by a Base Station.

**Offline-First Operation**
A mode of operation in which Actors are able to read and mutate state without network connectivity, with synchronization occurring opportunistically when connectivity is available.

**Eventual Consistency**
A property whereby independent Actor replicas, given sufficient time and communication, converge to equivalent authoritative state.
