# Interoperable Message Handling (IMH)

## Status

This document specifies **Interoperable Message Handling (IMH)** as a single logical object implemented in any language/runtime.

IMH consumes and produces **Message Frames** (byte sequences) and emits events that application code can register handlers for using an EventTarget-style model (or a conforming equivalent event sink).

This specification defines:

- the Message Frame envelope  
- the IMH object surface  
- type → code mapping rules (hardcoded per profile)  
- event emission rules for valid and invalid frames  

This specification **does not define** transports, networking, routing, identity, authorization, or payload semantics.

RFC 2119 / RFC 8174 keywords apply.

---

## Terms

- **Message Frame**: the atomic byte unit consumed and produced by IMH.  
- **type**: a string identifier for an operation/event (API level).  
- **code**: an 8-bit numeric identifier derived from `type` (wire level).  
- **payload**: opaque bytes carried by a Message Frame. Payload is **never empty**.

---

## Message Frame

### Envelope (normative)

A Message Frame MUST use the envelope:

[ 1 byte VERSION ][ 1 byte CODE ][ N bytes PAYLOAD ]


Where:

- `VERSION` is an unsigned 8-bit integer.  
- `CODE` is an unsigned 8-bit integer.  
- `PAYLOAD` is an opaque byte sequence of length **N >= 1**.

A frame with zero-length payload is **invalid**.

---

### Parsing rules (normative)

Given `frame` as a byte sequence:

- A Message Frame MUST be at least **3 bytes**.  
- `VERSION = frame[0]`  
- `CODE = frame[1]`  
- `PAYLOAD = frame[2..]` (length ≥ 1)

---

## IMH Object

### Required surface (normative)

An IMH implementation MUST expose a single object `imh` with:

- `imh.version: number`  
  The single supported `VERSION` value.

- `imh.typeMap: Map<string, number>`  
  Hardcoded mapping from `type` strings to numeric `CODE`s (0–255).  
  This map is immutable for the lifetime of the object.

- `imh.consumeMessage(frame: Uint8Array): void`  
  Consumes a Message Frame and emits events.

- `imh.produceMessage(type: string, payload: Uint8Array): Uint8Array`  
  Produces a Message Frame.

- `imh.addEventListener(eventType: string, listener: Function, options?): void`  
- `imh.removeEventListener(eventType: string, listener: Function, options?): void`

The event subscription model MUST be compatible with EventTarget semantics or provide a conforming equivalent.

---

### Hardcoded mapping rules (normative)

- `typeMap` MUST define a total mapping for all supported `type` values.  
- The inverse mapping (`CODE → type`) MUST be derivable and used for decoding.  
- Duplicate `CODE` values MUST NOT exist.

---

## consumeMessage(frame)

### Input validation (normative)

`consumeMessage` MUST:

1. Reject non-byte input.  
2. Reject frames where `frame.length < 3`.

On rejection, it MUST emit `imh:invalid-frame`.

---

### Version validation (normative)

If `VERSION !== imh.version`, `consumeMessage` MUST emit  
`imh:invalid-version` and MUST reject the frame.

---

### Code / type resolution (normative)

If `CODE` does not resolve to a `type` via the inverse of `typeMap`,  
`consumeMessage` MUST emit `imh:unknown-type` and MUST reject the frame.

---

### Event emission (normative)

For a valid frame, IMH MUST emit, in order:

1) `imh:message`  
2) `<type>`

Both events MUST include identical `detail`:

- `detail.version: number`  
- `detail.code: number`  
- `detail.type: string`  
- `detail.payload: Uint8Array`

IMH MUST NOT interpret payload semantics.

---

## produceMessage(type, payload)

### Validation (normative)

`produceMessage` MUST reject if:

- `type` is not a key in `typeMap`  
- `payload` is not a byte sequence  
- `payload.length === 0`

On rejection, the implementation MUST throw or MUST emit  
`imh:produce-error`. A conforming profile MUST choose exactly one.

---

### Encoding (normative)

On success:

- `CODE = typeMap[type]`  
- The returned frame MUST be:

[ imh.version ][ CODE ][ payload... ]


Payload bytes MUST be copied or referenced without modification.

---

## Required events

IMH MUST support:

- `imh:message`  
- `imh:invalid-frame`  
- `imh:invalid-version`  
- `imh:unknown-type`

And dynamic event types equal to every `type` key in `typeMap`.

If emit-on-error is chosen for production, it MUST also support:

- `imh:produce-error`

---

## Conformance

An implementation conforms iff it:

- implements the IMH object surface  
- enforces non-empty payload frames  
- validates version strictly  
- resolves `CODE` via the inverse of `typeMap`  
- emits required events with required detail  
- produces frames matching the envelope exactly
