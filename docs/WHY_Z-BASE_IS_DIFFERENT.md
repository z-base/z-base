# Why z-base Is Different

Most decentralized systems try to decentralize infrastructure.

**z-base decentralizes authority.**

Infrastructure — cloud providers, blockchains, peer networks, relays, CDNs — is treated as a swappable transport and persistence layer. It may scale, autoscale, federate, or disappear entirely. z-base does not depend on which one you choose.

The first implementation may run on Cloudflare.  
Another could run on a blockchain.  
Another on a peer network.  

It does not matter.

Because in z-base:

- Application state is treated like media.
- Actors are cryptographic authorities over that state.
- Capabilities — not servers — define who can view, modify, relay, or persist it.
- Infrastructure only moves and stores encrypted state.
- Truth is derived from signatures and capability chains, not from database custody.

This makes decentralization structural rather than topological.

The system does not care where the bytes live.  
It only cares who holds the keys.

That is why z-base is infrastructure-agnostic and deployable at scale:

- Cloud-compatible.
- Blockchain-compatible.
- Peer-network-compatible.
- Replaceable without migrating authority.

z-base does not decentralize servers.  
It decentralizes control.

---

## Application State as Media

The “application state as media” framing clarifies the model.

Media has:

- A format.
- Viewers.
- Editors.
- Directors.
- Owners.
- Rights.

If application state becomes signed, capability-governed media:

- Anyone can host it.
- Anyone can relay it.
- Only capability holders can interpret or mutate it meaningfully.

This model avoids consensus overhead and treats infrastructure as transport, not authority.

---

## The Critical Requirements

For this architecture to hold, the following must be precisely defined:

- Deterministic ordering rules.
- Conflict resolution semantics.
- Capability revocation.
- Replay and fork handling.

If these are crisp and verifiable, infrastructure truly becomes irrelevant to authority.

If they are ambiguous, infrastructure quietly reasserts itself as the source of truth.

Real decentralization is not about topology.  
It is about where epistemic authority resides.
