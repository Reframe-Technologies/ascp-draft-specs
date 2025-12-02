# **ASCP LogSync Protocol (ALSP)**

**Layer 0 - ASCP Protocol Stack Logging and Synchronization Layer**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.4 — Informational (Pre-RFC Working Draft)  
November 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the ASCP specification suite and defines Layer 0 of the protocol stack: ALSP (ASCP LogSync Protocol), the transport-agnostic synchronization layer that ensures deterministic convergence of append-only logs across distributed replicas. It is published at this time to gather community feedback on the synchronization semantics, Lamport clock ordering, authentication mechanisms, and transport binding specifications.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# **2. Abstract**

The ASCP LogSync Protocol (ALSP) specifies the Layer 0 transport and replication mechanism of the Agents Shared Cognition Protocol (ASCP). ALSP defines a transport-agnostic protocol for synchronizing append-only Channel Logs that contain immutable, semantically opaque Layer 1 message envelopes. These envelopes encapsulate signed and optionally encrypted Layer 2 articulation sequences, the structured content of the ASCP cognition model. ALSP ensures deterministic, convergent ordering of these messages across replicas using Lamport clocks and a stable global tie-breaking rule.

This document defines the ALSP message formats, authentication and authorization procedures, replica behavior, and transport bindings necessary for reliable log replication in centralized, partially connected, and fully distributed environments. It normatively specifies the deterministic CBOR envelope and Dot-Preserving Binary (DPB) encoding used to transport Layer 1 JOSE compact serialization without semantic interpretation, as well as mechanisms for pull- and push-based synchronization, replay protection, and Channel Access Key–based authorization.

Together, these mechanisms provide interoperable and fault-tolerant synchronization of ASCP Channel Logs while preserving the immutability, ordering guarantees, and cryptographic integrity required by higher layers of the ASCP protocol stack.

# **3. Introduction**

The ASCP LogSync Protocol (ALSP) defines the Layer 0 transport and replication layer of the Agents Shared Cognition Protocol (ASCP). ALSP provides a transport-agnostic mechanism for synchronizing append-only Channel Logs containing immutable, semantically opaque Layer 1 message envelopes. These envelopes encapsulate Layer 2 articulation sequences but are treated by ALSP as uninterpreted binary payloads. The purpose of ALSP is to ensure that all authorized replicas converge on a bit-identical, canonically ordered view of each Channel Log regardless of network topology or connectivity conditions.

ALSP operates beneath all semantic interpretation, above the physical transport, and forms the foundation upon which the higher layers of ASCP construct structured shared cognition, secure authorship, policy enforcement, and collaboration semantics. This document defines the behavior, guarantees, and on-wire formats required for interoperable ALSP implementations.

## **3.1 Motivations Behind ALSP Design**

ALSP is motivated by the need for a deterministic, secure, and transport-independent replication substrate for ASCP’s distributed cognition model. Channels in ASCP represent persistent, append-only streams of articulated meaning; their correct operation depends on the ability of replicas to converge reliably even in environments with intermittent connectivity, partial replication, or multiple writers.

Several design goals guided the creation of ALSP:

ALSP provides **deterministic convergence** across replicas through a combination of Lamport clocks and globally stable tie-breaking rules. It supports **local-first, eventually consistent operation**, allowing replicas to operate offline and later reconcile logs without conflict. It adopts a **semantically opaque content model**: Layer 0 is not permitted to inspect, reorder, or interpret Layer 1 or Layer 2 payloads, ensuring that all semantics remain at higher layers. ALSP also enforces **strong authentication and authorization**, ensuring only valid identities and channel members may participate in replication. Finally, ALSP is **agnostic to deployment topology**, supporting centralized, peer-to-peer, and hybrid configurations with equal fidelity.

This combination of determinism, minimalism, and robust security provides a stable substrate for the higher layers of ASCP to express meaning, authorship, governance, and encrypted content without embedding transport concerns in those layers.

## **3.2 Position of ALSP in the ASCP Layer Model**

ASCP is defined as a four-layer protocol stack. ALSP occupies **Layer 0**, directly above the physical transport and below all semantic layers. Its role is to replicate and order Layer 1 message envelopes, not to interpret them.

- **Layer 0 — ALSP (This Specification):** Synchronizes append-only Channel Logs, enforces deterministic ordering, transports JOSE-formatted Layer 1 envelopes efficiently using DPB and deterministic CBOR, and manages replica-level authentication and channel-level authorization.
- **Layer 1 — Message Security Layer:** Defines the JOSE-based signatures and optional encryptions that protect the integrity, authorship, and confidentiality of Layer 2 articulations. ALSP treats all Layer 1 content as opaque.
- **Layer 2 — Articulation Layer:** Defines the structured representation of meaning (articulations) and the relationships forming the ASCP knowledge graph. Layer 0 has no access to or awareness of these structures.
- **Layer 3 — Application Layer:** Hosts agents, clients, and collaborative processes that interpret ASCP state and produce new articulated content.

This strict layering ensures that ALSP concerns itself solely with the reliable distribution and ordering of messages, while all semantic interpretation and policy enforcement occurs above Layer 0.

## **3.3 Scope of This Specification**

This document specifies the full behavioral and technical requirements for ALSP. It includes:

- The deterministic ordering model and Lamport-based logical clock semantics
- The append-only Channel Log replication model
- Replica responsibilities for deduplication, insertion, and convergence
- Authentication, authorization, and replay-protection mechanisms
- The synchronization modes (pull and push) and negotiation procedures
- The WebSocket-over-TLS transport binding and on-wire message structure
- Deterministic CBOR encoding, DPB encoding rules, and ALSP envelope format
- Error signaling, connection handling, and resilience expectations

The specification defines all normative requirements for interoperability among independently developed ALSP implementations.

## **3.4 Out of Scope**

Several components essential to complete ASCP interoperability are defined in other documents and are explicitly out of scope for ALSP. These include:

- **Semantic interpretation of messages:** ALSP does not inspect or understand Layer 1 JOSE envelopes or the Layer 2 articulations they protect.
- **Identity and trust semantics:** ALSP depends on identity certificates, claims bundles, and binding rules defined in the *ASCP Trust and Identity Architecture* specification.
- **Channel creation, governance, and membership:** Channel manifests, access-control policies, and Channel Access Key lifecycle rules are defined in the *ASCP Channels* and *ASCP Governance and Access Control* specifications.
- **Articulation grammar and structured cognition model:** The structure and semantics of articulated meaning are defined in the *ASCP Artipoint Grammar* and related Layer 2 specifications.
- **Peer discovery, routing, and topology management:** ALSP assumes an established connection between replicas and does not define overlay networks or discovery protocols.

By limiting ALSP to synchronization, ordering, and replica-level security, the specification maintains a narrow, stable contract between the transport layer and the semantic layers above it.

# **4. Terminology**

This section defines terms used normatively throughout this specification. Terms not defined here carry their customary meaning within the ASCP architecture.

### **ALSP (ASCP LogSync Protocol)**

The Layer 0 protocol of the ASCP stack responsible for transport-agnostic replication of append-only Channel Logs using deterministic ordering.

### **Articulation**

A Layer 2 semantic unit representing structured shared cognition. Articulations are conveyed to Layer 0 as opaque JOSE JWS/JWE compact sequences encapsulated within Layer 1 message envelopes.

### **ASCP (Agents Shared Cognition Protocol)**

A four-layer protocol suite enabling persistent, structured collaboration among humans and agents. ALSP defines Layer 0 of this stack.

### **Bootstrap Log**

A log containing initial Channel references, identity material, certificates, and cryptographic keys. It serves as the trust root for Channel manifests, identity binding, and authorization.

### **Canonical Order**

The total ordering over Channel Log entries defined as the tuple **(lamport\_time, message\_id)**, sorted first by Lamport clock value and then by the lexicographic ordering of the 16-byte message identifier.

### **Challenge Flow**

A server-side handshake path triggered when the receiving replica cannot fully validate the client’s identity using cached or known materials and therefore sends an auth\_challenge requesting additional identity data. Challenge Flow describes server behavior independent of the client’s credential mode.

### **Channel**

A uniquely identified, append-only stream of Layer 1 message envelopes. Channels define the unit of distribution and authorization within ASCP.

### **Channel Access Key (CAK)**

An Ed25519 key pair used to authenticate access to a Channel at Layer 0. The public key is distributed in the Channel Manifest; the private key is held by authorized participants to produce Channel Access Proofs.

### **Channel Log**

A replica’s ordered sequence of Layer 1 message envelopes for a specific Channel. The log is append-only and synchronized across replicas using ALSP.

### **Channel Manifest**

Channel metadata distributed via the bootstrap process, including Channel identifiers, access-control configuration, and the CAK public key.

### **Digest Hash Exchange**

A Channel Log consistency check in which replicas compare a SHA-256 digest of message identifiers in canonical order to detect divergence.

### **Direct Mode**

A client credential-supply mode in which the initiating replica includes all identity materials necessary for validation in its initial auth\_request. Direct Mode describes how the client packages credentials and does not imply anything about the server’s validation flow.

### **Dot-Preserving Binary (DPB)**

A reversible binary encoding that transforms JOSE compact serialization into a more efficient on-wire representation by decoding Base64url segments while preserving literal dot separators. Used by Layer 0 to transport Layer 1 payloads without semantic interpretation.

### **Hello Message**

The message exchanged after authentication to negotiate session parameters, exchange lamport\_max values, advertise capabilities, and confirm mutual authorization before synchronization.

### **Identity Key**

The long-term cryptographic key pair representing a participant (human or agent) within ASCP. Represented as a JWK and bound to an identity via signed claim bundles defined outside this specification.

### **Immediate Flow**

A server-side handshake path in which the receiving replica can validate the client’s auth\_requestimmediately based on locally available trust material, allowing it to respond directly with hello without issuing an auth\_challenge.

### **JWK (JSON Web Key)**

A JOSE structure used to encode public and symmetric keys, including Identity Keys, Bootstrap channel keys, and Channel Access Keys.

### **JWS (JSON Web Signature)**

The JOSE signature format used to authenticate all ALSP messages and to carry Channel Access Proofs.

### **JWT (JSON Web Token)**

The JOSE token format used within ASCP for identity claims and trust establishment. JWT content is not interpreted by ALSP.

### **JOSE (JavaScript Object Signing and Encryption)**

A family of standards (JWS, JWE, JWK) used by ASCP for signatures, encryption, and key representation. ALSP is aware only of compact serialization encoding and does not interpret JOSE semantics.

### **kid (Key Identifier)**

A structured identifier of the form ascp:\<type>:\<uuid> referencing key material obtained through the bootstrap process. Used in JWS and JWE protected headers.

### **Lamport Clock**

A monotonically increasing logical timestamp used by ALSP to provide deterministic ordering across distributed replicas.

### **lamport\_max**

The highest Lamport clock value known to a replica at the time a message is sent. Propagated during sync to maintain logical time coherence across replicas.

### **log\_digest**

A SHA-256 digest of the sequence of message identifiers in canonical order, used for Channel Log consistency checks.

### **message\_id**

A 16-byte universally unique identifier (UUID) assigned by ALSP to each Layer 1 message envelope for idempotence and ordering.

### **node\_id**

A universally unique identifier for a replica participating in ALSP synchronization.

### **payload**

The DPB-encoded JOSE compact serialization of a Layer 1 JWS/JWE message. ALSP treats payloads as opaque byte sequences.

### **Provisioned Mode**

A client credential-supply mode in which the initiating replica includes only minimal identity references in its initial auth\_request, supplying full credentials only if requested via auth\_challenge. Provisioned Mode concerns client behavior, not server response.

### **Pull Sync**

A synchronization mode in which replicas request Channel Log updates via sync\_request messages.

### **Push Sync**

A synchronization mode in which replicas automatically deliver new Channel Log entries via sync\_update messages after an initial synchronization.

### **Replica**

An authorized node storing and synchronizing one or more Channel Logs.

### **Session**

A mutually authenticated communication context established between two ALSP replicas for the duration of message exchange. A session begins with the ALSP authentication handshake, binds all subsequent messages through session-specific nonces and signature rules, and persists until closed, terminated, or expired.

### **session\_nonce**

A per-session, randomly generated 128-bit nonce created independently by each endpoint. During authentication, each replica includes its own `session_nonce` in the JWS protected header's `nonce` field. After authentication completes, each replica includes the peer's `session_nonce` in the `nonce` field instead. This cross-use of session nonces binds messages to a specific authenticated session and prevents replay attacks across connections.

### **sync\_request**

An ALSP message used by a replica to request Channel Log entries or verify log state.

### **sync\_response**

An ALSP message containing Channel Log entries provided in response to a sync\_request.

### **sync\_update**

An ALSP message used in push mode to deliver newly appended Channel Log entries without an explicit request.

# **5. Architectural Overview**

## **5.1 Layer Separation and Security Model**

A critical architectural principle underlies ALSP: **everything from Layer 1 is treated as completely semantically opaque to Layer 0**. ALSP knows nothing about the contents, structure, or meaning of Layer 1 payloads. While Layer 0 understands that these payloads use JOSE compact serialization format (enabling the DPB transport optimization), it remains semantically agnostic to the protected content within these structures. As such, it treats them purely as base64url payloads that need secure, identical replication across authorized replicas.

This separation serves distinct security purposes across layers:

**Layer 0 (ALSP) Security**: Protects the integrity of the replication transport itself:

- **Authenticate peers** (who is allowed to sync)
- **Authorize channel access** (which logs can this peer replicate)
- **Ensure identical replication** (same payload bytes in same order across all replicas)
- **Prevent tampering/replay** at the transport layer

**Layer 1 (ASCP Message Layer) Security**: Protects the meaning and content of articulation statements through JWS signatures, JWE encryption, keyframes, and certificate chains. While Layer 0 understands the JOSE compact serialization format for transport optimization, it remains semantically agnostic to the protected content within these structures.

This architectural separation means Layer 0 ALSP operates as a **secure replication and synchronization mechanism** that can efficiently encode and transmit Layer 1's JOSE-formatted messages while Layer 1 handles all semantic security. The two security models are complementary, not redundant: Layer 0 ensures that Layer 1's cryptographically secured articulation statements reach all authorized replicas with identical content and ordering, while Layer 1 ensures that the meaning and content itself remains cryptographically protected regardless of transport encoding optimizations.

## **5.2 Relationship to Channels and Higher ASCP Layers**

Building on this separation principle, ASCP operates as a four-layer protocol stack with ALSP serving as Layer 0:

**Layer 0 (ALSP Transport Layer)**: Handles channel log synchronization, ordering, and distribution across replicas. This layer is transport-agnostic and provides deterministic convergence using Lamport clocks, ensuring all authorized replicas eventually hold identical, canonically ordered logs per channel. Layer 0 supports selective replication—nodes only replicate channels they are authorized to participate in or monitor.

**Layer 1 (ASCP Message Layer)**: Contains the actual articulation content—signed, optionally encrypted articulation statements that represent structured shared cognition. Layer 1 sends individual articulation sequences down to Layer 0 for storage and distribution as semantically opaque payloads. ALSP understands these payloads use JOSE compact serialization format, enabling transport encoding optimizations through DPB, while delegating all semantic interpretation of the protected content to higher application layers.

## **5.3 Channel Model and Distribution Semantics (Non-Normative)**

Channels serve as the fundamental unit of both layers:

- **Channel ID**: A universally unique identifier for associated Channel which also defines the scope of both distribution and access
- **Channel Log**: A linear causal sequence of Layer 1 messages, maintained per channel with universally deterministic Layer 0 ordering across all Channels and replicas.
- **Multicast Distribution**: Channels act as secure multicast groups rather than logical partitions—authorized members receive all messages for their accessible channels

During synchronization, Layer 0 transmits received messages up to Layer 1 in clear causal order but not necessarily exact time order. The Layer 1 timestamp can be used for semantic indication of actual message order within the accuracy of each node's wall clock time.

This architecture enables ALSP to synchronize structured articulation content while remaining semantically agnostic, with each channel log transmitted independently but merged into a universally consistent timeline at the application layer.

## **5.4 Consistency Guarantees**

ALSP guarantees **eventual consistency** across authorized replicas through a hierarchical model of immutability and deterministic ordering.

**At the articulation staement level**, each articulation is globally unique and immutable with verifiable authorship and timestamps. The protocol prevents duplication or mutation once messages are replicated, ensuring message integrity across the network.

**At the log level**, each channel maintains an append-only stream where deletions and overwrites are prohibited. Messages are stored and transmitted in canonical lamport clock order using the message\_id for tie-breaking the order, providing deterministic sequencing within each channel and also globaly across logs.

**For cross-channel coordination**, while messages are transported per-channel, all valid messages received by a client are merged into a single global timeline. This unified timeline serves as the foundation for DAG construction and user-facing coordination history, where channel membership determines visibility rather than creating logical isolation between channels.

This consistency model enables multi-writer safety without conflicts, full offline write and sync capabilities are replayable, globally deterministic views for every participant in the network.

# **6. Articulation Payload Handling (Layer 1 Encapsulation)**

## **6.1 Semantically Opaque Treatment of Layer 1**

This section defines how the opaque JOSE compact serialized encodings of Articulation Sequences are represented in the wire protocol and provides normative guidance for local channel log storage as well. Layer 1 articulation sequences arrive at Layer 0 as JOSE compact serialized strings, encoded into DPB (Dot Preserving Binary) encoded payloads, which are then wrapped in a CBOR-encoded Layer 0 envelope containing ordering and identity metadata. Each channel log is usually maintained locally as a sorted list of these wrapped messages, with the same basic format typically used both on the wire and in local storage.

Layer-0 does **not** interpret payloads.

## **6.2 DPB Encoding Overview**

JOSE compact serialization represents JWS and JWE tokens as sequences of Base64url-encoded segments separated by ASCII dot characters (0x2E). To optimize transport efficiency while maintaining perfect reversibility, ALSP introduces the **ASCP Dot Preserving Binary (DPB)** format for encoding Layer 1 JOSE payloads.

DPB transforms JOSE compact serialization into a more compact binary representation by converting Base64url segments to raw binary while preserving structural dot separators. This eliminates Base64url overhead while maintaining bidirectional conversion and exact semantic boundaries.

The complete normative specification of DPB encoding rules, ULEB128 length encoding, and reversibility requirements is provided in Section 7.

## **6.3 Deterministic CBOR Layer-0 Wrapper**

This section defines the **binary** Layer-0 wrapper used to store and carry semantically opaque Layer-1 articulation sequences over ALSP. The wrapper is encoded with **CBOR** (IETF RFC 8949) using **deterministic CBOR** rules. Layer-1 payloads are **opaque bytes** containing the JOSE compact serialization encoded via **Dot-Preserving Binary (DPB)**. Layer-0 does **not** interpret payloads.

> Channel identifiers and other ALSP sync metadata are conveyed by the surrounding ALSP messages and **do not** appear in this wrapper.

### **Data Model (Single Entry)**

A Layer-0 entry wraps exactly one Layer-1 payload together with ordering and identity metadata.

### **CDDL (normative)**

```
; Integer-keyed, compact map
L0-Entry = {
  0: lamport-time,       ; uint / uint64
  1: bstr .size 16,      ; message_id (UUID, raw 16 bytes per RFC 4122)
  2: bstr                ; payload (DPB bytes; Layer-1 opaque)
}

lamport-time = uint      ; producers/consumers MUST support up to 64-bit
```

#### **Field Semantics**

- **0 — lamport\_time (uint)**
- Lamport clock tick for deterministic ordering. MUST be treated as an **unsigned 64-bit** value (even if many entries fit in 32 bits).
- **1 — message\_id (bstr .size 16)**
- Message UUID encoded as **16 raw bytes** (RFC 4122 canonical byte order). **Textual UUIDs are not permitted** in this wrapper.
- **2 — payload (bstr)**
- Opaque DPB bytes (the exact Layer-1 JOSE compact sequence encoded with DPB). Layer-0 MUST NOT transform or re-serialize this value.

### **Deterministic CBOR Requirements (normative)**

To ensure signatures and hashes are stable:

1. **Key ordering:** Map keys MUST be encoded in ascending order (0, 1, 2).
2. **Shortest form:** Integers MUST use the shortest-length CBOR major type encodings that represent the value.
3. **Byte strings:** MUST use definite-length encoding with exact length.
4. **No extras:** No duplicate keys; no semantic tags; no indefinite-length items.

### **Example (informative)**

Given:

- lamport\_time = 1\_345\_678
- message\_id = 550e8400-e29b-41d4-a716-446655440000 (raw 16 bytes)
- payload = h'65794a2e686247382e736967' (example DPB)

#### **Diagnostic notation:**

```
{
 0: 1345678,
 1: h'550e8400e29b41d4a716446655440000',
 2: h'65794a2e686247382e736967'
}
```

#### **Wire bytes (hex):**

```
A3                                  # map(3)
  00                                # key: 0
  1A 00 14 88 8E                    # uint32 1,345,678  (shortest form)
  01                                # key: 1
  50                                # bstr, length 16
     55 0E 84 00 E2 9B 41 D4
     A7 16 44 66 55 44 00 00
  02                                # key: 2
  4C                                # bstr, length 12
     65 79 4A 2E 68 62 47 38 2E 73 69 67
```

### **Error Handling (normative)**

Receivers MUST reject (and not insert) an entry if any of the following hold:

- The map is not exactly 3 keys {0,1,2} in ascending order.
- lamport\_time is not an unsigned integer or exceeds 64-bit range supported by the implementation.
- message\_id is not a 16-byte bstr.
- payload is not a definite-length bstr (zero length is permitted but discouraged).
- Any indefinite-length item or semantic tag is present.

Rejected entries MUST NOT affect Lamport counters or ordering. Implementations SHOULD surface diagnostics.

## **6.4 Canonical Order and Insertion Rules**

ALSP's ordering architecture centers on a **logical clock mechanism** - a particular type of Lamport clock that establishes a consistent, deterministic sequence for all messages across distributed replicas. Rather than relying on synchronized wall clocks or centralized coordination, ALSP uses metadata embedded in each Channel Log entry to determine where new messages should appear relative to existing ones.

- **Canonical ordering key:**(lamport\_time, message\_id) with message\_id compared as raw 16-byte lexicographic tie-breaker.
- **Idempotence:** Receivers MUST de-duplicate **per channel** by (message\_id).
- **Atomicity:** Each L0-Entry is self-contained; batch boundaries (below) have no semantics.

This ordering mechanism is fundamental to Layer 0's core responsibility: ensuring that all replicas of a Channel Log converge to identical message sequences, regardless of network conditions, arrival order, or the timing of synchronization events. The architecture supports offline operation, concurrent authorship, and partial replication while maintaining causal consistency.

The detailed specifications for how this ordering works—including clock advancement rules, insertion algorithms, and tie-breaking procedures—are provided in Section 8.

# **7. Dot-Preserving Binary (DPB) Format**

Layer 0 must transport Layer 1 JOSE payloads efficiently without interpreting their cryptographic content. The Dot-Preserving Binary (DPB) format eliminates the 33% Base64url encoding overhead present in JOSE compact serialization while maintaining perfect reversibility to the original format, ensuring that Layer 1 can parse and validate signatures over the exact byte sequences that were signed.

**ALSP implementations MUST support DPB encoding and decoding.** Layer 1 payloads transmitted over ALSP MUST be DPB-encoded; non-DPB encodings are non-conformant and SHOULD be rejected.

## **7.1 Encoding Rules**

The transformation follows these principles:

1. **Dot preservation**: Literal dot characters (0x2E) remain unchanged to preserve segment boundaries
2. **Base64url segment conversion**: Each Base64url segment is converted to a binary block consisting of:
   - An escape header (0x1F)
   - The segment length encoded in ULEB128 format
   - The raw decoded bytes from the Base64url segment
3. **Empty segment handling**: Empty segments between consecutive dots require no encoding (dots remain consecutive)
4. **Raw ASCII segments**: Raw ASCII middle segments as defined in RFC 7797 remain as literal text without binary block encoding

## **7.2 ULEB128 Encoding**

The segment length uses ULEB128 (Unsigned Little Endian Base 128), a variable-length integer encoding widely adopted in binary protocols including DWARF debugging information, WebAssembly, and Protocol Buffers. ULEB128 represents integers by breaking them into 7-bit chunks, with each byte's most significant bit serving as a continuation flag: 1 indicates more bytes follow, while 0 marks the final byte. The encoding reads little-endian, with the lowest 7 bits transmitted first.

For example, the value 1 encodes as a single byte 0x01, while 128 requires two bytes: 0x80 0x01 (the first byte's high bit indicates continuation, its low 7 bits represent zero, and the second byte contains the remaining value 1). The value 624485 encodes as 0xE5 0x8E 0x26, demonstrating the format's efficiency for larger integers.

## **7.3 Reversibility Requirements**

1. Scan for 0x1F escape sequences and literal dots
2. For each binary block: decode ULEB128 length, extract raw bytes, re-encode as Base64url
3. Reconstruct JOSE compact serialization with original dot separators
4. Result is bit-for-bit identical to original JOSE string

## **7.4 Example (Non-Normative)**

Please see Section 21.3 for a fully worked example of DPD encoding.

# **8. ALSP Session Model**

This section defines how two ALSP replicas establish a mutually authenticated session, how session-scoped nonces bind subsequent messages to that session, and how the system transitions from the authentication handshake to steady-state synchronization. All requirements in this section are normative unless explicitly marked non-normative.

A *session* is defined in Section 4.

ALSP uses a two-layer authentication model:

1. **Session authentication** — performed once per connection to establish peer identity and freshness before any Channel Log synchronization begins.
2. **Message-level authentication** — applied to every ALSP message via JWS signatures, binding each message to the authenticated session using dual session nonces and preventing replay across connections.

Session authentication itself is carried out using ALSP messages that are cryptographically authenticated. After the handshake completes, message-level authentication transitions into its post-authentication mode, ensuring all subsequent messages are explicitly tied to the negotiated session.

## **8.1 Authentication Layers in ALSP**

### **8.1.1 Session Authentication and Nonce Binding**

Session authentication establishes cryptographic binding between two replicas. Each endpoint generates a 128-bit **session\_nonce** (defined in Section 4) that uniquely identifies the local half of the session.

ALSP uses a single `nonce` field in the JWS protected header with the following rules:

- **Before authentication completes**: Each replica MUST include its **own** `session_nonce` in the `nonce` field.
- **After authentication completes**: Each replica MUST include the **peer's** `session_nonce` in the `nonce` field.

This cross-use of nonces ensures that:

1. Both endpoints prove possession of their `session_nonce` during the handshake,
2. All post-authentication messages are bound to the authenticated peer and cannot be replayed across sessions, and
3. Exactly one nonce value is present in the JWS protected header at any given time.

The `session_nonce` field in the ALSP message body MUST always contain the sender's own nonce.

### **8.1.2 Message-Level Authentication**

Message-level authentication uses JWS signatures to provide per-message integrity and authenticity.

The nonce usage rules are defined exclusively in Section 8.1 and apply identically to all message types, including authentication messages, sync messages, and operational updates.

Message-level authentication validates that:

1. the JWS signature was created by the sender’s private key;
2. the protected header includes the correct nonce per Section 8.1;
3. the message is fresh according to timestamp and replay-detection rules.

Message-level authentication does not introduce additional nonce semantics and does not modify which nonce is placed in the protected header.

## **8.2. Goals and Requirements**

The ALSP session authentication process establishes a secure, replay-protected, and identity-bound communication context between replicas before any synchronization activity begins. The following goals and requirements apply to all ALSP sessions.

**Identity Verification:**

Each peer MUST validate the identity of the other using signed credentials anchored in the ASCP Trust & Identity architecture. ALSP does not define identity semantics but requires that the resulting key material be verified before a session transitions to the AUTHENTICATED state.

**Freshness and Replay Protection:**

All authentication messages MUST include a `session_nonce` generated by the sender, and peers MUST validate that these nonces are fresh and not reused across sessions. Timestamp validation MUST be applied to authentication messages to prevent replay of stale credentials.

**Mutual Authentication:**

Both parties MUST complete the authentication handshake and confirm each other’s identity before any Channel Log operations are permitted. Authentication MUST be mutual; unilateral authentication is not allowed.

**Session Binding:**

Upon successful authentication, both peers MUST have exchanged and validated each other’s `session_nonce`. All subsequent ALSP messages MUST be bound to this session by including the peer’s session\_nonce in the JWS protected header; messages with an unexpected `nonce` MUST be rejected, ensuring that messages cannot be replayed across sessions or connections.

**Transport Independence:**

The session authentication mechanism MUST operate correctly over any transport that satisfies ALSP’s framing and ordering requirements. Session semantics are defined entirely within ALSP and MUST NOT depend on transport-specific features.

**No Session Resumption:**

ALSP sessions MUST NOT be resumed across connections. A new authentication handshake is required after any disconnect to ensure fresh nonces, updated trust validation, and to eliminate replay risk.

**Single Authoritative Session:**

At any given time, only one authenticated session MAY exist between two replicas. If a second connection attempt occurs while an authenticated session is active, the recipient MUST reject the new session or explicitly terminate the prior one before proceeding.

**Post-Authentication Requirements:**

After the session is authenticated, all subsequent ALSP messages MUST be signed using the identity key validated during the handshake. Messages failing signature validation, nonce validation, or timestamp requirements MUST be rejected with an error and connection termination.

*Non-normative note:*

These requirements ensure that ALSP does not rely on transport features for security, guarantees that all post-authentication messages are tightly bound to a specific session, and provides a clear separation between session establishment and ongoing log replication.

## **8.3. Session Handshake Overview**

The ALSP session handshake establishes mutual authentication between replicas and derives the session parameters that bind all subsequent messages. The handshake always begins with an `auth_request` sent by the initiating replica. The receiving replica then determines, based on its local trust material, whether the request can be validated immediately or whether additional information is required.

Two server-side validation flows are supported:

1. **Immediate Flow** — the server has sufficient cached or known trust materials to validate the client’s `auth_request` directly. The server responds with `hello`, and then the client sends its `hello` message to complete the handshake.
2. **Challenge Flow** — the server cannot fully validate the client’s identity based on cached material. The server responds with `auth_challenge`, requesting additional credentials. The client responds with an updated `auth_request`, after which the server validates the supplied materials and returns `hello`.

The choice of flow is **entirely server-driven** and **independent** of whether the client is operating in Direct Mode or Provisioned Mode.

In both flows:

- Each peer generates a fresh `session_nonce` and includes it in all authentication messages.
- Each peer MUST validate the peer’s identity, `session_nonce`, and `timestamp` freshness before proceeding.
- No Channel Log queries or updates may occur until the handshake completes.

The handshake concludes when each side has received a valid hello from the other, at which point the session transitions into the AUTHENTICATED state (Section 8.4). After this transition, all ALSP messages MUST use the post-authentication signature and nonce-binding rules defined in Section 8.1.

*Non-normative note:*

The handshake design allows authentication to succeed even in asymmetric trust-discovery scenarios, such as when only one peer realizes a challenge is required or when caches differ across replicas.

Narrative examples of both the Immediate Flow and Challenge Flow are provided in Appendix A (Informative).

## **8.4. Session States**

Session authentication proceeds through a well-defined sequence of states. All transitions are normative unless explicitly marked otherwise.

```
    +-------------+
    | CONNECTING  |
    +-------------+
           |
           | Send auth_request
           v
    +------------------------------+
    |     AUTH_REQUEST_SENT        |
    +------------------------------+
       |                       |
       | recv                  | recv
       | auth_challenge        | hello
       v                       v
+--------------------+   +----------------------+
| CHALLENGE_RECEIVED |   | HELLO_EXCHANGE       |
+--------------------+   +----------------------+
           |                  |
           | send hello       | send hello (if not sent)
           v                  v
        +------------------------+
        |   AUTHENTICATED        |
        +------------------------+
                    |
                    | Session binding established
                    v
        +------------------------+
        |       SYNC_READY       |
        +------------------------+
```

### **CONNECTING**

A transport-level connection has been established but no ALSP authentication messages have been exchanged.

### **AUTH\_REQUEST\_SENT**

The initiating peer has sent an `auth_request` and awaits the server’s response.

The response MUST be either `auth_challenge` or `hello`.

### **CHALLENGE\_RECEIVED**

The initiating peer has received an `auth_challenge` and MUST respond with an updated `auth_request` containing the requested identity materials.

After sending this updated `auth_request`, the client transitions back into AUTH\_REQUEST\_SENT.

### **HELLO\_EXCHANGE**

The peer has received a valid `hello` from the other side.

Both peers MUST send exactly one `hello` message per session.

If a peer has not yet sent its `hello` upon receiving the peer’s `hello`, it MUST send one immediately.

### **AUTHENTICATED**

Both peers have:

- validated the peer’s `identity`
- validated the peer’s `session_nonce`
- ensured freshness and replay protection
- exchanged `hello` messages

At this point, the session is fully authenticated.

### **SYNC\_READY**

Entering SYNC\_READY signals that:

- dual-nonce binding is in effect
- message-level authentication operates in post-authentication mode
- Channel Log synchronization messages MAY now be exchanged

### **Error Conditions**

If any of the following checks fail:

- JWS signature validation
- timestamp freshness
- nonce validation
- credential validation
- malformed authentication messages

…the recipient MUST send an error with `disconnect: true` and immediately terminate the connection.

*Non-normative note:*

This strict failure behavior ensures that misconfigured clients, replay attempts, or injection attacks cannot pollute session state or reach log synchronization.

## **8.5. Authentication Modes**

The ALSP session authentication handshake supports two client-side credential-supply modes. These modes define *how the initiating replica packages its identity materials* in the initial `auth_request`. The authentication **flow** taken by the receiving replica is independent and is determined solely by whether the server has sufficient trust material to validate the client.

## **8.5.1 Direct Mode**

In **Direct Mode**, the initiating replica includes all identity materials required for validation in its initial `auth_request`. This includes the identity certificate, any intermediate certificates, and any proof-of-possession structures required to validate the presented identity key.

A receiver **MUST NOT assume** that a Direct Mode request guarantees successful immediate validation. Even when all materials are provided, the receiver MAY still need to issue an `auth_challenge` if:

- its local trust cache is outdated,
- it lacks a necessary trust anchor, or
- it requires additional freshness proofs.

*Non-normative note:*

Direct Mode optimizes for the common case where replicas have previously exchanged or cached the relevant trust materials, enabling an Immediate Flow (Section 8.3). However, no such outcome is guaranteed.

## **8.5.2 Provisioned Mode**

In **Provisioned Mode**, the initiating replica includes only a minimal identity reference in its initial `auth_request`. The client expects the server to request additional materials if needed.

When operating in Provisioned Mode:

- The initial `auth_request` MUST advertise the identity reference(s) necessary for the server to determine whether a challenge is required.
- If the server issues an `auth_challenge`, the client MUST respond with an updated `auth_request` containing the requested credential materials.
- The client MUST NOT send partial or incremental credential updates; each response to an `auth_challenge` MUST include all materials requested in the challenge.

Provisioned Mode is functionally equivalent to Direct Mode from ALSP’s perspective; it merely defers credential presentation until requested. Provisioned Mode does **not** imply a Challenge Flow, and Direct Mode does **not** imply an Immediate Flow.

## **8.5.3 Independence of Modes and Flows**

ALSP explicitly separates *client mode* from *server flow*:

- **Modes** (Direct / Provisioned) describe how the **initiating replica** packages credentials.
- **Flows** (Immediate / Challenge) describe how the **receiving replica** validates the request.

Thus:

- A Direct Mode request MAY result in either Immediate Flow or Challenge Flow.
- A Provisioned Mode request MAY result in either Immediate Flow or Challenge Flow.

Receivers MUST determine the appropriate flow based solely on their local trust material—not the client’s chosen mode.

## **8.5.4 Initial Identity Bootstrap (Informative)**

In deployments where a replica possesses only a bootstrap identity (e.g., at first activation), its identity materials may not yet be known by peers. This may cause servers to consistently take the Challenge Flow until normal trust material circulation occurs.

Bootstrap identity establishment, credential issuance, and long-term trust establishment are defined in the **ASCP Trust & Identity Architecture** and are outside the scope of ALSP. ALSP’s only requirement is that, regardless of bootstrap state, replicas MUST complete the authentication handshake before entering the AUTHENTICATED state.

## 8.6 Transition to Authenticated Message Mode

A replica enters authenticated message mode after both peers have exchanged valid `hello` messages and verified each other's `identity`, `session_nonce`, and `timestamp` freshness. At this point, the session transitions from pre-authentication to post-authentication message rules as defined in Section 8.1.1.

### Transition point

A peer MUST NOT consider the session authenticated until it has both:

1. **sent** its own `hello`, and
2. **received** a valid `hello` from the peer.

Once both conditions are met, the session enters authenticated message mode and the nonce binding rules defined in Section 8.1.1 take effect.

### Post-authentication requirements

After authentication completes:

1. All ALSP messages MUST follow the post-authentication nonce binding rules specified in Section 8.1.1.
2. A replica MUST sign all messages using the identity key validated during the handshake.
3. A receiver MUST reject any message whose `nonce` does not match the expected value for the current authentication state (Section 8.1.1).
4. A receiver MUST reject any message signed with a different key than the key authenticated during the session.

*Non-normative note:*  
This transition model ensures that authentication is mutual and complete before any channel synchronization occurs, and that all subsequent messages carry cryptographic proof of session binding through the cross-nonce mechanism defined in Section 8.1.1.

## **8.7. Session Lifecycle and Resilience**

A session becomes active when both peers have exchanged valid hello messages and have entered the AUTHENTICATED state (Section 8.4). A session remains valid until explicitly terminated or invalidated due to error or freshness failure.

### **8.7.1 Session Termination**

A replica MUST terminate the session when:

- the transport connection is closed,
- an ALSP error is received with disconnect: true,
- a protocol violation occurs,
- authentication state becomes invalid, or
- the implementation’s inactivity timeout elapses.

ALSP sessions MUST NOT be resumed.

Any reconnection requires a full authentication handshake and new session nonces.

### **8.7.2 Conditions That Invalidate a Session**

A replica MUST terminate the session if it detects any condition defined elsewhere in this specification as invalidating authentication, including:

- invalid or stale timestamps (Section 10.4),
- incorrect or unexpected session\_nonce usage (Sections 8.1 and 8.6),
- signature validation failure (Sections 8.1 and 10.4), or
- malformed protected headers (Sections 10.1 and 10.4).

### **8.7.3 Session Reestablishment**

After termination, peers MAY immediately establish a new session by initiating a new authentication handshake. New session\_nonces MUST be generated for each new session, even if the underlying transport is reused.

*Non-normative note:*

This strict session model avoids ambiguity across reconnects, enforces replay protection, and ensures that each authenticated session represents a fresh, well-defined security context.

## **8.10 Channel Access Authorization**

Session authentication (described in this section) establishes the identity and trustworthiness of a peer replica. However, session authentication does **not** grant access to any specific Channel Logs.

Channel access is controlled through **Channel Access Keys (CAKs)**, which operate as a separate authorization layer on top of the authenticated session. A replica with a valid authenticated session must still provide a valid Channel Access Proof—a JWS signature created with the Channel Access Key—to synchronize a specific channel.

This separation ensures that:

1. Session authentication validates **who** the peer is,
2. Channel Access Proofs validate **what** the peer may access, and
3. The two authorization mechanisms remain independent and composable.

The complete specification for Channel Access Keys and Channel Access Proofs is provided in Section 11.

# **9. Global Synchronization Model**

## **9.1 Lamport Clock Model**

ALSP uses **Lamport clocks** to provide ordering across all channel logs. Each replica MUST maintain a **single, unsigned 64-bit Lamport counter** that spans all channels.

All Channel Log entries MUST be ordered using the canonical tuple:

1. `lamport_time` — unsigned integer (numeric ascending)
2. `message_id` — deterministic tie-breaker (lexicographic ordering of the **raw 16-byte value**)

This tuple defines a minimal set of metadata to create a **totally deterministic, replica‑independent ordering** across all Layer-0 wrappers without requiring synchronized wall clocks. Any two replicas possessing the same set of Layer-1 will converge on the same canonical ordering.

## **9.2 Canonical Ordering Definition**

ALSP implementations **MUST**:

- Treat `lamport_time` as an unsigned 64‑bit integer
- Compare `message_id` as a raw 16‑byte sequence
- Sort messages strictly by the tuple (`lamport_time`, `message_id`)
- Preserve this ordering for insertion, storage, synchronization, and retransmission
- Treat this ordering as canonical and immutable once a message is inserted

All replicas reading the same set of messages will converge to an identical canonical log order.

Multiple messages MAY legitimately share the same Lamport value; this does not represent a conflict. Deterministic ordering is preserved through the message\_id tie‑breaker.

## **9.3 Global vs. Per-Channel Clock Semantics**

ALSP uses a single, replica‑wide Lamport counter rather than maintaining separate counters per channel. This design enables consistent ordering across channels even when replicas:

- Possess only a subset of channels,
- Later gain access to older channels,
- Operate offline for extended periods, or
- Receive messages from peers with larger Lamport clocks.

A replica **MUST NOT** maintain separate Lamport counters per channel.

## **9.4 Lamport Max Propagation**

Replicas advertise their current logical time using the lamport\_max field in all synchronization messages. This value represents the highest logical time known to the sender.

Upon receiving any ALSP message, a replica **MUST** update its local Lamport counter to:

```
max(local_lamport,
    received_lamport_max,
    any_lamport_time_in_received_messages)
```

lamport\_max values are **monotonic advisory hints**:

- A replica **MUST NOT** reduce its Lamport counter based on a received value.
- A replica **MUST** advance its counter to at least the received `lamport_max`.
- A replica **MUST** treat `lamport_max` as a lower bound on the peer’s logical time.

Architecturally, this mechanism is **not required for correctness** but significantly improves ordering quality in offline and partially replicated environments.

## **9.5 Initialization and Persistence Requirements**

Each replica maintains a single Lamport counter that persists across restarts.

### 9.5.1 Initialization

On startup, a replica MUST initialize its Lamport counter to the maximum of:

- The persisted counter value (if available)
- The highest Lamport value among locally stored messages
- Any previously preserved lamport\_max value
- The value 0.

### 9.5.2 Local Message Creation

When creating a new message locally for any channel:

- Increment the Lamport counter by 1
- Assign the incremented value to the Layer-0 wrapper's `lamport_time`

### 9.5.3 Receiving Messages

When receiving messages or synchronization metadata, replicas MUST set their Lamport counter to the maximum of all relevant values per Section 9.4.

### 9.5.4 Offline Operation

Replicas operating offline **MUST** continue incrementing their Lamport counter for each locally created message. Offline state does not alter Lamport semantics.

### 9.5.5 Counter Overflow and Anomaly Detection

#### Local Counter Overflow

If a replica's own Lamport counter approaches the maximum 64-bit value, the replica **MUST** prevent the creation of further local messages and **MUST** terminate all active sessions with an `internal_error` and `disconnect: true`.

*Non-normative note:* Given the Lamport clock is a 64-bit value, this condition is assumed unreachable for correctly operating implementations.

#### Received Message Validation

Implementations **MAY** reject received messages whose `lamport_time` or `lamport_max` value would cause the local counter to advance by more than a configurable threshold (e.g., eliminating more than 50% of the remaining dynamic range). Such messages likely indicate corruption or malicious activity and **SHOULD** be rejected with an appropriate error response.

## **9.6 Normative Insertion & Ordering Rules**

During synchronization, replicas **MUST** insert all received messages into local logs strictly according to canonical ordering:

```
(lamport_time, message_id)
```

Messages MAY arrive out of order due to batching or network conditions. Implementations MUST reorder them before insertion.

Idempotence is required:

- Replicas MUST deduplicate by `message_id` per channel.
- Duplicate messages MUST NOT affect Lamport counters.

Once inserted, message ordering is immutable.

## **9.7 Rationale (Non-Normative)**

Wall‑clock timestamps are unsuitable for canonical ordering due to drift, skew, and offline operation. Lamport clocks provide deterministic ordering without requiring synchronized clocks.

Vector clocks offer explicit causality detection but introduce significant metadata overhead. ASCP already expresses causality at Layer 2 through DAG references, so ALSP does not require vector‑clock semantics.

Lamport clocks provide the right balance: deterministic global ordering with minimal state and excellent behavior under concurrency.

## 9.8 Additional Notes (Non‑Normative)

Lamport ordering provides **transport‑level sequencing only**. Semantic ordering—such as authorship precedence or DAG traversal—is defined entirely at Layer 2. Applications MUST use articulation‑level DAG edges, not Lamport values, to determine semantic causality or meaning relationships.

# **10. ALSP Protocol Messages**

This section defines the complete **Layer-0 ALSP message model**, including the binary envelope format, JSON message header conventions, and how channel log entries are batched for synchronization. All ALSP messages are integrity-protected by JWS as defined in Section 6 and Section 7. Transport bindings (for example, WebSocket) carry these envelopes as opaque binary units (see Section 14).

## **10.1 Message Envelope Structure**

This section specifies the on‑wire encoding for ALSP messages. It defines a compact CBOR envelope that batches Layer‑0 entries while carrying a flexible message header as a JSON blob. The entire CBOR envelope is integrity‑protected via a JWS signature forms the JWS payload. JWS must sign the entire CBOR bytes verbatim.

- **Envelope:** The ALSP envelope is encoded as **deterministic CBOR**.
- **Header:** `alsp_msg_header` is a **CBOR byte string** whose contents are **UTF‑8 JSON**. The CBOR layer treats it as opaque.
- **Payload:** `alsp_msg_payload` is an optional **CBOR array** of Layer‑0 entries for efficient batching.

This design yields a compact, canonical, and extensible envelope while preserving maximum flexibility for header fields.

### **CDDL (normative)**

```cddl
; Deterministic CBOR is required: ascending map keys, shortest
; integer encodings, definite lengths, and no semantic tags
; or indefinite-length items.

ALSP-Envelope = {
  0: tstr,            ; alsp_version (e.g., "0.1")
  1: bstr,            ; alsp_msg_header: UTF-8 JSON bytes (opaque)
  ?2: [* L0-Entry]    ; alsp_msg_payload: batch of Layer-0 entries
}

; L0-Entry is defined elsewhere and remains CBOR (not repeated here).
; It includes the lamport time, message_id (UUID bytes), and DPB-encoded Layer-1 JOSE payload.

```

### **Deterministic CBOR requirements (normative)**

1. Map keys **MUST** be encoded in ascending order (for ALSP-Envelope: 0, 1, 2).
2. Integers **MUST** use the shortest valid encodings.
3. Arrays and byte strings **MUST** use **definite lengths**.
4. Implementations **MUST NOT** use semantic tags or indefinite-length items in the envelope.

These requirements ensure that all implementations produce comparable encodings for the same logical envelope, which is helpful for stable hashing, signatures, and comparison between implementations.

### **alsp\_msg\_header (JSON-in-bytes)**

- The value of `alsp_msg_header` **MUST** be a valid **UTF‑8** encoded **JSON** document (RFC 8259). I‑JSON (RFC 7493) is RECOMMENDED.
- To promote stable signatures and easy comparisons between implementations, producers **SHOULD** serialize this JSON in **canonical form** (e.g., JCS / RFC 8785).
- Receivers **MUST** treat the header as an **opaque byte string** at the CBOR layer and **MUST NOT** re‑serialize it when verifying signatures.
- The schema of the JSON header is intentionally flexible but MUST NOT be reserialized when verfied. It is the entire CBOR envelope that MUST be signed as sent and verified as received. 

The JSON schema for `alsp_msg_header` is defined in the remainder of this section 10 for each concrete ALSP message type (e.g., hello, sync\_request, sync\_response, sync\_update, error).

### **Example in CBOR diagnostic notation**

This example is informative only. The normative encoding is defined by the CDDL and deterministic CBOR rules above.

```cbor-diag
{
  0: "0.1",
  1: h'7B22616C73705F6D73675F74797065223A2273796E635F726573706F6E7
       365222C2274696D657374616D70223A22323032352D30392D3035543132
       3A33343A35365A222C226C616D706F72745F6D6178223A3136353639393
       0392C226D6F7265223A66616C73657D',
  2: [
    { 0: 1345678,
      1: h'550e8400e29b41d4a716446655440000',
      2: h'...DPB bytes...' }
  ]
}

# Field-by-field breakdown
# a3                          : map(3)
#   00                        : key 0 (uint 0)
#     63 30 2e 31             : tstr len=3  → "0.1"
#   01                        : key 1 (uint 1)
#     58 68                   : bstr len=0x68=104 (header JSON)
#       7b .. 7d              : 104 bytes UTF-8 JSON
#   02                        : key 2 (uint 2)
#     81                      : array(1)
#       a3                    : map(3) — L0 entry
#         00                  : key 0 (uint 0)
#           1a 00 14 88 8e    : uint(1345678)
#         01                  : key 1 (uint 1)
#           50                : bstr len=16 (UUID)
#             55 0e 84 00 e2 9b 41 d4 a7 16 44 66 55 44 00 00
#                             : 16 bytes UUID
#         02                  : key 2 (uint 2)
#           44                : bstr len=4 (DPB bytes)
#             de ad be ef     : example DPB payload bytes
```

All multi-byte integers in CBOR use network byte order (big-endian) as per RFC 8949. The value at key 1 above is a **bstr** whose hex decodes to this JSON (UTF‑8):

```json
{
  "alsp_msg_type":"sync_response",
  "timestamp":"2025-09-05T12:34:56Z",
  "lamport_max":16569909,
  "more":false
}
```

## **10.2 Authentication Messages**

Authentication messages establish a mutually authenticated ALSP session prior to any Channel Log synchronization. Authentication semantics are defined in **Section 8**; this section defines the **message-specific JSON fields** that MUST appear in the alsp\_msg\_header for each authentication message type.

All authentication messages:

- MUST use the ALSP Envelope format defined in Section 10.1.
- MUST include a session\_nonce in the **message body** per Section 8.
- MUST include a nonce field in the **JWS protected header** per Section 8.1.
- MUST reference identity and key material in accordance with **ASCP Trust & Identity Architecture** .

Authentication messages do **not** grant channel access; they only establish the authenticated session.

### **10.2.1 Auth Request Message**

```json
{
  "alsp_msg_type": "auth_request",     // Explicit ALSP Auth Request
  "auth_mode": "direct" | "provisioned",
  "timestamp": "<timestamp>",          // RFC 3339 UTC
  "session_nonce": "utf-8-string",     // Sender's session nonce
  "identity_cert": "<JWK+JWS>",        // Signed public key package
  "recovery_cert": "<JWK+JWS>",        // Signed recovery key package
  "user_identity": "utf-8-string",     // Email address or URN
  "node_id": "uuid"                    // Sender's replica node UUID
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "auth\_request".
  - Identifies this header as an ALSP authentication request.
- **auth\_mode**
  - Type: string ("direct" or "provisioned")
  - **MUST** be present.
  - **MUST** indicate the credential-supply mode as defined in the ALSP authentication section (Direct vs Provisioned).
  - Implementations **MUST** interpret this field consistently with the **ASCP Trust & Identity Architecture** for how identity and key material are supplied and validated.
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the time at which the message was generated.
  - Receivers **MUST** apply freshness and replay checks as specified in the ALSP authentication section (Section 8.1).
- **session\_nonce**
  - Type: string (UTF-8, fixed length as specified in Section 8)
  - **MUST** be present.
  - **MUST** be freshly generated for this session by the sender.
  - **MUST** be used in the JWS protected header for auth\_request messages according to the nonce rules defined in Section 8.1.
  - Receivers **MUST** treat this value as the sender’s session nonce for the remainder of the handshake.
- **identity\_cert**
  - Type: string (JWS-encoded JWK package)
  - In **Direct Mode**, this field **MUST** be present and **MUST** contain the sender’s identity certificate as defined in the **ASCP Trust & Identity Architecture**.
  - In **Provisioned Mode**, this field **MAY** be omitted if the identity will be provisioned via client\_key\_env in a subsequent flow, as defined in the Trust & Identity Architecture.
  - ALSP **MUST NOT** interpret the contents beyond ensuring it is well-formed per the Trust & Identity specification; semantic validation is delegated to that specification.
- **recovery\_cert**
  - Type: string (JWS-encoded JWK package)
  - In **Provisioned Mode**, this field **MUST** be present and **MUST** contain the recovery key used to encrypt the client’s user-key-envelope, as defined by the Trust & Identity Architecture.
  - In **Direct Mode**, this field **MAY** be omitted.
  - ALSP treats this field as opaque and only requires consistency with subsequent auth\_challenge messages that reference it.
- **user\_identity**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **MUST** identify the user or agent associated with this session (for example an email address or URN) using the identity formats defined in the Trust & Identity Architecture.
  - **MUST** match, or be resolvable to, the identity bound to identity\_cert or the identity resulting from the provisioned identity flow.
- **node\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** uniquely identify the sender’s replica node within the ALSP deployment.
  - Implementations **MUST** use this identifier in connection- and session-management logic (for example, enforcing single active session per pair of replicas).

### **10.2.2 Auth Challenge Message**

```json
{
  "alsp_msg_type": "auth_challenge",     // Explicit ALSP Auth Challenge
  "timestamp": "<timestamp>",            // RFC 3339 UTC
  "session_nonce": "utf-8-string",       // Challenger's session nonce
  "client_key_env": "<JWE-encoded UKE>", // JWE of user-key-envelope
  "identity_cert": "<JWK+JWS>",          // Challenger's public key package
  "user_identity": "utf-8-string",       // Challenger’s identity
  "node_id": "uuid"                      // Challenger’s replica node UUID
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "auth\_challenge".
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the generation time.
  - Receivers **MUST** apply freshness and replay validation as specified for authentication messages.
- **session\_nonce**
  - Type: string (UTF-8, fixed length as specified in Section 8)
  - **MUST** be present.
  - **MUST** be freshly generated by the challenger for this session.
  - **MUST** be treated as the challenger’s session nonce and used for subsequent JWS nonce values according to the rules in Section 8.1.
  - This value serves as the challenge nonce for identity and key-binding proof, as defined in the Trust & Identity Architecture.
- **client\_key\_env**
  - Type: string (JWE compact serialization of the user-key-envelope)
  - In **Provisioned Mode**, **MUST** be present and **MUST** be a JWE encrypted with the recovery\_cert supplied in the corresponding auth\_request, as defined in the Trust & Identity Architecture.
  - In **Direct Mode**, **MAY** be omitted and **MUST NOT** be interpreted if present in violation of the mode.
  - ALSP treats this field as an opaque JWE; structure and validation are governed exclusively by the Trust & Identity Architecture.
- **identity\_cert**
  - Type: string (JWS-encoded JWK package)
  - **MUST** be present.
  - **MUST** contain the challenger’s own session identity certificate as defined in the Trust & Identity Architecture.
  - **MUST** be consistent with the kid used in the JWS protected header for this message.
- **user\_identity**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **MUST** identify the challenger’s user or agent identity as recorded in the ASCP bootstrap / trust material.
  - **MUST** be consistent with the identity bound to identity\_cert per the Trust & Identity Architecture.
- **node\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** identify the challenger’s replica node.
  - Receivers **MUST** apply the same uniqueness and single-session rules as for auth\_request.

### **10.2.3 Hello Message**

```json
{
  "alsp_msg_type": "hello",            // ALSP hello message
  "timestamp": "<timestamp>",          // RFC 3339 UTC
  "session_nonce": "utf-8-string",     // Sender's session nonce
  "boot_keys_jwe": "<JWE of BCK>",     // JWE-encoded bootstrap channel keys
  "lamport_max": 16569909,             // Local Lamport global max value
  "node_id": "uuid",                   // Sender's replica node UUID
  "push_enabled": true,                // Request/confirm push mode
  "node_description": "utf-8-string",  // Friendly node description
  "max_alsp_length": 262144,           // Maximum receive length in bytes
  "user_auth_cert": "cert-or-kid",     // Identity token package or certificate reference
  "user_identity": "utf-8-string",     // User / agent identity
  "status_message": "utf-8-string"     // Optional status or error information
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "hello".
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the generation time.
  - Receivers **MUST** apply the same freshness and replay checks as for other authenticated messages.
- **session\_nonce**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **MUST** equal the sender’s session nonce used during the authentication handshake (see Section 8).
  - After both peers have exchanged hello messages, all subsequent ALSP messages **MUST** use the peer’s session\_nonce in the JWS protected header as specified in Section 8.1.
- **boot\_keys\_jwe**
  - Type: string (JWE compact serialization)
  - **MAY** be present.
  - When present, **MUST** contain the Bootstrap Channel key array as specified in the ALSP bootstrap key distribution section and the ASCP Trust & Identity Architecture.
  - If omitted, the receiver **MUST NOT** assume encrypted bootstrap content is available until keys are obtained via another mechanism.
- **lamport\_max**
  - Type: integer (unsigned)
  - **MUST** be present.
  - **MUST** represent the sender’s current global Lamport maximum, as defined in the Lamport ordering section.
  - Receivers **MUST** use this value when updating local Lamport counters according to the rules in Section 9.
- **node\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** uniquely identify the sending replica.
  - Peers **MUST** enforce single active authenticated session per (local\_node\_id, remote\_node\_id) pair.
- **push\_enabled**
  - Type: boolean
  - **MAY** be present.
  - When present in an initiating hello, it **MUST** indicate the sender’s request to enable push mode for this session.
  - When present in a responding hello, it **MUST** indicate whether push mode is accepted (true) or declined (false) for this session, as specified in the sync mode negotiation rules.
  - If omitted, receivers **MUST** treat it as false (pull-only mode).
- **node\_description**
  - Type: string (UTF-8)
  - **MAY** be present.
  - When present, **MUST** be treated as informational metadata only (for logging, diagnostics, or UI); it has no protocol semantics.
- **max\_alsp\_length**
  - Type: integer (unsigned 32-bit)
  - **MAY** be present.
  - When present, **MUST** indicate the maximum ALSP envelope size in bytes that the sender is willing to receive on this session.
  - Implementations **MUST** support at least 32 KiB and **SHOULD** support at least 128 KiB.
  - If omitted, peers **MUST** assume a default maximum of 131072 bytes (128 KiB).
  - Senders **MUST NOT** exceed the peer’s advertised limit.
- **user\_auth\_cert**
  - Type: string
  - **MUST** be present.
  - In challenge flows, **MUST** contain the Identity Token Package or equivalent credential specified by the **ASCP Trust & Identity Architecture** to bind user\_identity to key material.
  - In other flows, this **MAY** *instead* be a kid referencing the certificate used for ongoing message authentication, as defined in the ASCP Trust & Identity Architecture and key-identifier resolution rules covered in Section 13.
  - Receivers **MUST** use this field (directly or via kid resolution) to validate the authenticated identity for the session.
- **user\_identity**
  - Type: string (UTF-8)
  - **MAY** be present.
  - When present, **MUST** identify the user or agent associated with this node and **MUST** be consistent with the identity conveyed or referenced by user\_auth\_cert.
  - If omitted, receivers **MUST** derive the effective identity from user\_auth\_cert per the Trust & Identity Architecture.
- **status\_message**
  - Type: string (UTF-8)
  - **MAY** be present.
  - When present, **MUST** be treated as informational or diagnostic text.
  - If the sender is rejecting or degrading capabilities for the session (for example, refusing push mode or indicating a constraint), it **SHOULD** set status\_message to describe the reason and **SHOULD** follow with an ALSP error message if the session cannot proceed.

## **10.3 Synchronization Phase**

### **10.3.1 Sync Request Message**

```json
{
  "alsp_msg_type": "sync_request",     // Explicit ALSP poll / subscription
  "credentials": "eyJaiJFZERTQSJ9...", // Channel access proof (JWS)
  "timestamp": "<timestamp>",          // RFC 3339 UTC
  "lamport_max": 16569909,            // Local Lamport global max value
  "from_lamport": 10500,              // Lower bound (inclusive)
  "to_lamport": 11000,                // Upper bound (inclusive, optional)
  "node_id": "uuid",                  // Requesting replica node UUID
  "channel_id": "uuid",               // Channel requested
  "log_digest": "sha256:abcd..."      // Optional log digest up to from_lamport
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "sync\_request".
- **credentials**
  - Type: string (JWS compact serialization)
  - **MUST** be present.
  - **MUST** contain a channel access proof whose payload and signing semantics are defined in the **ASCP Trust & Identity Architecture** (Channel Access Key / CAK).
  - The JWS payload **MUST** at minimum contain the channel\_id, a freshness timestamp, and the peer session nonce as required by the Trust & Identity Architecture.
  - Receivers **MUST** validate this JWS before processing the request.
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the message generation time and is subject to standard replay-protection checks.
- **lamport\_max**
  - Type: integer
  - **MUST** be present.
  - **MUST** represent the requester’s current global Lamport maximum.
  - Receivers **SHOULD** use this to reconcile Lamport counters following the rules in Section 9.
- **from\_lamport**
  - Type: integer
  - **MUST** be present.
  - **MUST** specify the inclusive lower bound of the requested Lamport range for the given channel.
  - **MUST NOT** exceed to\_lamport when both present
  - Receivers **MUST** use this value when selecting which log entries to include in one or more sync\_response messages.
- **to\_lamport**
  - Type: integer
  - **MAY** be present.
  - When present, **MUST** specify the inclusive upper bound of the requested Lamport range.
  - When omitted, receivers **MUST** interpret the request as “from from\_lamport to the end of the channel log”.
- **node\_id**
  - Type: string (UUID)
  - **MAY** be present.
  - When present, receivers **MUST** use this to exclude messages originated by the requesting node from the response (to avoid redundant transmission).
  - When omitted, receivers **MUST** treat the request as asking for all matching messages, regardless of origin.
- **channel\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** identify the specific channel whose log is requested.
  - When used with push mode, **MUST** also be treated as a subscription request for this channel for the remainder of the session.
- **log\_digest**
  - Type: string ("sha256:\<hex>")
  - **MAY** be present.
  - When present, **MUST** be computed as the SHA-256 digest over the concatenation of UTF-8 message\_id values for all Layer-0 entries in the channel log strictly before from\_lamport, in canonical log order. See Section 17 for detailed semantics.
  - Receivers **MAY** use this digest for log health checking and divergence detection per Section 17.

### **10.3.2 Sync Response**

```json
{
  "alsp_msg_type": "sync_response", // Response(s) to a sync_request
  "timestamp": "<timestamp>",       // RFC 3339 UTC
  "lamport_max": 16569909,          // Responder's Lamport global max
  "channel_id": "uuid",             // Channel being supplied
  "log_digest": "sha256:abcd..."    // Optional log digest up to from_lamport
  "more": false                     // Additional responses to follow?
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "sync\_response".
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the generation time and is subject to replay checks.
- **lamport\_max**
  - Type: integer
  - **MUST** be present.
  - **MUST** represent the responder’s current global Lamport maximum.
  - Receivers **MUST** process this value using the same Lamport counter rules as other messages (Section 9).
- **channel\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** identify the channel whose entries are carried in the alsp\_msg\_payload.
  - A single sync\_response **MUST NOT** include entries from multiple channels.
- **log\_digest**
  - Type: string ("sha256:\<hex>")
  - **MUST** be present if the most recent triggering "sync\_request" message included its own log\_digest field.
  - When present, **MUST** be computed as the SHA-256 digest over the concatenation of UTF-8 message\_id values for all Layer-0 entries in the channel log strictly up to and including the Layer-0 entries included in this message, in canonical log order. See Section 17 for detailed semantics.
  - Receivers **MAY** use this digest for log health checking and divergence detection per Section 17.
- **more**
  - Type: boolean
  - **MUST** be present.
  - When true, indicates that additional sync\_response messages for the same (session, channel\_id) and requested range will follow.
  - When false, indicates that the responder has completed sending all entries matching the corresponding sync\_request.
  - Whether true or false, it is valid for a Sync Response message to have an empty alsp\_msg\_payload.
  - Receivers **MUST NOT** assume completion of a request until they have observed a sync\_response with more: false (or a terminal error).

### **10.3.3 Sync Update (Push Mode)**

```json
{
  "alsp_msg_type": "sync_update",  // Asynchronous live update
  "timestamp": "<timestamp>",      // RFC 3339 UTC
  "lamport_max": 16569909,         // Sender's Lamport global max value
  "channel_id": "uuid"             // Channel being supplied (optional)
  "log_digest": "sha256:abcd..."   // Optional log digest up to from_lamport
}
```

**Field requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "sync\_update".
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the generation time and participates in replay protection.
- **lamport\_max**
  - Type: integer
  - **MUST** be present.
  - **MUST** represent the sender’s current global Lamport maximum.
  - Receivers **MUST** use this value when updating local Lamport counters and **SHOULD** enforce any safety thresholds defined for large counter jumps.
- **channel\_id**
  - Type: string (UUID)
  - **MUST** be present when the alsp\_msg\_payload contains channel log entries
  - When present, **MUST** identify the channel whose new entries are included in alsp\_msg\_payload for this update.
  - When omitted, the message **MUST NOT** carry a payload of channel log entries and **MUST** be interpreted as a Lamport-only update (for example, broadcasting an updated lamport\_max).
  - Senders **MUST** send separate sync\_update messages per channel and **MUST NOT** mix entries from multiple channels in a single update.
- **log\_digest**
  - Type: string ("sha256:\<hex>")
  - **MUST** always be present
  - **MUST** be computed as the SHA-256 digest over the concatenation of UTF-8 message\_id values for all Layer-0 entries in the channel log strictly up to and including the Layer-0 entries included in this message, in canonical log order. See Section 17 for detailed semantics.
  - Receivers **MUST** use this digest for log health checking and divergence detection per Section 17.

## **10.4 Message Authentication (JWS Layer)**

All ALSP messages are secured using JWS compact serialization, where the complete message structure (including headers and payload array) is cryptographically signed and encoded into the ALSP Dot Preserving Binart (DPB).

After Session authentication completes, every message's JWS protected header must carry the peer's session nonce. All messages are signed with the sender's private key and verified using their registered public key, providing cryptographic proof of identity and session binding.

Each ALSP session begins with authentication using one of two flows: Auth\_Request/Hello exchange for known peers, or Auth\_Request/Auth\_Challenge/Hello sequence for new identity binding. During establishment, both peers generate unique 32-character hexadecimal `session_nonce` values that provide replay protection for the entire session.

### **10.4.1 JWS Header Requirements**

All ALSP messages are integrity-protected using JWS.

The **JWS Protected Header** MUST be a JSON object with the fields defined below and MUST be encoded using UTF-8 prior to BASE64URL encoding as specified in RFC 7515.

#### **Protected Header:**

```clike
{
  "alg": "ES256",                // Signing algorithm
  "kid": "ascp:cert:<uuid>",     // Identity or session key reference
  "typ": "alsp",                 // Identifies ALSP message
  "nonce": "<nonce>"             // Nonce per Section 10
}
```

### **Field Requirements**

**alg:**

- **Type:** string
- **MUST** be present.
- **MUST** indicate the JWS algorithm used to sign the ALSP message.
- For this version of ALSP, **MUST** be "ES256".
- Future ALSP versions MAY allow additional algorithms only if explicitly registered in the ALSP IANA registry.

**kid:**

- **Type:** string
- **MUST** be present on all ALSP messages
- When constructing the initial auth\_request in Provisioned Mode, the value must be an empty string other the kid **MUST** uniquely reference the signing key (identity key) as defined by Section 13 and the **ASCP Trust & Identity Architecture**.
- The receiver **MUST** use the kid value to resolve the appropriate public key for signature verification.
- The sender **MUST** ensure that the referenced key is the same key whose private component generated the JWS signature.

**typ:**

- **Type:** string
- **MUST** be present.
- **MUST** equal "alsp".
- Identifies the JWS as carrying an ALSP message and prevents cross-protocol substitution attacks.

**nonce:**

- **Type:** string (UTF-8)
- **MUST** be present with contents as specified in Section 8.1
- Receivers **MUST** reject any message whose protected header contains an unexpected nonce to ensures session binding, replay protection, and cross-session separation.

#### **JWS Payload:**

The JWS payload contains the complete ALSP envelope encoded as deterministic CBOR bytes, as defined in the Transport Bindings section. This envelope includes the ALSP version, message header (containing alsp\_msg\_type, timestamp, channel\_id, lamport\_max, etc. as UTF-8 JSON bytes), and optional payload array of Layer-0 wrapped Layer-1 articulation statement entries. This ensures cryptographic integrity protection for both routing metadata and articulation content at the transport layer. The `timestamp` field within the ALSP message header must be populated with the current time in RFC 3339 UTC format at message generation for replay protection.

#### **Wire Format:**

```clike
<DPB_encoded_JWS_bytes>
```

Where the DPB-encoded bytes represent the JWS compact serialization: `<JWSprotectedHeader>.<JWSpayload>.<JWSsignature>`

### **10.4.2 Serialization and Encoding Requirements**

#### **Canonical JSON**

- The Protected Header JSON **SHOULD** be serialized using **RFC 8785 JCS canonical JSON** before BASE64URL encoding.

#### **Encoding**

- After JSON serialization, the Protected Header **MUST** be UTF-8 encoded, then BASE64URL encoded without padding, as defined in RFC 7515.

### **10.4.3 Verification Rules**

Receivers **MUST** validate, in order:

1. BASE64URL decoding
2. UTF-8 validity
3. JSON object structure
4. Presence and correctness of required fields
5. Algorithm and key resolution
6. Nonce correctness
7. Signature correctness over the CBOR envelope bytes

Messages failing any requirement **MUST** be rejected with an ALSP error message (Section 16).

### **10.4.6 Error Cases**

Use the standard ALSP error message. For more guidance please reference to Section 15 on Error handling.

- `invalid_auth`: Bad signature, unknown kid, or cert mismatch.
- `unauthorized`: Token or CAK proof rejected.
- `stale_timestamp`: Outside allowed skew window.
- `protocol_violation`: Wrong typ/nonce usage or message order.
- `re-auth-required`: Force a full restart of the auth sequence.

# **11. Channel Access Authorization**

This section defines the authorization mechanism by which a client proves it is permitted to synchronize a specific ASCP Channel Log. Channel access authorization is distinct from session authentication. Session authentication (Section 8) establishes *who* the peer is; channel authorization establishes *what* that peer may access.

Channel access is enforced through a **Channel Access Key (CAK)**—a per-channel Ed25519 keypair defined at channel creation time. A client that wishes to replicate a channel protected by a CAK MUST present a valid **Channel Access Proof** (CAP). The CAP is a JWS object signed using the CAK private key and verified using the corresponding public key distributed through the ASCP bootstrap process.

If a channel is configured with a CAK, the server MUST verify a CAP before providing any replication content for that channel.

Non-normative note: This authorization mechanism permits Layer 0 to validate access without inspecting Layer-1 encrypted payloads, preserving end-to-end confidentiality while enforcing strict channel-level access control.

## **11.1 Channel Access Keys (CAKs)**

A **Channel Access Key (CAK)** is an Ed25519 keypair provisioned at channel creation time. Each channel has at most one active CAK at any moment. The CAK enables clients to prove authorization to access the channel’s log.

### **11.1.1 Key Structure and Distribution**

- A CAK consists of:
  - **Public Key (pk):** Distributed via the ASCP bootstrap process.
  - **Private Key (sk):** Distributed only to authorized channel participants via secure, out-of-band mechanisms.
- The CAK public key:
  - **MUST** be referenced by a stable kid value of the form: `ascp:cak:<channel_uuid>`
  - **MUST** be present in the channel’s bootstrap manifest.
  - **MUST** be integrity-protected by a signature from the channel creator or governing authority (per bootstrap specification).
- The CAK private key:
  - **MUST** be kept confidential.
  - **MUST NOT** be transmitted within ALSP or other ASCP protocol layers.
  - **MUST** be accessible only to participants authorized to access the channel log.

### **11.1.2 Rotation**

A CAK MAY be rotated. When rotated:

- A new CAK public key **MUST** be published in an updated bootstrap manifest.
- The previous CAK MAY remain valid for a transition period, but servers **MUST** accept only CAKs explicitly listed as active.
- Servers **MUST** reject CAPs referencing a kid not listed in the current or transitional manifest set.

Non-normative note: CAK rotation is typically triggered by membership changes, compromise, or organizational policy.

## **11.2 Channel Access Proof**

A **Channel Access Proof (CAP)** is a detached JWS object attached to a sync\_request to prove that the client holds the CAK private key for the requested channel.

A client **MUST** include a CAP via the `credentials` field of every sync\_request for any channel with an active CAK. A server **MUST** reject any request for such a channel that omits, corrupts, or invalidates its CAP.

### **11.2.1 CAP Structure**

A CAP uses JWS Compact Serialization:

```json
<JWSprotectedHeader>.<payload>.<JWSsignature>
```

#### **JWS Protected Header**

The protected header **MUST** contain:

```json
{
  "alg": "EdDSA",
  "kid": "ascp:cak:<channel_uuid>",
  "typ": "alsp+cak"
}
```

- `alg` **MUST** be "EdDSA" (Ed25519).
- `kid` **MUST** match a CAK public key published via bootstrap channel manifest
- `typ` **MUST** be "alsp+cak".

The protected header **SHOULD** be serialized using RFC 8785 Canonical JSON before BASE64URL encoding.

#### **JWS Payload**

The payload **MUST** be a JSON object containing:

```json
{
  "channel_id": "550e8400-e29b-41d4-a716-446655440002",
  "nonce": "abcd1234567890abcdef1234567890ab",
  "timestamp": "2024-01-15T10:30:45.123Z"
}
```

- `channel_id` **MUST** equal the channel\_id field of the associated sync\_request.
- `nonce` **MUST** be as specified by Section 8.1 for authenticated sessions.
- `timestamp` **MUST** follow RFC 3339 UTC format with mandatory ‘Z’ suffix.
- Servers **MUST** reject CAPs with timestamps outside the configured validity window specified by Section 19.1 Replay Protection.

#### **Signature Requirements**

The signature **MUST** be computed over the standard JWS signing input using the CAK private key.

### **11.2.2 Attachment to sync\_request**

The CAP MUST be included in the credentials field of a sync\_request, e.g.:

```asciidoc
"credentials": "<protected>.<payload>.<signature>"
```

If the credentials field is present when a channel does *not* require a CAK, servers **MAY** ignore it.

## **11.3 Server Verification Algorithm**

## **11.3 Server Verification Algorithm**

Upon receiving a sync\_request for a channel with an active CAK, the server **MUST** validate the CAP using the following procedure:

1. **Parse JWS:** The server **MUST** split the CAP into protected header, payload, and signature fields. Invalid formatting **MUST** cause the server to reject the request.
2. **Validate Protected Header:** The server **MUST** ensure:
   - alg is "EdDSA".
   - typ is "alsp+cak".
   - kid corresponds to a known CAK public key for the requested channel.
3. **Resolve Public Key:** The kid value **MUST** be resolved using the current bootstrap manifest. Failure to resolve **MUST** result in an unauthorized error.
4. **Verify Signature:** The server **MUST** verify the JWS signature using the resolved CAK public key. Signature failures **MUST** result in invalid\_auth.
5. **Validate Payload Fields:** The server **MUST** ensure:
   - `channel_id` matches the sync\_request.channel\_id.
   - `nonce` matches the expected client-side session nonce for this request.
   - `timestamp` is within the server’s acceptable time window.
6. **Authorize or Reject:**
   - If all validations succeed, the server **MUST** authorize the channel sync.
   - On failure, the server **MUST** reply with an ALSP error message with:
     - error\_code: "unauthorized" or "invalid\_auth"
     - disconnect: true if the failure invalidates the session
     - (e.g., signature mismatch, malformed JWS)

### **Non-Normative Notes**

- This authorization scheme ensures that Layer 0 enforces channel access without knowledge of Layer-1 payload encryption.
- The use of a detached JWS avoids exposing CAK signatures to unnecessary contexts and maintains integrity across transports.
- CAP nonce reuse or timestamp replay indicates potential attack activity and should be logged for diagnostic purposes.

# **12. Bootstrap Channel Key Handling**

This section defines how ALSP replicas obtain and manage the cryptographic keys required to decrypt bootstrap material. The bootstrap process distributes a set of **Bootstrap Channel Keys (BCKs)**, which are used to unwrap (decrypt) protected channel manifests, trust roots, and other initialization artifacts necessary to authenticate and authorize participation in ASCP Channels.

Bootstrap keys are provided outside the ALSP channel log synchronization protocol itself and are consumed only during bootstrap or manifest refresh operations. Bootstrap keys do not grant channel access; they are distinct from Channel Access Keys (CAKs) defined in Section 11.

Non-normative note: Bootstrap keys form the root of trust for discovering channel metadata and identities. Because bootstrap artifacts may contain sensitive keying material (e.g., channel encryption keys), their confidentiality must be preserved, and key lifecycle must be well-defined.

## **12.1 Bootstrap Key Format**

Each Bootstrap Channel Key (BCK) is distributed as a JWE-encrypted envelope containing one or more bootstrap artifacts. A BCK envelope conforms to the standard JWE Compact Serialization:

```asciidoc
<protected-header>.<encrypted-key>.<iv>.<ciphertext>.<tag>
```

### **12.1.1 Protected Header Requirements**

The JWE protected header **MUST** include:

```json
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "kid": "<alsp:bck:index>"
}
```

Normative requirements:

- `alg` **MUST** be "ECDH-ES+A256KW".
- `enc` **MUST** be "A256GCM".
- `kid` **MUST** uniquely identify the bootstrap key used to encrypt the payload using the `<alsp:bck:index>` encoding.

### **12.1.2 Payload Requirements**

The JWE ciphertext **MUST** contain a JSON object representing a bootstrap artifact or array of artifacts. The structure of these artifacts is defined in the ASCP Bootstrap specification and is outside the scope of ALSP.

A replica **MUST** be able to decrypt the payload using the private key associated with the relevant bootstrap key.

## **12.2 Bootstrap Key Distribution**

Bootstrap keys are distributed out-of-band during initial provisioning and during any subsequent bootstrap metadata updates.

Replica behavior:

- A replica **MUST** receive at least one valid bootstrap key during session initialization via the hello message and be encrypted using the agreed content public encryption key.
- The contained bootstrap keys **MUST** be integrity-protected and authenticated using the bootstrap trust root.
- Bootstrap keys **MUST** be stored securely and **MUST NOT** be exposed via ALSP or other untrusted channels.
- A replica **MAY** receive multiple bootstrap keys, for example during key rotation or multi-tier organizational provisioning.

### **12.2.1 Required Capabilities**

A replica:

- **MUST** be able to resolve a bootstrap key by its kid.
- **MUST** reject bootstrap envelopes whose kid does not match any locally installed bootstrap key.
- **MAY** support multiple private keys for decrypting bootstrap material.
- **MAY** request new bootstrap keys or updated manifests as part of the higher-layer bootstrap refresh procedure.

Non-normative note:

Some deployments may maintain both a long-term master bootstrap key and periodically rotated operational bootstrap keys.

## **12.3 Bootstrap Key Rotation**

Bootstrap keys may be rotated by an organization or channel owner. Rotation ensures forward secrecy, revocation capability, and alignment with organizational policy.

### **12.3.1 Replica Handling of Rotation**

A replica:

- **MUST** treat newly distributed bootstrap keys as active after successful validation.
- **MUST** continue to accept previously valid bootstrap keys until the bootstrap manifest declares them expired.
- **MUST** reject bootstrap artifacts encrypted with bootstrap keys not present in the current or transitional key set.
- **SHOULD** remove expired bootstrap keys from local storage once they are no longer required for decrypting historical material.

### **12.3.2 Manifest Updates**

The bootstrap manifest:

- **MUST** specify the currently active bootstrap key(s).
- **MUST** indicate when previous bootstrap keys become invalid.
- **MAY** include a scheduled or immediate rotation event.
- **MUST** be signed by the bootstrap authority.

Replica behavior:

- Upon receiving a new manifest, a replica **MUST** validate the signature and update its bootstrap key set accordingly.
- If a manifest indicates that a previously valid bootstrap key is revoked, replicas **MUST** immediately cease accepting bootstrap envelopes encrypted under the revoked key.

## **12.4 Error Handling**

When handling bootstrap keys or decrypting bootstrap artifacts, a replica:

- **MUST** return the ALSP error code "invalid\_auth" when decryption fails due to an unrecognized or revoked bootstrap key.
- **MUST** return "protocol\_violation" if the JWE structure is malformed.
- **SHOULD** return "unauthorized" if the bootstrap key is recognized but does not authorize access to the requested artifact.
- **MAY** include diagnostic information in the reason field, provided it does not disclose sensitive key material.

Non-normative note: For security reasons, error messages should not disclose whether the failure was due to unknown kid, decryption failure, or payload tampering.

## **12.5 Security Considerations**

Bootstrap Channel Keys protect sensitive initialization material and therefore have strict confidentiality requirements:

- Bootstrap private keys **MUST** be stored in a secure key store or equivalent.
- Replicas **MUST** minimize exposure by wiping decrypted bootstrap artifacts from memory when no longer needed.
- Implementations **SHOULD** log bootstrap key failures or key rotations for auditability.
- Implementations **SHOULD** rate-limit repeated decryption failures to mitigate oracle attacks.

Non-normative note: Compromise of a bootstrap key does not directly grant channel write or read permissions, but may permit an attacker to obtain metadata (e.g., channel encryption keys or identity information) depending on the bootstrap structure.

## **12.6 Decryptability Guarantees (Informational)**

By referencing the channel\_key\_envelope format for the AES key package and applying it to an array of Bootstrap keys, this approach maintains uniformity across the protocol while allowing the Bootstrap channel to support multi-epoch keying without special-case handling.

# **13. Key Identifier (kid) Resolution**

ALSP uses structured `kid` values in JWS headers to reference cryptographic material obtained through the bootstrap process. The `kid` format follows the same pattern as Layer 1 but resolves through different mechanisms appropriate to Layer 0's requirements.

## **13.1 Format & Semantics**

```
ascp:<type>:<uuid>
```

**Supported Types at Layer 0:**

- `ascp:cert:<uuid>` - References an ASCP authentication certificate for message authentication
  - Used in JWS protected headers for peer authentication
  - Resolved from the bootstrap process certificate directory
  - Contains the public key corresponding to the `user_auth_cert` exchanged during hello
- `ascp:cak:<uuid>` - References a Channel Access Key for channel authorization
  - Used in JWS protected headers for channel access proofs
  - Resolved from the channel manifest obtained during bootstrap
  - Contains the Ed25519 public key for the specific channel

## **13.2 Resolution Rules**

1. Parse the `kid` to extract type and UUID
2. Look up the key material in the appropriate bootstrap-provided directory
3. Extract the public key from the JWK representation
4. Use for JWS signature verification

This approach maintains consistency with Layer 1's key referencing model while operating within Layer 0's bootstrap-based key distribution mechanism.

## **13.3 JWK Requirements**

Unlike Layer 1 where keys are embedded in articulation statements, Layer 0 key material is distributed via the bootstrap process and stored in **JWK format** for consistency.

### EC Public Key JWK Format:

```json
{
  "kty": "EC",
  "crv": "P-256",
  "x": "<base64url X-coordinate>",
  "y": "<base64url Y-coordinate>",
  "alg": "ECDH-ES",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}

```

- `kty`: Key type (Elliptic Curve)
- `crv`: Curve (NIST P-256)
- `x`, `y`: Coordinates of the EC public key
- `alg`: Suggested algorithm (ECDH-ES)
- `kid`: Key identifier used for JWE or JWS headers and directory lookup

**Note:** The `use` field is intentionally omitted. This allows the same key to be used for both signature verification in JWS or key agreement in JWE.

This is compliant with RFC 7517 §4.2, which permits keys to omit the use field to avoid unnecessary restriction.

### Ed25519 Public Key JWK Format:

```json
{
  "kty": "OKP",
  "crv": "Ed25519",  
  "x": "<base64url-encoded-public-key>",
  "alg": "EdDSA",
  "kid": "ascp:cak:550e8400-e29b-41d4-a716-446655440002"
}
```

## **13.4 Caching & Lifetimes**

While Layer 1 manages all key lifecycle operations (rotation, expiration, revocation), Layer 0 has specific responsibilities for key resolution and caching:

**Static Key-ID Binding**: Each `kid` represents an immutable key. If a key needs to change, it must receive a new `kid`. This ensures that Layer 0 can trust that any given `kid` always resolves to the same cryptographic material.

**Session-Based Key Resolution**:

- Layer 0 implementations should resolve JWK keys from the bootstrap process/channel manifest at the time of use
- All cached keys should be purged at the start of each new ALSP session
- Keys should be resolved fresh from the bootstrap data when first encountered in a session

**Security-Based Cache Management**: For security best practices, any cached keys should be purged if they haven't been referenced for 10 minutes, ensuring that Layer 0 operates with fresh key material and reducing the window for potential key compromise.

**JWK Validation**: When resolving a key via `kid`, Layer 0 must verify that the `kid` field in the JWS header matches the `kid` field in the resolved JWK object to prevent key confusion attacks.

**Active session handling during CAK rotation:** Active sessions may continue with their session-cached keys until natural termination, at which point fresh sessions will resolve updated keys. If a peer wishes to force revalidation of a rotated CAK, it can send a 're-auth-required' error message. After sending this error, push syncs should cease and further sync\_requests must be refused for the remainder of this session. Note that in pull mode, each sync\_request is individually authorized against current credentials, while push mode relies on session-cached authorization from the initial subscription.

## 13.5 Interaction with Bootstrap Channel

**Trust Model**: Layer 0 operates under a **log-anchored trust model** similar to Layer 1 - key material from the bootstrap process and channel manifests is trusted as it existed at the time those artifacts were created and signed. Layer 0 does not perform live PKI validation or revocation checking; instead, it trusts the cryptographic relationships established when the bootstrap document and channel manifests were originally articulated into the log. This ensures that historical sync operations remain verifiable using the credentials that were valid when those log entries were created, regardless of subsequent key lifecycle events handled at higher layers.

# **14. Transport Bindings**

## **14.1 Transport Independence Statement**

While ALSP is designed to be transport-agnostic, this specification defines a WebSocket-based binding as the primary implementation. WebSockets provide the optimal characteristics for log synchronization: persistent bidirectional connections that support both push and pull sync models, low-latency message delivery, and efficient batching of articulation entries. Other transport bindings such as HTTPS and gRPC may be considered as future options.

All transport endpoints MUST enforce TLS 1.3 or higher for secure connection establishment and ALSP-level peer session authentication. ALSP implements a dual authentication approach:

1. **Transport Level (TLS)**: Implementations MUST establish a strong cipher based TLS connection to provide end-to-end encryption with a unique session key. For client-server configurations operating on public internet or corporate infrastructure, implementations SHOULD validate server certificates against PKI infrastructure to authenticate the server domain. For peer-to-peer configurations, implementations MAY use alternative authentication techniques, though such mechanisms are outside the scope of ASCP. 
2. **Protocol Level (ALSP)**: Once the secure transport connection is established, ALSP auth\_request, auth\_challenge (as needed), and hello message exchange provide the mandatory peer authentication and authorization that applies at the channel replica level. This sequence establishes the session and forms the basis for ongoing message authentication.

## **14.2 WebSocket-over-TLS Binding (Normative)**

The WebSocket binding leverages the full-duplex, persistent nature of WebSocket connections combined with Dot Preserving Binary (DPB) encoding to support high-throughput, resilient synchronization even in the face of large backlogs or offline replay scenarios.

### **Transport Characteristics**

WebSockets provide optimal support for ALSP's bidirectional synchronization requirements:

- Full-duplex communication allows both client and server to initiate sync operations without polling
- Persistent connections reduce overhead compared to repeated HTTPS requests
- Native support for live updates and push-based delivery

### **TLS Connection Setup**

This section normatively specifies the default ALSP transport over TLS. All ALSP connections establish secure communication through a standardized process that combines transport-level security via TLS 1.3 WebSocket connections with protocol-level authentication through the ALSP auth\_request, auth\_challenge, and hello message exchange. The following requirements define the specific connection establishment procedures:

1. **Endpoint and scheme.** For a given domain, the canonical ALSP endpoint **SHOULD** be a WebSocket Secure (WSS) origin at port **443**, e.g., wss\://ascp.\<domain>. Clients **MUST** use the wss scheme and TCP port 443 unless explicitly configured otherwise.
2. **TLS and ALPN.** The client **MUST** initiate a TLS handshake to the origin on TCP/443. During the handshake, the client **MUST** offer **ALPN** protocols h2 and http/1.1; the server **MUST** select one. Implementations **MUST**support TLS 1.3; support for TLS 1.2 is **OPTIONAL**.
3. **WebSocket establishment.**
   - If ALPN selects **HTTP/2**, the client and server **MUST** establish the WebSocket using the **Extended CONNECT** mechanism per RFC 8441 with :protocol = "websocket".
   - If ALPN selects **HTTP/1.1**, the client and server **MUST** establish the WebSocket using the standard HTTP/1.1 **Upgrade** handshake.
4. **Connection model.** After the WebSocket is established, ALSP **MUST** operate over a **persistent, full-duplex** TCP connection. Application data **MUST** be carried as **WebSocket binary messages**.
5. **Compression.** Implementations **SHOULD NOT** negotiate per-message compression extensions (e.g., per-message-deflate) for ALSP frames by default, as Layer 1 payloads already employ JWE compression where appropriate, and compression can introduce security vulnerabilities like CRIME and BREACH attacks.
6. **Keepalive and liveness.** Endpoints **SHOULD** use WebSocket **Ping/Pong** frames to detect half-open connections and to maintain NAT/firewall mappings at reasonable intervals.
7. **Subprotocol identification.** Servers **SHOULD** advertise, and clients **SHOULD** request, the WebSocket subprotocol token `ascp.alsp.v1` (where v1 corresponds to ALSP version 0.x for backward compatibility) via Sec-WebSocket-Protocol to enable policy and routing at intermediaries. If the subprotocol is not accepted, the connection **MAY** proceed without it.
8. **Proxy traversal.** When traversing explicit HTTP proxies, clients **MAY** use HTTP CONNECT tunneling prior to TLS, per HTTP semantics. Once tunneled, the requirements in (2) and (3) apply.
9. **Error handling and reconnect.** On transport failure (TLS or WebSocket), clients **SHOULD** immediately re-establish a new WSS connection and resume ALSP synchronization per the ALSP state machine (hello/auth and channel resubscription rules defined elsewhere in this specification).
10. **WebSocket close handling.** When ALSP protocol errors occur, implementations **SHOULD** close the WebSocket with code 1002 (Protocol Error) and include the ALSP error\_code in the close reason where possible.

## **14.3 Message Framing Rules**

**Framing and message unit.** Each WebSocket **binary message** **MUST** contain exactly one ALSP envelope (the deterministic-CBOR Layer‑0 envelope, that contains the DPB JWS payload as defined elsewhere in this specification). WebSocket fragmentation is transport-internal and **MUST NOT** be observable at the ALSP layer.

## **14.4 Performance Notes (Non-Normative)**

*(Optional—summaries related to batching/effects.)*

## **14.5 Future Transport Notes (Non-Normative)**

*(Keep short.)*

# **15. Node Topology and Deployment Models**

ALSP's symmetric protocol design enables flexible deployment topologies ranging from centralized client-server architectures to distributed peer-to-peer networks. The choice of architecture is orthogonal to the protocol itself—any replica can serve sync requests to any other authorized replica regardless of network topology.

## **15.1 Client/Server Model**

**Centralized Architecture**: The most common deployment pattern involves clients connecting to a centralized "server" that acts as both a data backup repository and primary replication partner. This server maintains authoritative copies of all channel logs and coordinates synchronization across multiple client replicas.

## **15.2 Peer-to-Peer Configurations**

**Distributed Architecture**: ALSP also supports loosely distributed sets of nodes where peers connect to peers without requiring a central authority. The specific peer discovery and connection configuration mechanisms are deployment-specific and not defined by this protocol specification.

## **15.3 Connection Constraints**

**Servers**: Must accept inbound connections and serve sync requests from authorized clients. Servers never initiate outbound connections to clients, operating purely in a responsive mode for connection establishment. To classify as a server in the protocol, a server MUST take connections from multiple nodes. Servers MUST refuse to authenticate a session with a node\_id replica that already has an established connection.

**Clients/Peers**: Typically initiate outbound connections to their configured sync partners. Clients may optionally accept inbound connections from other peers, but this is not required for basic operation. Given the full-duplex and symmetrical nature of the protocol, the topology MUST be constructed such that peers only maintain a single point-to-point connection with each other. For peer-to-peer operation, a peer MUST NOT attempt to connect to another node that already has a connection established regardless of who established it.

## **15.4 Role Symmetry**

By definition, servers not only respond to sync requests and send channel updates to clients, but also actively request live updates from connected clients. When a client supports push mode, the server automatically receives new content via sync\_update messages. However, if a client does not support push mode, the server must periodically issue sync\_request messages to collect new client content and maintain up-to-date channel logs.

This bidirectional synchronization ensures that servers maintain comprehensive, current views of all channel activity across their connected client base, regardless of individual client capabilities.

## **15.5 Push Mode Synchronization**

ALSP operates in one of two synchronization modes:

**Pull Mode (Default)**: Updates must be explicitly requested via sync\_request messages. The peer responds with available messages but does not send automatic updates.

**Push Mode (Optional)**: After initial synchronization, the peer automatically sends sync\_update messages when new messages become available for subscribed channels.

### **15.5.1** Negotiating Push Mode

During the hello exchange, peers negotiate push capabilities:

- Connecting peer sets `push_enabled: true` to request push mode
- Receiving peer confirms or denies based on its own `push_enabled` response
- Server nodes MUST implement push mode; client/distributed peers SHOULD implement it for efficiency

When push mode is negotiated successfully:

1. **Channel Subscription**: Each sync\_request acts as a subscription request for that channel\_id
2. **State Transition**: Once the sync\_response completes (bringing the peer up to date), that channel enters push mode for the remainder of the session
3. **Automatic Updates**: The peer automatically sends sync\_update messages whenever new messages become available for subscribed channels, without requiring further sync\_requests

This eliminates polling overhead and reduces latency for real-time synchronization scenarios.

**Important**: Push subscriptions are unidirectional. If both peers want to receive automatic updates for the same channel, each peer must send its own sync\_request to subscribe. Otherwise, push updates will only flow in one direction.

### **15.5.2 Channel Subscription State Machine**

Channel Sync States are maintained per peer connection for nodes that handle multiple inbound connections:

- **Unsubscribed**: No sync relationship established for this channel
- **Pull Sync**: Channel actively being synchronized via sync\_request/sync\_response cycles
- **Push Sync**: Channel subscribed for real-time sync\_update delivery
- **Error State**: Channel temporarily unavailable due to auth/transport failures

#### **State Transition Rules:**

1. A channel enters **Pull Sync** when a sync\_request is received and authorized
2. A channel transitions to **Push Sync** when:
   - The initial sync\_request has been completely fulfilled (more = false in final sync\_response)
   - Both peers negotiated push\_enabled = true during hello
3. A channel returns to **Pull Sync** if push delivery fails after 3 consecutive attempts

#### **Replica State Tracking:**

Each replica MUST maintain a per-peer, per-channel subscription table tracking which remote peers subscribed for push updates for which specific channels. This table is populated when:

- A sync\_request completes successfully
- Explicitly cleared when a connection closes or authentication fails

# **16. Error Handling**

ALSP defines a structured error message used to signal protocol violations, authentication failures, authorization problems, and capability mismatches between replicas. Error messages are carried in the standard ALSP envelope and follow the same JWS signing and nonce-binding rules as all other ALSP messages.

## **16.1 Error Message Format**

```json
{
  "alsp_msg_type": "error",             // Error message type identifier
  "timestamp": "<rfc-3339-timestamp>",  // Timestamp of this message
  "error_code": "invalid_auth",         // Machine-readable error code
  "reason": "utf-8-string",             // Human-readable error explanation
  "suggested_action": "utf-8-string",   // Corrective action guidance
  "disconnect": true                    // Whether session should be closed
}
```

### **Field Requirements**

- **alsp\_msg\_type**
  - Type: string
  - **MUST** be present.
  - **MUST** equal "error".
  - Identifies this header as an ALSP error message.
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the time at which the error message was generated.
  - Receivers **MUST** apply the same freshness and replay-protection rules to error messages as to other ALSP messages.
- **error\_code**
  - Type: string
  - **MUST** be present.
  - **MUST** be one of the error codes defined in Section 16.2.
  - Implementations **MUST NOT** rely on reason or suggested\_action for machine-interpreted behavior; decisions **MUST** be based on error\_code and disconnect.
- **reason**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **SHOULD** provide a concise, human-readable explanation of the error condition (for example, *“certificate does not match claimed identity”*).
  - Receivers **MAY** log or display this text for diagnostics, but **MUST NOT** treat it as authoritative for protocol semantics.
- **suggested\_action**
  - Type: string (UTF-8)
  - **MAY** be present.
  - When present, **SHOULD** describe a recommended corrective action (for example, *“Reconnect with a valid identity certificate”*).
  - Receivers **MAY** use this for operator guidance or UI messaging, but it has no normative effect on protocol behavior.
- **disconnect**
  - Type: boolean
  - **MUST** be present.
  - When true, the sender **MUST** close the ALSP session immediately after sending the error, and the receiver **MUST** treat the session as terminated once the error is processed.
  - When false, the sender **MAY** keep the session open and **MAY** continue to process and send non-error messages, subject to the semantics of the indicated error\_code.
  - For fatal conditions (for example, invalid\_auth, protocol\_violation), senders **SHOULD** set disconnect to true.

## **16.2 Error Codes**

| Description          |                                                                                                                               |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| invalid\_auth        | Certificate does not match claimed identity or is untrusted.                                                                  |
| unauthorized         | Replica is not permitted to access the requested channel.                                                                     |
| hash\_mismatch       | A validation SHA-256 hash mismatch has occurred and this implies log divergence.                                              |
| stale\_timestamp     | The message was rejected due to the timestamp being out-of -range. Replay attack is suspected.                                |
| protocol\_violation  | Message structure or sequence violated ALSP semantics.                                                                        |
| unsupported\_version | Peer sent an incompatible message format or spec version.                                                                     |
| payload\_too\_large  | Message exceeds max\_alsp\_length or violates buffer constraints.                                                             |
| internal\_error      | Sender encountered an unexpected failure.                                                                                     |
| re-auth-required     | This error can be generated to force the termination of a sync session. The disconnect parameter MUST be true for this error. |

## **16.3 Required Behavior**

Implementations MUST handle malformed or invalid messages gracefully without terminating the ALSP session unless explicitly required by an error message with `disconnect: true`.

When encountering invalid messages, replicas SHOULD log the error details and continue processing any remaining valid messages in the same sync batch.

Replicas MUST validate that all required fields are present and correctly typed before processing any sync message, and SHOULD send appropriate error responses for protocol violations.

**Recovery model:** All error recovery follows the same pattern as normal sync operations - establish connection, negotiate capabilities via hello, and request missing data via sync\_request. The protocol's idempotent design ensures that reconnection after any failure is equivalent to resuming sync after offline operation.

- Error messages may be sent in response to any ALSP message.
- A hello response with an error code is a valid rejection of the handshake.
- The receiver of an error message should log or report the error and must honor the disconnect directive if set to true.

## **16.4 Error Handling During Sync**

**Failure Detection:**

- Transport-level failures (WebSocket disconnect, timeout)
- Protocol-level failures (invalid JWS, authorization expired)

**Fallback Behavior:**

1. On push delivery failure, mark the channel as **Error State** for that peer
2. The failed peer MUST re-initiate sync via sync\_request to resume
3. Senders SHOULD NOT automatically retry push delivery to avoid message storms
4. Optionally, send an ALSP error message with error\_code "push\_failed" and suggested\_action "Re-sync via pull request"

**Graceful Degradation:**  
Replicas operating in unreliable network conditions MAY disable push mode entirely and rely on periodic pull sync to ensure eventual consistency.

## **16.5 Reconnection & Backoff**

When ALSP connections fail or are rejected, replicas SHOULD implement exponential backoff for successive connection attempts to prevent overwhelming servers during outages.

Recommended backoff strategy:

- Initial retry delay: 1 second
- Maximum retry delay: 300 seconds (5 minutes)
- Backoff multiplier: 2.0 with jitter

Replicas MAY implement more aggressive retry behavior for user-initiated sync operations, but automated background sync SHOULD respect these limits.

# **17. Channel Log Health Check & Recovery**

While ALSP ensures deterministic, convergent log synchronization under normal operation, real-world systems may encounter channel divergence due to implementation bugs, storage faults, or partial sync failures. This section defines normative and recommended practices for detecting and recovering from such divergence scenarios.

## **17.1 Digest Hash Exchange**

Implementations MAY periodically compute a deterministic hash of the message log for each channel based on the canonical ordering of message IDs. This digest can be validated with any peer via any sync\_request message. This is done via the optional log\_digest parameter of the sync\_request message.

- Hash inputs MUST include all logged message\_id values in canonical order.
- Hashing only the Layer 0 message\_id metadata (not payloads) is sufficient for detection.
- A mismatch signals possible divergence or incomplete sync.
- One should only perform this check when one presumed that the peer has the specified range of messages available to it therefore this is best done once one believes the log is already up to date.

Important: Any receiver of a log\_digest value MUST compute the corresponding local log\_digest for the same set of messages its own replica, and if there is a mismatch, it MUST send a hash\_mismatch error to the original sender. No other actions are required, but the receiver MAY attempt a recovery operation per Section 17.3.

## **17.2 Range Consistency Checking**

During sync\_request or sync\_response, replicas MAY evaluate:

- Number of log entries
- First and last lamport\_times in the log.

Peers can use this to identify range inconsistencies or unexpected log truncation.

## **17.3 Divergence Recovery Options**

If divergence is detected, replicas MAY perform the following recovery options:

### **1. Targeted Rehydration**

- Re-request a specific Lamport range via sync\_request with from\_lamport and to\_lamport set.

### **2. Full Re-Sync**

- Issue a sync\_request from from\_lamport = 0 to rehydrate the complete channel log.
- Validate canonical ordering and deduplicate on receipt.

### **3. Replica Rollback (Manual or Audited)**

- A faulty replica MAY delete and rebuild the affected channel log from scratch.
- However, a replica MUST NOT delete channel log records for which it is the originating party (i.e., messages it authored), as it is the canonical source for these messages and deleting them could cause permanent data loss.
- Operators SHOULD log the rollback event and verify post-recovery state with digest checks.

## **17.4 Merkle Tree Extensions (Non-Normative)**

A Merkle tree structure per channel log MAY be implemented to support efficient range comparison, similar to Git or Hypercore. This enables detection of localized divergence with minimal hash exchange.

## **17.5 Operational Best Practices**

Divergence Prevention Guidelines:

- Always persist Lamport counters and last-known lamport\_max
- Perform message insertion atomically and in canonical order
- Never insert messages into a log without full canonical sort metadata

While divergence is expected to be rare, these tools ensure that detection and repair can be handled gracefully without compromising global convergence.

# **18. Normative Implementation Guidance**

## **18.1 Deduplication**

Upon receiving messages, replicas MUST deduplicate based on message\_id before inserting into local logs.

If a message with the same message\_id already exists in the local log, it MUST NOT be reinserted or affect the Lamport counter. However, the replica MUST still process any lamport\_max values from the sync message envelope for counter synchronization.

This ensures idempotent sync and prevents duplication during retries, reconnects, or when receiving the same message from multiple peers.

## **18.2 Channel Message Scope**

All messages included in a single sync\_response or sync\_update MUST belong to the same channel\_id as declared in the message envelope.

Messages from different channels MUST NOT be intermixed in the same ALSP sync message. Implementations MUST send separate sync messages for each channel, even when responding to multiple concurrent sync requests.

## **18.3 Push Mode Lamport Rules**

In push mode, replicas MUST always transmit sync\_update messages when they have new locally generated (or received) messages that have advanced their local clock beyond the `lamport_max` value last transmitted or received from that specific peer in any previous ALSP message. This sync\_update must go out even when the receiving peer is not monitoring the associated channel(s).

This ensures:

- Proper advancement of the receiver's Lamport counter to the highest observed value.
- Prevents rollback or reordering anomalies
- Maintains causal consistency under concurrent sync conditions

## **18.4 Ordering Guarantees**

When inserting received messages into local logs, replicas MUST maintain canonical ordering using the tuple (lamport\_time, message\_id) regardless of the order in which messages were received or transmitted.

Messages MAY be received out of canonical order due to network conditions or batching optimizations. The `more: true` field in sync responses provides natural flow control, allowing senders to segment large message sets across multiple responses while respecting the `max_alsp_length` negotiated during hello. Implementations MUST sort and insert them in the correct positions within the local log structure.

## **18.5 Persistence Requirements**

Replicas SHOULD persist their current Lamport counter value to stable storage periodically and SHOULD persist it during graceful shutdowns as well.

Upon restart, replicas MUST initialize their counter to the maximum of:

- The persisted counter value
- The highest Lamport value across all locally stored messages
- Any previously received lamport\_max value from peers (if preserved)

**Startup Sync Optimization**: To maximize wall-clock time ordering quality, replicas SHOULD attempt to sync with at least one peer and process any received `lamport_max` values before encoding new local articulations into the Layer 0 channel logs. This ensures that new messages receive Lamport values that are temporally consistent with current network activity rather than artificially low values that would cause them to appear out of chronological sequence in the global timeline.

This prevents counter regression and maintains the best possible wall clock sorted global ordering consistency across restarts.

# **19. Security Considerations**

*(To be built out more—will summarize threats, transport security, CAK hazards, identity handling.)*

### **19.1 Replay Protection**

To prevent replay attacks, receiving peers MUST validate the `timestamp` field in the `alsp_msg_header` against their local clock.

Implementations MUST NOT accept messages with timestamps that differ from the receiver's local time by more than 300 seconds (5 minutes), as accepting messages beyond this threshold significantly increases replay attack vulnerability.

Implementations SHOULD reject messages whose `timestamp` value differs from the receiver's local time by more than 60 seconds. This recommended default provides strong replay protection while accommodating reasonable clock skew between peers and network latency.

When a message is rejected due to timestamp violation, the receiver MUST send an ALSP error message (Section 16) with an appropriate error code indicating the timestamp validation failure.

# **20. IANA Considerations**

*(Currently empty placeholder.)*

# **21. Examples (Non-Normative)**

## **21.1 Example ALSP Message**

This example shows a fully-formed sync\_response message using the standardized JSON structure. It demonstrates how multiple messages from a single channel are returned in a sync response.

### **JWS Protected Header:**

```json
BASE64URL(UTF8({
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "alsp",
  "nonce": "peer_session_nonce_here"
}))
```

### **JWS Payload (the complete ALSP message structure):**

```json
BASE64URL({
  "alsp_version": "0.1",
  "alsp_msg_header": {
    "alsp_msg_type": "sync_response",
    "timestamp": "2024-01-15T10:30:45.123Z",
    "channel_id": "550e8400-e29b-41d4-a716-446655440002",
    "lamport_max": 23456,
    "more": false
  },
  "alsp_msg_payload": [
    {
      "lamport_time": 23451,
      "message_id": "850e8400-e29b-41d4-a716-212554400020",
      "payload": "<DPB_encoded_bytes>"
    },
    {
      "lamport_time": 23452,
      "message_id": "50e8400-f29b-41d4-b716-446655448010",
      "payload": "<DPB_encoded_bytes>"
    }
  ]
})
```

The complete JWS compact serialization (`header.payload.signature`) is sent over the WebSocket in DPB format.

This representation applies equally to sync\_update messages. The `alsp_msg_payload` field is only present for messages that carry articulation data. Note that payloads in the message array are treated as opaque binary by ALSP and interpreted only at Layer 1.

## **21.2 Example Session Flow**

The typical ALSP session progresses through the following phases:

```asciidoc
Client                       Server
  |                            |
  | -- auth_request ---------> |
  |                            |
  | <------ auth_challenge --- |
  |                            |
  | -- hello (nonce=S) ------> |
  |                            |
  | <------ hello (nonce=C) -- |
  |                            |
  | --------- sync_request --> |
  |                            |
  | <--- sync_response ------  |
  |                            |
  | -- sync_update --------->  | (if using push model)
  |                            |
  | <--- sync_update --------  | (peer pushes back updates)
  |                            |
  | -- error ----------------> |  (optional)
  | <---------- error -------- |  (if needed)
  |                            |
  |       connection close     |
```

### **Notes**

- auth\_request is always required and initiated by connecting party
- auth\_challenge only happens if/as needed
- hello is always required and a symmetric to complete mutual authentication
- sync\_request initiates pull of sync information.
- sync\_update is optional and used for live push.
- error may be sent at any phase to signal a problem.
- Sessions may close cleanly or due to protocol violations.

## **21.3 DPB Encoding Example**

Consider a JOSE JWS with the following compact serialization:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibm
FtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.SflKx
wRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

This breaks down into three segments:

- **Header**: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
- **Payload**: eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWR taW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0
- **Signature**: SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV\_adQssw5c

### **Decoded lengths (bytes)**

- Header = **27**, Payload = **68**, Signature = **32**

### **DPB bytes (hex)**

```
// Header segment: 1f 1b (escape + ULEB128(27)) + 27 bytes
1f 1b
7b 22 61 6c 67 22 3a 22 48 53 32 35 36 22 2c 22
74 79 70 22 3a 22 4a 57 54 22 7d

// Dot separator
2e

// Payload segment: 1f 44 (escape + ULEB128(68)) + 68 bytes  
1f 44
7b 22 73 75 62 22 3a 22 31 32 33 34 35 36 37 38
39 30 22 2c 22 6e 61 6d 65 22 3a 22 4a 6f 68 6e
20 44 6f 65 22 2c 22 61 64 6d 69 6e 22 3a 74 72
75 65 2c 22 69 61 74 22 3a 31 35 31 36 32 33 39
30 32 32 7d

// Dot separator
2e

// Signature segment: 1f 20 (escape + ULEB128(32)) + 32 bytes
1f 20
49 f9 4a c7 04 49 48 c7 8a 28 5d 90 4f 87 f0 a4
c7 89 7f 7e 8f 3a 4e b2 25 5f da 75 0b 2c c3 97
```

### **Notes:**

- This example shows a JWS with header, payload and signature in JOSE compact serialization form, but DPB encoding is format-agnostic—any sequence of dot-separated Base64url encoded streams can be represented in DPB format
- `1f 1b` → binary block opener (0x1F) + ULEB128(27) indicating 27-byte segment length
- Dots (0x2E) remain as literal separators preserving JOSE structure
- All Base64url segments round-trip perfectly via raw bytes and canonical Base64url re-encoding (no padding required)

# **22. Future Extensions (Non-Normative)**

- Merkle tree summary per log for integrity verification
- Log compression and chunking
- Gossip-based peer-to-peer sync overlay
- Snapshotting and checkpointing

# **23. References**

## **23.1 Normative References**

## **23.2 Informative References**

# **Appendix A — Authentication Flow Examples (Informative)**

This appendix provides narrative examples of the two possible ALSP authentication flows. These examples illustrate the interaction between session nonces, authentication messages, and the session state machine defined in Section 8. They are informative only; the normative behavior is defined entirely in Sections 8 and 10.

## **A.1 Immediate Flow Example (Informative)**

In the Immediate Flow, the receiving replica has sufficient trust material cached to validate the client’s identity directly. No auth\_challenge is needed.

**Preconditions**

- The client constructs an auth\_request containing either:
  - all required identity materials (Direct Mode), or
  - an identity reference (Provisioned Mode), which the server already trusts.
- The server already possesses the necessary trust anchors, claim bundles, or certificate chains.

**Sequence Explanation**

1. The client sends an auth\_request containing its own session\_nonce (C\_nonce).
2. The server successfully validates the request using its cached trust materials.
3. The server responds directly with hello, carrying its own session\_nonce (S\_nonce).
4. When the client receives the server’s hello, it sends its own hello containing S\_nonce.
5. At this point, both peers have exchanged hello messages, and the session enters the AUTHENTICATED state.
6. All subsequent messages use post-authentication mode, where each side signs using the peer’s session\_nonce.

## **A.2 Challenge Flow Example (Informative)**

In the Challenge Flow, the server cannot validate the identity presented in the initial auth\_request with its local trust material. The server requests additional information.

**Preconditions**

- The client may be operating in Direct or Provisioned Mode.
- The server requires more identity materials than were included in the initial auth\_request.

**Sequence Explanation**

1. The client sends an initial auth\_request with its own C\_nonce.
2. The server attempts validation but lacks some identity information.
3. The server sends an auth\_challenge, including S\_nonce and requests for missing credential elements.
4. The client responds with a second auth\_request, providing all requested identity materials.
5. The server validates the response using its trust mechanisms.
6. The server sends hello with its S\_nonce.
7. The client, upon receiving this hello, sends its own hello echoing S\_nonce.
8. The session transitions to AUTHENTICATED, and post-authentication message rules apply.

## **A.3 Post-Authentication Message Exchange (Informative)**

Regardless of whether the Immediate or Challenge Flow occurs, authenticated message mode works identically.

After the session is authenticated:

- The client uses the **server’s** **session\_nonce** in its JWS protected headers.
- The server uses the **client’s** **session\_nonce** in its JWS protected headers.
- Only *one* nonce appears in each protected header.
- All messages are signed with the identity key validated in the handshake.
- Any mismatch in the expected nonce MUST result in session termination.

**Example:**

```
Client → Server:
  sync_request (nonce = S_nonce)

Server → Client:
  sync_response (nonce = C_nonce)
```

This bilateral “cross-nonce” pattern binds every message to the authenticated session.

## **A.4 Relationship Between Modes and Flows (Informative)**

The following combinations are all valid:

| **Client Mode**  | **Server Flow** | **Possible?** | **Explanation**                                          |
| ---------------- | --------------- | ------------- | -------------------------------------------------------- |
| Direct Mode      | Immediate Flow  | ✔ Common      | Server already trusts client credentials.                |
| Direct Mode      | Challenge Flow  | ✔ Possible    | Server may require updated or additional trust material. |
| Provisioned Mode | Immediate Flow  | ✔ Possible    | Server may already have required trust material cached.  |
| Provisioned Mode | Challenge Flow  | ✔ Common      | Server typically needs additional identity information.  |

Modes describe **what the client sends**.

Flows describe **what the server needs**.

They are independent.

# **Appendix B — Authentication Flow Tutorial (Informative)**

*This section is informative, but draws on normative details found in the ASCP Trust and Identity Architecture specification.*

### **B.1 Challenge Flow (New Identity Binding)**

The Challenge Flow is used for first-time connections where new identity binding is required, or when the server needs to re-authenticate the connecting peer's identity. This flow involves an additional round-trip through the auth\_challenge message.

The message sequence proceeds as follows:

- Client sends auth\_request with their identity information and JWK for their public identity key (direct mode) or their public recovery key (provisioned mode). The JWS protected header typically omits the `kid` field since no binding exists yet.
- Server responds with auth\_challenge, which provides the server's session nonce and, in provisioned mode, may include the **client\_key\_env** JWE containing the generated or recovered client identity encrypted using the `recovery_cert` from the auth\_request.
- Client sends hello, including the JWS-signed Identity Token Package in the `user_auth_cert` field to establish identity binding.
- Server responds with hello to complete mutual authentication, or sends an error if identity binding fails.

### **B.2 Immediate Flow (Known Identity)**

The Immediate Flow is the standard authentication process when peers already have established, trusted identity-key bindings. This flow skips the challenge step, reducing the handshake to a single round-trip.

The message sequence proceeds as follows:

- Client sends auth\_request with their identity and JWK for their public identity key. The JWS protected header includes the `kid` for their signing certificate.
- Server validates the key-identity binding immediately and responds directly with hello, skipping the challenge step.
- Client responds with their own hello to complete mutual authentication.