# **MiniDoc Design Note**

**Proto-RFC Working Draft for the ASCP MiniDoc Protocol**

Version: 0.3 — Informational Design Note (Pre-RFC Working Draft)  
May 2026

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document defines the current design direction for **MiniDocs** within the Agents Shared Cognition Protocol (ASCP) ecosystem. It is an **informational design note** being developed in a form intentionally closer to an ASCP protocol draft so that the MiniDoc model, transport boundaries, and security assumptions can converge toward a future protocol specification.

This document is **not** yet a complete ASCP protocol specification. It does not yet freeze the complete wire format, complete HTTP header profile, or complete server interoperability rules required for a full standards-track style specification. However, it does define the current intended architectural model, protocol boundaries, initial MiniDoc classes, document-model subtypes, and the security and admission model expected to shape the eventual protocol.

This document is **not** an Internet Standards Track specification. It has not undergone IETF review and has no formal standing within the IETF process. The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. In this design note, such requirements are provisional and indicate the intended direction of the future protocol.

# **2. Abstract**

This document defines the current architectural design for **MiniDocs**, a Channel-scoped mutable content mechanism for ASCP-native coordination workflows.

MiniDocs are distinct from immutable ASCP Artipoints. Artipoints provide durable semantic coordination structure, while MiniDocs provide evolving content bodies associated with those structures. MiniDoc content is carried using the same Layer-1 Channel security model used elsewhere in ASCP: a protected **Channel Envelope** providing signatures, optional encryption, and Channel-scoped access control.

This document introduces the **ASCP MiniDoc Protocol (AMDP)** as the protocol used to create, update, retrieve, and version MiniDocs. The initial AMDP transport profile is **AMDP over HTTPS**, but HTTPS is only the first binding profile for the protocol, not the defining identity form of a MiniDoc.

This document also introduces **MiniDoc Records** as the MiniDoc-side analogue of immutable **Artipoint Records**. In the immutable ASCP path, Layer-1 Channel Envelopes become Artipoint Records and are replicated through ALSP. In the MiniDoc path, Layer-1 Channel Envelopes become MiniDoc Records and are exchanged between MiniDoc clients and MiniDoc servers through AMDP.

This document defines two initial MiniDoc forms:

- **MiniDoc Document** — the general mutable document class.
- **MiniDoc Log** — a transcript-oriented append-state MiniDoc class.

And a **MiniDoc Document** has two different document interaction models:

- **Snapshot MiniDoc Document** — each accepted update is a complete snapshot of the document.
- **Collaborative MiniDoc Document** — each accepted update carries collaborative differential change while preserving one stable document identity.

The document further defines the intended relationship between MiniDoc transport, `minidoc://` URIs, Layer-1 codecs, Channel Access Keys (CAKs), Channel Access Proofs (CAPs), HTTPS authentication, version history, and future OT/CRDT-style collaborative evolution.

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
- User-agent conversational transcripts

MiniDocs provide a mechanism for representing these mutable bodies while preserving ASCP’s immutable coordination semantics. A MiniDoc is therefore best understood as a mutable, Channel-scoped content object whose evolving states are referenced by immutable ASCP Artipoints.

## **3.1 Design goals**

MiniDocs are intended to satisfy the following goals:

1. Support lightweight mutable coordination content.
2. Preserve immutable ASCP coordination history.
3. Reuse ASCP Channel security and admission semantics.
4. Reuse ordinary HTTP implementation patterns and tooling where practical.
5. Support a simple centralized transactional implementation first.
6. Preserve stable, implementation-independent references to immutable MiniDoc states.
7. Support future evolution toward local-first and collaborative conflict-avoiding models.
8. Avoid introducing a separate security or trust model unrelated to ASCP Channels.
9. Preserve one stable MiniDoc reference and versioning model across multiple document update models.

## **3.2 Position of MiniDocs in the ASCP model**

MiniDocs are part of the ASCP ecosystem, but they are not themselves Artipoints and they are not Layer-0 log records.

Instead:

- Artipoints define immutable semantic coordination structure.
- MiniDocs define mutable content evolution.
- Layer-1 Channel codecs secure MiniDoc payloads in the same way they secure other Channel-scoped payloads.
- The MiniDoc client/server protocol is **AMDP**.
- **AMDP over HTTPS** is the first transport profile for that protocol.

This creates a deliberate symmetry:

| **Concern** | **Immutable ASCP path** | **MiniDoc path** |
| --- | --- | --- |
| Protected payload creation | Layer-1 Channel Envelope | Layer-1 Channel Envelope |
| Transport unit | Artipoint Record | MiniDoc Record |
| Transport substrate | ALSP / Layer-0 log sync | AMDP |
| Initial binding profile | ALSP over WebSocket/TLS | AMDP over HTTPS |
| Persistence model | Append-only Channel Log | MiniDoc server document history |
| Content model | Immutable coordination record | Mutable document or transcript evolution |

## **3.3 Scope of this document**

This design note covers:

- the architectural role of MiniDocs in ASCP,
- the AMDP protocol boundary,
- the MiniDoc object and record model,
- the `minidoc://` identity model,
- the canonical ASCP MiniDoc URI path profile,
- the initial MiniDoc classes,
- the two initial MiniDoc Document models,
- the intended AMDP-over-HTTPS transport profile,
- the relationship between HTTP authentication and ASCP identity,
- the CAK/CAP-style admission model for Channel-scoped MiniDoc access,
- the initial transactional synchronization profile,
- and the future OT/CRDT-compatible evolution path.

## **3.4 Out of scope**

This document does not yet define:

- the final complete HTTP endpoint inventory,
- the final wire serialization for every request and response,
- the full HTTP header profile for authentication and admission carriage,
- the final metadata field names for MiniDoc Records,
- the exact Collaborative MiniDoc Document update encoding,
- the complete AMDP-over-HTTPS endpoint inventory corresponding to each MiniDoc URI form,
- UI behavior,
- or application-level semantics of the MiniDoc content beyond its transport and versioning model.

# **4. Relationship to companion ASCP specifications**

MiniDocs intentionally reuse existing ASCP concepts rather than replacing them.

- The ASCP architectural and layering model is defined by **The Agents Shared Cognition Protocol** and the **ASCP Terminology Primer**.
- The semantics of **Layer-1 Channel Envelopes** are owned by the ASCP Channel specification.
- The immutable log-replication model for **Artipoint Records** is owned by **ALSP**.
- Trust anchoring, identity/certificate meaning, and bootstrap-distributed key material remain owned by the ASCP Identity & Trust and Bootstrap specifications.

This document does **not** redefine those companion specifications. Instead, it defines how MiniDocs reuse those concepts in a mutable-content transport path.

The key boundary is:

- Layer-1 owns signing, optional encryption, and validation of protected MiniDoc payloads.
- AMDP owns MiniDoc-specific identity, request/response behavior, document history coordination, and Channel-scoped document access procedures.
- AMDP over HTTPS owns the first concrete HTTP/TLS realization of AMDP.
- MiniDoc servers do not redefine Channel meaning, governance meaning, or Layer-1 cryptographic semantics.

# **5. Terminology**

## **5.1 MiniDoc**

A **MiniDoc** is a mutable, Channel-scoped content object addressed by a stable MiniDoc URI.

## **5.2 MiniDoc URI**

A **MiniDoc URI** is the protocol-level identifier for a MiniDoc. MiniDocs are identified using a URL-shaped URI form:

```text
minidoc://<host>[:<port>]/ascp/<channel-uuid>/<doc-id>.<suffix>
```

This URI form is part of the MiniDoc reference model. It allows MiniDocs to appear naturally as URI/URL payloads within Artipoints while remaining independent of any one transport binding.

For the current ASCP MiniDoc path profile:

- `<host>[:<port>]` identifies the MiniDoc server authority in ordinary URI form,
- `/ascp/` signals the ASCP MiniDoc encapsulation profile for the pathname,
- `<channel-uuid>` identifies the Channel scope,
- `<doc-id>` is an opaque MiniDoc identifier unique within that Channel on that server,
- and `<suffix>` identifies the MiniDoc class or document-model subtype.

The current suffix set is:

- `.txt` for **Snapshot MiniDoc Documents**
- `.collab` for **Collaborative MiniDoc Documents**
- `.log` for **MiniDoc Logs**

The `/ascp/` path segment is used for both explicitness and future-proofing. It makes the ASCP MiniDoc path profile visible while leaving room for future non-ASCP MiniDoc path profiles under the same `minidoc:` scheme if such profiles are ever needed.

## **5.3 MiniDoc Record**

A **MiniDoc Record** is the transport and persistence unit exchanged between a MiniDoc client and a MiniDoc server. It is the MiniDoc-side analogue of an **Artipoint Record**.

For the purposes of this document, a MiniDoc Record consists of:

1. transport-visible metadata sufficient to identify the Channel scope, MiniDoc identity, MiniDoc class, document-model subtype where applicable, and immutable state identity, and
2. an opaque **Layer-1 Channel Envelope** payload carried without semantic interpretation by the MiniDoc transport layer.

## **5.4 MiniDoc State**

A **MiniDoc State** is an immutable reconstructable state of a MiniDoc. A MiniDoc evolves by creating additional MiniDoc States.

## **5.5 `state-id`**

A **state-id** is a stable identifier for one immutable MiniDoc State.

A `state-id` **MUST** identify an immutable reconstructable state rather than a mutable version counter or storage-specific row identifier.

The canonical current profile uses:

- opaque server-issued 64-bit `doc-id` values,
- opaque server-issued or server-recognized 64-bit `state-id` values,
- and unpadded base64url text encoding for both.

Under that profile, each encoded `doc-id` and `state-id` is represented as an 11-character base64url string.

## **5.6 MiniDoc Document**

A **MiniDoc Document** is the general mutable document class in AMDP. It has:

- stable MiniDoc identity,
- stable read, update, version, and history semantics,
- immutable state identifiers,
- and support for multiple document-model subtypes that preserve the same external MiniDoc reference model.

## **5.7 Snapshot MiniDoc Document**

A **Snapshot MiniDoc Document** is a MiniDoc Document model in which each accepted MiniDoc Record carries a whole-document state and becomes the next coherent document snapshot or revision.

## **5.8 Collaborative MiniDoc Document**

A **Collaborative MiniDoc Document** is a MiniDoc Document model in which accepted MiniDoc Records carry incremental collaborative change rather than whole-document replacement, while preserving the same external MiniDoc identity, read, update, version, and history semantics as other MiniDoc Documents.

Collaborative MiniDoc Documents will likely be implemented using OT, CRDT, or related conflict-avoiding collaborative techniques, but those techniques are informative implementation strategies rather than the defining semantic meaning of the document model.

## **5.9 MiniDoc Log**

A **MiniDoc Log** is a MiniDoc class whose semantics are transcript-oriented and append-state. Each accepted MiniDoc Record extends the running log or transcript.

## **5.10 MiniDoc transaction atomicity**

Each accepted **MiniDoc Record** is atomic: it is fully applied or not applied at all.

Each accepted MiniDoc Record yields exactly one new identifiable immutable MiniDoc State. The historical state sequence is therefore aligned 1:1 with accepted MiniDoc Record application boundaries across:

- Snapshot MiniDoc Documents
- Collaborative MiniDoc Documents
- MiniDoc Logs

## **5.11 MiniDoc server**

A **MiniDoc server** is the AMDP-speaking server responsible for authenticating clients, enforcing Channel-scoped admission policy, coordinating MiniDoc history progression, and serving MiniDoc Records or reconstructable states.

## **5.12 MiniDoc client**

A **MiniDoc client** is an ASCP stack client that uses the Layer-1 codec to produce and validate Channel Envelopes and that exchanges MiniDoc Records with a MiniDoc server using AMDP.

# **6. Architectural overview**

## **6.1 MiniDocs are not Artipoints**

MiniDocs are distinct from immutable ASCP Artipoints.

- Artipoints represent immutable semantic coordination statements.
- MiniDocs represent mutable content bodies or transcript resources.
- Artipoints MAY reference immutable MiniDoc States using MiniDoc URIs and, where needed, `state-id` selectors.

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

The AMDP transport layer and MiniDoc server **MUST NOT** assign application-level semantic meaning to the protected Layer-1 payload carried in a MiniDoc Record beyond what is required to store, retrieve, order, and coordinate MiniDoc history.

As with ALSP:

- Layer-1 produces and validates the protected Channel Envelope.
- The transport and persistence layer carries that envelope as an opaque protected object.

## **6.4 MiniDoc identity and state identity**

A MiniDoc has a stable MiniDoc URI across its lifetime.

Example:

```text
minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/6m2rQ8TnP4A.txt
```

Each immutable state has a separate state identity.

Example:

```text
state-id = "bQ7mK9rTx2M"
```

The MiniDoc identity remains stable while MiniDoc States evolve.

# **7. MiniDoc Record model**

## **7.1 Conceptual structure**

A MiniDoc Record conceptually binds:

- `channel_id`
- MiniDoc URI
- MiniDoc class
- document-model subtype where applicable
- immutable state identity
- any transport-level predecessor or history metadata required by the protocol profile
- and an opaque Layer-1 Channel Envelope payload

This document does not yet freeze the exact wire encoding of that metadata, but the Channel scope, MiniDoc class, document-model subtype where applicable, and immutable state identity are protocol-essential.

## **7.2 Metadata-driven client behavior**

MiniDoc metadata **MUST** provide enough information for a client to determine how to interpret and submit records for a given MiniDoc.

At minimum, a client must be able to determine:

- whether the resource is a **MiniDoc Document** or a **MiniDoc Log**, and
- if it is a MiniDoc Document, whether it is a **Snapshot MiniDoc Document** or a **Collaborative MiniDoc Document**.

This is required because MiniDoc Record contents differ across these forms:

- Snapshot MiniDoc Documents carry whole-document states,
- Collaborative MiniDoc Documents carry incremental collaborative change,
- MiniDoc Logs carry append-oriented transcript or log extensions.

## **7.3 Relationship to Layer-1**

The Layer-1 Channel codec is used on the MiniDoc path in the same way it is used on the immutable ASCP path:

- On submission, the MiniDoc client uses Layer-1 to secure the payload before sending it to the server.
- On retrieval, the MiniDoc client validates and decodes the returned protected payload using Layer-1.

The MiniDoc server coordinates storage and history, but it does not replace Layer-1 cryptographic validation as the authoritative content-security mechanism.

## **7.4 Binding-level retrieval addresses**

The primary MiniDoc identity form is the `minidoc://` URI. A concrete transport profile such as AMDP over HTTPS may define HTTP retrieval addresses or endpoint paths used to operate on that MiniDoc URI.

Those HTTP addresses are binding-level realization details, not the primary MiniDoc identity itself.

## **7.5 Optional immutable state selection**

The base MiniDoc URI identifies the MiniDoc resource generically:

```text
minidoc://<host>[:<port>]/ascp/<channel-uuid>/<doc-id>.<suffix>
```

The current profile uses an optional query parameter to select a specific immutable historical state:

```text
?state-id=<state-id>
```

Accordingly:

- absence of `state-id` means the current effective head of the MiniDoc resource,
- presence of `state-id` means the exact immutable historical state named by that value.

This allows Artipoints to carry a stable base MiniDoc URI while later attributes or annotations pin a specific state without rewriting the base payload URI.

# **8. MiniDoc classes and document models**

## **8.1 MiniDoc Document**

MiniDoc Document is the high-level mutable document class. It is defined by stable document semantics rather than by one specific record encoding or coherency mechanism.

### **8.1.1 Snapshot MiniDoc Document**

A Snapshot MiniDoc Document is the initial interoperable document model.

For a Snapshot MiniDoc Document:

- each accepted MiniDoc Record represents one complete document state,
- each new accepted MiniDoc Record becomes the next coherent document snapshot or revision,
- and the MiniDoc server maintains an ordered history of those immutable versions.

This model is straightforward for server-authoritative transactional implementations.

### **8.1.2 Collaborative MiniDoc Document**

A Collaborative MiniDoc Document is the future collaborative document model.

For a Collaborative MiniDoc Document:

- each accepted MiniDoc Record represents incremental collaborative document evolution rather than full replacement,
- the MiniDoc remains one stable document resource with one stable identity and version/history model,
- and clients materialize document state from the collaborative update history or from an equivalent retained representation consistent with that history.

Collaborative MiniDoc Documents are expected to use OT, CRDT, or related collaborative-conflict-avoidance techniques informatively, but AMDP should define the document model in a way that does not depend on naming one such technique as normative.

### **8.1.3 Shared MiniDoc Document invariants**

Snapshot and Collaborative MiniDoc Documents share:

- the same MiniDoc URI model,
- the same Artipoint reference model,
- the same concept of immutable referenced states,
- and the same high-level read, update, version, and history semantics.

The difference between them is the record content model and the coherency model, not the external document identity model.

## **8.2 MiniDoc Log**

MiniDoc Log is a distinct MiniDoc class rather than a MiniDoc Document subtype.

Its primary intended use is transcript-oriented append semantics, especially:

- user-agent conversational transcripts,
- chat histories,
- session transcripts,
- and similar ordered interaction records.

For a MiniDoc Log:

- each accepted MiniDoc Record extends the log,
- the authoritative history is the ordered record sequence,
- and the semantic effect is a growing transcript or log-shaped resource.

Unlike a Collaborative MiniDoc Document, a MiniDoc Log is fundamentally transcript-oriented rather than document-oriented.

## **8.3 History retention and pruning**

MiniDoc servers **MAY** maintain a rolling history and **MAY** prune older stored MiniDoc Records according to retention policy.

Pruning does not change the abstract model:

- retained MiniDoc Records remain immutable,
- retained `state-id` values remain stable,
- and the MiniDoc class and document-model semantics remain the same.

Pruning does affect availability of pruned historical states at that server. Implementations that prune history **SHOULD** make that retention policy explicit at the operational level.

# **9. ASCP MiniDoc Protocol (AMDP)**

## **9.1 AMDP as the protocol layer**

AMDP defines the MiniDoc-specific client/server protocol semantics for:

- MiniDoc identity resolution,
- create, read, update, and history interactions,
- MiniDoc Record submission and retrieval,
- client authentication and Channel-scoped admission boundaries,
- and version or state progression semantics.

AMDP is not inherently tied to HTTPS, even though HTTPS is the first transport profile.

## **9.2 AMDP over HTTPS**

The initial AMDP transport profile uses HTTPS over TLS as its transport substrate.

AMDP over HTTPS should be defined so that it maps as directly as possible onto standard modern HTTP request and response patterns and can be implemented using ordinary HTTP client and server libraries.

Conforming AMDP-over-HTTPS deployments **SHOULD** use HTTPS over TLS 1.3 or higher as the baseline profile. TLS provides network-path confidentiality and resistance to active transport interference, but it does not by itself establish ASCP identity semantics or Channel admission.

## **9.3 Core AMDP operations**

The initial transactional AMDP profile is expected to support at least the following logical operations:

1. Create a new MiniDoc.
2. Submit a new MiniDoc Record for an existing MiniDoc.
3. Fetch the current state of a MiniDoc.
4. Fetch a specific immutable MiniDoc State by `state-id`.
5. Fetch the ordered history of MiniDoc Records for a MiniDoc.

This document does not yet freeze the exact endpoint shapes for those operations.

## **9.4 Retrieval behavior**

Fetch operations **SHOULD** return MiniDoc Records, or material sufficient to reconstruct and validate MiniDoc Records, rather than returning only already-decoded application content.

This preserves the Channel-protected model in which MiniDoc clients validate the protected payload on receipt using the same Layer-1 codec semantics used during submission.

## **9.5 Server role in the initial profile**

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

- **session and client authentication** — establishing who the AMDP client is in ASCP identity terms, and
- **MiniDoc admission** — establishing whether the client may access MiniDoc resources for the referenced Channel.

This mirrors ALSP’s distinction between session authentication and replication admission.

## **10.2 AMDP over HTTPS authentication model**

In the initial AMDP-over-HTTPS profile, session authentication is intended to use standard HTTPS and HTTP authentication mechanisms rather than ALSP-specific message framing.

The current design direction is:

- the connection is established over HTTPS/TLS,
- the client authenticates using an ASCP identity and certificate-backed HTTP authentication scheme,
- the server may challenge using ordinary HTTP authentication challenge and response behavior,
- and successful HTTP authentication establishes the ASCP identity binding for the request or session context.

This is conceptually analogous to ALSP direct authentication, but adapted to standard HTTP request and response semantics.

## **10.3 Channel admission using CAK and CAP**

MiniDoc access remains Channel-governed. It does not introduce a separate document-specific authorization scheme unrelated to ASCP Channels.

Where a Channel requires access admission:

- the client **MUST** prove possession of the appropriate Channel Access Key (CAK) or equivalent active authorization material,
- the proof is expressed through a Channel Access Proof (CAP)-like mechanism analogous to ALSP,
- and the server **MUST** verify that proof before serving protected MiniDoc content or accepting new MiniDoc Records for that Channel.

Authentication establishes who the client is. CAK and CAP-based admission establishes whether that client may access MiniDoc resources in the referenced Channel.

## **10.4 HTTP carriage boundary**

This design note intentionally does not yet freeze the final HTTP carriage details for CAP material. However, the intended profile is that:

- identity authentication uses standard HTTP authentication challenge and response behavior,
- admission proof is carried as request-associated protocol material,
- and admission verification occurs in the context of the authenticated request before protected MiniDoc access is granted.

# **11. Versioning and synchronization semantics**

## **11.1 Initial transactional profile**

The MVP AMDP implementation is expected to use a centralized online transactional model.

Properties:

- online-only editing,
- server-authoritative updates,
- HTTPS API in the initial transport profile,
- no offline editing requirement,
- and no Collaborative MiniDoc Document synchronization in the initial interoperable profile.

In this model, MiniDoc updates are coordinated through the MiniDoc server.

Each accepted MiniDoc Record creates exactly one new externally identifiable MiniDoc State boundary.

## **11.2 Snapshot MiniDoc Document progression**

For a Snapshot MiniDoc Document, the server accepts an ordered series of full-state MiniDoc Records.

Conceptually:

```text
Client submits snapshot S2 as successor to S1
Server accepts new MiniDoc Record R2
R2 becomes the current document version
```

## **11.3 Collaborative MiniDoc Document progression**

For a Collaborative MiniDoc Document, the server accepts an ordered series of incremental collaborative MiniDoc Records.

Conceptually:

```text
Client submits collaborative update U3
Server accepts new MiniDoc Record R3
R3 advances the collaborative document history
```

The external MiniDoc Document identity, version semantics, and immutable reference model remain stable even though the record contents are not whole-document snapshots.

## **11.4 MiniDoc Log progression**

For a MiniDoc Log, the server accepts an ordered series of append-state MiniDoc Records.

Conceptually:

```text
Client submits transcript append A3
Server accepts new MiniDoc Record R3
R3 extends the MiniDoc Log
```

The authoritative history of a MiniDoc Log is the accepted ordered record sequence, and the current state is the materialized transcript represented by that sequence.

## **11.5 Current-state semantics**

For a Snapshot MiniDoc Document, the current state is the latest accepted whole-document snapshot.

For a Collaborative MiniDoc Document, the current state is the materialized result of the accepted collaborative update history, or an equivalent retained representation consistent with that history.

For a MiniDoc Log, the current state is the materialized result of the accepted ordered transcript or log sequence, or an equivalent retained representation consistent with that sequence.

## **11.6 Future collaborative evolution**

Future AMDP implementations **MAY** use OT, CRDT, or related conflict-avoiding collaborative techniques for Collaborative MiniDoc Documents.

The intended invariant is that:

- MiniDoc identity remains stable,
- MiniDoc URIs remain stable,
- Artipoint references remain stable,
- and Channel security and admission semantics remain stable.

Only the internal collaborative coherency mechanism changes.

# **12. Relationship to ASCP Artipoints and external artifacts**

## **12.1 Relationship to ASCP Artipoints**

Artipoints **MAY** reference MiniDoc States.

Examples:

- a Task Artipoint referencing a task-body MiniDoc State,
- a Decision Artipoint referencing a rationale MiniDoc State,
- a Goal Artipoint referencing a strategy MiniDoc State,
- an interaction Artipoint referencing a conversational MiniDoc Log.

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

- whole-document snapshots,
- content-addressed blobs,
- database rows,
- append logs,
- transcript event sequences,
- collaborative update logs,
- snapshot objects,
- or future OT and CRDT-oriented retained forms.

Implementations **SHOULD** preserve stable immutable state identifiers independent of the underlying storage technique.

Operational concerns that the eventual protocol specification will need to define more fully include:

- retention and pruning policy,
- durability expectations,
- admission key rotation behavior,
- server recovery behavior,
- mapping from `minidoc://` identity to binding-level retrieval addresses,
- and migration from Snapshot to Collaborative MiniDoc Document deployment models.

# **14. Error handling**

This design note does not yet freeze a complete AMDP error vocabulary, but the protocol should distinguish at least the following failure classes:

- authentication failed,
- Channel admission proof missing or invalid,
- unknown Channel,
- unknown MiniDoc,
- unknown historical state,
- stale or conflicting update basis,
- malformed MiniDoc Record,
- unsupported MiniDoc class,
- unsupported MiniDoc Document model,
- and transport-level HTTPS or TLS failure in the AMDP-over-HTTPS profile.

Transport-level failures are distinct from protocol-level MiniDoc errors in the same way that ALSP transport failures are distinct from ALSP protocol errors.

# **15. Security considerations**

MiniDoc security is layered across the ASCP stack.

- Layer-1 provides protected payload semantics including signatures and optional encryption.
- HTTPS/TLS provides network-path confidentiality and resistance to active transport interference for the AMDP-over-HTTPS profile.
- HTTP authentication provides ASCP identity binding for the client session or request context in that profile.
- CAK and CAP-based admission provides Channel-scoped access control for MiniDoc resources.

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
- identity-binding mistakes between HTTP authentication and Channel admission,
- downgrade or misconfiguration of TLS,
- unauthorized semantic effect despite authenticated transport,
- denial of service against MiniDoc servers,
- and retention-policy surprises that expose or remove historical content unexpectedly.

Collaborative MiniDoc Documents may use OT, CRDT, or related collaborative-conflict-avoidance techniques as an implementation strategy, but the AMDP security boundary remains the same: Layer-1 protects payloads, and AMDP governs transport, admission, and history semantics.

As with ALSP, TLS is necessary but not sufficient for ASCP trust. TLS alone does not establish Channel admission, Layer-1 authorship, or higher-layer governance meaning.

# **16. Privacy considerations**

MiniDocs can expose privacy-sensitive metadata even when protected content is encrypted.

Relevant privacy surfaces include:

- stable MiniDoc URIs,
- stable `state-id` values,
- Channel identifiers,
- access timing,
- authorship metadata,
- document-history visibility,
- transcript visibility and interaction cadence,
- and cross-Channel or cross-server correlation risk if identifiers are reused carelessly.

Deployments should assume that some metadata may remain observable even when payload confidentiality is otherwise protected. Future protocol work should further specify privacy expectations around request logging, history retention, transcript sensitivity, and metadata minimization.

# **17. IANA considerations**

This document has no IANA actions.

# **18. Examples (Informative)**

## **18.1 Snapshot MiniDoc Document example**

```text
MiniDoc URI: minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/6m2rQ8TnP4A.txt
Channel UUID: 018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d
Class: MiniDoc Document
Model: Snapshot MiniDoc Document

R1 -> full state "Draft task text"
R2 -> full state "Revised task text"
R3 -> full state "Approved task text"
```

Each accepted record replaces the current document snapshot while preserving immutable history.

Pinned historical state example:

```text
minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/6m2rQ8TnP4A.txt?state-id=bQ7mK9rTx2M
```

## **18.2 Collaborative MiniDoc Document example**

```text
MiniDoc URI: minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/J4pLm82QeYs.collab
Channel UUID: 018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d
Class: MiniDoc Document
Model: Collaborative MiniDoc Document

R1 -> initial document state
R2 -> collaborative update adding section text
R3 -> collaborative update revising earlier content without replacing the whole document
```

The document remains one stable MiniDoc Document even though record contents represent incremental collaborative evolution rather than whole-document replacement.

## **18.3 MiniDoc Log example**

```text
MiniDoc URI: minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/2xRfQ1mNc6U.log
Channel UUID: 018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d
Class: MiniDoc Log

R1 -> "User asks opening question"
R2 -> "Agent replies with analysis"
R3 -> "User clarifies requirements"
R4 -> "Agent produces updated answer"
```

The current MiniDoc Log is the accumulated conversational transcript represented by the ordered record sequence.

## **18.4 Artipoint reference example**

```text
Goal G payload:
minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/6m2rQ8TnP4A.txt

Later annotation:
state-id=bQ7mK9rTx2M

Pinned retrieval form:
minidoc://collab.example:443/ascp/018f7d7a-b35c-7d24-b7e2-4df10f6d8b4d/6m2rQ8TnP4A.txt?state-id=bQ7mK9rTx2M
```

The Artipoint payload can remain the stable generic MiniDoc URI while later annotations pin one specific immutable state in effect.

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
