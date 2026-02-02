# Interoperable Message Handling (IMH)

## Status

This document specifies **Interoperable Message Handling (IMH)** as a single logical object implemented in any language/runtime.

IMH consumes and produces **Message Frames** (byte sequences) and emits events that application code can register handlers for using an EventTarget-style `addEventListener` model.

This specification defines:
- the Message Frame envelope
- the IMH object surface (properties and methods)
- type→code mapping rules (hardcoded within the object)
- event emission rules for valid and invalid frames

This specification does not define transports, networking, routing policy, authorization, identity, or payload semantics.

## Terms

- **Message Frame**: the atomic byte unit consumed/produced by IMH.
- **type**: a string identifier for an operation/event (API-level).
- **code**: a numeric identifier derived from `type` using `typeMap` (wire-level).
- **payload**: opaque bytes carried by a Message Frame.

RFC 2119 / RFC 8174 keywords apply.

---

## Message Frame

### Envelope (normative)

A Message Frame MUST use the envelope:

```

[ 1 byte VERSION ][ 1 byte CODE ][ N bytes PAYLOAD ]

```

- `VERSION` is an unsigned 8-bit integer.
- `CODE` is an unsigned 8-bit integer.
- `PAYLOAD` is an opaque byte sequence of length `N >= 0`.

### Parsing rules (normative)

Given `frame` as a byte sequence:

- A Message Frame MUST be at least 2 bytes.
- `VERSION = frame[0]`
- `CODE = frame[1]`
- `PAYLOAD = frame[2..]` (>1)

---

## IMH Object

### Required surface (normative)

An IMH implementation MUST provide a single object `imh` with:

- `imh.version: number`  
  The single supported `VERSION` value (0–255).

- `imh.typeMap: Map<string, number>` (or equivalent)  
  Hardcoded mapping from `type` strings to `CODE` numbers (0–255).

- `imh.consumeMessage(frame: Uint8Array): void`  
  Consumes a Message Frame and emits events.

- `imh.produceMessage(type: string, payload?: Uint8Array): Uint8Array`  
  Produces a Message Frame for `type` and optional payload bytes.

- `imh.addEventListener(eventType: string, listener: Function, options?): void`
- `imh.removeEventListener(eventType: string, listener: Function, options?): void`

The event subscription model MUST be compatible with the EventTarget pattern.

### Hardcoded mapping rules (normative)

- `typeMap` MUST be present on the IMH object and treated as immutable for conformance.
- `typeMap` MUST define a total mapping for all supported `type` strings in this IMH profile.
- The inverse mapping (`CODE -> type`) MUST be derivable from `typeMap` and used for decoding.
- Duplicate CODE assignments MUST NOT exist.

---

## consumeMessage(frame)

### Input validation (normative)

`consumeMessage` MUST:

1. Reject non-byte input.
2. Reject frames where `frame.length < 2`.

On rejection, it MUST emit `imh:invalid-frame`.

### Version validation (normative)

If `VERSION != imh.version`, `consumeMessage` MUST emit `imh:invalid-version` and MUST reject the frame.

### Code/type resolution (normative)

If `CODE` does not map to any `type` (via inverse of `typeMap`), `consumeMessage` MUST emit `imh:unknown-type` and MUST reject the frame.

### Event emission (normative)

For a valid frame:

- IMH MUST emit a generic event `imh:message`.
- IMH MUST emit a second event whose `eventType` equals the resolved `type` string.

Both events MUST include the same `detail` payload:

- `detail.version: number` (the parsed VERSION)
- `detail.code: number` (the parsed CODE)
- `detail.type: string` (the resolved type)
- `detail.payload: Uint8Array` (the payload bytes)

Event order MUST be:
1) `imh:message`
2) `<type>`

IMH MUST NOT interpret payload semantics.

---

## produceMessage(type, payload)

### Validation (normative)

`produceMessage` MUST:

- Reject if `type` is not a key in `typeMap`.
- Reject if `payload` is provided and is not a byte sequence.

On rejection, the implementation MUST throw or MUST emit `imh:produce-error`. A conforming profile MUST choose exactly one of these behaviors and document it.

### Encoding (normative)

On success:

- `CODE = typeMap[type]`
- The returned frame MUST be:
  - `[imh.version][CODE][payload...]`
- Payload bytes MUST be copied or referenced without modification.

---

## Required events

IMH MUST support these event types:

- `imh:message` (emitted for every valid consumed frame)
- `imh:invalid-frame`
- `imh:invalid-version`
- `imh:unknown-type`

And MUST support dynamic event types equal to every `type` key in `typeMap`.

If the profile uses emit-on-error for `produceMessage`, it MUST also support:

- `imh:produce-error`

---

## Conformance

An implementation conforms to this specification iff it:

- implements the IMH object surface
- parses Message Frames exactly as specified
- enforces version equality against `imh.version`
- resolves `CODE` to `type` using the inverse of `typeMap`
- emits required events with required `detail`
- produces frames that follow the Message Frame envelope
```
