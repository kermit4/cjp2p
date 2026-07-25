# How LCDP Contrasts with the P2P / Local-First Landscape

> Status claims below were spot-checked against live sources on 2026-07-25 for: SSB/Manyverse, Cabal, Hypercore/DAT (Pear/Holepunch), Earthstar, Willow, P2Panda, NextGraph, Holochain, GNUnet, Veilid, Qaul, Briar, Pijul, and Nostr's Negentropy extension — sources are linked inline next to the claims that depend on them. The iroh and Nostr core comparisons reuse the existing repo docs ([how_lcdp_contrasts_to_iroh.md](how_lcdp_contrasts_to_iroh.md), [how_lcdp_contrasts_to_nostr.md](how_lcdp_contrasts_to_nostr.md), written 2026-07-22/19) and were not independently re-checked in this pass. Anything without a citation is background knowledge, not a freshly verified fact — treat it accordingly.

This is not a "why LCDP is better" doc. Most of the field below solves a problem LCDP explicitly does not try to solve: durable, multi-writer data replication. LCDP solves a narrower problem — what bytes go on the wire, forever, without version negotiation. Read this as a map, not a sales pitch.

## 0. What LCDP is, in one paragraph

LCDP (`draft-pearson-lcdp-04`) is a UTF-8 JSON array of single-key objects, sent over a datagram (UDP by convention, port 24254). `[{"ChatMessage":{"message":"hi"}}]`. Unknown keys and fields MUST be ignored. Field semantics MUST NOT change once shipped. There is no mandatory identity, no mandatory crypto, no connection state, no persistence model, and no version number — extension happens by adding new keys, never by bumping a version. Everything else — security, reliability, discovery, congestion control, storage, replication — is an optional message type layered on top, defined by whoever needs it. See [README.md](README.md) and [draft-pearson-lcdp-04.txt](draft-pearson-lcdp-04.txt).

That narrowness is the whole point of this document: almost everything below is solving problems LCDP deliberately leaves unsolved (multi-writer conflict resolution, durable storage, DHT discovery, NAT traversal). LCDP is closer to a floor these could stand on than a member of the same category.

## 1. Where LCDP sits in the stack

```
Layer 4  Applications        Nostr (social) · Qaul (mesh chat) · Briar (offline/Tor chat) · Cabal (group chat)
Layer 3  Data sync/storage   SSB · Hypercore/DAT · Earthstar · Willow · P2Panda · NextGraph · Holochain · Pijul
Layer 2  Transport/session   libp2p · iroh · GNUnet · Veilid
Layer 1  Wire framing        LCDP  --  no persistence, no session, no identity, no version, by default
Layer 0  Datagrams / bytes   UDP, WebSocket, TCP, Bluetooth, carrier pigeon, whatever moves bytes
```

This is a simplification — several projects straddle layers (Holochain bundles transport + storage; Cabal is really "an app + a data-sync engine" wearing one name; Nostr skips straight from WebSocket to signed application events with no separate session layer). But it's the right first mental model: most of this list answers "how do N devices agree on a growing pile of data," and LCDP answers "how do two processes exchange a message without ever needing to agree on a version."

## 2. The landscape at a glance

| Protocol | Category | Core data model | Identity mandatory? | Sync/consistency model | Extensibility model |
|---|---|---|---|---|---|
| **LCDP** | Wire framing | None (out of scope) | No (optional pubkey) | None (app-defined) | Ignore-unknown, additive only, no versions |
| libp2p | Transport/session toolkit | None prescribed | Yes (PeerID) | App-chosen (pubsub/DHT/Bitswap) | Multistream-select, versioned protocol IDs |
| iroh | Transport/session toolkit | Blobs (BLAKE3) + gossip + docs KV | Yes (EndpointId) | iroh-docs (eventual consistency) | ALPN-negotiated composable protocols |
| GNUnet | Full alternative net stack | R5N DHT records | Yes (own PKI + GNS) | DHT replication/routing | Coordinated versioned service suite |
| Veilid | Transport/session toolkit (privacy-first) | DHT records | Yes (Node ID) | DHT replication factor | Versioned framework releases |
| SSB | Social log replication | Per-identity append-only hash chain | Yes (feed = ed25519 pubkey) | Gossip replication (later: EBT) | Loose JSON `type` + coordinated spec revamps |
| Hypercore/DAT (Pear) | Log replication engine | Per-writer append-only Merkle log | Yes (per-log keypair) | Sparse Merkle-verified block sync | Coordinated breaking versions (8→10) |
| Cabal | Chat app, own "Cable" wire protocol | Content-hash addressed signed posts, per channel | Yes | Cable's own request/exchange handshake, small-scale | Small coordinated protocol spec, JS + Rust impls |
| Earthstar | Local-first doc sync | Path-addressed documents, LWW | Yes (author keypair) | Doc-level sync, moving to range reconciliation | Coordinated major-version rewrites |
| Willow | General-purpose data model + sync spec | (namespace, subspace, path, time)-keyed entries | Yes (Meadowcap capabilities) | Range-based set reconciliation | Formally specified, versioned, test vectors |
| P2Panda | Local-first structured-data protocol | Per-author operation logs (event sourcing) | Yes (author pubkey) | Log-height ordered catch-up sync | Explicit hashed schemas — new shape, new hash |
| NextGraph | Semantic-graph CRDT platform | RDF graph + CRDT deltas in Merkle-DAG repos | Yes (identity + device keys) | CRDT merge over signed commits | RDF/ontology versioning |
| Holochain | Agent-centric DHT app framework | Per-agent hash chain + sharded validating DHT | Yes (agent pubkey) | DHT sharding + gossip + app-defined validation | Isolated WASM rules per app (different DNA = different network) |
| Nostr | Social app protocol | Signed JSON events, `kind`-tagged, held by relays | Yes (secp256k1 pubkey + sig) | Client↔relay query/subscribe (+ Negentropy reconciliation) | NIP registry, numbered `kind`s |
| Qaul | Offline mesh messaging app | Routed message envelopes, no shared data model | Yes (ed25519) | Epidemic store-and-forward routing | Protobuf schema, coordinated releases |
| Briar | Offline/Tor messaging app | Pairwise synced encrypted message logs | Yes | Pairwise transport sync | Coordinated protocol versioning |
| Pijul | Distributed VCS (patch algebra) | Commutative patch graph | Optional (signing for provenance) | Patch DAG diff/exchange, guaranteed convergent merge | Coordinated patch-format versioning |

## 3. Transport/session frameworks — libp2p, iroh, GNUnet, Veilid

These answer "how do two devices get a live, secure, addressable channel." LCDP explicitly refuses to answer that question — it assumes you already have a way to send a datagram and starts from there.

| | LCDP | libp2p | iroh | GNUnet | Veilid |
|---|---|---|---|---|---|
| Unit of work | A datagram of tagged messages | A stream over a negotiated protocol | A QUIC connection to an EndpointId | A message routed through the R5N DHT / GNUnet services | A DHT record or private route |
| Identity | Optional | PeerID (mandatory) | ed25519 EndpointId (mandatory) | Own PKI + GNS (mandatory) | Node ID keypair (mandatory) |
| Crypto | Optional, per-message | TLS/Noise, mandatory per connection | QUIC + TLS 1.3, mandatory | EdDSA-based, onion routing optional | Mandatory, with anonymous "safety routes" |
| NAT traversal | None built in — fails visibly, "surface the policy" | AutoNAT + Circuit Relay v2 (default fallback) | Hole-punch via relay coordination, ~90% success rate | Pluggable transports incl. Bluetooth/WLAN, own onion routing | Built in, privacy-route-aware |
| Discovery | Optional gossip message (`PleaseSendPeers`) | Kademlia DHT, mDNS, rendezvous | pkarr/DNS + mDNS + tickets | R5N DHT | R5N-like DHT |
| Philosophy | "Be the IP of p2p" — minimal floor, build up | "Be the TCP/IP of p2p" — composable toolkit | Fast direct connectivity for NAT'd/mobile devices | Full alternative internet stack (DNS, VPN, routing, anonymity) | Privacy-first app framework, "cloud alternative" |

- Full deep-dives already exist for two of these: [how_lcdp_contrasts_to_iroh.md](how_lcdp_contrasts_to_iroh.md) and [lcdp_description_for_libp2p_users.md](lcdp_description_for_libp2p_users.md).
- **GNUnet** is the oldest and broadest thing on this whole list (development traces to the early 2000s). It isn't really "a protocol" so much as a from-scratch alternative internet: its own DHT, its own onion routing, its own DNS replacement (GNS, delegation-based rather than a global namespace), its own VPN and file-sharing layer. That breadth is also the criticism people make of it — it's heavyweight, has a steep deployment curve, and after two decades its own 0.27.0 release notes (March 2026) still flag open usability issues and "critical privacy issues, particularly for mobile users." [[gnunet.org, 0.27.0 release](https://lists.gnu.org/archive/html/info-gnu/2026-03/msg00007.html)] Worth noting for this doc specifically: that same 0.27.0 release **broke protocol compatibility with the 0.26.x line** — a real, current example of exactly the coordinated-version-break cost LCDP's "extend, never break" rule is designed to avoid entirely. LCDP's whole spec fits in a page; GNUnet's does not fit in a book.
- **Veilid** (Cult of the Dead Cow, publicly launched 2023; VeilidChat went to open beta at DEF CON 32 in 2024 and remains under active development) is the newest entrant in this cluster and the closest in *spirit* to "privacy by default, developer framework" as a goal — closer to iroh's ambitions than to LCDP's. Worth knowing about if you liked iroh but want anonymity properties baked in rather than layered on.

## 4. Local-first log & CRDT data-sync engines — SSB, Hypercore/DAT, Cabal, Earthstar, Willow, P2Panda, NextGraph

This is the largest cluster, and it's the one where "LCDP vs X" is the most apples-to-oranges comparison, because these all solve durable multi-device data replication — a problem LCDP has no opinion about. If you wanted to build one of these on top of LCDP, you'd define the sync logic as an optional message type; LCDP would just be the envelope.

| | LCDP | SSB | Hypercore/DAT | Cabal | Earthstar | Willow | P2Panda | NextGraph |
|---|---|---|---|---|---|---|---|---|
| Fundamental unit | A message | An append-only per-identity hash chain | An append-only per-writer Merkle log | A signed, content-hash-addressed post under a channel (Cable protocol) | A path-addressed document | An entry keyed by (namespace, subspace, path, time) | A per-author operation log | An RDF graph delta in a Merkle-DAG repo |
| Conflict resolution | N/A — no shared state | N/A per-feed (each identity is authoritative over its own chain) | N/A per-writer; Autobase adds multi-writer causal merge | N/A per-author, append-only per channel | Last-write-wins by default | Prefix-pruning rules over ranges | Materialized from ordered operations (event sourcing) | CRDT merge over the graph |
| Discovery/transport | Optional, app-defined | Secret-handshake over TCP, LAN broadcast, "pubs" as relays | Hyperswarm DHT + Noise-encrypted streams | Its own "Cable" wire protocol as of the current generation (see note below) — no longer Hypercore-based | Transport-agnostic (HTTP/WS/LAN), explicit "syncers" | Transport-agnostic, its own resumable session protocol (now called Confidential Sync, formerly WGPS) | `p2panda-net` is built on **iroh** — QUIC, gossip, confidential/discovery over iroh's transport | Broker/relay architecture, can run peer-to-peer or via relay |
| Sync efficiency | N/A | Historically full-feed replication (unbounded log growth was a known pain point); EBT improved this | Sparse — request only the Merkle-tree ranges you're missing | Content-hash addressed post exchange over Cable's own handshake + wire protocol | Doc-level, improving via Willow adoption | Range-based set reconciliation — a genuine strength, efficient partial sync | Log-height/sequence-range catch-up, "local-first sync for append-only logs" per its own docs | Merkle-DAG commit diffing, Git-like |
| Extensibility stance | Ignore-unknown, no versions, ever | Loose JSON `type` field; the community concluded incremental patching had hit its limits (see note below) | Coordinated breaking rewrites (Hypercore 8 → 10) | Coordinated spec, own JS + Rust implementations (`cable.js`, `cable.rs`) | Coordinated major rewrites — v11 (beta as of mid-2026) is the Willow-powered rewrite | Formally specified with test vectors — closer to an RFC than a convention, still revising (hardening the sync handshake against MITM in 2026) | Explicit hashed schemas: a shape change is a new hash, not a silently-ignored field | RDF/ontology versioning |
| Status, 2026-07-25 | — | Original architecture has an acknowledged structural flaw (clients must download gigabytes of history to participate) and is being succeeded by a new protocol, PZP/"ppppp", led by ex-SSB/Manyverse devs — see note below. [[Manyverse blog](https://www.manyver.se/blog/2024-07-03/)] | Actively developed, backed by Tether/Bitfinex via Holepunch; Keet Mobile launched an alpha in 2026 (text-only so far), 150%+ download growth reported Q1 2026. [[CIO](https://www.cio.com/article/4151267/keet-is-untethering-social-media-from-big-tech.html)] | Actively maintained, multiple clients (desktop/CLI/mobile-in-progress) under the `cabal-club` org. [[GitHub](https://github.com/cabal-club)] | v11 in beta, actively developed. [[earthstar-project.org](https://earthstar-project.org/docs/future)] | Active, informal updates moved to worm-blossom.org; FOSDEM 2026 talk given. [[willowprotocol.org](https://willowprotocol.org/more/changes/index.html)] | Active, NLnet/NGI-funded (grant window ran through April 2026), pre-1.0. [[NLnet](https://nlnet.nl/project/P2Panda/)] | **Alpha — explicitly "not stable, should not be used for productive work."** Core protocol not yet finalized; broker self-hosting planned later in 2026. [[nextgraph.org roadmap](https://nextgraph.org/roadmap/)] |

A few things worth knowing that don't fit a table cell:

- **SSB is the field's clearest cautionary tale for LCDP's core bet, and it's playing out right now.** SSB tried to evolve by patching an identity-centric, full-feed-replication design for years. As of 2026 the project's own long-time maintainers have concluded that isn't tenable anymore — the architecture has a structural flaw (a new client has to download gigabytes of history just to participate), and rather than patch it again, the response was a clean-slate successor protocol, **PZP** (working title "ppppp"/PPPPP), that Manyverse is migrating toward. [[Manyverse: Launch of the PZP protocol](https://www.manyver.se/blog/2024-07-03/)] This is exactly the failure mode LCDP's "extend, never version, never break" rule is trying to avoid — SSB extended for years and still hit a wall requiring a hard fork of the ecosystem. Whether LCDP's much narrower scope (a message envelope, not a whole social-replication architecture) lets it dodge the same fate long-term is an open question, not a settled one.
- **Cabal → Earthstar → Willow is a lineage of "what we learned went wrong," but Cabal's own story corrects what I first wrote here.** Cabal originally reused the Hypercore log stack, but the current generation (`cabal-club/cable`) replaced that with its own purpose-built, lightweight binary wire protocol — a handshake plus content-hash-addressed request/exchange of signed posts, with independent JS (`cable.js`) and Rust (`cable.rs`) implementations. [[cabal-club/cable](https://github.com/cabal-club/cable)] Earthstar generalized past "social feed" to "arbitrary small-community documents" and its v11 (in beta as of mid-2026) is the rewrite powered by Willow's data model. [[earthstar-project.org](https://earthstar-project.org/docs/future)] Willow itself is the current end state of that lineage: not an app, but a rigorously specified, transport-agnostic data model + range-reconciliation sync protocol meant to be a shared foundation multiple local-first apps can build on, still under active revision in 2026 (hardening its sync handshake against man-in-the-middle attacks) — the closest thing in this list to "a spec people are trying to make outlast any one implementation," which is also LCDP's stated goal, just for a much richer problem (structured multi-writer data vs. a bare message envelope).
- **Hypercore/DAT's rename is worth knowing if you find old links.** The original Dat protocol (~2013-2018) was deprecated in favor of the Hypercore protocol, now developed under the Holepunch company/Pear runtime umbrella. Worth knowing: Holepunch is backed by Tether/Bitfinex, and their flagship app, the Keet chat/video app, launched a mobile alpha in 2026 with reported triple-digit download growth. [[CIO, 2026](https://www.cio.com/article/4151267/keet-is-untethering-social-media-from-big-tech.html)] That's a very different funding model from most of this list — worth factoring in if provenance/funding-source matters to your evaluation. If you land on a "Dat protocol" doc, assume it's superseded.
- **P2Panda's transport layer is now built on iroh**, not raw libp2p as I'd first guessed — its `p2panda-net` stack layers confidential topic/node discovery, gossip, and local-first log sync on top of iroh's QUIC transport, alongside BLAKE3, Ed25519, CBOR, and UCAN. [[p2panda.org](https://p2panda.org/)] That makes P2Panda a good concrete example of layer 3 (data sync) being built cleanly on top of layer 2 (transport) rather than reinventing it — the same composition pattern section 9 below suggests for LCDP.
- **NextGraph is explicitly pre-production as of 2026** — its own docs state the current alpha "should not be used for any productive work or to store personal documents," and the core protocol isn't finalized yet. [[nextgraph.org roadmap](https://nextgraph.org/roadmap/)] Both P2Panda and NextGraph share LCDP's non-profit-tool ethos (grant-funded, small-team, not-a-company) but make the opposite extensibility bet: explicit, hashed, typed schemas rather than "ignore what you don't recognize." That's a real tradeoff, not a flaw — typed schemas give you compile-time-ish guarantees LCDP never will; LCDP's looseness gives you a 2040 node still talking to a 2026 node, which typed schema hashes don't guarantee unless someone maintains a migration path.

## 5. Agent-centric DHT framework — Holochain

Holochain doesn't fit cleanly into either the transport cluster or the log-sync cluster because it bundles both, with a genuinely different foundational idea: no global ledger, no global consensus. Each agent keeps their own append-only hash chain (a "source chain") of their own actions; entries are also published into a DHT sharded across peers, who validate them against application-defined rules (WASM "zomes" bundled into a "DNA"). There's no blockchain-style global ordering to agree on — bad actors get rejected/blacklisted locally by peers who catch invalid entries against the shared validation rules, not rolled back globally.

| | LCDP | Holochain |
|---|---|---|
| Trust model | None assumed; every message optionally signed | Peer validation against app-defined WASM rules; no global consensus |
| Identity | Optional | Mandatory agent pubkey, is your DHT address |
| Networking | Any datagram transport, unopinionated | Custom gossip layer ("Kitsune") for DHT storage/validation |
| Scope of a "network" | Whatever peers you're exchanging JSON with | Bound to one hApp's DNA hash — different app, different isolated network |
| Extensibility | Add a message key, old nodes ignore it | Define new validation logic in WASM; incompatible apps are simply different networks |
| Maturity, 2026-07-25 | Draft-04, informational IETF submission | Multi-year project (since ~2017); actively shipping in 2026 — Holochain 0.6.2 released July 1, 2026, and the Moss app-launcher hit 0.15.7 in June 2026 — but still explicitly pre-mainnet, working toward that launch. [[Holochain roadmap](https://www.holochain.org/roadmap/)] |

The interesting contrast: Holochain's "no global consensus, agents are sovereign over their own chain" instinct rhymes with LCDP's "no version negotiation, peers are sovereign over what they implement" instinct. Both reject a global coordination point on principle. Holochain spends that philosophy on a full DHT+validation runtime; LCDP spends it on staying small enough to implement in an afternoon in Bash.

## 6. Application-layer protocols — Nostr, Qaul, Briar

These operate one level above transport, with an opinion about what the message is *for* (social posts, offline chat), unlike LCDP which has no opinion about payload semantics at all.

- **Nostr** — already has a full write-up: [how_lcdp_contrasts_to_nostr.md](how_lcdp_contrasts_to_nostr.md). Short version: Nostr mandates a signed JSON event schema and relay-based client/server delivery; LCDP mandates nothing but the envelope and is relay-agnostic (peer-to-peer by default). You could carry Nostr events as an LCDP message type; you can't carry LCDP inside Nostr's event schema without just reimplementing LCDP as `content`. Worth noting for this doc specifically: Nostr's relay reconciliation extension, **NIP-77 "Negentropy"**, does range-based set reconciliation between clients/relays or relay/relay — the same core idea Willow uses for efficient partial sync, arrived at independently in a very different part of this landscape. It's not experimental in name only anymore: implementations like the `strfry` relay use it routinely on datasets in the tens of millions of events, with reported bandwidth reductions of 10-100x over naive full transfer. [[NIP-77](https://github.com/nostr-protocol/nips/blob/master/77.md)]
- **Qaul** — a mesh-networking chat app built to work with zero internet infrastructure: Bluetooth Low Energy, WiFi Direct/ad-hoc, LAN, and internet-as-just-another-link, with epidemic store-and-forward routing when peers aren't simultaneously reachable. Its wire format is protobuf-defined (`libqaul`), so it's schema-rigid where LCDP is schema-tolerant, but the deployment goal — communicate when the normal infrastructure is unavailable or hostile — is close to LCDP's "protocol honesty" framing in [lcdp_description_for_libp2p_users.md](lcdp_description_for_libp2p_users.md): both care about working, visibly, when the network is unfriendly, rather than degrading silently. As of 2026 it's at release-candidate 2.0.0-rc.5, describes itself as a production-ready toolkit, and reports real use by disaster-response teams and human rights organizations. [[qaul.net](https://qaul.net/)]
- **Briar** — secure messaging for activists/journalists, Tor by default over the internet, direct Bluetooth/WiFi when there's no internet at all, no phone number or central directory for adding contacts. Closer in mission to Qaul than to Nostr, but privacy/anonymity-first (Tor) rather than mesh/offline-first. Important status update as of 2026: **Briar entered maintenance mode this year** (latest release 1.5.19, July 13, 2026) — the project remains active but the team has stopped new feature work, shipping only bug/security fixes going forward. Their own announcement cites longstanding practical pain: high battery usage, unreliable Android background operation, missing account backup and file attachments, and a difficult UX for adding contacts and communicating offline. [[Briar: maintenance mode](https://briarproject.org/news/2026-maintenance-mode/)] It's a useful real-world data point on how much unglamorous engineering "just works reliably on a phone" costs for a genuinely offline/anonymity-first p2p messenger, independent of the protocol design being sound.

## 7. The odd one out — Pijul

Pijul is a distributed version control system (a Git/Mercurial alternative), not a messaging protocol, but it belongs on this list because it's genuinely peer-to-peer — any two repos can push/pull directly, no server required — and because its core idea, a mathematically commutative "patch algebra," is a different answer to the same underlying question every CRDT-flavored project above is also answering: *how do independent peers converge on the same state without coordinating first.* Pijul's patches are proven to commute and associate, so applying them in any order (including cherry-picking one out of sequence) produces the same result and never produces the kind of merge conflict Git can produce. Sync is a matter of diffing patch DAGs and exchanging what's missing over SSH, HTTPS, or a direct connection. There's no "ignore unknown fields" story here — the patch format is tightly, coordinated-version specified by a small core team (mainly Pierre-Étienne Meunier), the opposite extensibility bet from LCDP, made for a domain (source code history) where silent tolerance of unrecognized data would be actively dangerous. It's licensed GPL-2.0-or-later and, as of 2026, is still officially in alpha with active commits, having never fully graduated to a stable 1.0 despite years of real-world use by a niche audience. [[pijul.org FAQ](https://pijul.org/faq)]

## 8. What's genuinely rare in each direction

Being honest about tradeoffs rather than selling anything:

**What almost nothing else in this list gives you, that LCDP does:**
- A spec that fits in one paragraph and can be implemented from scratch by reading `tcpdump -A` output.
- Zero mandatory identity, zero mandatory crypto, zero mandatory persistence — you can speak a subset of one message type and be a fully conformant node.
- Compatibility guaranteed by never changing existing semantics, rather than by a coordinated migration, a schema hash, or a spec committee.

**What almost everything else in this list gives you, that LCDP does not:**
- An actual answer to "how do two devices with the same data converge after being offline" — LCDP has no data model to converge. Every project in sections 3-7 does real work here (CRDTs, patch algebras, Merkle-tree diffing, hash-chain validation) that a real app eventually needs and that LCDP explicitly punts to "define it as an optional message type."
- NAT traversal that actually works most of the time by default (libp2p, iroh, Holochain's Kitsune) — LCDP fails visibly on purpose rather than solving this, which is a stated design choice, not an oversight, but it does mean you're doing more work than a libp2p or iroh app would need to.
- A community-maintained registry of message/event types so independent implementations don't collide on meaning (Nostr's NIPs, P2Panda's schema hashes) — LCDP deliberately has no registry, which is either a feature (no gatekeeper) or a foot-gun (namespace collisions are possible, if unlikely at LCDP's current scale) depending on how many independent implementations start inventing message types with the same name for different things.

## 9. How LCDP could carry, or be carried by, any of these

The general pattern, already spelled out for iroh and Nostr specifically:

```
Some richer protocol's own transport   // fast/native path
  |
  +-- ALPN or message-type = /lcdp/0.4  // JSON array, slow/debuggable/universal path
```

Concretely: a Hypercore or Willow sync session could be tunneled as an LCDP `EncryptedMessages` payload for the cases where you want it to survive being carried over something LCDP-only speaking. An SSB pub, a Nostr relay, a Cabal cabal, or a Qaul mesh node could all expose an LCDP-speaking side purely for debuggability and for talking to constrained/legacy/AI-generated clients that don't want to link a whole replication engine just to say hello. None of them need to; LCDP doesn't require anyone's permission to be used this way, which is the point.

## Further reading in this repo

- [how_lcdp_contrasts_to_iroh.md](how_lcdp_contrasts_to_iroh.md)
- [how_lcdp_contrasts_to_nostr.md](how_lcdp_contrasts_to_nostr.md)
- [lcdp_description_for_libp2p_users.md](lcdp_description_for_libp2p_users.md)
- [why-messages-not-connections.md](why-messages-not-connections.md)
- [draft-pearson-lcdp-04.txt](draft-pearson-lcdp-04.txt)
