# **MiniDoc Design Note**

**Proto-RFC Working Draft for Channel-Scoped Mutable Content over HTTPS**

Version: 0.2 — Informational Design Note (Pre-RFC Working Draft)  
May 2026

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document defines the current design direction for **MiniDocs** within the Agents Shared Cognition Protocol (ASCP) ecosystem. It is an **informational design note** being developed in a form intentionally closer to an ASCP protocol draft so that the MiniDoc model, transport boundaries, and security assumptions can converge toward a future protocol specification.

This document is **not** yet a complete ASCP protocol specification. It does not yet freeze the complete wire format, complete HTTP header profile, or complete server interoperability rules required for a full standards-track style specification. However, it does define the current intended architectural model, protocol boundaries, initial MiniDoc profiles, and the security and admission model expected to shape the eventual protocol.

This document is **not** an Internet Standards Track specification. It has not undergone IETF review and has no formal standing within the IETF process. The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. In this design note, such requirements are provisional and indicate the intended direction of the future protocol.

# **2. Abstract**

This document defines the current architectural design for **MiniDocs**, a Channel-scoped mutable content mechanism for ASCP-native coordination workflows.

MiniDocs are distinct from immutable ASCP Artipoints. Artipoints provide durable semantic coordination structure, while MiniDocs provide evolving content bodies associated with those structures. MiniDoc content is carried using the same Layer-1 Channel security model used elsewhere in ASCP: a protected **Channel Envelope** providing signatures, optional encryption, and Channel-scoped access control.

This document introduces **MiniDoc Records** as the MiniDoc-side analogue of immutable **Artipoint Records**. In the immutable ASCP path, Layer-1 Channel Envelopes become Artipoint Records and are replicated through ALSP. In the MiniDoc path, Layer-1 Channel Envelopes become MiniDoc Records and are exchanged between MiniDoc clients and MiniDoc servers over HTTPS/TLS.

This document also defines two initial MiniDoc classes:

- **MiniDoc Document** — a full-state replacement profile in which each new MiniDoc Record becomes the next complete document version.
- **MiniDoc Log** — an append-state profile in which MiniDoc Records accumulate as an ever-expanding document history.

The document further defines the intended relationship between MiniDoc transport, Layer-1 codecs, Channel Access Keys (CAKs), Channel Access Proofs (CAPs), HTTPS authentication, version history, and future CRDT-based evolution.

# **3. Introduction**

ASCP Artipoints are intentionally immutable. This property provides durable provenance, auditability, semantic clarity, and append-only coordination semantics. However, many coordination workflows require mutable content bodies associated with those immutable coordination structures.

Examples include:

- Task descriptions
- Goal rationale
- Decision notes
- Agent scratchpads
- Working drafts
- Shared annotations
- Lightweight markdown documents

MiniDocs provide a mechanism for representing these mutable bodies while preserving ASCP’s immutable coordination semantics. A MiniDoc is therefore best understood as a mutable, Channel-scoped content object whose evolving states are referenced by immutable ASCP Artipoints.

## **3.1 Design goals**

MiniDocs are intended to satisfy the following goals:

1. Support lightweight mutable coordination content.
2. Preserve immutable ASCP coordination history.
3. Reuse ASCP Channel security and admission semantics.
4. Reuse ordinary HTTPS-based transport and tooling.
5. Support a simple centralized transactional implementation first.
6. Preserve stable, implementation-independent references to immutable MiniDoc states.
7. Support future evolution toward local-first and CRDT-based models.
8. Avoid introducing a separate security or trust model unrelated to ASCP Channels.

## **3.2 Position of MiniDocs in the ASCP model**

MiniDocs are part of the ASCP ecosystem, but they are not themselves Artipoints and they are not Layer-0 log records.

Instead:

- Artipoints define immutable semantic coordination structure.
- MiniDocs define mutable content evolution.
- Layer-1 Channel codecs secure MiniDoc payloads in the same way they secure other Channel-scoped payloads.
- The MiniDoc client/server protocol transports MiniDoc Records over HTTPS/TLS rather than through ALSP.

This creates a deliberate symmetry:

| **Concern** | **Immutable ASCP path** | **MiniDoc path** |
| --- | --- | --- |
| Protected payload creation | Layer-1 Channel Envelope | Layer-1 Channel Envelope |
| Transport unit | Artipoint Record | MiniDoc Record |
| Transport substrate | ALSP / Layer-0 log sync | HTTPS/TLS client-server protocol |
| Persistence model | Append-only Channel Log | MiniDoc server document history |
| Content model | Immutable coordination record | Mutable document evolution |

## **3.3 Scope of this document**

This design note covers:

- the architectural role of MiniDocs in ASCP,
- the MiniDoc object and record model,
- the two initial MiniDoc classes,
- the intended HTTPS/TLS transport model,
- the relationship between HTTP authentication and ASCP identity,
- the CAK/CAP-style admission model for Channel-scoped MiniDoc access,
- the initial transactional synchronization profile,
- and the future CRDT-compatible evolution path.

## **3.4 Out of scope**

This document does not yet define:

- the final complete HTTP endpoint inventory,
- the final wire serialization for every request and response,
- the full HTTP header profile for authentication and admission carriage,
- CRDT-specific state formats or merge algorithms,
- UI behavior,
- or application-level semantics of the MiniDoc content itself.

# **4. Relationship to companion ASCP specifications**

MiniDocs intentionally reuse existing ASCP concepts rather than replacing them.

- The ASCP architectural and layering model is defined by **The Agents Shared Cognition Protocol** and the **ASCP Terminology Primer**.
- The semantics of **Layer-1 Channel Envelopes** are owned by the ASCP Channel specification.
- The immutable log-replication model for **Artipoint Records** is owned by **ALSP**.
- Trust anchoring, identity/certificate meaning, and bootstrap-distributed key material remain owned by the ASCP Identity & Trust and Bootstrap specifications.

This document does **not** redefine those companion specifications. Instead, it defines how MiniDocs reuse those concepts in a mutable-content transport path.

The key boundary is:

- Layer-1 owns signing, optional encryption, and validation of protected MiniDoc payloads.
- The MiniDoc protocol owns HTTPS transport, request/response behavior, document history coordination, and Channel-scoped document access procedures.
- MiniDoc servers do not redefine Channel meaning, governance meaning, or Layer-1 cryptographic semantics.

# **5. Terminology**

## **5.1 MiniDoc**

A **MiniDoc** is a mutable, Channel-scoped content object addressed by a stable MiniDoc identifier.

## **5.2 MiniDoc Record**

A **MiniDoc Record** is the transport and persistence unit exchanged between a MiniDoc client and a MiniDoc server. It is the MiniDoc-side analogue of an **Artipoint Record**.

For the purposes of this document, a MiniDoc Record consists of:

1. transport-visible metadata sufficient to identify the Channel scope, MiniDoc identity, and immutable state identity, and
2. an opaque **Layer-1 Channel Envelope** payload carried without semantic interpretation by the MiniDoc transport layer.

## **5.3 MiniDoc State**

A **MiniDoc State** is an immutable reconstructable state of a MiniDoc. A MiniDoc evolves by creating additional MiniDoc States.

## **5.4 `state_ref`**

A **state reference** (`state_ref`) is a stable identifier for one immutable MiniDoc State.

A `state_ref` **MUST** identify an immutable reconstructable state rather than a mutable version counter or storage-specific row identifier.

## **5.5 MiniDoc Document**

A **MiniDoc Document** is a MiniDoc profile in which each new MiniDoc Record represents a complete replacement state of the document.

## **5.6 MiniDoc Log**

A **MiniDoc Log** is a MiniDoc profile in which MiniDoc Records accumulate as an append-state sequence whose semantic effect is to extend an ever-growing document.

## **5.7 MiniDoc server**

A **MiniDoc server** is the HTTPS-speaking server responsible for authenticating clients, enforcing Channel-scoped admission policy, coordinating MiniDoc history progression, and serving MiniDoc Records or reconstructable states.

## **5.8 MiniDoc client**

A **MiniDoc client** is an ASCP stack client that uses the Layer-1 codec to produce and validate Channel Envelopes and that exchanges MiniDoc Records with a MiniDoc server.

# **6. Architectural overview**

## **6.1 MiniDocs are not Artipoints**

MiniDocs are distinct from immutable ASCP Artipoints.

- Artipoints represent immutable semantic coordination statements.
- MiniDocs represent mutable content bodies.
- Artipoints MAY reference immutable MiniDoc States.

This preserves immutable coordination history while allowing the associated content to evolve.

## **6.2 Channel-scoped transport model**

Every MiniDoc is scoped to an ASCP Channel. The Channel provides:

- membership and admission context,
- trust domain,
- encryption scope,
- replication scope,
- and authorization semantics for access to MiniDoc content.

For MiniDocs, Channel scoping is realized through MiniDoc Record metadata and the Layer-1 protected payload model rather than through a separate document-specific security system.

## **6.3 Layer-1 payload opacity**

The MiniDoc transport layer and MiniDoc server **MUST NOT** assign application-level semantic meaning to the protected Layer-1 payload carried in a MiniDoc Record beyond what is required to store, retrieve, order, and coordinate MiniDoc history.

As with ALSP:

- Layer-1 produces and validates the protected Channel Envelope.
- The transport/persistence layer carries that envelope as an opaque protected object.

## **6.4 MiniDoc identity and state identity**

A MiniDoc has a stable MiniDoc identifier across its lifetime.

Example:

```text
minidoc_id = "task-123.body"
```

Each immutable state has a separate state identity.

Example:

```text
state_ref = "sha256:9f2c..."
```

The MiniDoc identity remains stable while MiniDoc States evolve.

# **7. MiniDoc Record model**

## **7.1 Conceptual structure**

A MiniDoc Record conceptually binds:

- `channel_id`
- `minidoc_id`
- profile/type information
- immutable state identity
- any transport-level predecessor or history metadata required by the protocol profile
- and an opaque Layer-1 Channel Envelope payload

This document does not yet freeze the exact wire encoding of that metadata, but the Channel scope and immutable state identity are protocol-essential.

## **7.2 Relationship to Layer-1**

The Layer-1 Channel codec is used on the MiniDoc path in the same way it is used on the immutable ASCP path:

- On submission, the MiniDoc client uses Layer-1 to secure the payload before sending it to the server.
- On retrieval, the MiniDoc client validates and decodes the returned protected payload using Layer-1.

The MiniDoc server coordinates storage and history, but it does not replace Layer-1 cryptographic validation as the authoritative content-security mechanism.

## **7.3 URI model**

MiniDocs are intended to be referenced using ordinary HTTPS URLs.

Example:

```text
https://host.example/channels/{channel_id}/minidocs/{doc_id}/states/{state_ref}
```

Example:

```text
https://collab.example/channels/acme-product/minidocs/task-123.body/states/sha256-9f2c...
```

MiniDoc URIs **SHOULD** be stable and durable for retained states.

# **8. Initial MiniDoc classes**

## **8.1 MiniDoc Document**

A MiniDoc Document is the full-state replacement profile.

For a MiniDoc Document:

- each accepted MiniDoc Record represents one complete document state,
- each new accepted MiniDoc Record becomes the next document version,
- and the MiniDoc server maintains an ordered history of those immutable versions.

This profile is analogous to a per-document commit history in which each accepted record is a complete snapshot of that document.

## **8.2 MiniDoc Log**

A MiniDoc Log is the append-state accumulation profile.

For a MiniDoc Log:

- each accepted MiniDoc Record extends the document,
- the authoritative document history is the ordered sequence of accepted MiniDoc Records,
- and the semantic effect is an ever-growing document or log-shaped object.

Unlike a MiniDoc Document, a MiniDoc Log is not conceptually a series of whole-document replacements. Its evolution is additive.

## **8.3 Shared invariants**

Both MiniDoc classes share the following properties:

- they are Channel-scoped,
- they evolve through immutable MiniDoc Records,
- they expose immutable state references,
- they are secured using the Layer-1 Channel model,
- and they are retrieved and updated through the MiniDoc HTTPS protocol.

## **8.4 History retention and pruning**

MiniDoc servers **MAY** maintain a rolling history and **MAY** prune older stored MiniDoc Records according to retention policy.

Pruning does not change the abstract model:

- retained MiniDoc Records remain immutable,
- retained `state_ref` values remain stable,
- and the MiniDoc class semantics remain the same.

Pruning does affect availability of pruned historical states at that server. Implementations that prune history **SHOULD** make that retention policy explicit at the operational level.

# **9. HTTPS protocol model**

## **9.1 Baseline transport profile**

The initial MiniDoc protocol profile uses HTTPS over TLS as its transport substrate.

Conforming deployments **SHOULD** use HTTPS over TLS 1.3 or higher as the baseline profile. TLS provides network-path confidentiality and resistance to active transport interference, but it does not by itself establish ASCP identity semantics or Channel admission.

## **9.2 Core client operations**

The initial transactional MiniDoc profile is expected to support at least the following logical operations:

1. Create a new MiniDoc.
2. Submit a new MiniDoc Record for an existing MiniDoc.
3. Fetch the current state of a MiniDoc.
4. Fetch a specific immutable MiniDoc State by `state_ref`.
5. Fetch the ordered history of MiniDoc Records for a MiniDoc.

This document does not yet freeze the exact endpoint shapes for those operations.

## **9.3 Retrieval behavior**

Fetch operations **SHOULD** return MiniDoc Records, or material sufficient to reconstruct and validate MiniDoc Records, rather than returning only already-decoded application content.

This preserves the Channel-protected model in which MiniDoc clients validate the protected payload on receipt using the same Layer-1 codec semantics used during submission.

## **9.4 Server role in the initial profile**

In the initial profile, the MiniDoc server acts as the authoritative coordinator for:

- client authentication,
- Channel-scoped document admission,
- ordered acceptance of MiniDoc Records,
- current-state determination,
- and retained-history retrieval.

The server-authoritative role in this profile does not change the protected-payload boundary: the server coordinates history, while Layer-1 remains authoritative for content protection and validation.

# **10. Session authentication and Channel admission**

## **10.1 Separation of concerns**

The MiniDoc protocol separates:

- **session/client authentication** — establishing who the HTTP client is in ASCP identity terms, and
- **MiniDoc admission** — establishing whether the client may access MiniDoc resources for the referenced Channel.

This mirrors ALSP’s distinction between session authentication and replication admission.

## **10.2 HTTPS authentication model**

MiniDoc session authentication is intended to use standard HTTPS and HTTP authentication mechanisms rather than ALSP-specific message framing.

The current design direction is:

- the connection is established over HTTPS/TLS,
- the client authenticates using an ASCP identity/certificate-backed HTTP authentication scheme,
- the server may challenge using ordinary HTTP authentication challenge/response behavior,
- and successful HTTP authentication establishes the ASCP identity binding for the session.

This is conceptually analogous to ALSP direct authentication, but adapted to standard HTTP request/response semantics.

## **10.3 Channel admission using CAK and CAP**

MiniDoc access remains Channel-governed. It does not introduce a separate document-specific authorization scheme unrelated to ASCP Channels.

Where a Channel requires access admission:

- the client **MUST** prove possession of the appropriate Channel Access Key (CAK) or equivalent active authorization material,
- the proof is expressed through a Channel Access Proof (CAP)-like mechanism analogous to ALSP,
- and the server **MUST** verify that proof before serving protected MiniDoc content or accepting new MiniDoc Records for that Channel.

Authentication establishes who the client is. CAK/CAP-based admission establishes whether that client may access MiniDoc resources in the referenced Channel.

## **10.4 HTTP carriage boundary**

This design note intentionally does not yet freeze the final HTTP carriage details for CAP material. However, the intended profile is that:

- identity authentication uses standard HTTP authentication challenge/response behavior,
- admission proof is carried as request-associated protocol material,
- and admission verification occurs in the context of the authenticated request before protected MiniDoc access is granted.

# **11. Versioning and synchronization semantics**

## **11.1 Initial transactional profile**

The MVP MiniDoc implementation is expected to use a centralized online transactional model.

Properties:

- online-only editing,
- server-authoritative updates,
- HTTPS API,
- no offline editing requirement,
- and no CRDT synchronization in the initial interoperable profile.

In this model, MiniDoc updates are coordinated through the MiniDoc server.

## **11.2 MiniDoc Document progression**

For a MiniDoc Document, the server accepts an ordered series of full-state MiniDoc Records.

Conceptually:

```text
Client submits state S2 as successor to S1
Server accepts new MiniDoc Record R2
R2 becomes the current document version
```

## **11.3 MiniDoc Log progression**

For a MiniDoc Log, the server accepts an ordered series of append-state MiniDoc Records.

Conceptually:

```text
Client submits append record A3
Server accepts new MiniDoc Record R3
R3 extends the MiniDoc Log
```

The authoritative history of a MiniDoc Log is the accepted ordered record sequence rather than a sequence of full replacements.

## **11.4 Current-state semantics**

For a MiniDoc Document, the current state is the latest accepted full-state MiniDoc Record.

For a MiniDoc Log, the current state is the materialized result of the accepted ordered record sequence, or an equivalent retained representation consistent with that sequence.

## **11.5 Future CRDT evolution**

Future MiniDoc implementations **MAY** use CRDT-based synchronization.

In that model:

- MiniDocs become local-first objects,
- clients maintain local MiniDoc replicas,
- CRDT updates synchronize through MiniDoc servers and/or peer exchange,
- and immutable external MiniDoc state references remain stable.

The intended invariant is that:

- MiniDoc identity remains stable,
- URI structure remains stable,
- Artipoint references remain stable,
- and Channel security and admission semantics remain stable.

Only the coherency mechanism changes.

# **12. Relationship to ASCP Artipoints and external artifacts**

## **12.1 Relationship to ASCP Artipoints**

Artipoints **MAY** reference MiniDoc States.

Examples:

- a Task Artipoint referencing a task-body MiniDoc State,
- a Decision Artipoint referencing a rationale MiniDoc State,
- a Goal Artipoint referencing a strategy MiniDoc State.

This preserves:

- immutable coordination history,
- durable references,
- semantic clarity,
- and auditability,

while allowing mutable coordination content.

## **12.2 Relationship to external artifacts**

MiniDocs are distinct from externally governed artifacts such as:

- Google Docs,
- OneDrive files,
- Dropbox files,
- Slack messages,
- GitHub resources,
- and generic web URLs.

External artifacts are governed externally and may be referenced through other ASCP mechanisms. MiniDocs are native ASCP Channel resources.

# **13. Operational considerations**

MiniDoc implementations may vary internally, but they should preserve the externally visible model defined here.

Implementations **MAY** store MiniDoc content using:

- inline content,
- content-addressed blobs,
- database rows,
- append logs,
- snapshot objects,
- or future CRDT update logs.

Implementations **SHOULD** preserve stable immutable state references independent of the underlying storage technique.

Operational concerns that the eventual protocol specification will need to define more fully include:

- retention and pruning policy,
- durability expectations,
- admission key rotation behavior,
- server recovery behavior,
- and migration from transactional to CRDT-backed deployment models.

# **14. Error handling**

This design note does not yet freeze a complete HTTP error vocabulary, but the MiniDoc protocol should distinguish at least the following failure classes:

- authentication failed,
- Channel admission proof missing or invalid,
- unknown Channel,
- unknown MiniDoc,
- unknown historical state,
- stale or conflicting update basis,
- malformed MiniDoc Record,
- unsupported MiniDoc profile,
- and transport-level HTTPS/TLS failure.

Transport-level failures are distinct from protocol-level MiniDoc errors in the same way that ALSP transport failures are distinct from ALSP protocol errors.

# **15. Security considerations**

MiniDoc security is layered across the ASCP stack.

- Layer-1 provides protected payload semantics including signatures and optional encryption.
- HTTPS/TLS provides network-path confidentiality and resistance to active transport interference.
- HTTP authentication provides ASCP identity binding for the client session.
- CAK/CAP-based admission provides Channel-scoped access control for MiniDoc resources.

The resulting security model is intended to provide:

- integrity of protected MiniDoc payloads,
- confidentiality where Layer-1 encryption is used,
- authenticated client identity,
- and admitted access to Channel-scoped MiniDoc resources.

Threats that the MiniDoc protocol must account for include:

- replay of admission material,
- MiniDoc Record substitution,
- forged or stale CAP material,
- Channel confusion,
- identity-binding mistakes between HTTP auth and Channel admission,
- downgrade or misconfiguration of TLS,
- unauthorized semantic effect despite authenticated transport,
- denial of service against MiniDoc servers,
- and retention-policy surprises that expose or remove historical content unexpectedly.

As with ALSP, TLS is necessary but not sufficient for ASCP trust. TLS alone does not establish Channel admission, Layer-1 authorship, or higher-layer governance meaning.

# **16. Privacy considerations**

MiniDocs can expose privacy-sensitive metadata even when protected content is encrypted.

Relevant privacy surfaces include:

- stable MiniDoc identifiers,
- stable state references,
- Channel identifiers,
- access timing,
- authorship metadata,
- document-history visibility,
- and cross-Channel or cross-server correlation risk if identifiers are reused carelessly.

Deployments should assume that some metadata may remain observable even when payload confidentiality is otherwise protected. Future protocol work should further specify privacy expectations around request logging, history retention, and metadata minimization.

# **17. IANA considerations**

This document has no IANA actions.

# **18. Examples (Informative)**

## **18.1 MiniDoc Document example**

```text
MiniDoc: task-123.body
Channel: acme-product
Profile: MiniDoc Document

R1 -> full state "Draft task text"
R2 -> full state "Revised task text"
R3 -> full state "Approved task text"
```

Each accepted record replaces the current state while preserving immutable history.

## **18.2 MiniDoc Log example**

```text
MiniDoc: decision-77.log
Channel: acme-product
Profile: MiniDoc Log

R1 -> "Initial decision note"
R2 -> append "Rationale update"
R3 -> append "Follow-up annotation"
```

The current MiniDoc Log is the accumulated result of the ordered record sequence.

## **18.3 Artipoint reference example**

```text
Decision D references MiniDoc state S
```

The Artipoint remains immutable even though the MiniDoc may later evolve to additional states.

# **19. References**

## **19.1 Normative references**

- RFC 2119, *Key words for use in RFCs to Indicate Requirement Levels*.
- RFC 8174, *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*.
- RFC 8446, *The Transport Layer Security (TLS) Protocol Version 1.3*.
- ASCP Channels specification.
- ASCP LogSync Protocol (ALSP).
- ASCP Trust and Identity Architecture.
- ASCP Bootstrap Process and Channel Discovery.

## **19.2 Informative references**

- *The Agents Shared Cognition Protocol*.
- *ASCP Terminology Primer*.
- RFC 3552, *Guidelines for Writing RFC Text on Security Considerations*.
- RFC 6973, *Privacy Considerations for Internet Protocols*.
