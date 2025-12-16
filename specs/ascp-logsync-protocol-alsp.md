# **ASCP LogSync Protocol (ALSP)**

**Layer 0 - ASCP Protocol Stack Logging and Synchronization Layer**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.52 — Informational (Pre-RFC Working Draft)  
December 2025

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

### **4.1 ALSP (ASCP LogSync Protocol)**

The Layer 0 protocol of the ASCP stack responsible for transport-agnostic replication of append-only Channel Logs using deterministic ordering.

### **4.2 Articulation**

A Layer 2 semantic unit representing structured shared cognition. Articulations are conveyed to Layer 0 as opaque JOSE JWS/JWE compact sequences encapsulated within Layer 1 message envelopes.

### **4.3 ASCP (Agents Shared Cognition Protocol)**

A four-layer protocol suite enabling persistent, structured collaboration among humans and agents. ALSP defines Layer 0 of this stack.

### **4.4 Bootstrap Key Package (BKP)**

A JWE-encoded, bootstrap-scoped package delivered out-of-band via the `boot_keys_jwe` field in the ALSP hello message, used to convey bootstrap-scoped key material required by higher layers to enable decryptability of the encrypted @references channel for deterministic discovery during join bootstrap.

### **4.5 Bootstrap Log**

A minimal, append-only log used to establish the initial trust context for an ASCP organizational instance. The bootstrap log anchors the organizational trust root (RootCA) and provides the information required to locate the authoritative channel discovery registry.

The bootstrap log **does not contain cryptographic keys**, channel membership data, or authorization state, and **does not function as a channel discovery registry**.  
All trust evaluation, discovery, and authorization semantics beyond initial anchoring are derived from validated history in subsequent channels under normal ASCP rules.

### **4.6 Canonical Order**

The total ordering over Channel Log entries defined as the tuple **(lamport\_time, message\_id)**, sorted first by Lamport clock value and then by the lexicographic ordering of the 16-byte message identifier.

### **4.7 Challenge Flow**

A server-side handshake path triggered when the receiving replica cannot fully validate the client's identity using cached or known materials and therefore sends an auth\_challenge requesting additional identity data. Challenge Flow describes server behavior independent of the client's credential mode.

### **4.8 Channel**

A uniquely identified, append-only stream of Layer 1 message envelopes. Channels define the unit of distribution and authorization within ASCP.

### **4.9 Channel Access Key (CAK)**

An Ed25519 key pair used to authenticate access to a Channel at Layer 0. The public key is distributed in the Channel Manifest; the private key is held by authorized participants to produce Channel Access Proofs.

### **4.10 Channel Log**

A replica's ordered sequence of Layer 1 message envelopes for a specific Channel. The log is append-only and synchronized across replicas using ALSP.

### **4.11 Channel Manifest**

Channel metadata distributed via the bootstrap process, including Channel identifiers, access-control configuration, and the CAK public key.

### **4.12 Digest Hash Exchange**

A Channel Log consistency check in which replicas compare a SHA-256 digest of message identifiers in canonical order to detect divergence.

### **4.13 Direct Mode**

A client credential-supply mode in which the initiating replica includes all identity materials necessary for validation in its initial auth\_request. Direct Mode describes how the client packages credentials and does not imply anything about the server's validation flow.

### **4.14 Dot-Preserving Binary (DPB)**

A reversible binary encoding that transforms JOSE compact serialization into a more efficient on-wire representation by decoding Base64url segments while preserving literal dot separators. Used by Layer 0 to transport Layer 1 payloads without semantic interpretation.

### **4.15 Hello Message**

The message exchanged after authentication to negotiate session parameters, exchange lamport\_max values, advertise capabilities, and confirm mutual authorization before synchronization.

### **4.16 Identity Key**

The long-term cryptographic key pair representing a participant (human or agent) within ASCP. Represented as a JWK and bound to an identity via signed claim bundles defined outside this specification.

### **4.17 Immediate Flow**

A server-side handshake path in which the receiving replica can validate the client's auth\_requestimmediately based on locally available trust material, allowing it to respond directly with hello without issuing an auth\_challenge.

### **4.18 JWK (JSON Web Key)**

A JOSE structure used to encode public and symmetric keys, including Identity Keys, Bootstrap channel keys, and Channel Access Keys.

### **4.19 JWS (JSON Web Signature)**

The JOSE signature format used to authenticate all ALSP messages and to carry Channel Access Proofs.

### **4.20 JWT (JSON Web Token)**

The JOSE token format used within ASCP for identity claims and trust establishment. JWT content is not interpreted by ALSP.

### **4.21 JOSE (JavaScript Object Signing and Encryption)**

A family of standards (JWS, JWE, JWK) used by ASCP for signatures, encryption, and key representation. ALSP is aware only of compact serialization encoding and does not interpret JOSE semantics.

### **4.22 kid (Key Identifier)**

A structured identifier of the form ascp:\<type>:\<uuid> referencing key material obtained through the bootstrap process. Used in JWS and JWE protected headers.

### **4.23 Lamport Clock**

A monotonically increasing logical timestamp used by ALSP to provide deterministic ordering across distributed replicas.

### **4.24 lamport\_max**

The highest Lamport clock value known to a replica at the time a message is sent. Propagated during sync to maintain logical time coherence across replicas.

### **4.25 log\_digest**

A SHA-256 digest of the sequence of message identifiers in canonical order, used for Channel Log consistency checks.

### **4.26 message\_id**

A 16-byte universally unique identifier (UUID) assigned by ALSP to each Layer 1 message envelope for idempotence and ordering.

### **4.27 node\_id**

A universally unique identifier for a replica participating in ALSP synchronization.

### **4.28 payload**

The DPB-encoded JOSE compact serialization of a Layer 1 JWS/JWE message. ALSP treats payloads as opaque byte sequences.

### **4.29 Provisioned Mode**

A client credential-supply mode in which the initiating replica includes only minimal identity references in its initial auth\_request, supplying full credentials only if requested via auth\_challenge. Provisioned Mode concerns client behavior, not server response.

### **4.30 Pull Sync**

A synchronization mode in which replicas request Channel Log updates via sync\_request messages.

### **4.31 Push Sync**

A synchronization mode in which replicas automatically deliver new Channel Log entries via sync\_update messages after an initial synchronization.

### **4.32 Replica**

An authorized node storing and synchronizing one or more Channel Logs.

### **4.33 Session**

A mutually authenticated communication context established between two ALSP replicas for the duration of message exchange. A session begins with the ALSP authentication handshake, binds all subsequent messages through session-specific nonces and signature rules, and persists until closed, terminated, or expired.

### **4.34 session\_nonce**

A per-session, randomly generated 128-bit nonce created independently by each endpoint. During authentication, each replica includes its own `session_nonce` in the JWS protected header's `nonce` field. After authentication completes, each replica includes the peer's `session_nonce` in the `nonce` field instead. This cross-use of session nonces binds messages to a specific authenticated session and prevents replay attacks across connections.

### **4.35 sync\_request**

An ALSP message used by a replica to request Channel Log entries or verify log state.

### **4.36 sync\_response**

An ALSP message containing Channel Log entries provided in response to a sync\_request.

### **4.37 sync\_update**

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

This architectural separation means Layer 0 ALSP operates as a **secure replication and synchronization mechanism** that can efficiently encode and transmit Layer 1's JOSE-formatted messages while Layer 1 handles all semantic security.

**Note on Transport Security:**  
While ALSP is commonly deployed over TLS-secured transports, TLS authentication provides only baseline network-level security (e.g., confidentiality in transit and resistance to opportunistic MITM attacks). TLS **does not establish ASCP identity, instance trust, or authorization semantics**. All ASCP-relevant authentication, session binding, and trust anchoring occur within ALSP and higher layers, not at the transport layer.

The two security models are complementary, not redundant: Layer 0 ensures that Layer 1's cryptographically secured articulation statements reach all authorized replicas with identical content and ordering, while Layer 1 ensures that the meaning and content itself remains cryptographically protected regardless of transport encoding optimizations.

### **Note on Transparency Systems**

ALSP’s security model is intentionally limited to authenticated transport, deterministic ordering, and integrity of replicated logs. Unlike Certificate Transparency (CT) or other Verifiable Data Structure (VDS) architectures, ALSP does not require globally comparable Merkle trees nor an untrusted log operator. Trust derives from authorship signatures at Layer 1 and scoped channel authorization, not from public auditability.

**Future revisions of this specification** *may* introduce Merkle-based summary structures solely to assist with replica-convergence checks (See Section 9.9), but such mechanisms would serve as transport-level optimizations rather than trust anchors and would not alter any Layer-1 or Layer-2 semantics.

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

**At the articulation statement level**, each articulation is globally unique and immutable with verifiable authorship and timestamps. The protocol prevents duplication or mutation once messages are replicated, ensuring message integrity across the network.

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

## 8.1 Authentication Layers

ALSP defines two distinct but complementary layers of authentication: **message-level authentication** and **session-level authentication**. Message-level authentication ensures the authenticity and integrity of individual protocol messages, while session-level authentication establishes cryptographic continuity, replay resistance, and privileged operation eligibility across a bounded interaction between two replicas.

Session-level authentication builds upon message-level authentication and does not replace it. Both layers are required for correct and secure ALSP operation.

### 8.1.1 Message-Level Authentication

Message-level authentication applies to **all ALSP messages**, independent of session state. It provides per-message cryptographic authenticity, integrity protection, and freshness validation.

Message-level authentication uses JWS signatures to provide per-message integrity and authenticity.

The nonce usage rules are defined exclusively in Section 8.1 and apply identically to all message types, including authentication messages, sync messages, and operational updates.

Message-level authentication validates that:

1. the JWS signature was created by the sender’s private key;
2. the protected header includes the correct nonce per Section 8.1;
3. the message is fresh according to timestamp and replay-detection rules.

Message-level authentication does not introduce additional nonce semantics and does not modify which nonce is placed in the protected header.

Message-level authentication allows recipients to detect message tampering, spoofing, and stale replays. Prior to session binding, replay may be detectable but not cryptographically impossible; no privileged operations are permitted during this phase.

### 8.1.2 Session-Level Authentication

Session-level authentication establishes a mutually authenticated, replay-safe cryptographic session between two replicas. It enables privileged ALSP operations such as log synchronization, bootstrap artifact exchange, and channel interaction.

Session-level authentication is **session-oriented**, not trust-authoritative. A successfully authenticated session guarantees cryptographic continuity with a peer for the lifetime of the session, but does not by itself establish ASCP instance membership, authorization, or governance legitimacy.

#### 8.1.2.1 Session Authentication Semantics

Session authentication establishes **cryptographic continuity and replay-safe message authenticity** between two replicas for the lifetime of a single ALSP session. Each endpoint generates a 128-bit **session\_nonce** (defined in Section 4) that uniquely identifies the local half of the session.

**ALSP session authentication is explicitly** ***session-oriented*****, not trust-authoritative.**  
A successfully authenticated ALSP session guarantees only that messages originate from the same cryptographic peer for the duration of the session. It does **not** establish authoritative ASCP instance membership, RootCA legitimacy, governance approval, or long-term trust standing. Those determinations are made exclusively by higher layers after bootstrap artifacts are acquired and validated.

Session authentication defines the boundary after which privileged protocol operations may occur and before which all interactions are constrained to non-privileged, message-authenticated exchanges.

#### 8.1.2.2 Nonce Generation and Session Binding

ALSP uses a single `nonce` field in the JWS protected header with the following rules:

- **Before authentication completes**: Each replica MUST include its **own** `session_nonce` in the `nonce` field.
- **After authentication completes**: Each replica MUST include the **peer's** `session_nonce` in the `nonce` field.

This cross-use of nonces ensures that:

1. Both endpoints prove possession of their `session_nonce` during the handshake,
2. All post-authentication messages are bound to the authenticated peer and cannot be replayed across sessions, and
3. Exactly one nonce value is present in the JWS protected header at any given time.

The `session_nonce` field in the ALSP message body MUST always contain the sender's own nonce.

Successful nonce exchange and validation cryptographically bind both peers to a shared session context and establish continuity across subsequent ALSP messages.

#### 8.1.2.3 Progressive Replay Protection Model

This ALSP replay protection model is intentionally **progressive** across the session establishment lifecycle:

- **Pre-binding phase (initial** `auth_request`**)**
  - Messages are signed and timestamped.
  - Replays are detectable but not cryptographically impossible.
  - No privileged operations are permitted.
- **Challenge phase (when applicable)**
  - The server introduces a fresh challenge nonce.
  - Identity material becomes bound to server-provided freshness.
- **Session-bound phase (post-**`hello` **exchange)**
  - Each peer possesses the other’s `session_nonce`.
  - All messages are cryptographically bound to the authenticated session.
  - Replay across connections becomes impossible within the ALSP threat model.

Replay exposure is limited exclusively to the pre-authentication phase, during which ALSP prohibits log access, bootstrap retrieval, and all authorization-sensitive operations.

## **8.2. Goals and Requirements**

The ALSP session authentication process establishes a secure, replay-protected, and identity-bound communication context between replicas before any synchronization activity begins. The following goals and requirements apply to all ALSP sessions.

**Identity Verification:**

Each peer MUST validate the identity of the other using signed credentials anchored in the ASCP Trust & Identity architecture. ALSP does not define identity semantics but requires that the resulting key material be verified before a session transitions to the AUTHENTICATED state.

**Freshness and Replay Protection:**

All authentication messages MUST include a `session_nonce` generated by the sender, and peers MUST validate that these nonces are fresh and not reused across sessions. Timestamp validation MUST be applied to authentication messages to prevent replay of stale credentials.

**Mutual Authentication:**

Both parties MUST complete the authentication handshake and confirm each other’s identity before any Channel Log operations are permitted. Authentication MUST be mutual; unilateral authentication is not allowed.

**Session Binding and Replay Resistance:**

Upon successful authentication, both peers MUST have exchanged and validated each other’s `session_nonce`. After this point:

- Every ALSP message is cryptographically bound to the authenticated session.
- Any replayed message **MUST** be rejected due to nonce mismatch.
- This property applies uniformly to all ALSP messages, including `sync_request`, `sync_response`, `sync_update`, and error messages.

Within the ALSP threat model, replay after session binding is **cryptographically impossible**.

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

- Each peer generates a fresh `session_nonce` and includes it in all message headers.
- Each peer MUST validate the peer’s identity, `session_nonce`, and `timestamp` freshness before proceeding.
- **No Channel Log operations of any kind MAY occur prior to successful session authentication**, including:
  - log synchronization,
  - bootstrap artifact retrieval,
  - channel discovery, or
  - authorization checks.

This invariant ensures that ALSP never exposes log material or bootstrap state prior to establishing a mutually authenticated, replay-safe session.

The handshake concludes when each side has received a valid `hello` from the other, at which point the session transitions into the AUTHENTICATED state (Section 8.4). After this transition, all ALSP messages MUST use the post-authentication signature and nonce-binding rules defined in Section 8.1.

*Non-normative note:* The handshake design allows authentication to succeed even in asymmetric trust-discovery scenarios, such as when only one peer realizes a challenge is required or when caches differ across replicas.

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

*Non-normative note:* This strict failure behavior ensures that misconfigured clients, replay attempts, or injection attacks cannot pollute session state or reach log synchronization.

## **8.5. Authentication Modes**

During session authentication, a client establishes its identity using one of two credential-supply modes: **Direct Mode** or **Provisioned Mode**. These modes describe *how* the client supplies identity material to the server. The concrete fields used to carry this material are defined in Section 10.2; this section defines only the normative behavior and constraints associated with each mode.

Authentication mode selection is a **handshake-time decision** determined **solely** by the contents of the **initial** `auth_request`. The selected mode governs how identity credentials are supplied during the handshake but does **not** create separate session types or require session restart.

In Provisioned Mode, identity credentials may be upgraded during the Challenge Flow while **preserving session continuity**. The session is established once and transitions smoothly from provisional to fully bound identity authentication without renegotiation or reset.

A server determines the client’s mode based solely on the presence or absence of certain authentication fields in the **initial authentication request**, as described in Section 8.5.3. Mode selection is not signaled explicitly by the client.

### **8.5.1 Direct Mode**

In **Direct Mode**, the client already possesses its long-term identity credentials. The client authenticates by presenting its identity certificate (or equivalent identity package) in the initial authentication request.

A client operating in Direct Mode:

- **MUST** include its identity certificate field in the initial authentication request.
- **MUST** sign all authentication messages with the private key corresponding to that certificate.
- **MUST** include a key identifier referencing that certificate’s verification key.
- **MUST NOT** include a recovery certificate in the initial authentication request.
- **MUST** continue to supply identity material in any retransmission of the authentication request.

If the server possesses sufficient trust metadata to validate the client’s identity, it **MAY** complete authentication immediately using the Immediate Flow defined in Section 8.4.1. Otherwise, the server **MAY** issue an authentication challenge to request additional identity artifacts.

Direct Mode is appropriate for clients that have been fully provisioned by their operators or by prior participation in the ASCP Bootstrap Process.

### **8.5.2 Provisioned Mode**

In **Provisioned Mode**, the client does not yet possess its long-term identity credentials and relies on the server to provision them during authentication. The client initiates authentication using a recovery certificate that permits the server to deliver identity key material securely.

A client operating in Provisioned Mode:

- **MUST NOT** include an identity certificate field in the initial authentication request.
- **MUST** include a recovery certificate field in the initial authentication request.
- **MUST** sign the initial authentication request using the private key associated with the recovery certificate.
- **MUST** use an empty key identifier value in the protected header of the initial authentication request.

If the server determines that additional identity material is required, it **MUST** issue an authentication challenge. The client’s response to this challenge:

- **MUST** include its newly provisioned identity certificate or identity package.
- **MUST** sign subsequent authentication messages using the newly provisioned identity key.
- **MUST** include a key identifier referencing that identity key.
- **MUST NOT** include the recovery certificate.

Provisioned Mode allows clients without preexisting identity credentials to participate in ALSP Channels using identity materials issued securely during the session establishment.

### **8.5.3 Mode Determination (Server Behavior)**

The server determines the client’s credential-supply mode by inspecting the **initial authentication request**. The following rules apply:

A server **MUST** classify the client as operating in:

**Direct Mode** if and only if:

- the identity certificate field is present in the initial authentication request.

**Provisioned Mode** if and only if:

- the identity certificate field is absent, and
- the recovery certificate field is present, and
- the key identifier in the protected header of the request is an empty string.

Any other combination of these fields in the initial authentication request:

- **MUST** result in an authentication\_error,
- **MUST** terminate the session, and
- **MUST NOT** trigger a challenge flow.

These rules ensure that mode detection is deterministic and that clients cannot enter inconsistent or partially defined authentication states.

### **8.5.4 Invalid Combinations**

A server **MUST** treat the following conditions as invalid and terminate authentication with an appropriate error:

- Both an identity certificate field **and** a recovery certificate field are present in the initial authentication request.
- An identity certificate field is absent in the initial authentication request and the recovery certificate is also absent.
- A recovery certificate is present in the challenge-response authentication request.
- The initial authentication request contains a non-empty key identifier while omitting the identity certificate field.
- The challenge-response authentication request omits the identity certificate field.

Invalid combinations prevent interoperable implementations and may indicate misconfiguration or an attempted downgrade attack.

### **8.5.5 Relation to Authentication Flows**

Credential-supply mode selection is orthogonal to the handshake flow selection defined in Section 8.4:

- Direct Mode **may** use either Immediate Flow or Challenge Flow, depending on the server’s trust state.
- Provisioned Mode **always** begins with a preliminary request and, if successful, continues via a Challenge Flow to complete identity provisioning.

Mode selection determines *what* identity material the client supplies; flow selection determines *how and when* that material is validated by the server.

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

## **9.9 Merkle-Based Replica Validation (Future)**

ALSP’s deterministic Lamport ordering model ensures that any two replicas with the same set of messages will converge to identical Channel Logs without requiring auxiliary data structures. However, certain deployments may benefit from more efficient mechanisms for detecting localized divergence during synchronization.

**Future revisions of this specification may define an optional per-channel Merkle summary**, allowing replicas to exchange compact proofs of log prefixes or ranges. Such structures would serve only as *replica health and convergence aids*, not as trust mechanisms:

- They would not replace authorship signatures or Layer-1 cryptographic guarantees.
- They would not introduce public auditability or CT-style global consistency requirements.
- They would remain strictly below the Channel trust boundary and would not affect visibility or encryption semantics.

Any incorporation of Merkle summaries would therefore be **purely a Layer-0 optimization**, consistent with ALSP’s role as a transport and replication substrate.

# **10. ALSP Protocol Messages**

This section defines the complete **Layer-0 ALSP message model**, including the binary envelope format, JSON message header conventions, and how channel log entries are batched for synchronization. All ALSP messages are integrity-protected by JWS as defined in Section 6 and Section 7. Transport bindings (for example, WebSocket) carry these envelopes as opaque binary units (see Section 14).

## **10.1 Message Envelope Structure**

This section specifies the on‑wire encoding for ALSP messages. It defines a compact CBOR envelope that batches Layer‑0 entries while carrying a flexible message header as a JSON blob. The CBOR envelope serves as the JWS payload and is integrity-protected by a JWS signature that signs the entire CBOR byte sequence verbatim.

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

- JSON is embedded as a CBOR byte string (bstr) to ensure deterministic encoding and avoid whitespace or ordering variability per RFC 8785.
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
- MUST include a session\_nonce in the ALSP **message header** per Section 8.
- MUST include a nonce field in the **JWS protected header** per Section 8.1.
- MUST NOT include message header fields other than those explicitly defined in this section unless negotiated by future protocol extensions.
- MUST reference identity and key material in accordance with **ASCP Trust & Identity Architecture.**

Authentication messages do **not** grant channel access; they only establish the authenticated session.

See Section 8.5.4 for normative constraints on valid and invalid field combinations in authentication messages. Implementations MUST reject any combination that violates those constraints.

### **10.2.1 Auth Request Message**

The `auth_request` message initiates the ALSP session handshake. The client sends this message to establish its identity with the server and begin mutual authentication.

Field requirements depend on the client's credential-supply mode (Direct Mode or Provisioned Mode), as defined in Section 8.5. Not all field combinations are valid; implementations MUST enforce the mode-specific constraints specified in Section 8.5 when constructing or validating this message.

```json
{
  "alsp_msg_type": "auth_request",     // Explicit ALSP Auth Request
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
- **timestamp**
  - Type: string (RFC 3339 UTC)
  - **MUST** be present.
  - **MUST** contain the time at which the message was generated.
  - Receivers **MUST** apply freshness and replay checks as specified in the ALSP authentication section (Section 8.1).
- **session\_nonce**
  - Type: string (UTF-8, fixed length as specified in Section 8)
  - **MUST** be present.
  - **MUST** be freshly generated for this session by the sender and MUST be the same in all messages of the session.
  - **MUST** be used in the JWS protected headers according to the nonce rules defined in Section 8.1.
  - Receivers **MUST** treat this value as the sender’s session nonce for the entire session and **MUST** validate that it does not change in future messages.
- **identity\_cert**
  - Type: string (JWS-encoded Certificate Artipoint)
  - A JWS-encoded JWK containing the client’s identity certificate. This field **MUST** be included in the initial authentication request for **Direct Mode** (Section 8.5.1) and **MUST NOT** be included in the initial authentication request for **Provisioned Mode**.
  - When included, this certificate **SHALL** be used by the server to authenticate the client and to validate all JWS signatures during the session (Trust & Identity §7.2).
  - Informational Note: In **Provisioned Mode**, the identity will be provisioned via `client_key_env` in a subsequent message.
- **recovery\_cert**
  - Type: string (JWS-encoded Certificate Artipoint)
  - This field is used exclusively in **Provisioned Mode** (Section 8.5.2).
  - A recovery certificate **MUST** be included in the initial authentication request for **Provisioned Mode** and **MUST NOT** be included in **Direct Mode**.
- **user\_identity**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **MUST** contain a UTF-8 encoded client side identity reference conforming to the Identity Artipoint structure defined in Trust & Identity §7.1.2. The reference **MUST** be either the `payload` field content (email address or URN, typically) from the Identity Artipoint, OR the UUID of the Identity Artipoint itself.
  - **MUST** match, or be resolvable to, the identity bound to identity\_cert or the identity resulting from the provisioned identity flow.
  - **SHALL** be used for correlation with certificate material but **SHALL NOT** be treated as authoritative without certificate validation.
- **node\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** uniquely identify the sender’s replica node within the ALSP deployment.
  - Implementations **MUST** use this identifier in connection- and session-management logic (for example, enforcing single active session per pair of replicas).

### **10.2.2 Auth Challenge Message**

The `auth_challenge` message is sent by the server during the Challenge Flow when it requires additional identity material from the client beyond what was provided in the initial `auth_request` (see Section 8.4).

The server **MUST** include its identity certificate in all `auth_challenge` messages, and the client **MUST** validate this certificate before using it to verify any subsequent server signatures. In Provisioned Mode, the server also returns the encrypted client key envelope (`client_key_env`) that enables the client to complete authentication.

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
  - In **Direct Mode**, this field **MUST** be omitted.
  - In **Provisioned Mode**, **MUST** be present and the server **MUST** encrypt the user-key-envelope with the `recovery_cert` supplied in the initial authentication request with structure and validation are governed exclusively by Trust & Identity §11.2–§11.3.
- **identity\_cert**
  - Type: string (JWS-encoded Certificate Artipoint)
  - **MUST** provide the challenging server's JWS-signed Certificate Artipoint that binds its identity to a signing key. This certificate **MUST** be used by the client to authenticate the server and to validate all JWS signatures during the session (Trust & Identity §7.2).
  - **MUST** be consistent with the kid used in the JWS protected header for this and all following messages.
- **user\_identity**
  - Type: string (UTF-8)
  - **MUST** be present.
  - **MUST** contain a UTF-8 encoded server side identity reference conforming to the Identity Artipoint structure defined in Trust & Identity §7.1.2. The reference **MUST** be either the `payload` field content (email address or URN, typically) from the Identity Artipoint, OR the UUID of the Identity Artipoint itself.
  - **MUST** match, or be resolvable to, the identity bound to identity\_cert or the identity resulting from the provisioned identity flow.
  - **SHALL** be used for correlation with certificate material but **SHALL NOT** be treated as authoritative without certificate validation.
- **node\_id**
  - Type: string (UUID)
  - **MUST** be present.
  - **MUST** identify the challenger’s replica node.
  - Receivers **MUST** apply the same uniqueness and single-session rules as for auth\_request.

### **10.2.3 Hello Message**

The `hello` message completes mutual authentication and establishes the authenticated session between two ALSP replicas. Both client and server exchange `hello` messages to finalize authentication validation, transitioning the session to the AUTHENTICATED state as defined in Section 8.6.

The sender MUST include either the full identity certificate or a `kid` referencing a certificate already available to the receiver via bootstrap. A `kid` reference MUST NOT be used unless the receiver has previously obtained and validated the referenced certificate through out-of-band provisioning or prior credential exchange.

```json
{
  "alsp_msg_type": "hello",            // ALSP hello message
  "timestamp": "<timestamp>",          // RFC 3339 UTC
  "session_nonce": "utf-8-string",     // Sender's session nonce
  "boot_keys_jwe": "<JWE of BKP>",     // JWE-encoded Bootstrap Key Package
  "lamport_max": 16569909,             // Local Lamport global max value
  "node_id": "uuid",                   // Sender's replica node UUID
  "push_enabled": true,                // Request/confirm push mode
  "node_description": "utf-8-string",  // Friendly node description
  "max_alsp_length": 262144,           // Maximum receive length in bytes
  "user_auth_cert": "cert-or-kid",     // Identity Claim Bundle or kid
  "user_identity": "utf-8-string",     // User / agent identity
  "status_message": "utf-8-string"     // Optional status or error info
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
  - When present, **MUST** contain a Bootstrap Key Package (BKP).
  - The BKP payload structure, semantics, and lifecycle constraints are defined by the **ASCP Bootstrap Process** specification (Bootstrap Section 10.7.6).
  - ALSP treats the decrypted BKP payload as **semantically opaque** and uses it only as an out-of-band bootstrap visibility seed required by higher layers to decrypt encrypted @references entries for deterministic discovery.
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
  - This field **MUST** be present.
  - In **Challenge Flow**, this field **MUST** contain a freshly generated **Identity Claim Bundle (ICB)** bound to the peer’s session\_nonce. The ICB **MUST** satisfy all requirements of Trust & Identity §9 and **MUST** correspond to the identity key used to sign subsequent ALSP messages.
  - In **Immediate Flow**, this field **MAY** instead contain a JOSE key identifier (kid) of the form `ascp:cert:<uuid>` referencing a certificate already available to, and previously validated by, the receiver through bootstrap or prior exchange. A sender **MUST NOT** provide a kid unless the receiver can resolve the referenced certificate without additional provisioning.
  - Receivers **MUST** validate the identity conveyed by this field. When an ICB is supplied, the receiver **MUST** validate the bundle and treat the contained certificate as authoritative for the session. When a kid is supplied, the receiver **MUST** resolve it to a previously validated certificate and **MUST NOT** accept the value if resolution is not possible.
- **user\_identity**
  - Type: string (UTF-8)
  - **MAY** be present and when present:
    - **MUST** identify the user or agent associated with this node and **MUST** be consistent with the identity conveyed or referenced by user\_auth\_cert.
    - **MUST** contain a UTF-8 encoded server side identity reference conforming to the Identity Artipoint structure defined in Trust & Identity §7.1.2. The reference **MUST** be either the `payload` field content (email address or URN, typically) from the Identity Artipoint, OR the UUID of the Identity Artipoint itself.
    - **MUST** match, or be resolvable to, the identity bound to identity\_cert or the identity resulting from the provisioned identity flow.
    - **SHALL** be used for correlation with certificate material but **SHALL NOT** be treated as authoritative without certificate validation.
  - If omitted, receivers **MUST** derive the effective identity from user\_auth\_cert in a manner consistent to the above.
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

## **10.4 Message Authentication**

ALSP protects message integrity and authenticity by wrapping each message in a JWS signature. The complete ALSP Envelope—including both the message header and any payload entries—is signed using the sender's private key, creating cryptographic proof of the message's origin and binding it to the active session. The resulting signed structure is then encoded using DPB (Section 7) to produce the final ALSP frame for transmission over the underlying transport (Section 14).

This approach ensures that every message can be verified against the sender's registered public key, establishing both identity proof and session binding throughout the connection lifecycle. The authentication flows, nonce generation rules, and session establishment procedures that govern this signing process are detailed in Section 8.

### **10.4.1 JWS Header Requirements**

The **JWS Protected Header** MUST be a JSON object containing the fields defined below, SHOULD be serialized using RFC 8785 JCS canonical JSON, and MUST be encoded as UTF-8 prior to BASE64URL encoding as specified in RFC 7515.

#### **JWS Protected Header:**

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
- **MUST** be present on all ALSP messages except when constructing the initial auth\_request in Provisioned Mode. The missing `kid` signifies the absence of a resolvable identity key at this point in the authentication process.
- When present, **MUST** uniquely reference the signing key (identity key) as defined by Section 13 and the **ASCP Trust & Identity Architecture**.
- Implementations **MUST** emit proper kid for all messages once the authentication completes.
- The receiver **MUST** use the kid value to resolve the appropriate public key for signature verification.
- The sender **MUST** ensure that the referenced key is the same key whose private component generated the JWS signature of the message.

**typ:**

- **Type:** string
- **MUST** be present.
- **MUST** equal "alsp".
- Identifies the JWS as carrying an ALSP message and prevents cross-protocol substitution attacks.

**nonce:**

- **Type:** string (UTF-8)
- **MUST** be present with contents as specified in Section 8.1
- Receivers **MUST** reject any message whose protected header contains an unexpected nonce to ensures session binding, replay protection, and cross-session separation.

### **10.4.2 JWS Payload**

The ALSP Envelope (as defined in Section 10.1) serves as the JWS payload. The JWS signature cryptographically protects the entire envelope, including all routing metadata and message content.

### **10.4.3 ALSP Frame (Wire Format)**

An ALSP frame MUST be constructed as a DPB-encoded JWS compact serialization (see Section 7 for DPB encoding), formatted as:

```clike
DPB( <JWSprotectedHeader>.<JWSpayload>.<JWSsignature> )
```

It is the ALSP frame that travels "on the wire" using an underlying transport binding per section 14.

### **10.4.4 Verification Rules**

Receivers **MUST** validate, in order:

1. BASE64URL decoding
2. UTF-8 validity
3. JSON object structure
4. Presence and correctness of required fields
5. Algorithm and key resolution
6. Nonce correctness
7. Signature correctness over the CBOR envelope bytes
8. Timestamp validation per Section 10.4.5

Messages failing any requirement **MUST** be rejected with an ALSP error message per Section 10.5, with one exception: when constructing the initial auth\_request in Provisioned Mode, the receiver **MUST** **defer** signature verification (step 7 above) until the referenced key is available. All other validation steps must pass immediately.

### **10.4.5 Timestamp Validation**

Receiving peers MUST validate the `timestamp` field in the `alsp_msg_header` against their local clock.

Implementations MUST NOT accept messages with timestamps that differ from the receiver's local time by more than 300 seconds (5 minutes), as accepting messages beyond this threshold significantly increases replay attack vulnerability.

Implementations SHOULD reject messages whose `timestamp` value differs from the receiver's local time by more than 60 seconds. This recommended default provides strong replay protection while accommodating reasonable clock skew between peers and network latency.

When a message is rejected due to timestamp violation, the receiver MUST send an ALSP error message (Section 16) with the `stale_timestamp` error code indicating the timestamp validation failure.

## **10.5 Error Cases**

Use the standard ALSP error message. For more guidance please reference to Section 16 on Error handling.

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

ALSP permits a replica to synchronize a channel log **without possessing the keys required to decrypt or interpret the channel’s payloads**. Possession of a valid CAK private key allows a replica to present a Channel Access Proof (CAP) and retrieve channel log entries, regardless of whether the replica can decrypt the encrypted payloads carried within those entries.

Replicas that do not possess the corresponding payload decryption keys MUST treat replicated entries as opaque data. Authorization to replicate a channel MUST NOT be interpreted as authorization to access or interpret channel content.

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

# **12. Bootstrap Key Package Transport**

This section defines how ALSP transports the `boot_keys_jwe` field during session establishment and the constraints placed on ALSP implementations when handling it.

A **Bootstrap Key Package (BKP)** is delivered during the ALSP `hello` exchange and is **not transmitted via ALSP channel logs**. The payload structure, semantics, trust interpretation, and lifecycle of the BKP are defined **exclusively** by the *ASCP Bootstrap Process* specification. ALSP treats the BKP as an **opaque, session-scoped JWE payload**.

Bootstrap Key Packages **do not grant channel access**, **do not authorize replication**, and are **distinct from Channel Access Keys (CAKs)** defined in Section 11.

---

## **12.1 Transport Semantics**

- A BKP, when present, **MUST** be carried only in the `boot_keys_jwe` field of the ALSP `hello` message.
- BKPs **MUST NOT** be transmitted via ALSP Channel Logs, sync messages, or any other ALSP message type.
- ALSP implementations **MUST NOT** persist BKPs beyond the lifetime of the session in which they are received.
- ALSP implementations **MUST NOT** interpret, inspect, or derive semantic meaning from BKP payload contents.

From the perspective of ALSP, the BKP is a **binary artifact delivered out-of-band** whose purpose is to enable higher layers to proceed with bootstrap discovery.

## **12.2 JWE Envelope Requirements**

The `boot_keys_jwe` value **MUST** be encoded using **JWE Compact Serialization**.

ALSP imposes **no requirements** on the internal payload format beyond successful JWE decryption by the recipient.

### **12.2.1 Protected Header Constraints**

ALSP places the following minimal constraints on the JWE protected header:

- The header **MUST** be syntactically valid JOSE JSON.
- The `kid` field:
  - **SHOULD** reference the recipient’s identity certificate using the form  
    `ascp:cert:<uuid>` *when such a reference is known and resolvable by the sender*.
  - **MUST** be the empty string (`""`) if no such certificate reference exists at the time of wrapping.
- ALSP **MUST NOT** assign or interpret any semantic meaning to the `kid` value beyond enabling JWE processing.

All other protected header fields (including `alg`, `enc`, and key agreement parameters) are governed by the *ASCP Bootstrap Process* specification and applicable JOSE standards, not by ALSP.

## **12.3 Session Binding and Authenticity**

A BKP is accepted solely on the basis that it was delivered over an **authenticated ALSP session**.

ALSP authentication establishes **provisional cryptographic trust sufficient for secure transport**. Authoritative trust anchoring — including RootCA validation, governance enforcement, and legitimacy evaluation — occurs **only after bootstrap artifacts are acquired and validated by higher layers**.

This separation is intentional and allows secure bootstrap discovery without circular trust dependencies.

- ALSP **does not define**, require, or prohibit any additional trust validation of BKPs.
- ALSP **MUST NOT** specify bootstrap trust interpretation rules or mandate validation against bootstrap trust roots.

Any trust evaluation related to BKP contents is performed entirely by higher layers under the ASCP Bootstrap Process.

## **12.4 Lifetime and Scope (ALSP View)**

From the ALSP perspective only:

- A BKP is **session-scoped**.
- A BKP is **ephemeral**.
- A BKP **MUST NOT** be reused across sessions.
- A BKP **MUST NOT** be cached, replayed, or referenced after session termination.

ALSP does not track bootstrap epochs, rotation state, or historical validity of BKPs.

## **12.5 Error Handling**

When handling `boot_keys_jwe`, ALSP implementations:

- **MUST** return `protocol_violation` for malformed JWE structures.
- **MUST** return `invalid_auth` when JWE decryption fails due to cryptographic errors.
- **MUST NOT** use authorization-related error codes (e.g., `unauthorized`) to signal BKP handling failures.

Error responses **SHOULD NOT** disclose whether failures were caused by key absence, decryption failure, or payload corruption.

## **12.6 Out-of-Scope Semantics**

The following are explicitly **out of scope** for ALSP and are defined by the ASCP Bootstrap Process specification:

- BKP payload structure
- Channel Key Envelope semantics
- Discovery rules
- Key indexing and selection
- Key rotation and epoch overlap
- Decryptability guarantees for any channel

ALSP’s role is limited to **opaque, authenticated transport** of the Bootstrap Key Package during session establishment.

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

## **13.5 Relationship to Bootstrap Artifacts (Informative)**

Layer 0 relies on key material that is introduced via the ASCP Bootstrap Process and referenced by stable `kid` values. From the perspective of ALSP, these materials function as **immutable inputs** for signature verification and authorization checks during the lifetime of an authenticated session.

ALSP:

- Does **not** define or evaluate trust semantics
- Does **not** perform revocation checking or policy interpretation
- Does **not** reason about historical or future validity of keys

Instead, ALSP assumes that any key material it is asked to resolve via a supported `kid` was already validated, authorized, and published according to higher-layer rules at the time it was made available through bootstrap artifacts (e.g., certificate directories or channel manifests).

This design allows Layer 0 to:

- Perform deterministic verification of signatures and access proofs
- Preserve historical verifiability of replicated logs
- Remain agnostic to identity lifecycle, governance, and trust policy evolution

All trust interpretation, lifecycle management, revocation semantics, and governance rules are defined exclusively by the *ASCP Trust & Identity Architecture* and related higher-layer specifications.

### **13.5.1 Bootstrap Key Index Identifiers (Informative)**

As part of higher-layer bootstrap operations, encrypted payloads MAY reference key identifiers of the form:

```asciidoc
ascp:bkp:<index>
```

These identifiers are defined by the *ASCP Bootstrap Process* specification and are used exclusively for bootstrap-scoped key selection above Layer-0.

ALSP implementations:

- **MUST NOT** attempt to resolve, validate, or interpret `ascp:bkp:*` identifiers
- **MUST NOT** treat such identifiers as UUID-addressable keys
- **MUST NOT** apply authorization, trust evaluation, or key lookup semantics to them

From the ALSP perspective, `ascp:bkp:*` identifiers may appear only as opaque values within encrypted payloads processed by higher layers. Their structure, indexing rules, lifecycle, and semantics are entirely outside the scope of this specification.

# **14. Transport Bindings**

ALSP is transport-agnostic and can operate over any bidirectional, ordered, reliable delivery mechanism. This section defines the baseline requirements for interoperable deployments and provides guidance for WebSocket/TLS bindings, which serve as the default transport profile.

**Unless explicitly stated otherwise, descriptive text in this section is informational; only statements using RFC 2119 terminology are normative.**

## **14.1 Baseline Transport Profile**

Implementations **MUST** support WebSockets over TLS 1.3 as the baseline ALSP transport. Other transports (e.g., raw TCP, HTTP/2 CONNECT, HTTP/3, gRPC) **MAY** be supported but are not required for baseline interoperability.

All ALSP messages sent over WebSockets **MUST** use binary frames; text frames **MUST NOT** be used.

Transport-level compression (e.g., permessage-deflate) **SHOULD** be disabled to avoid known compression side-channel attacks.

Each new transport connection **MUST** establish a new ALSP session; ALSP sessions **MUST NOT** span multiple transport connections.

Transport-level failures (TLS alerts, WebSocket close frames, underlying socket failures) are **not** represented using ALSP error messages. Upon any transport failure, peers **MUST** initiate a new ALSP session and resume synchronization via sync\_request.

## **14.2 TLS Requirements**

All ALSP deployments using TLS **MUST** use TLS 1.3 or higher.

Clients **MUST** validate the server certificate when establishing a TLS-protected transport, unless operating in an explicitly configured development or testing mode. Certificate validation **SHOULD** follow standard PKIX verification procedures.

Server-side certificate rotation **MAY** occur without affecting ALSP session semantics; new connections will naturally establish new ALSP sessions.

TLS authentication provides baseline network-level security and resistance to active man-in-the-middle attacks. It does **not** establish ASCP identity, instance membership, or channel trust.

ALSP session authentication and ASCP Bootstrap trust anchoring remain the authoritative mechanisms for identity verification and trust establishment within the ASCP protocol stack. TLS is treated as **necessary but insufficient** for ASCP trust.

## **14.3 WebSocket Binding**

When using WebSockets:

- The client **MUST** initiate the WebSocket handshake.
- The server **MUST** advertise acceptance of the ALSP subprotocol (if defined).
- The connection **MUST** be established over TLS 1.3.
- WebSocket extensions providing compression **SHOULD NOT** be negotiated.
- Implementations **MAY** use WebSocket close code 1000 (normal closure) or 1002 (protocol error) to terminate connections.

WebSocket pings/pongs are transport operations and do not affect ALSP message sequencing.

## **14.4 HTTP Upgrade and Extended CONNECT**

ALSP **MAY** be deployed using HTTP/1.1 Upgrade, HTTP/2 extended CONNECT, or HTTP/3 CONNECT. These profiles **SHOULD** maintain the same security properties and MUST preserve ordered, bidirectional message delivery.

Implementations supporting multiple HTTP transports **MUST** ensure consistent ALSP session semantics across all bindings.

## **14.5 Alternative Transports**

Deployments **MAY** define additional bindings (e.g., raw TCP with TLS, QUIC streams, or gRPC streaming APIs). Any alternative binding:

- **MUST** support ordered, reliable delivery.
- **MUST** preserve ALSP message boundaries exactly as encoded.
- **MUST** provide equivalent security properties to WebSocket/TLS.
- **MUST** begin a new ALSP session on transport establishment.

Future companion documents may specify additional transport profiles.

## **14.6 WebSocket Connections (Non-Normative)**

This section provides implementation guidance for establishing WebSocket connections that satisfy the normative requirements defined in sections 14.1-14.5. The following procedure describes a typical connection establishment sequence:

1. **Endpoint and scheme.** For a given domain, implementations typically use a WebSocket Secure (WSS) origin at port 443, such as wss\://ascp.\<domain>. Clients generally use the wss scheme and TCP port 443 unless explicitly configured otherwise.
2. **TLS and ALPN.** The client initiates a TLS handshake to the origin on TCP/443. During the handshake, the client offers ALPN protocols h2 and http/1.1, and the server selects one. Implementations MUST support TLS 1.3; TLS 1.2 support is optional.
3. **WebSocket establishment.**
   - If ALPN selects HTTP/2, the client and server establish the WebSocket using the Extended CONNECT mechanism per RFC 8441 with :protocol = "websocket".
   - If ALPN selects HTTP/1.1, the client and server establish the WebSocket using the standard HTTP/1.1 Upgrade handshake.
4. **Connection model.** After the WebSocket is established, ALSP operates over a persistent, full-duplex TCP connection. Application data is carried as WebSocket binary messages.
5. **Compression.** Implementations typically avoid negotiating per-message compression extensions (e.g., per-message-deflate) for ALSP frames, as Layer 1 payloads already employ JWE compression where appropriate, and compression can introduce security vulnerabilities like CRIME and BREACH attacks.
6. **Keepalive and liveness.** Endpoints typically use WebSocket Ping/Pong frames to detect half-open connections and to maintain NAT/firewall mappings at reasonable intervals.
7. **Subprotocol identification.** Servers typically advertise, and clients typically request, the WebSocket subprotocol token `ascp.alsp.v1` (where v1 corresponds to ALSP version 0.x for backward compatibility) via Sec-WebSocket-Protocol to enable policy and routing at intermediaries. If the subprotocol is not accepted, the connection may proceed without it.
8. **Proxy traversal.** When traversing explicit HTTP proxies, clients may use HTTP CONNECT tunneling prior to TLS, per HTTP semantics. Once tunneled, the requirements in (2) and (3) apply. Proxy traversal is outside the scope of ALSP and follows standard HTTP semantics.
9. **Error handling and reconnect.** On transport failure (TLS or WebSocket), clients typically immediately re-establish a new WSS connection and resume ALSP synchronization per the ALSP state machine (hello/auth and channel resubscription rules defined elsewhere in this specification).
10. **WebSocket close handling.** When ALSP protocol errors occur, implementations typically close the WebSocket with code 1002 (Protocol Error) and include the ALSP error\_code in the close reason where possible.

### **Message Framing**

Each WebSocket binary message MUST contain exactly one ALSP Frame as defined in Section 10.4.3. WebSocket fragmentation is handled by the transport layer and is transparent to ALSP.

# **15. Node Topology and Deployment Models**

ALSP can operate in a variety of network configurations due to the symmetry of the protocol. Every replica implements the same ALSP logic; the only practical distinction between nodes is whether they **accept inbound connections**, **initiate outbound connections**, or **do both**.

**Unless explicitly stated otherwise, topology descriptions in this section are informational; only statements using RFC 2119 terminology are normative.**

Deterministic ordering (Section 12), authentication, and session semantics apply uniformly across all topologies. Each transport connection **MUST** establish a new ALSP session (Section 14).

## **15.1 Client–Server Model (Informational)**

In a client–server deployment, multiple clients establish outbound connections to a common server. The server maintains a consolidated replica of each Channel Log and participates in synchronization with all connected clients.

This model is common when replicas cannot directly reach each other or when a single shared replica simplifies operational management. The protocol behavior remains identical for all participants.

## **15.2 Peer-to-Peer Model (Informational)**

In a peer-to-peer deployment, replicas connect directly to one another in an ad hoc manner. Any replica may initiate outbound connections, accept inbound connections, or both. Nodes may maintain simultaneous sessions with multiple peers to improve resilience and reduce latency.

Peer-to-peer topology does not alter ALSP semantics: ordering, divergence detection, and authentication all function exactly as in the client–server model.

## **15.3 Hybrid Model (Informational)**

Hybrid deployments combine elements of both previous models. Some replicas form direct peer-to-peer connections, while others connect through one or more nodes that accept inbound connections. Hybrid topologies often arise naturally in environments with heterogeneous network constraints.

ALSP does not impose any restrictions on such mixed deployments. All replicas follow the same synchronization rules regardless of how they are connected.

## **15.4 Push-Mode Subscription (Partially Normative)**

ALSP operates in one of two synchronization modes:

**Pull Mode (Default)**: Updates must be explicitly requested via sync\_request messages. The peer responds with available messages but does not send automatic updates.

**Push Mode (Optional)**: After initial synchronization, the peer automatically sends sync\_update messages when new messages become available for subscribed channels.

Push-mode is an optional optimization where a replica requests proactive delivery of new Channel Log entries. A replica requesting push-mode **MUST** be prepared to fall back to pull-mode if the peer does not advertise support.

The sequence descriptions and examples for push-mode are **informational** and do not define a prescriptive state machine; only statements using RFC 2119 terminology are normative.

### **15.4.1 Negotiating Push Mode**

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

### **15.4.2 Channel Subscription State Machine**

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

## **15.5 Multi-Peer Connectivity (Informational)**

Replicas **MAY** maintain concurrent ALSP sessions with multiple remote endpoints. This is common in peer-to-peer and hybrid deployments and increases robustness against network partitions or partial failures.

Regardless of connectivity pattern:

- Ordering semantics **MUST** follow Section 12.
- Session establishment and authentication **MUST** follow Section 8 and Section 14.
- Divergence detection and recovery **MUST** follow Section 17.

Topology does not affect any protocol guarantees.

# **16. Error Handling**

ALSP defines a structured error message used to signal protocol violations, authentication failures, authorization problems, and capability mismatches between replicas. Error messages are carried in the standard ALSP envelope and follow the same JWS signing and nonce-binding rules as all other ALSP messages.

**Transport-Level Failures:**

Transport-level failures (e.g., WebSocket disconnects, TLS alerts, timeouts) are not conveyed using ALSP error messages. Implementations MUST treat these events as error conditions and proceed with reconnection and resynchronization behavior as described in Section 16.5.

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

| Description          | Force Disconnect?                                      | Notes                                                                                          |
| -------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| invalid\_auth        | YES                                                    | Certificate does not match claimed identity or is untrusted.                                   |
| unauthorized         | NO unless it appears to be a DoS attack                | Replica is not permitted to access the requested channel.                                      |
| hash\_mismatch       | NO unless it appears to be a DoS attack                | A validation SHA-256 hash mismatch has occurred and this implies log divergence.               |
| stale\_timestamp     | NO unless it appears to be a DoS attack                | The message was rejected due to the timestamp being out-of -range. Replay attack is suspected. |
| protocol\_violation  | YES                                                    | Message structure or sequence violated ALSP semantics.                                         |
| unsupported\_version | YES - after session authentication, otherwise OPTIONAL | Peer sent an incompatible message format or spec version.                                      |
| payload\_too\_large  | NO unless it appears to be a DoS attack                | Message exceeds max\_alsp\_length or violates buffer constraints.                              |
| internal\_error      | OPTIONAL                                               | Sender encountered an unexpected failure.                                                      |
| re-auth-required     | YES                                                    | This error can be generated to force the termination of a sync session.                        |

## **16.3 Required Behavior**

Implementations MUST handle malformed or invalid messages gracefully without terminating the ALSP session unless explicitly required by an error message with `disconnect: true`.

When encountering invalid messages, replicas SHOULD log the error details and continue processing any remaining valid messages in the same sync batch.

Replicas MUST validate that all required fields are present and correctly typed before processing any sync message, and SHOULD send appropriate error responses for protocol violations.

**Recovery model:** All error recovery follows the same pattern as normal sync operations - establish connection, negotiate capabilities via hello, and request missing data via sync\_request. After any transport or protocol failure, peers **MUST** initiate a new ALSP session and resume synchronization using sync\_request. ALSP’s idempotent message semantics ensure that this behavior results in deterministic recovery and eventual convergence.

- Error messages may be sent in response to any ALSP message.
- A hello response with an error code is a valid rejection of the handshake.
- The receiver of an error message SHOULD log the error and MUST honor the disconnect directive if set to true.

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

## 16.6 Error Message Example (Non-Normative)

```c
{
  "alsp_msg_type": "error",             // Error message type identifier
  "timestamp": "<rfc-3339-timestamp>",  // Timestamp of this message
  "error_code": "invalid_auth",
  "reason": "Signature did not validate with referenced key",
  "disconnect": true,
  "suggested_action": "Re-establish trust material or re-bootstrap"
}
```

# **17. Channel Log Health Check & Recovery**

While ALSP ensures deterministic, convergent log synchronization under normal operation, real-world systems may encounter channel divergence due to implementation bugs, storage faults, or partial sync failures. This section defines normative and recommended practices for detecting and recovering from such divergence scenarios.

## **17.1 Digest Hash Exchange**

Implementations **SHOULD** periodically compute a deterministic digest representing the current state of a Channel Log. When supported, the digest **MUST** be computed using the algorithm and canonical ordering defined below:

- The digest **MUST** be sha256: followed by the hexadecimal SHA-256 output.
- The digest **MUST** be computed over the concatenation of message\_id values (UUID value of 16 raw bytes) for all Layer-0 entries included in the Channel Log in **canonical log order**.
- Canonical log order **MUST** be defined by the ordering rules of Section 12 (Lamport ordering with global tie-breaking).

Any implementation that includes log\_digest in a sync\_update message **MUST** follow this computation procedure.

Any receiver of a log\_digest value in an ALSP message MUST compute the corresponding local log\_digest for the same set of messages its own replica, and if there is a mismatch, it MUST send a hash\_mismatch error to the original sender.

No other actions are required, but the receiver MAY attempt a recovery operation per Section 17.3.

## **17.2 Range Consistency Checking**

A mismatch **MUST** be interpreted as one or more of the following conditions:

- The local log is missing entries.
- The local log includes extra entries not present on the peer.
- The local log contains entries in a different order.
- Local storage has been corrupted.
- The peer is sending incorrect data due to misbehavior or implementation error.

If a digest mismatch is detected, the receiving node **SHOULD** initiate divergence recovery as defined in Section 17.3.

## **17.3 Divergence Recovery Options**

If divergence is detected, implementations **SHOULD** attempt recovery using one or more of the following mechanisms:

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

Future iterations of ALSP may define optional Merkle tree summaries per channel log to support more efficient divergence detection. Such structures would be used only for replica health and convergence validation and would not participate in the trust model defined at higher layers. This enables detection of localized divergence with minimal hash exchange. See Section 9.9.

## **17.5 Operational Best Practices (Informational)**

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

Security in ALSP is layered across the ASCP stack. Layer-0 provides integrity, ordering, authentication, and replay protection for Channel Log replication; confidentiality, authorship, and access control are defined in the Layer-1 and Layer-2 specifications. The following considerations summarize the intended security posture of ALSP. These topics will be expanded into a full Security Considerations section as the specification matures toward an Internet-Draft.

## **19.1 Intended Guarantees (High-Level)**

ALSP aims to provide:

- **Authenticated messaging** — all ALSP messages are JWS-signed and bound to a session nonce.
- **Integrity and non-malleability** — tampered messages are rejected before processing.
- **Replay resistance** — stale timestamps, reused nonces, or duplicate message identifiers are rejected.
- **Deterministic ordering** — adversaries cannot induce divergent log orderings.
- **Authorized replication** — only replicas with valid Channel Access Keys may replicate logs.
- **Divergence detection** — digest mismatches allow detection of tampering or implementation faults.

These form the minimum required security properties at Layer-0; confidentiality and authorship claims are handled by higher layers.

## **19.2 Threats Under Consideration**

The protocol is designed to resist:

- Passive or active network attackers (tampering, replay, message substitution)
- Malicious or misconfigured peers
- Accidental or intentional divergence of channel logs
- Resource-exhaustion behavior during sync sessions

Further formal threat modeling will be incorporated in future drafts following RFC 3552 guidance.

## **19.3 Interaction With Higher Layers**

Because ALSP transports semantically opaque Layer-1 envelopes:

- Confidentiality is provided by **Layer-1 JWE encryption**, not ALSP.
- Authorship and governance semantics are provided by **Layer-2 articulation signatures**.
- Identity binding and trust root management rely on the **ASCP Identity & Trust** specification.

ALSP implementers MUST validate signatures and MUST NOT alter or reinterpret Layer-1 payloads.

## **19.4 Areas for Future Expansion**

As this specification moves toward an RFC track, this section will be expanded to include:

- A formal threat model
- Detailed attack classes and required mitigations
- Privacy considerations
- Operational guidance for key rotation, trust-root updates, and divergence handling
- Cross-protocol substitution and downgrade-resistance analysis
- DoS considerations and rate-limiting guidance
- Security interactions with push-mode sync and multi-transport deployments

These topics are intentionally deferred until the protocol has undergone further public review and practical implementation feedback.

# **20. IANA Considerations**

At this stage of the ALSP specification, **no IANA actions are requested**. As the protocol matures toward an Internet-Draft, this section will be expanded to define any required registries, including:

- ALSP message types
- ALSP error codes
- ALSP WebSocket subprotocol identifiers
- JOSE header parameter values specific to ALSP
- Version negotiation tokens or transport-binding identifiers

These registries will be specified once the wire format, error taxonomy, and transport bindings have stabilized through community review and early implementation experience.

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