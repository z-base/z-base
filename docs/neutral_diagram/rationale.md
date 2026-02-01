# Rationale: Why these components exist in a Digital-Sovereignty-Enabling Architecture

This component set exists because **sovereignty is not a policy property**. It is a **structural property**: what an interface _can_ do, what an operator _can_ observe, what infrastructure _can_ retain, and what authority _can_ be exercised must be constrained by construction.

Conventional web architectures violate sovereignty by default because they routinely collapse:

- **execution context into consent**
- **availability infrastructure into data custody**
- **service operators into semantic authorities**
- **interoperability into durable correlation**

This system prevents those collapses by enforcing boundaries, capability-scoped authority, and meaning locality.

---

## The 10 principles as structural requirements

Below, each component is justified against these principles (not as aspirations, but as constraints):

1. **Existence** — the user’s identity cannot be fully digital; systems must treat the user as the root reality, not the service.
2. **Control** — the user is the final authority over their identifiers and the use of their cryptographic capabilities.
3. **Access** — the user can retrieve their own data without gatekeepers.
4. **Transparency** — mechanisms must be inspectable and independent of institutional trust.
5. **Persistence** — identity/state must remain usable long-term, even across operator failure.
6. **Portability** — identity/state must be movable; not trapped behind a provider.
7. **Interoperability** — identity/state must function across services without losing user control.
8. **Consent** — disclosure/use must be deliberate and scoped (not implied).
9. **Minimalization** — disclose the minimum necessary; avoid unnecessary linkage/correlation.
10. **Protection** — system design must privilege user rights over network/operator convenience; resist coercion and capture.

---

## The core invariants this architecture enforces

All components exist to enforce three non-negotiables:

1. **Meaning locality:** meaning-bearing state must not exist where the user cannot technically prevent capture.
2. **Authority locality:** authority must emerge only from **user-held** cryptographic capabilities and signed intent.
3. **Useful-but-untrusted infrastructure:** infrastructure must provide availability and transport **without becoming custody or authority**.

Everything below is a boundary or lifecycle needed to keep those invariants true.

---

## Component rationales (each prevents a specific collapse)

### Unspecified Intent

**Why it exists:** Execution context is not consent. A UI load or code execution must never imply authorization.  
**Principles enforced:** Consent, Control, Protection, Minimalization.  
**What collapses without it:** Any service could treat “user opened the app” as implicit permission to observe, export, or correlate state. That reintroduces coercive defaults and dark-pattern authorization.

---

### Uncontrolled Exposure

**Why it exists:** You cannot prevent what you refuse to model. This names the unacceptable condition: provider-controlled code can observe/transmit/retain without technical restriction.  
**Principles enforced:** Protection, Consent, Minimalization, Control, Transparency.  
**What collapses without it:** Sovereignty becomes “trust us” policy. The architecture loses a testable boundary between safe and unsafe states.

---

### Service Provider

**Why it exists:** Operators inevitably provide compute and distribution. The system must explicitly deny them structural authority over identity and state.  
**Principles enforced:** Control, Portability, Protection, Transparency.  
**What collapses without it:** Operational convenience becomes custodianship: the provider becomes the de facto identity/state authority because nothing prevents it.

---

### Service Boundary

**Why it exists:** Sovereignty requires a _maximum_ scope of trust. This boundary formalizes: inside = untrusted-by-default execution; crossing = cryptographically verified, explicitly authorized.  
**Principles enforced:** Consent, Minimalization, Protection, Transparency, Control.  
**What collapses without it:** Trust becomes ambient. Any shared runtime, shared storage context, or shared auth session becomes a correlation and exfiltration surface.

---

### Actor Software

**Why it exists:** Interpretation, merge logic, attribution, and state transitions must happen without granting semantic authority to infrastructure. Actor Software is the semantic engine, but confined to boundary rules.  
**Principles enforced:** Transparency, Interoperability, Control, Minimalization.  
**What collapses without it:** Either:

- infrastructure must interpret state (semantic authority leak), or
- services become the only place logic can live (custody/lock-in), or
- collaboration breaks (no consistent merge/attribution rules).

---

### Offline Storage

**Why it exists:** Access and persistence cannot depend on provider uptime. Offline encrypted state ensures the user can retain continuity even when the network or provider disappears.  
**Principles enforced:** Access, Persistence, Portability, Protection.  
**What collapses without it:** The user’s ability to exist digitally becomes conditional on service continuity—turning “sovereign identity” into “subscription identity”.

---

### Base Stations

**Why it exists:** Collaboration and multi-device sync need availability infrastructure—but that infrastructure must remain **opaque** and **non-semantic**. Base Stations provide authenticated relay and opaque persistence without state authority.  
**Principles enforced:** Interoperability, Persistence, Minimalization, Protection, Transparency.  
**What collapses without it:** Sync requires either:

- a semantic server (infrastructure becomes authority), or
- direct peer-to-peer availability dependence (fragile persistence), or
- provider custody (lock-in).

---

### Device Capability Pairing

**Why it exists:** Establishes a relationship between a boundary and a user-controlled capability without granting authority. It separates “this boundary may later request capability use” from “capability use is now authorized”.  
**Principles enforced:** Consent, Control, Protection.  
**What collapses without it:** Either capabilities must be global/ambient (unsafe correlation), or every authorization becomes a full re-bootstrap (usability collapse that pushes users back to custodial accounts).

---

### Device Capability Discovery

**Why it exists:** Makes a previously established capability available only after explicit user verification, without transferring secret custody.  
**Principles enforced:** Consent, Control, Minimalization, Protection.  
**What collapses without it:** Capabilities become either:

- silently usable by provider code (consent collapse), or
- exportable into service custody (control collapse).

---

### PDS Credential Enrollment

**Why it exists:** Creates or re-binds the root credential material as **opaque persisted state**, encrypted under a valid device capability, so the PDS can be re-materialized without provider custody.  
**Principles enforced:** Control, Persistence, Portability, Access, Protection.  
**What collapses without it:** The root of state depends on service-managed identity accounts or provider-side key custody.

---

### PDS Credential Discovery

**Why it exists:** A PDS must not be “always there”. It must be **materialized only when a valid capability is used** to retrieve and decrypt the credential that gates it.  
**Principles enforced:** Consent, Minimalization, Protection, Access.  
**What collapses without it:** The PDS becomes implicitly accessible to the boundary or provider logic, enabling silent traversal and correlation.

---

### PDS Credential Recovery

**Why it exists:** Total device loss must not equal total identity loss, but recovery must not reintroduce custodianship. Recovery produces material that still must be bound to a valid capability before use.  
**Principles enforced:** Persistence, Protection, Control, Portability.  
**What collapses without it:** Users either lose identities permanently (persistence failure) or accept provider custodianship “for convenience” (control failure).

---

### Device Capability Re-establishment

**Why it exists:** After recovery, authority must be re-anchored in a new user-held capability. This prevents recovered material from becoming a provider-managed credential.  
**Principles enforced:** Control, Protection, Persistence.  
**What collapses without it:** Recovery becomes a custodial reset path; the provider becomes the practical identity authority.

---

### Private Data Space (PDS)

**Why it exists:** Defines the only domain where meaning-bearing state is allowed to exist: it materializes only under valid credentials and is interpreted under user authority. Outside it, state remains opaque.  
**Principles enforced:** Existence, Control, Access, Minimalization, Protection, Portability.  
**What collapses without it:** Services/infrastructure become semantic authorities because meaning must exist somewhere—and if not in a user-authorized domain, it ends up in provider systems.

---

### Identity Registry

**Why it exists:** Interoperability needs public key discovery, but registries must not become identity authorities. This is lookup-only.  
**Principles enforced:** Interoperability, Transparency, Portability, Protection.  
**What collapses without it:** Either:

- services cannot verify identities across boundaries (interop failure), or
- registries become update-authorities (control failure).

---

### Identity Control Capability

**Why it exists:** Identity authority must be confined to the user-controlled domain (the PDS). Only this capability can sign identity actions and manage published identifiers.  
**Principles enforced:** Control, Consent, Protection, Persistence.  
**What collapses without it:** Identity updates drift to providers/registries, creating de facto centralized identity control.

---

### Resource Control Capability

**Why it exists:** Authority must be partitioned. Resource-bound capabilities scope what can be done to a specific resource without granting traversal of the whole PDS.  
**Principles enforced:** Minimalization, Consent, Control, Protection, Interoperability.  
**What collapses without it:** Authorization becomes all-or-nothing: either broad access (privacy collapse) or unusable micro-flows (usability collapse that pushes toward custodial models).

---

### Services

**Why it exists:** Automation is useful, but automation must not imply custody. Services are narrowly scoped workflows that operate only on explicitly submitted authorized data, with custody limited to the workflow.  
**Principles enforced:** Consent, Minimalization, Protection, Transparency, Control.  
**What collapses without it:** “Convenient automation” turns into durable data retention, PDS traversal, and provider correlation.

---

### Authorized Exchange

**Why it exists:** Interoperability across boundaries must not require shared sessions, reusable tokens, or durable correlation. Authorized Exchange provides explicit, scoped, verifiable cross-boundary transfer without secret sharing or custody transfer.  
**Principles enforced:** Interoperability, Consent, Minimalization, Protection, Portability, Transparency.  
**What collapses without it:** Cross-service use devolves into OAuth-style token delegation and shared identities that create long-lived linkage and expand provider authority.

---

## Summary: what this component set prevents

This system exists because sovereignty collapses in predictable ways unless prevented:

- **Consent collapse:** execution context treated as authorization
- **Custody collapse:** availability infrastructure becomes state owner
- **Semantic collapse:** servers/services become interpreters of meaning
- **Correlation collapse:** interoperability relies on durable tokens/sessions
- **Recovery collapse:** loss of devices forces custodial account resets

Each component is a structural countermeasure to one of these collapses, justified directly against the ten principles.

Sovereignty is not achieved by promise.

It is achieved by construction.
