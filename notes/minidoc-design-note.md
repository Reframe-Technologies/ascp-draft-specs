# **MiniDoc Design Note**

**Proto-RFC Working Draft for the ASCP MiniDoc Protocol**

Version: 0.3 — Informational Design Note (Pre-RFC Working Draft)  
May 2026

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document defines the current design direction for **MiniDocs** within the Agents Shared Cognition Protocol (ASCP) ecosystem. It is an **informational design note** being developed in a form intentionally closer to an ASCP protocol draft so that the MiniDoc model, transport boundaries, and security assumptions can converge toward a future protocol specification.

This document captures the current intended architectural model, protocol boundaries, initial MiniDoc classes, document-model subtypes, and the security and admission model expected to shape the eventual protocol. Complete wire formats, complete HTTP header profiles, and full server interoperability rules remain for a future protocol specification.

This document is an informational pre-RFC working draft. It is outside the IETF standards-track process and has not undergone IETF review. The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. In this design note, such requirements are provisional and indicate the intended direction of the future protocol.

# **2. Abstract**

This document defines the current architectural design for **MiniDocs**, a Channel-scoped mutable content mechanism for ASCP-native coordination workflows.

MiniDocs provide evolving content bodies associated with ASCP coordination structures. Artipoints provide durable semantic coordination structure, while MiniDocs carry mutable Channel-scoped content using the same Layer-1 Channel security model used elsewhere in ASCP: a protected **Channel Envelope** providing signatures, optional encryption, and Channel-scoped access control.

This document introduces the **ASCP MiniDoc Protocol (AMDP)** as the protocol used to create, update, retrieve, and version MiniDocs. The initial AMDP transport profile is **AMDP over HTTPS**, while the MiniDoc identity form remains the transport-independent `minidoc://` URI.

This document also introduces **MiniDoc Records** as the MiniDoc-side analogue of immutable **Artipoint Records**. In the immutable ASCP path, Layer-1 Channel Envelopes become Artipoint Records and are replicated through ALSP. In the MiniDoc path, Layer-1 Channel Envelopes become MiniDoc Records and are exchanged between MiniDoc clients and MiniDoc servers through AMDP.

This document defines two initial MiniDoc forms:

- **MiniDoc Document** — the general mutable document class.
- **MiniDoc Log** — a transcript-oriented append-state MiniDoc class.

The **MiniDoc Document** class has two different document interaction models:

- **Snapshot MiniDoc Document** — each accepted update is a complete snapshot of the document in a most recent update wins kind of way.
- **Collaborative MiniDoc Document** — each accepted update carries collaborative differential change while preserving one stable document identity. These documents resolve deterministically in a conflict-free way.

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

MiniDocs provide a mechanism for representing these mutable bodies securely and efficiently while preserving ASCP’s immutable coordination semantics. A MiniDoc is therefore best understood as a mutable, Channel-scoped content object whose evolving states are referenced by immutable ASCP Artipoints via a URI stored in their payload field.

MiniDocs provide the native default mutable content resource for ASCP coordination when no separate external system of record is being referenced. ASCP constructs may instead reference externally governed artifacts when those systems are the operational source of record. A MiniDoc may therefore serve either as the long-term content resource for a construct or as an emergent interim resource later superseded by a reference to an external system of record.

## **3.1 Design goals**

MiniDocs are intended to satisfy the following goals:

1. Support lightweight mutable coordination content.
2. Preserve immutable ASCP coordination history.
3. Reuse ASCP Channel security and admission semantics.
4. Reuse ordinary HTTP implementation patterns and tooling.
5. Provide a simple centralized transactional implementation for early deployments.
6. Preserve stable, implementation-independent references to immutable MiniDoc states.
7. Support future evolution toward local-first and collaborative conflict-free models.
8. Avoid introducing a separate security or trust model unrelated to ASCP Channels.
9. Preserve one stable MiniDoc reference and versioning model across multiple document update models.

## **3.2 Position of MiniDocs in the ASCP Stack model**

MiniDocs are part of the ASCP ecosystem and occupy the mutable-content path for Channel-scoped coordination data.

- Artipoints define immutable semantic coordination structure.
- MiniDocs define mutable content evolution.
- Layer-1 Channel codecs secure MiniDoc payloads in the same way they secure other Channel-scoped payloads. ie: via a cryptographically signed and optionally encrypted Channel Envelope format.
- The MiniDoc client/server protocol isn ASCP MiniDoc Protocol abbreviated as **AMDP**.
- **AMDP over HTTPS** is the first transport profile for that protocol.

This creates a deliberate symmetry:

| **Concern** | **Immutable ASCP path** | ASCP **MiniDoc path** |
| --- | --- | --- |
| Protected payload creation | Layer-1 Channel Envelope | Layer-1 Channel Envelope |
| Transport unit | Artipoint Record | MiniDoc Record |
| Transport substrate | ALSP / Layer-0 log sync | AMDP |
| Initial binding profile | ALSP over WebSocket/TLS | AMDP over HTTPS |
| Persistence model | Append-only Channel Log | MiniDoc server document history |
| Content model | Immutable coordination record | Mutable documents and transcripts |

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

This document defines how MiniDocs reuse those companion concepts in a mutable-content transport path.

The key boundary is:

- Layer-1 owns signing, optional encryption, and validation of protected MiniDoc payloads.
- AMDP owns MiniDoc-specific identity, request/response behavior, document history coordination, and Channel-scoped document access procedures.
- AMDP over HTTPS owns the first concrete HTTP/TLS realization of AMDP.
- ASCP Channels and companion trust specifications continue to own Channel meaning, governance meaning, and Layer-1 cryptographic semantics.

# **5. Terminology**

## **5.1 MiniDoc**

A **MiniDoc** is a mutable, Channel-scoped AMDP content resource addressed by a stable MiniDoc URI. Its content evolves through an ordered series of immutable MiniDoc States that can be referenced from ASCP coordination structures.

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

A **MiniDoc Record** is the retained state-transition unit exchanged through AMDP. It is the MiniDoc-side analogue of an **Artipoint Record**.

In the current profile, an accepted retained MiniDoc Record carries:

1. a self-describing `record_type` corresponding to the MiniDoc class or document model declared for that MiniDoc,
2. `prev_state_id`, identifying the prior state boundary this record follows,
3. `state_id`, identifying the resulting immutable state boundary created by acceptance of the record, and
4. an opaque **Layer-1 Channel Envelope** payload carried without semantic interpretation by the MiniDoc transport layer.

On client submission, `state_id` is absent and is assigned by the server when the record is accepted. On later retrieval, retained MiniDoc Records include both `prev_state_id` and `state_id`.

For MiniDoc creation, the submitted record uses the reserved `prev_state_id` sentinel `origin`. This token is outside the normal base64url `state-id` encoding space and cannot collide with an issued `state-id`.

AMDP request and response operations may carry additional protocol metadata alongside a MiniDoc Record. That metadata can include the target MiniDoc URI or `doc-id`, `base-state-id`, `from-state-id`, `to-state-id`, retention cutoff information, and authenticated session or admission context. Those operation fields are part of AMDP processing and are not intrinsic retained-record fields.

## **5.4 MiniDoc State**

A **MiniDoc State** is an immutable reconstructable state of a MiniDoc. A MiniDoc evolves by creating additional MiniDoc States.

## **5.5 `state-id`**

A **state-id** is a stable identifier for one immutable MiniDoc State.

A `state-id` **MUST** identify the immutable reconstructable state boundary created by acceptance of a MiniDoc Record. It remains independent of mutable version counters or storage-specific row identifiers.

The canonical current profile uses:

- opaque server-issued 64-bit `doc-id` values,
- opaque server-issued 64-bit `state-id` values,
- and unpadded base64url text encoding for both.

Under that profile, each encoded `doc-id` and `state-id` is represented as an 11-character base64url string.

The initial profile treats `state-id` as a server-assigned resulting-state identifier. Clients do not choose `state-id` values on submission.

## **5.6 MiniDoc Document**

A **MiniDoc Document** is the general mutable document class in AMDP. It has:

- stable MiniDoc identity,
- stable read, update, version, and history semantics,
- immutable state identifiers,
- and support for multiple document-model subtypes that preserve the same external MiniDoc reference model.

## **5.7 Snapshot MiniDoc Document**

A **Snapshot MiniDoc Document** is a MiniDoc Document model in which each accepted MiniDoc Record carries a whole-document state and becomes the next coherent document snapshot or revision.

## **5.8 Collaborative MiniDoc Document**

A **Collaborative MiniDoc Document** is a MiniDoc Document model in which accepted MiniDoc Records carry incremental collaborative change while preserving the same external MiniDoc identity, read, update, version, and history semantics as other MiniDoc Documents.

Collaborative MiniDoc Documents will likely be implemented using OT, CRDT, or related conflict-avoiding collaborative techniques. Those techniques are informative implementation strategies for this document model.

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

## **6.1 MiniDocs as mutable coordination content**

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

For MiniDocs, Channel scoping is carried through MiniDoc Record metadata and the Layer-1 protected payload model. MiniDoc access therefore follows ASCP Channel security, admission, and trust semantics.

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

## **6.5 Server-visible and client-visible semantics**

AMDP separates server-coordinated protocol state from client-interpreted content state.

The MiniDoc server coordinates and acts on:

- the MiniDoc URI and Channel UUID,
- the MiniDoc class and document-model subtype,
- `doc-id`, `state-id`, and predecessor or basis metadata,
- authenticated session state and admitted Channel session state,
- ordered record acceptance,
- retained-history boundaries,
- and retention-policy effects on historical availability.

The MiniDoc client interprets and validates:

- Layer-1 protected payload content,
- decrypted document or transcript content,
- application-level document meaning,
- collaborative operation meaning for OT, CRDT, or related models,
- and any semantic validation that depends on the protected content itself.

This split allows the server to coordinate admissibility, sequencing, and retained history without requiring application-level interpretation of MiniDoc Record payloads.

# **7. MiniDoc Record model**

## **7.1 Retained record structure**

A retained MiniDoc Record in the current profile conceptually binds:

- `record_type`
- `prev_state_id`
- `state_id`
- and an opaque Layer-1 Channel Envelope payload

The exact wire encoding of those fields remains for the protocol specification, but their retained semantic roles are part of the current AMDP design.

AMDP operations supply additional request and response context outside the retained record itself. That context includes:

- target MiniDoc identity,
- `base-state-id`,
- `from-state-id`,
- `to-state-id`,
- retention cutoff metadata,
- and authenticated session or admission context.

## **7.2 Metadata-driven client behavior**

MiniDoc metadata **MUST** provide enough information for a client to determine how to interpret and submit records for a given MiniDoc.

At minimum, a client must be able to determine:

- whether the resource is a **MiniDoc Document** or a **MiniDoc Log**, and
- if it is a MiniDoc Document, whether it is a **Snapshot MiniDoc Document** or a **Collaborative MiniDoc Document**.

This is required because MiniDoc Record contents differ across these forms:

- Snapshot MiniDoc Documents carry whole-document states,
- Collaborative MiniDoc Documents carry incremental collaborative change,
- MiniDoc Logs carry append-oriented transcript or log extensions.

In the current profile, each MiniDoc has one declared type and all retained MiniDoc Records for that MiniDoc **MUST** carry the matching `record_type`. Mixed-type record histories within one MiniDoc are out of scope for the current profile.

## **7.3 Common state-transition invariant**

AMDP uses one shared state-transition invariant across all MiniDoc forms:

- one accepted MiniDoc Record,
- one atomic state transition,
- and one new immutable `state-id`.

This invariant allows Snapshot MiniDoc Documents, Collaborative MiniDoc Documents, and MiniDoc Logs to share one common protocol model even though their record contents differ.

For protocol purposes, the server needs to coordinate:

- the predecessor state boundary named by `prev_state_id`,
- the resulting state boundary named by `state_id`,
- the client-supplied operation basis state,
- the ordered retained history,
- and the resulting new immutable state boundary.

This explicit predecessor and resulting-state representation makes retained records auditable and self-describing even though the protected payload content remains opaque at the AMDP transport layer.

## **7.4 Relationship to Layer-1**

The Layer-1 Channel codec is used on the MiniDoc path in the same way it is used on the immutable ASCP path:

- On submission, the MiniDoc client uses Layer-1 to secure the payload before sending it to the server.
- On retrieval, the MiniDoc client validates and decodes the returned protected payload using Layer-1.

The MiniDoc server coordinates storage and history, while Layer-1 remains authoritative for cryptographic validation and protected content semantics.

## **7.5 Binding-level retrieval addresses**

The primary MiniDoc identity form is the `minidoc://` URI. A concrete transport profile such as AMDP over HTTPS may define HTTP retrieval addresses or endpoint paths used to operate on that MiniDoc URI.

Those HTTP addresses are binding-level realization details used to operate on the primary MiniDoc identity.

## **7.6 Optional immutable state selection**

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

MiniDoc Document is the high-level mutable document class. It is defined by stable document semantics, with multiple record and coherency models sharing that external behavior.

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

- each accepted MiniDoc Record represents incremental collaborative document evolution,
- the MiniDoc remains one stable document resource with one stable identity and version/history model,
- and clients materialize document state from the collaborative update history or from an equivalent retained representation consistent with that history.

Collaborative MiniDoc Documents are expected to use OT, CRDT, or related collaborative-conflict-avoidance techniques informatively. AMDP should define the document model through its externally visible semantics so that one specific technique does not become normative.

At the protocol layer, a Collaborative MiniDoc Document remains compatible with opaque server handling because the server coordinates only:

- record acceptance,
- ordered retention,
- state lineage,
- span retrieval boundaries,
- and compaction boundaries.

A collaborative MiniDoc Record may contain one or more incremental collaborative operations, but the enclosing MiniDoc Record remains the atomic accepted update unit. Client-side materialization and conflict-resolution behavior therefore remain above the server's opaque-record coordination role.

### **8.1.3 Shared MiniDoc Document invariants**

Snapshot and Collaborative MiniDoc Documents share:

- the same MiniDoc URI model,
- the same Artipoint reference model,
- the same concept of immutable referenced states,
- and the same high-level read, update, version, and history semantics.

They differ in record content model and coherency model while sharing one external document identity model.

## **8.2 MiniDoc Log**

MiniDoc Log is a distinct MiniDoc class with transcript-oriented append semantics.

Its primary intended use is transcript-oriented append semantics, especially:

- user-agent conversational transcripts,
- chat histories,
- session transcripts,
- and similar ordered interaction records.

For a MiniDoc Log:

- each accepted MiniDoc Record extends the log,
- the authoritative history is the ordered record sequence,
- and the semantic effect is a growing transcript or log-shaped resource.

MiniDoc Logs are optimized for transcript and event-sequence resources, while Collaborative MiniDoc Documents preserve general document semantics under collaborative evolution.

## **8.3 History retention and pruning**

MiniDoc servers **MAY** maintain a rolling history and **MAY** prune older stored MiniDoc Records according to retention policy.

Pruning preserves the abstract model for retained history:

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

AMDP defines the MiniDoc protocol semantics. AMDP over HTTPS is the first transport profile for those semantics.

## **9.2 Protocol operation model**

AMDP is a record-coordination protocol with client-side content interpretation and server-side history coordination.

In the initial profile, every AMDP operation proceeds through these phases:

1. Authenticate or resume an AMDP session.
2. Establish or reuse admitted access for the referenced Channel.
3. Resolve the target MiniDoc resource or create a new one.
4. Perform the requested read, write, span, or retention operation against the retained MiniDoc history.
5. Return MiniDoc Records, or sufficient retained record material, for the requested state boundary.
6. Validate and materialize the protected content client-side through the Layer-1 codec.

This model keeps content security and interpretation at the client boundary while allowing the server to enforce admission, sequencing, and retained-history semantics.

## **9.3 AMDP over HTTPS**

The initial AMDP transport profile **MUST** use HTTPS over TLS as its transport substrate. Conforming AMDP-over-HTTPS deployments **MUST** use TLS 1.3 or higher.

AMDP over HTTPS **MUST** map MiniDoc resources and operations onto standard HTTP request and response patterns closely enough that ordinary HTTP servers, clients, cookies, headers, status codes, and JSON request bodies can realize the protocol directly.

In this profile:

- TLS provides transport confidentiality and resistance to network-path interference,
- HTTP authentication establishes ASCP client identity,
- a server-issued cookie carries the authenticated AMDP session,
- a dedicated admission header carries Channel Access Proof material,
- and HTTPS resource paths mirror the `minidoc://` identity structure.

## **9.4 Core AMDP operations**

The initial transactional AMDP-over-HTTPS profile **MUST** support the following operations:

1. Create a new MiniDoc and receive its canonical MiniDoc URI.
2. Submit a new MiniDoc Record for an existing MiniDoc.
3. Fetch the current state of a MiniDoc.
4. Fetch a specific immutable MiniDoc State by `state-id`.
5. Fetch a retained record span for a MiniDoc using `from-state-id` and `to-state-id`.
6. Truncate or compact retained history according to the MiniDoc class and document model.

The HTTPS binding for those operations **MUST** use the following resource and method model:

- `POST /ascp/<channel-uuid>/`
  Creates a new MiniDoc in the identified Channel. The request body **MUST** declare the MiniDoc `record_type` and include the initial submitted MiniDoc Record with `prev_state_id=origin` and no `state_id`. On success, the server **MUST** allocate `doc-id`, assign `state_id`, return `201 Created`, and include the canonical HTTPS resource path in the `Location` header.
- `GET /ascp/<channel-uuid>/<doc-id>.<suffix>`
  Returns the current retained state of the identified MiniDoc.
- `GET /ascp/<channel-uuid>/<doc-id>.<suffix>?state-id=<state-id>`
  Returns the exact retained immutable state selected by `state-id`.
- `GET /ascp/<channel-uuid>/<doc-id>.<suffix>?from-state-id=<state-id>[&to-state-id=<state-id>]`
  Returns the retained ordered record span needed to materialize the requested state range. When `to-state-id` is omitted, the upper bound is the current head state.
- `PATCH /ascp/<channel-uuid>/<doc-id>.<suffix>`
  Submits a new MiniDoc Record or a retention-maintenance mutation for the identified MiniDoc. The request body **MUST** include `base-state-id` and the mutation payload appropriate to the MiniDoc class and operation.

The current profile **MUST NOT** tunnel all AMDP operations through a single generic operation endpoint.

## **9.5 HTTP semantic mapping model**

AMDP-over-HTTPS **MUST** preserve the higher-level AMDP semantics while using ordinary HTTP mechanisms directly.

At the semantic level:

- authenticated AMDP session maps onto HTTP authentication plus a server-issued session cookie,
- admitted Channel session maps onto per-Channel admission state cached against that authenticated session,
- MiniDoc identity maps onto the `minidoc://` resource identity plus the HTTPS transport path `/ascp/<channel-uuid>/<doc-id>.<suffix>`,
- current versus pinned state maps onto base resource retrieval versus `state-id`-qualified retrieval,
- retained-history reads map onto `from-state-id` and `to-state-id` query-qualified retrieval,
- and mutation preconditions map onto request-carried `base-state-id`.

In this profile, AMDP request and response bodies **MUST** use `application/json`. Protected Layer-1 Channel Envelopes **MUST** be carried as opaque JOSE compact serialization strings inside those JSON bodies. CAP values carried in headers **MUST** likewise use compact JOSE serialization.

The profile **MUST** use these HTTP status code classes:

- `201 Created` for successful MiniDoc creation,
- `200 OK` for successful retrieval and successful mutation returning protocol state,
- `400 Bad Request` for malformed or contradictory request parameters,
- `401 Unauthorized` for missing, invalid, or expired authenticated session state,
- `403 Forbidden` for missing or invalid Channel admission,
- `404 Not Found` for unknown MiniDoc or unavailable retained historical state,
- and `409 Conflict` for stale or conflicting `base-state-id` or `prev_state_id` basis.

## **9.6 Retrieval behavior**

Fetch operations **MUST** return MiniDoc Records and the material sufficient to reconstruct and validate MiniDoc Records, so that MiniDoc clients can apply the Layer-1 validation model directly.

This preserves the Channel-protected model in which MiniDoc clients validate the protected payload on receipt using the same Layer-1 codec semantics used during submission.

When retained MiniDoc Records are returned, they include `record_type`, `prev_state_id`, `state_id`, and the protected payload material for that accepted record.

The current read model is type-specific:

- Snapshot MiniDoc Documents **MUST** return exactly one retained record corresponding to the requested state.
- Collaborative MiniDoc Documents **MUST** return the retained ordered record span needed for the client to materialize the requested state.
- MiniDoc Logs **MUST** return the retained ordered record span needed for the client to materialize the requested transcript or log state.

The binding **MUST NOT** return only already-decoded application content in place of retained MiniDoc Record material.

## **9.7 Server role in the initial profile**

In the initial profile, the MiniDoc server acts as the authoritative coordinator for:

- client authentication,
- Channel-scoped document admission,
- ordered acceptance of MiniDoc Records,
- current-state determination,
- and retained-history retrieval.

The server-authoritative role in this profile preserves the protected-payload boundary: the server coordinates history, while Layer-1 remains authoritative for content protection and validation.

## **9.8 Server-issued document identity**

In the initial AMDP-over-HTTPS profile, the server **MUST** allocate `doc-id` values during MiniDoc creation and return the canonical MiniDoc URI to the client.

This keeps MiniDoc identity allocation consistent with the server’s authority over retained history, state progression, and Channel-scoped namespace management.

# **10. Session authentication and Channel admission**

## **10.1 Separation of concerns**

The MiniDoc protocol separates:

- **session and client authentication** — establishing who the AMDP client is in ASCP identity terms, and
- **MiniDoc admission** — establishing whether the client may access MiniDoc resources for the referenced Channel.

This mirrors ALSP’s distinction between session authentication and replication admission.

## **10.2 AMDP over HTTPS authentication model**

In the initial AMDP-over-HTTPS profile, session authentication **MUST** use standard HTTP authentication challenge and response semantics over HTTPS.

The binding **MUST** use a custom `ASCP` HTTP authentication scheme within those standard HTTP mechanisms:

- an unauthenticated request to a protected AMDP resource **MUST** receive `401 Unauthorized`,
- that response **MUST** include `WWW-Authenticate: ASCP ...`,
- the client **MUST** answer with `Authorization: ASCP ...`,
- the `ASCP` authentication material **MUST** carry the identity and certificate-backed proof material required for the server to validate ASCP protocol identity in the HTTPS binding,
- and successful authentication **MUST** establish the ASCP identity for the request and authenticated session.

This is analogous to ALSP direct authentication, but adapted to standard HTTP request and response semantics.

## **10.3 Session reuse**

After successful authentication:

- the server **MUST** issue `Set-Cookie: AMDP-Session=<opaque>; Secure; HttpOnly; SameSite=Strict; Path=/ascp/`,
- later AMDP-over-HTTPS requests **MUST** present `Cookie: AMDP-Session=<opaque>`,
- the server **MUST** reject requests with missing, invalid, expired, or invalidated session cookies using `401 Unauthorized`,
- and each such `401` response **MUST** include `WWW-Authenticate: ASCP ...` so that the client can re-establish the session.

TLS protects the transport. HTTP authentication establishes ASCP identity. The `AMDP-Session` cookie carries that authenticated session forward across later requests.

## **10.4 Channel admission using CAK and CAP**

MiniDoc access is Channel-governed and follows ASCP Channel authorization semantics.

Where a Channel requires access admission:

- the client **MUST** prove possession of the appropriate Channel Access Key (CAK) or equivalent active authorization material,
- the proof **MUST** be carried in the dedicated HTTP request header `AMDP-CAP`,
- the value of `AMDP-CAP` **MUST** be a JOSE compact serialization carrying the Channel Access Proof,
- and the server **MUST** verify that proof before serving protected MiniDoc content or accepting new MiniDoc Records for that Channel.

Authentication establishes who the client is. CAK and CAP-based admission establishes whether that client may access MiniDoc resources in the referenced Channel.

## **10.5 Channel admission caching**

In the initial AMDP-over-HTTPS profile, Channel admission **MUST** be session-cached on a per-Channel basis.

The binding **MUST** behave as follows:

- first access to a Channel within an authenticated session **MUST** either include a valid `AMDP-CAP` header or receive an admission failure,
- once the client satisfies Channel admission for that Channel, the server **MUST** record that admitted result in the `AMDP-Session`,
- later requests to the same Channel within that session **MUST** be accepted without requiring a repeated `AMDP-CAP` header,
- the server **MAY** accept redundant valid `AMDP-CAP` headers on an already admitted session,
- and the server **MUST** require renewed admission when the authenticated session expires, the server invalidates the admission state, CAK material rotates, or local policy requires renewed proof.

This mirrors ALSP’s separation between identity authentication and Channel-scoped admission while adapting it to ordinary HTTP request and response flow.

## **10.6 HTTP carriage boundary**

The HTTP carriage model for the initial AMDP-over-HTTPS profile is:

- `WWW-Authenticate: ASCP ...` carries the server’s identity-authentication challenge,
- `Authorization: ASCP ...` carries the client’s response to that challenge,
- `Set-Cookie: AMDP-Session=...` carries the authenticated session token,
- `Cookie: AMDP-Session=...` carries that session on later requests,
- `AMDP-CAP: <compact-jose>` carries Channel admission proof,
- `Location: /ascp/<channel-uuid>/<doc-id>.<suffix>` identifies a newly created MiniDoc resource,
- `state-id`, `from-state-id`, and `to-state-id` are carried as HTTP query parameters,
- and MiniDoc operation bodies are carried as `application/json`.

This profile keeps identity authentication, authenticated session continuity, and Channel admission as distinct HTTP-level mechanisms.

## **10.7 Session, admission, and operation sequence**

The AMDP-over-HTTPS sequence is:

1. Session authentication
   The client establishes an HTTPS connection and attempts the target request. If no valid `AMDP-Session` cookie is present, the server returns `401 Unauthorized` with `WWW-Authenticate: ASCP ...`. The client repeats the request with `Authorization: ASCP ...`. On successful identity validation, the server processes the request or continues to admission handling and returns `Set-Cookie: AMDP-Session=<opaque>`.
2. First Channel access
   On first access to a Channel within that authenticated session, the client presents `AMDP-CAP` for the referenced Channel. If the proof is absent or invalid, the server returns `403 Forbidden`. If the proof is valid, the server records the admitted result against the authenticated `AMDP-Session`.
3. Current-state read
   The client sends `GET /ascp/<channel-uuid>/<doc-id>.<suffix>` with `Cookie: AMDP-Session=...`. The server verifies authenticated session and admitted Channel access, resolves the current retained state boundary, and returns `200 OK` with JSON carrying the retained MiniDoc Record material.
4. Pinned-state read
   The client sends `GET /ascp/<channel-uuid>/<doc-id>.<suffix>?state-id=<state-id>` with `Cookie: AMDP-Session=...`. The server verifies authenticated session and admitted Channel access, resolves the named retained state boundary, and returns `200 OK` with JSON carrying the retained MiniDoc Record material.
5. Record submission
   For create, the client sends `POST /ascp/<channel-uuid>/` with JSON declaring `record_type` and the initial submitted record, `prev_state_id=origin`, and no `state_id`. For later mutation, the client sends `PATCH /ascp/<channel-uuid>/<doc-id>.<suffix>` with JSON carrying `base-state-id` and the submitted record or maintenance mutation. The server verifies authenticated session, admitted Channel access, target MiniDoc identity, record type consistency, and prior-state basis before accepting the mutation and issuing the resulting `state-id`. Successful create returns `201 Created`; successful mutation returns `200 OK`.
6. Span retrieval
   The client sends `GET /ascp/<channel-uuid>/<doc-id>.<suffix>?from-state-id=<state-id>[&to-state-id=<state-id>]` with `Cookie: AMDP-Session=...`. The server verifies authenticated session and admitted Channel access, resolves the retained span, and returns `200 OK` with JSON carrying the ordered retained records needed for client-side materialization.
7. Truncation or compaction
   The client sends `PATCH /ascp/<channel-uuid>/<doc-id>.<suffix>` with JSON carrying `base-state-id`, the retention cutoff, and any replacement baseline record required by the document model. The server verifies authenticated session, admitted Channel access, prior-state basis, and retention preconditions before updating the retained-history boundary and returning `200 OK` with the resulting `state-id`.

These flows define where authentication ends, where Channel admission begins, and where MiniDoc-specific history operations occur.

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

Every write in the initial profile **MUST** identify the client’s expected prior state using `base-state-id`.

The submitted MiniDoc Record also carries `prev_state_id` naming the predecessor state boundary for that retained transition. In ordinary updates, `base-state-id` and `prev_state_id` refer to the same prior accepted state. In creation, both use the reserved creation sentinel `origin`.

This provides one common sequencing rule across Snapshot MiniDoc Documents, Collaborative MiniDoc Documents, and MiniDoc Logs.

## **11.2 Snapshot MiniDoc Document progression**

For a Snapshot MiniDoc Document, the server accepts an ordered series of full-state MiniDoc Records.

Conceptually:

```text
Client submits snapshot S2 as successor to S1
Server accepts new MiniDoc Record R2
R2 becomes the current document version
```

Current read for a Snapshot MiniDoc Document returns the latest retained snapshot record. Historical read by `state-id` returns the exact retained record for that immutable state.

## **11.3 Collaborative MiniDoc Document progression**

For a Collaborative MiniDoc Document, the server accepts an ordered series of incremental collaborative MiniDoc Records.

Conceptually:

```text
Client submits collaborative update U3
Server accepts new MiniDoc Record R3
R3 advances the collaborative document history
```

The external MiniDoc Document identity, version semantics, and immutable reference model remain stable across collaborative record progression.

This remains compatible with opaque server handling because the server coordinates accepted record order, prior-state basis, retained spans, and compaction boundaries without interpreting the collaborative operation semantics inside the protected payload.

Current and historical reads for Collaborative MiniDoc Documents are span-oriented. The client requests the retained record span needed to materialize the target state using:

- `from-state-id`
- `to-state-id`

When `to-state-id` is omitted, the requested upper bound is the current head state.

## **11.4 MiniDoc Log progression**

For a MiniDoc Log, the server accepts an ordered series of append-state MiniDoc Records.

Conceptually:

```text
Client submits transcript append A3
Server accepts new MiniDoc Record R3
R3 extends the MiniDoc Log
```

The authoritative history of a MiniDoc Log is the accepted ordered record sequence, and the current state is the materialized transcript represented by that sequence.

Current and historical reads for MiniDoc Logs are likewise span-oriented. The client requests the retained record span needed to materialize the target transcript state using:

- `from-state-id`
- `to-state-id`

When `to-state-id` is omitted, the requested upper bound is the current head state.

## **11.5 Current-state semantics**

For a Snapshot MiniDoc Document, the current state is the latest accepted whole-document snapshot.

For a Collaborative MiniDoc Document, the current state is the materialized result of the accepted collaborative update history, or an equivalent retained representation consistent with that history.

For a MiniDoc Log, the current state is the materialized result of the accepted ordered transcript or log sequence, or an equivalent retained representation consistent with that sequence.

## **11.6 Retention-aware retrieval semantics**

AMDP distinguishes among:

- the current state,
- a pinned historical state selected by `state-id`,
- a retained span bounded by `from-state-id` and optional `to-state-id`,
- and the earliest retained state boundary still available at the server.

Retention changes historical availability, not the abstract MiniDoc state model.

For a Snapshot MiniDoc Document, current and pinned retrieval typically resolve to one retained record for the requested state boundary.

For a Collaborative MiniDoc Document, current and pinned retrieval resolve to the retained span or equivalent retained representation needed for the client to materialize the requested state boundary.

For a MiniDoc Log, current and pinned retrieval resolve to the retained ordered transcript span needed for the client to materialize the requested transcript state.

When historical content has been pruned, the server can still serve current and retained historical material within the retained boundary while earlier pruned state boundaries are no longer available at that server.

## **11.7 Truncation and compaction semantics**

Truncation and compaction are ordinary AMDP write operations with type-specific behavior. They update retained history boundaries as part of the protocol-visible MiniDoc lifecycle.

For all such operations:

- the request **MUST** identify the target MiniDoc,
- the request **MUST** include `base-state-id`,
- and successful acceptance yields exactly one new externally identifiable `state-id`.

### **11.7.1 MiniDoc Log truncation**

For a MiniDoc Log, truncation drops the retained prefix of the record history through a caller-specified cutoff `state-id`.

After successful truncation:

- records at or before the cutoff state are no longer retained,
- only later records remain available,
- and the earliest reconstructable retained transcript state advances to the retained post-cutoff boundary.

This means MiniDoc Log truncation advances the earliest retained transcript boundary without synthesizing a replacement baseline record.

### **11.7.2 Collaborative MiniDoc Document compaction**

For a Collaborative MiniDoc Document, truncation is a compaction operation.

The client supplies:

- the cutoff `state-id`, and
- one new baseline MiniDoc Record representing the materialized document state at that compaction boundary.

After successful compaction:

- the retained collaborative prefix through the cutoff is dropped,
- the replacement baseline record is retained,
- later collaborative records remain retained and continue to apply after that replacement baseline,
- and the compacted document remains readable without requiring the pruned collaborative prefix.

This is analogous to replacing an older edit history prefix with one retained baseline state while preserving later forward evolution.

### **11.7.3 Snapshot MiniDoc Document retention**

For a Snapshot MiniDoc Document, older retained snapshot records may be dropped without special compaction semantics because each retained record is already a self-contained document state.

## **11.8 Future collaborative evolution**

Future AMDP implementations **MAY** use OT, CRDT, or related conflict-avoiding collaborative techniques for Collaborative MiniDoc Documents.

The intended invariant is that:

- MiniDoc identity remains stable,
- MiniDoc URIs remain stable,
- Artipoint references remain stable,
- and Channel security and admission semantics remain stable.

This preserves one stable external MiniDoc model while allowing the internal collaborative coherency mechanism to evolve.

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

MiniDocs are native ASCP Channel resources. Other collaboration systems may also host mutable artifacts, such as:

- Google Docs,
- OneDrive files,
- Dropbox files,
- Slack messages,
- GitHub resources,
- and generic web URLs.

Externally governed artifacts may be referenced through other ASCP mechanisms when needed. MiniDocs provide the ASCP-native resource model for Channel-scoped mutable content.

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
- concrete endpoint and payload shapes for span reads and compaction writes,
- and migration from Snapshot to Collaborative MiniDoc Document deployment models.

## **13.1 Profile boundaries for the first RFC**

This design note is intended to provide enough protocol-shape detail for a first AMDP RFC to freeze:

- the AMDP-over-HTTPS binding,
- the HTTPS mapping of MiniDoc identity and operations,
- the authenticated-session and admitted-Channel flow,
- the exact request and response syntax for create, read, write, span, and retention operations,
- the exact MiniDoc Record metadata encoding,
- and the complete error vocabulary for interoperable implementations.

Within that framework, Snapshot MiniDoc Documents and MiniDoc Logs are ready to be specified as normative initial interoperable behaviors. Collaborative MiniDoc Documents already fit the AMDP architectural model and can either be specified in full or staged as a later interoperable profile while preserving the same core state-transition and retained-history model.

# **14. Error handling**

The eventual AMDP specification should define a complete error vocabulary. At minimum, it should distinguish the following failure classes:

- authentication failed,
- Channel admission proof missing or invalid,
- unknown Channel,
- unknown MiniDoc,
- unknown historical state,
- invalid session token or expired session,
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

Collaborative MiniDoc Documents may use OT, CRDT, or related collaborative-conflict-avoidance techniques as an implementation strategy. The AMDP security boundary remains the same: Layer-1 protects payloads, and AMDP governs transport, admission, and history semantics.

As with ALSP, TLS provides transport security for the HTTPS binding. ASCP trust additionally depends on Channel admission, Layer-1 authorship, and higher-layer governance meaning.

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

Illustrative retained-record lineage:

```text
Create submit:   record_type=.txt,     prev_state_id=origin,      state_id=<absent>
Server accepts:  record_type=.txt,     prev_state_id=origin,      state_id=bQ7mK9rTx2M
Later submit:    record_type=.txt,     prev_state_id=bQ7mK9rTx2M, state_id=<absent>
Server accepts:  record_type=.txt,     prev_state_id=bQ7mK9rTx2M, state_id=H3yLm2QvNc8
```

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

The document remains one stable MiniDoc Document while accepted records carry incremental collaborative evolution.

Illustrative retained-record lineage:

```text
R1 -> record_type=.collab, prev_state_id=origin,      state_id=S1
R2 -> record_type=.collab, prev_state_id=S1,          state_id=S2
R3 -> record_type=.collab, prev_state_id=S2,          state_id=S3
```

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

Illustrative retained-record lineage:

```text
R1 -> record_type=.log, prev_state_id=origin, state_id=L1
R2 -> record_type=.log, prev_state_id=L1,     state_id=L2
R3 -> record_type=.log, prev_state_id=L2,     state_id=L3
R4 -> record_type=.log, prev_state_id=L3,     state_id=L4
```

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
