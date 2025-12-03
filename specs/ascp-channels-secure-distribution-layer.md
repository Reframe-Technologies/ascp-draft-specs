# ASCP Channels: Secure Distribution Layer Specification

**Layer 1 of the ASCP Protocol Stack**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.41 — Informational (Pre-RFC Working Draft)  
November 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the ASCP specification suite and defines Layer 1 of the protocol stack: ASCP Channels, the secure distribution and encryption infrastructure that enables collaborative cognition between humans and AI agents. It is published at this time to gather community feedback on the cryptographic architecture, channel management semantics, and interoperability of the secure distribution layer.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# **2. Abstract**

The ASCP Channels layer defines the secure distribution substrate for articulated context within the Agents Shared Cognition Protocol. Channels provide a cryptographically verifiable, append-only transport for Artipoint payloads, enabling multi-writer replication across devices and agents in a local-first environment. Channels use JOSE-based mechanisms—JWS for authorship integrity and optional JWE for payload confidentiality—to produce tamper-evident, recipient-restricted message streams.

Channels operate strictly at Layer-1 of the ASCP architecture. They preserve envelopes in an immutable log and perform no semantic interpretation of the payload. All semantic governance—including membership, authorship permissions, and role semantics—is defined at higher layers and is not evaluated by this specification. Instead, Layer-1 receives externally provisioned key material and recipient sets derived from governance evaluation.

Keyframes, expressed as Layer-2 Artipoints, describe the intended cryptographic configuration of a Channel, including key versioning and rotation. Layer-1 executes these Keyframes by generating and consuming per-recipient JWE key envelopes and maintaining consistent cryptographic state across replicas.

This specification defines the Channel architecture, cryptographic model, envelope formats, Keyframe execution semantics, and normative sender/receiver processing rules. It does not define semantic visibility, access-control logic, identity trust roots, or governance interpretation, all of which are specified elsewhere in the ASCP suite.

# **3. Introduction**

## **3.1 Motivation: Secure Local-First Distribution**

Secure, durable, and verifiable distribution of articulated context is a foundational requirement for the Agents Shared Cognition Protocol (ASCP). Human–AI collaboration depends on the ability to exchange structured contributions reliably across devices and agents, maintain a consistent shared history, and ensure authenticity and confidentiality over time. Channels provide the Layer-1 substrate that enables this form of secure, local-first replication in multi-writer environments.

## **3.2 Position of Channels in the ASCP Layer Model**

A Channel is an append-only log of cryptographically protected envelopes. Every envelope contains a signed Artipoint payload expressed in the ASCP Grammar and MAY additionally be encrypted for restricted audiences. Channels authenticate and protect contributions using JOSE-based mechanisms—JWS for authorship integrity and optional JWE for confidentiality—while ensuring tamper-evident, convergent replication across replicas.

Channels operate strictly at Layer-1 and interpret no semantic meaning from payloads. Semantic governance—including membership, authorship permissions, roles, and inheritance—is defined at Layer-2 and resolved by Layer-3 view evaluators. Channels simply store and transport envelopes; they neither enforce governance policy nor infer semantic meaning. This separation preserves stability and determinism in the transport layer even as semantic structures evolve.

## **3.3 Scope of This Specification**

This specification defines:

- The architecture of ASCP Channels at Layer-1
- The JOSE-based cryptographic mechanisms used to sign and optionally encrypt payloads
- The normative structure of Channel envelopes
- The execution semantics for Keyframes as cryptographic configuration statements
- Replica processing rules for receiving, verifying, decrypting, and materializing envelopes
- Requirements for consistent, append-only log replication in a local-first environment

## **3.4 Out of Scope**

This specification does **not** define:

- Semantic governance, including membership, authorship permissions, roles, or visibility rules
- Identity trust roots or PKI/endorsement semantics (defined in ASCP Identity & Trust)
- Evaluation of governance attributes or inheritance logic (defined in ASCP Governance)
- Interpretation or materialization of Artipoint semantics (handled by Layer-2/Layer-3 components)
- Application-level behavior, UI conventions, or workflow semantics

Keyframes, while expressed as Artipoints at Layer-2, are only interpreted here for their **cryptographic** effects. All semantic reasons for issuing or modifying Keyframes belong to the Governance layer.

# **4. Terminology**

## **4.1 Channel**

A **Channel** is a Layer-1 secure distribution context represented as an append-only log of cryptographically protected envelopes. Channels preserve ordering and integrity but interpret no semantic meaning from payload content.

## **4.2 Envelope**

An **Envelope** is the JOSE-encoded container transported by a Channel. Each envelope is signed using JWS (authorship integrity) and MAY be encrypted using JWE (payload confidentiality). Envelopes form the immutable log entries replicated across Channel replicas.

## **4.3 Artipoint Payload**

The **Artipoint Payload** is the opaque, cleartext content carried inside an envelope. It is expressed using the ASCP Artipoint Grammar, but Channels do not parse or interpret it.

> *See: ASCP Grammar Specification.*

## **4.4 Replica**

A **Replica** is a local instance of a Channel hosted by a device, agent, or service. Replicas exchange append-only logs using the ASCP Log-Sync Protocol (ALSP), which is topology-neutral and supports peer-to-peer, local-first, and service-mediated synchronization.

> *ALSP does not depend on any centralized server, but may use one.*

## **4.5 Keyframe**

A **Keyframe** is a Layer-2 Artipoint that declares the intended cryptographic configuration of a Channel (e.g., symmetric keys, algorithm selection, key identifiers). Layer-1 does not parse Keyframes; instead, it receives externally provisioned key material and recipient sets derived from Keyframe evaluation performed by Layer-3.

## **4.6 Key Identifier (kid)**

A **Key Identifier (kid)** is an opaque, stable identifier used in JOSE headers to reference the key version for verifying signatures and decrypting payloads.

## **4.7 Channel AES Key**

The **Channel AES Key** is the symmetric encryption key associated with a specific Keyframe version via Keyframe provisioning from Layer-3. All recipients of an encrypted envelope receive (wrapped) access to the same symmetric key for that version.

Layer-1 receives this key from a higher-layer evaluation process; it is not derived from payload content.

## **4.8 Channel Access Key (CAK)**

The **Channel Access Key (CAK)** controls replication rights at the transport layer. It authorizes which replicas MAY exchange Channel envelopes, independent of semantic permissions.

## **4.9 Author**

The **Author** is the Layer-2 semantic identity responsible for the Artipoint payload.

Authors are determined by governance and identity verification rules. The Author’s certificate, bound via purpose::assert, is used to produce the JWS signature that asserts authorship over the payload.

Layer-1 verifies author signatures but does not determine authorship semantics.

## **4.10 Sender**

The **Sender** is the Layer-1 operational role that constructs an envelope. The Sender:

- renders the Artipoint payload into a JWS-signed envelope using the **Author’s key**
- optionally encrypts the signed payload using the Channel AES key
- commits the envelope into the Channel log via ALSP

The Sender is *typically* the Author but may be a delegated agent or service acting on behalf of the Author. Layer-1 verifies signatures using the Author’s key; Sender identity plays no direct role in cryptographic verification.

## **4.11 Recipient (Recipient Set)**

A **Recipient** is an identity for whom a decryptable envelope is produced. Recipients are supplied to Layer-1 as part of Keyframe-driven key provisioning. All recipients of a given Keyframe share the same symmetric Channel AES key for decrypting JWE envelopes.

Layer-1 does not determine who recipients are; it simply enforces encryption for the provided set.

> *Recipients are conceptually a set (Recipient Set), not a symmetric counterpart to Sender.*

## **4.12 ASCP Log-Sync Protocol (ALSP)**

The **ASCP Log-Sync Protocol (ALSP)** is the Layer-0 transport mechanism responsible for replicating append-only logs between replicas. It guarantees durability and convergence but performs no cryptographic validation.

## **4.13 Immutable Log**

The **Immutable Log** is the ordered sequence of envelopes stored by a Channel. Once appended, envelopes MUST NOT be modified or deleted.

## **4.14 JOSE (JWS and JWE)**

**JOSE** refers to the standards suite used by Channels.

- **JWS** provides authorship integrity and non-repudiation.
- **JWE** provides optional payload confidentiality and audience restriction.

Detailed algorithm requirements are defined in Section 6.

# **5. Architectural Overview**

## **5.1 Channels as Layer-1 Secure Transport**

Channels provide the secure distribution layer of ASCP. Each Channel is an append-only log of cryptographically protected envelopes, replicated across devices and agents. Envelopes carry Artipoint payloads expressed in the ASCP Grammar and are always signed; encryption may be applied when confidentiality is required. Channels guarantee that contributions remain authentic, tamper-evident, and durably replicated, even across intermittently connected or topology-diverse environments.

Although Channels are central to the integrity of ASCP collaboration, they remain structurally simple: they transport opaque payloads and do not interpret their semantic meaning.

## **5.2 Relationship to Layer-0 (ALSP)**

Channels operate directly above the **ASCP Log-Sync Protocol (ALSP)**, which synchronizes append-only logs across replicas. ALSP is topology-neutral: peers may synchronize directly, through shared services, or via a combination of both. ALSP provides ordered replication and convergence but performs no cryptographic validation. Channels rely on ALSP for durability and delivery while adding the cryptographic assurances that ALSP does not provide.

## **5.3 Relationship to Layer-2 (Grammar)**

All payloads carried by a Channel are expressed using the ASCP Artipoint Grammar. Channels do not parse or evaluate the grammar; to Layer-1, the payload is simply the cleartext (or decrypted) byte sequence that the sender has signed. Grammar defines the structured form of collaboration, but Channels concern themselves only with securely transporting those structures.

## **5.4 Relationship to Layer-3 (View Evaluation)**

Layer-3 evaluates the meaning of the payloads—governance, authorship semantics, visibility rules, inheritance, and effective state construction. Channels neither enforce nor derive these semantics. Instead, they receive cryptographic configuration from higher layers, including which keys to use and which recipients should receive decryptable envelopes. This allows Channels to remain stable and deterministic while the semantic layer above them evolves.

## **5.5 Channels Do Not Interpret Semantic Governance**

Channels maintain a strict separation from higher-layer governance and MUST NOT parse governance attributes, interpret membership or roles, or apply structural rules. Their responsibility is limited to validating signatures, applying encryption, and preserving envelopes in an immutable log. All meaning is resolved outside this layer.

## **5.6 Immutability, Ordering, and Multi-Writer Behavior**

Channels provide a uniform set of transport guarantees. The log is immutable: once appended, envelopes cannot be altered or removed. Replicas converge on a consistent order of envelopes through ALSP synchronization. Channels support multi-writer collaboration—any entity authorized by higher-layer governance may produce envelopes, and the Channel will accept them so long as they satisfy the required cryptographic checks. Replicas may temporarily diverge but will reconverge deterministically as ALSP exchanges histories.

Together, these properties establish Channels as a reliable, cryptographically sound substrate for distributed collaboration within ASCP.

# **6. Cryptographic Foundations**

Channels rely on the JOSE standards suite—**JWS** for authorship integrity and **JWE** for optional payload confidentiality—to secure the append-only logs replicated across ASCP replicas. Layer-1 applies these cryptographic protections deterministically and without interpreting the semantic content of payloads. All semantic authorization, role semantics, and governance logic occur at higher layers.

A Channel’s cryptographic configuration is expressed declaratively in Layer-2 Keyframes and interpreted by Layer-3. Layer-1 receives the resulting key material and cryptographic parameters through provisioning and applies them during envelope construction and verification.

The subsections that follow define the JWS requirements, optional JWE encryption, supported algorithms, the ASCP-specific kid format, symmetric key requirements, and the integrity model governing all Channel operations.

## **6.1 JWS for Authorship Signatures**

Every envelope written to a Channel MUST be signed using **JWS Compact Serialization** as defined in **RFC 7515 §3.1**.

The JWS signature provides authorship integrity for the Artipoint payload and protects the JOSE header.

Layer-1 MUST:

- verify the JWS signature using the public key identified by the JOSE kid header
- ensure that the signature algorithm and public corresponds to an Author Certificate  with `purpose::assert`
- reject any envelope with a missing or unknown `kid`
- treat the JWS payload as opaque byte content

The **Author** determines the payload and the signing key. The **Sender** may be the same entity or an agent acting on behalf of the Author; Layer-1 authenticates with the Author’s key, not the Sender’s identity.

## **6.2 JWE for Optional Payload Confidentiality**

Channels MAY encrypt signed payloads using **JWE Compact Serialization** as defined in **RFC 7516 §3.1**.

JWE encryption MUST follow the **sign-then-encrypt** pattern:

1. A JWS envelope is produced.
2. The JWS output becomes the payload for JWE encryption.
3. Encryption uses the Channel AES Key corresponding to the active Keyframe.

Key properties:

- All recipients for a given Keyframe share the same symmetric key.
- Channel AES Keys are provisioned to Layer-1 and MUST NOT be derived from payload content.
- The JOSE enc and alg fields MUST identify the content-encryption and key-agreement methods.
- Payloads MUST NOT be encrypted without a valid JWS signature.

Header usage is specified further in Section 8 (Envelope Format).

## **6.3 Supported Algorithms and Header Requirements**

### **6.3.1 JWS Signature Algorithms (Required)**

- `ES256` (ECDSA P-256 + SHA-256)
- `EdDSA` (Ed25519)

**Optional**: ES384 (ECDSA P-384)

The signature algorithm MUST match a purpose::assert certificate binding in the Author’s Identity.

### **6.3.2 JWE Content Encryption Algorithms**

**Required:**

- `A256GCM` (AES-GCM 256-bit)

### **6.3.3 Key Agreement / Wrapping Algorithms**

Used only for distributing Channel AES Keys to recipients:

**Required:**

- `ECDH-ES` (Direct key agreement; RFC 7518 §4.6)

**Optional:**

- ECDH-ES+A256KW

The algorithm MUST be compatible with the recipient’s purpose::keyAgreement certificate.

### **6.3.4 JOSE Header Requirements**

For all Channel envelopes:

- typ SHOULD be "ascp+jws" or "ascp+jwe".
- kid MUST reference the correct key version.
- alg MUST identify the signature or key-agreement method.
- enc MUST appear on JWE envelopes.

Sections 8.2–8.4 define normative header structure and error-handling rules.

### 6.3.4 Summary of JOSE Usage in ASCP

| Use Case                        | Protocol | Format                | Algorithm(s)        | Key Material Source                                                       |
| ------------------------------- | -------- | --------------------- | ------------------- | ------------------------------------------------------------------------- |
| Sign Artipoints                 | JWS \*   | Compact Serialization | ES256 (ECDSA P-256) | EC public identity key pair (public key referenced by kid for validation) |
| Encrypt Artipoint Messages      | JWE \*   | Compact Serialization | AES-256-GCM         | Channel AES key (out-of-band referenced via kid)                          |
| Distribute Channel AES Key      | JWE \*\* | JSON Serialization    | ECDH-ES + A256KW    | EC public identity key (referenced by kid)                                |
| Distribute Ed25519 Auth Key     | JWE \*\* | JSON Serialization    | ECDH-ES + A256KW    | EC public identity key (referenced by kid)                                |
| Identity for Signing / Wrapping | JWK      | JSON                  | ES256 / Ed25519     | Embedded or linked via kid                                                |

Notes for the table:

- \* - Messages use Compact Serialization
- \*\* - Keyframe envelopes (wrapped JWK) use Flattened JSON JWE.

## 6.4 kid Format and Interpretation

ASCP defines a structured `kid` namespace for deterministic key selection across Layers 1–3.. Each `kid` references cryptographic material stored out-of-band in articulation statements, acting as a pointer to a certificate or keyframe within the channel's log. This allows JOSE operations to resolve the appropriate material for message validation, decryption, or key recovery while remaining compliant with JOSE standards.

While `kid` is optional in JOSE specifications, ASCP **MUST** include `kid` explicitly in all message headers to ensure determinism during key rotation, avoiding ambiguity when multiple writers generate messages while updated key material is still propagating through the network.

**Note**: Layer-1 only uses the kid to select the key material supplied by provisioning from higher level layers.

### 6.4.1 Kid Format Specification

The `kid` values must be a string formatted according to the following:

```plaintext
ascp:<type>:<uuid>
```

Where:

- `ascp` identifies this as an ASCP-specific key identifier
- `<type>` indicates the kind of cryptographic material being referenced
- `<uuid>` is the UUID of the articulation statement containing the key material

### 6.4.2 Supported Types

`ascp:cert:<uuid>` - References an JWK encoded EC based Identity Key

- Used in JWS protected headers for signature verification
- Used in JWE protected headers when wrapping channel keys for recipients
- Points to an articulation statement containing the signer's or recipient's JWK encoded EC public identity key.

`ascp:keyframe:<uuid>` - References a channel keyframe

- Used in JWE protected headers for message encryption/decryption
- Points to a Keyframe articulation statement containing channel keys for authorized recipients
- The referenced keyframe must be the newest / active keyframe at the time of message encoding. That is, one must never use a key that has been rotated out already.

`alsp:bck:<index>` - References a Bootstrap channel key index.

- Used in JWE protected headers for message encryption/decryption of the Bootstrap channel messages specifically.
- The index is a zero-based offset into the key array provided during the ALSP Layer-0 hello exchange
- The index must reference the highest known key at the time of message encoding
- These keys exist out-of-band from the ASCP Keyframe database because no channel exists yet to store them
- The `alsp:bck:<index>` KID format is reserved for bootstrap Channels and is defined in detail in the companion document *ASCP: Bootstrap Process and Channel Discovery*. Layer-1 treats these identifiers as opaque and uses them only for initial key selection.
- See the Layer-0 ASCP Log-Sync Protocol Specification for more details as well.

> **Note:** The JOSE protected headers, even in JWE are ***not*** **encrypted**, but *are* **integrity-protected** as part of the AEAD encryption process. This means the `kid` is visible to the recipient prior to decryption, enabling proper key selection, but cannot be tampered with without causing decryption failure. This design ensures that key identifiers are safely discoverable while still cryptographically bound to the encrypted payload.

### 6.4.3 Usage Examples

#### **JWS Signing (Artipoint validation):**

```json
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "ascp+jws"
}
```

#### **JWE Message Encryption:**

```json
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
}
```

#### **JWE Key Wrapping (in channel-key-envelope):**

```json
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "typ": "jwk",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}
```

### 6.4.5 Lookup Process - Demonstrative

1. **Parse the kid** to extract the type and UUID
2. **Locate the associated Certificate** with the specified UUID provisioned into Layer-1 via Layers 2 and 3.
3. **Confirm the Certificate** purpose matches the needed purpose.
4. **Extract the cryptographic material** based on the type:
   - For `cert:` - retrieve the JWK encoded EC public identity key from the articulation statement
   - For `keyframe:` - locate the appropriate channel-key-envelope for the current recipient
5. **Use the material** for the intended cryptographic operation (signing, verification, encryption, or decryption)

This approach ensures that all cryptographic operations can be independently verified and audited by examining the articulation logs, while keeping individual messages compact and efficient.

## **6.5 Channel AES Key Requirements**

For each Keyframe, Layer-3 provisions a single symmetric **Channel AES Key** to Layer-1. This key:

- MUST be generated using a cryptographically secure random generator
- MUST be 256 bits when used with required AES-GCM
- MUST be stable for the duration of the Keyframe’s active period
- MUST be distributed to all recipients via wrapped key envelopes
- MUST NOT be reused across unrelated Keyframes except under explicit, well-scoped configuration

Rotation and supersession rules are defined in Section 9.

## **6.6 Integrity, Authenticity, and Replay Protection**

Channels combine JOSE cryptographic protections with immutable log semantics to ensure durable, verifiable history.

- **Integrity** is guaranteed by JWS signatures; any modification to headers or payload invalidates the envelope.
- **Authenticity** derives from the Author’s verified signing key.
- **Confidentiality**—when enabled—is enforced by JWE using the Channel AES Key.
- **Replay Safety** is achieved through ALSP’s append-only replication and Layer-1’s rejection of duplicates and stale kids.
- **Keyframe Transitions** ensure old symmetric keys cannot decrypt envelopes produced after rotation.

Channels intentionally do not provide per-recipient confidentiality or forward secrecy; such properties belong to higher-layer governance policy or future protocol extensions (Appendix B).

# **7. Channel Structure**

This section defines the structural properties of an ASCP Channel at Layer-1. Channels do not own or maintain append-only logs; that responsibility belongs entirely to Layer-0 (ALSP). Instead, Layer-1 constructs cryptographically protected envelopes and hands them **down** to Layer-0 for storage and replication. Conversely, Layer-1 receives envelopes **from** Layer-0, verifies and decrypts them, and passes the resulting cleartext payloads upward to the Layer-2 Grammar engine.

Layer-1 therefore functions as the cryptographic boundary between semantic content (Layer-2) and log replication (Layer-0).

## **7.1 Channel Identity (Artipoint Reference)**

A Channel is identified by the UUID of its **Channel Declaration Artipoint** in Layer-2. This UUID is used consistently across layers:

- Layer-1 uses it to select the correct key table, Keyframe configuration, and cryptographic state.
- Layer-0 uses it to reference the Channel’s append-only log.

This identifier is the **only** determinant of Channel identity; Layer-1 MUST NOT derive identity from JOSE headers, payloads, or governance attributes.

## **7.2 Log Characteristics and Naming (Layer-0 Responsibility)**

Channels are realized in storage as **Layer-0 append-only logs**, managed by the ASCP Log-Sync Protocol (ALSP). From Layer-1’s perspective:

- Layer-0 owns the log.
- Layer-0 enforces immutability.
- Layer-0 ensures deterministic convergence across replicas.
- Layer-1 never writes to or reorders log entries directly.

Layer-1 hands constructed envelopes **down** to Layer-0, which appends them exactly as received.

Similarly, Layer-0 delivers envelopes **up** to Layer-1, which verifies and decrypts them before handing payloads to Layer-2.

Logs SHOULD be named using the Channel UUID (e.g., channel/\<uuid>). Layer-1 treats this naming convention as opaque and MUST NOT assume any meaning or structure beyond the mapping supplied by Layer-0.

## **7.3 Channel Access Key (CAK)**

The **Channel Access Key (CAK)** is a replication credential used exclusively by Layer-0 to authorize participation in the Channel’s sync domain. Layer-1 does not consume the CAK directly.

Properties:

- CAK authorizes Layer-0 sync *only*.
- It has no semantic meaning at Layer-1 or Layer-2.
- It does not grant decryption rights.
- It does not grant authorship rights.
- Its key material is declared within a Keyframe but enforced by Layer-0.

Layer-1 MUST treat the CAK as opaque and MUST NOT attempt to validate or interpret it.

## **7.4 Replica Responsibilities (Layer-0 and Layer-1 Split)**

An ASCP **Replica** is the combination of:

- a Layer-0 ALSP instance that maintains the append-only log, and
- a Layer-1 Channels instance that validates and decrypts envelopes for higher layers.

To remove ambiguity, responsibilities are clarified explicitly.

### **7.4.1 Layer-0 Replica Responsibilities**

Layer-0 replicas MUST:

- store envelopes verbatim in the append-only log,
- maintain immutability and ordering guarantees,
- synchronize logs with other replicas via ALSP,
- deliver newly arrived envelopes **upward** to Layer-1,
- and accept envelopes from Layer-1 for append.

Layer-0 MUST NOT inspect or modify envelope contents.

### **7.4.2 Layer-1 Responsibilities**

Layer-1 MUST:

- construct new envelopes (JWS, optionally JWE) and hand them down to Layer-0,
- verify signatures on received envelopes,
- decrypt payloads when encryption is enabled,
- reject envelopes that fail verification or violate envelope format rules,
- and pass valid cleartext payloads **up** to the Layer-2 Grammar system unmodified.

Layer-1 MUST NOT write to logs directly; it always communicates through Layer-0’s append interface.

## **7.5 Local-First, Multi-Writer Characteristics (Corrected for Layer Boundaries)**

### **7.5.1 Multi-Writer Collaboration**

Layer-1 accepts envelopes from any Sender presenting a valid JWS signature referring to an identity certificate via ascp:cert:\<uuid>. Whether a Sender is *authorized* to write is determined by Layer-3 governance evaluation and enforced through provisioning of legitimate signing keys—not by Layer-1 logic.

Layer-0 then appends these envelopes verbatim.

### **7.5.2 Local-First Operation via Layer-0**

Local-first collaboration emerges from ALSP, not Layer-1:

- Layer-0 replicates logs peer-to-peer, via servers, or mixed topologies.
- Layer-1 remains topology-agnostic and simply reacts to incoming envelopes from Layer-0 or incoming Articulations Sequences from Layer-2.

### **7.5.3 Deterministic Convergence**

All replicas eventually converge to the same envelope ordering because ALSP ensures log alignment. Layer-1 MUST NOT attempt reordering, reconciliation, or semantic merging. Envelopes MUST be handed off in the order received.

Layer-1 only validates what ALSP has appended.

# **8. Envelope Format**

This section defines the normative structure of all envelopes used in ASCP Channels. Channels use compact JOSE encodings to minimize storage overhead, support efficient ALSP replication, and maintain compatibility with widely deployed cryptographic tooling. Every envelope is first represented as a **JWS Compact Serialization** string; encryption, when enabled, wraps that JWS output in a **JWE Compact Serialization** envelope.

Layer-1 is responsible for constructing and validating these envelopes but does not interpret or modify their cleartext payloads of the Layer-2 ASCP Grammar.

## 8.1 Envelope Encoding Overview

Channel envelopes follow a strict **two-stage** encoding process:

1. **JWS Signature (required)**
   - Produces a JWS Compact Serialization string (header.payload.signature).
   - Establishes authorship integrity and binds the JOSE header to the Artipoint payload.
2. **JWE Encryption (optional)**
   - Wraps the JWS string inside a JWE Compact Serialization structure.
   - Provides confidentiality and audience restriction using the Channel AES Key.
   - MAY also be used to enable payload compression (see §8.5), even when encryption is not required.

This layering ensures that encryption, when used, does not obscure authorship and integrity: the payload is always signed by the Author's key *before* encryption is applied.

Both JWS and JWE envelopes MUST use **Compact Serialization**, as defined in:

- **RFC 7515 §3.1 / §7.1** for JWS
- **RFC 7516 §3.1 / §7.1** for JWE

Compact Serialization provides a concise and deterministic format suited to append-only logs and is compatible with the JOSE ecosystem.

## 8.2 JWS Envelope Structure

All Artipoint payloads MUST be signed as JWS envelopes prior to optional encryption.

### **8.2.1 Inputs (Sender-Side)**

- Cleartext Articulation Sequence (UTF-8 string)
- Author’s private signing key (purpose::assert)
- Protected header which MUST be containing the following fields:
  - alg: signature algorithm (e.g., ES256)
  - kid: ascp:cert:\<uuid> referencing the Author’s identity certificate
  - typ: "ascp+jws"

### **8.2.2 Compact JWS Structure**

```
<header>.<payload>.<signature>
```

This compact format consists of three base64url-encoded components separated by periods:

- **Header**: JWS protected header JSON containing alg, kid, and typ
- **Payload**: Articulation Sequence UTF-8 string
- **Signature**: JOSE JWS signature (ie: ECDSA P-256, etc.).

### **8.2.3 Example: JWS Protected Header**

```
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "ascp+jws"
}
```

### **8.2.4 Why JWS Compact Serialization?**

- Extremely compact for ALSP logs
- Fully JOSE-compliant
- Easy to validate at Layer-1
- Tooling support across languages and platforms

Layer-1 MUST treat the JWS payload as opaque content and MUST NOT attempt to interpret the Artipoint Grammar.

## **8.3 JWE Envelope Structure**

JWE encryption is optional and depends on the Channel's `payload_cipher` configuration. When a Channel's `payload_cipher` is not "none", the JWS string MUST be encrypted using JWE. JWE MAY also be applied when `payload_cipher` is "none" solely to enable compression (see §8.5), in which case the JWE "alg" field is set to "dir" but no actual encryption key is used.

### **8.3.1 Inputs (Sender-Side)**

- JWS Compact Serialization string
- Channel AES Key (provisioned from Keyframe evaluation)
- Protected header including:
  - alg: "dir" (direct symmetric encryption)
  - enc: "A256GCM" (required)
  - typ: "ascp+jws+jwe"
  - kid: "ascp:keyframe:\<uuid>" referencing the active Keyframe
  - zip: "DEF" optional deflate compression.

### **8.3.2 JWE Compact Structure**

```
<header>.<encrypted-key>.<init-vector>.<ciphertext>.<auth-tag>
```

This compact format consists of five base64url-encoded components separated by periods:

- **header**: JWE protected header JSON
- **encrypted-key**: Encryption Key (omitted due to direct encryption mode)
- **init-vector**: initialization vector for AES-GCM
- **ciphertext**: Encrypted JWS payload
- **auth-Tag**: AES-GCM authentication tag

ASCP uses **direct encryption**, so \<encrypted-key> is always empty, generating two consecutive periods, which is correct per RFC 7516 §7.1.

### **8.3.3 Example: JWE Protected Header**

```
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
}
```

### **8.3.4 Initialization Vector (IV) Requirements**

To satisfy AES-GCM security guarantees:

- A **new 96-bit (12-byte) IV MUST be generated** for every encryption
- IVs MUST be cryptographically random
- IV reuse under the same AES key MUST NOT occur under any circumstances

Failure to follow these requirements may compromise confidentiality and authentication.

## **8.4 Header Fields (typ, kid, alg, enc, iv, apv, etc.)**

The following JOSE header fields are used in ASCP Channel envelopes:

### **8.4.1 Common Fields**

- `typ`: Identifies the envelope type and is required for tooling and media-type identification. Valid values are:
  - `"ascp+jws"` for JWS-only envelopes
  - `"ascp+jws+jwe"` for JWE-wrapped envelopes
- `kid`: Key identifier used for cryptographic key lookup. This field MUST reference either:
  - The Author's certificate (`ascp:cert:<uuid>`) for JWS signature verification, or
  - The active Keyframe (`ascp:keyframe:<uuid>` or `alsp:bck:<index>`) for JWE decryption.
  - Layer-1 MUST NOT infer semantics from the kid value's structure; it is used solely for key lookup.

### **8.4.2 JWS-Specific Fields**

- **alg**: Signature algorithm used to sign the envelope. MUST be one of the following allowed algorithms:
  - `ES256` (ECDSA using P-256 and SHA-256)
  - `EdDSA` (Ed25519)
  - `ES384` (ECDSA using P-384 and SHA-384)

### **8.4.3 JWE-Specific Fields**

- **alg**: Key management algorithm. MUST be `"dir"` (direct use of a shared symmetric key) for Channel message encryption.
- **enc**: Content encryption algorithm. MUST be `"A256GCM"` (AES-256 in Galois/Counter Mode).
- **iv**: Initialization vector for AES-GCM encryption. MUST be at least 96 bits (12 bytes) of cryptographically secure random data. See §8.3.4 for detailed IV requirements.
- **tag**: AES-GCM authentication tag generated during encryption and verified during decryption.

Additional JOSE header fields (e.g., `zip`, `apv`, `apu`) MAY appear in envelopes. Layer-1 implementations MUST preserve all unrecognized header fields even if they do not process them.

## **8.5 Payload Requirements**

### **8.5.1 JWS Payload**

The payload MUST be:

- the entire Articulation Sequence,
- encoded as UTF-8,
- base64url-encoded for Compact Serialization.

Layer-1 MUST NOT alter whitespace or structure.

### **8.5.2 Compression (JWE “zip”)**

The plaintext input to JWE encryption is the complete JWS Compact Serialization string. Implementations **SHOULD** apply DEFLATE compression (JWE header field "zip": "DEF") when the total JWS Compact string length exceeds **200 bytes**.

Compression, when applied, occurs as part of the JWE encryption step as defined in \[RFC 7516 §4.1.3]. The compression is applied to the JWS string before encryption; the "zip" parameter MAY be present even when the JWE "alg" is "dir" and no key encryption occurs.

Although JWE supports DEFLATE compression (zip) prior to encryption, it does not define an unencrypted ‘compression-only’ mode. In ASCP, use of zip implicitly requires encryption; Channels with payload\_cipher = none MUST NOT emit JWE envelopes.

**Rationale (Informative):**

The 200-byte threshold is selected based on empirical analysis of ASCP Articulation Sequences and JOSE envelope overhead:

- **Base64url expansion**: JWS Compact Serialization introduces approximately 33% size overhead due to base64url encoding of the UTF-8 payload.
- **Compression effectiveness**: DEFLATE with dynamic Huffman coding reliably recovers the base64url overhead and achieves additional reduction by exploiting repeated ASCII patterns common in ASCP grammar. Empirical testing shows consistent net compression gains once the full JWS string exceeds approximately 150 bytes (corresponding to roughly 32 bytes of raw Articulation Sequence payload).
- **Conservative threshold**: The normative 200-byte threshold provides a safety margin across different JOSE header sizes and signature algorithms. Below this threshold, compression overhead may equal or exceed size savings; above it, compression consistently yields net reductions of 40–60% as payloads reach 1K or more.

When enabled, the JWE protected header MUST include:

```asciidoc
"zip": "DEF"
```

## **8.6 Error Handling and Diagnostic Requirements**

Layer-1 MUST detect and surface the following classes of errors:

### **8.6.1 JWS Errors**

JWS validation failures occur when the envelope structure is invalid, when the `kid` field is missing or malformed, or when the signing `kid` references an unknown certificate. Signature verification may fail due to cryptographic mismatch or tampering, and envelopes using unsupported signature algorithms (outside the allowed set of ES256, EdDSA, ES384) MUST be rejected.

### **8.6.2 JWE Errors**

JWE decryption failures arise from invalid JWE structure, use of an incorrect AES key for the specified Keyframe, or invalid or reused initialization vectors that violate AES-GCM security requirements. Envelopes specifying incorrect or unsupported `alg` or `enc` values MUST be rejected, as must envelopes referencing unknown Keyframe `kid` values that cannot be resolved to provisioned key material.

### **8.6.3 General Envelope Errors**

Envelope parsing errors include malformed base64url segments that cannot be decoded, incorrect Compact Serialization structures with the wrong number of period-delimited parts, protected headers that do not contain valid JSON, and the presence of forbidden header parameters such as unencoded raw objects that violate JOSE Compact Serialization requirements.

### 8.6.4 Error handling rules

- **Layer-0 MUST still append malformed envelopes** (immutability)
- **Layer-1 MUST reject malformed or unverifiable envelopes** for semantic handoff
- Detailed diagnostics SHOULD be logged
- Cleartext MUST NOT be emitted to Layer-2 unless JWS verification succeeds and decryption (if applicable) succeeds

# **9. Keyframes**

Keyframes are Layer-2 Artipoints that declare the intended cryptographic configuration for a Channel. They are signed, durable, and auditable entries in the Channel’s log, but Layer-1 does not parse their contents. Instead, Layer-3 evaluates Keyframes, determines their meaning, and provisions Layer-1 with the corresponding cryptographic state.

Layer-1 therefore treats Keyframes as **externally supplied configuration statements**, not as artifacts to be parsed or semantically interpreted. This section defines only the **cryptographic consequences** of Keyframes—how they influence Layer-1 behavior, how rotation works, and how key versioning is resolved during envelope encoding and decoding.

## 9.1 Purpose of Keyframes in Layer-1

Keyframes provide the cryptographic material required for a Channel to function. Each Keyframe establishes:

- the **Channel AES Key** (symmetric key used for JWE message encryption),
- the **Channel Access Key (CAK)** (Ed25519 keypair for Layer-0 replication authentication), and
- the **recipient-specific wrapped key envelopes**, enabling each authorized participant to decrypt both keys.

Keyframes enable:

- **initial bootstrapping** of a Channel’s cryptographic configuration,
- **key rotation** to achieve forward secrecy or accommodate membership changes, and
- **per-recipient binding**, ensuring that only authorized participants receive decryptable key material.

Keyframes do **not** define governance semantics (e.g., “who is a member”). They carry only the *cryptographic consequences* of such governance decisions after Layer-3 evaluation. Membership semantics belong entirely to the Governance specification.

## 9.2 Keyframe Contents Required by Layer-1

Although Keyframes exist in Layer-2 and have rich semantic structure, Layer-1 relies on a small, well-defined subset of the information they express:

### **9.2.1 Channel AES Key (Symmetric Encryption Key)**

Provisioned to Layer-1 as a 256-bit AES key in JWK form:

```
{
  "kty": "oct",
  "k": "<base64url AES-256 key>",
  "alg": "A256GCM",
  "use": "enc"
}

```

Used exclusively for encrypting and decrypting JWE envelopes carrying JWS signed Articulation Sequences.

### **9.2.2 Channel Access Key (CAK)**

Ed25519 keypair used by Layer-0 to authenticate sync access:

```
{
  "kty": "OKP",
  "crv": "Ed25519",
  "x": "<base64url pubkey>",
  "d": "<base64url privkey>",
  "alg": "EdDSA",
  "use": "sig"
}
```

Layer-1 does not use the CAK itself but it is contained in the same envelope as the AES encryption key and is documented here as part of everything managed by Layer-2 and Layer-3. 

**Note:** For the sake of clarity, this Ed25519 key is **not** used to sign Artipoints. Artipoints are always signed by the originator’s personal identity key (typically ECDSA P-256 private key. The CAK functions as a shared *token* to authorize log access, not authorship.

### **9.2.3 Recipient Public Keys for Key Wrapping**

This is the JWK representation of a recipient's EC public key. It is stored in the directory of public keys and referenced via the associated `kid`.

```
{
  "kty": "EC",
  "crv": "P-256",
  "x": "<base64url X-coordinate>",
  "y": "<base64url Y-coordinate>",
  "alg": "ECDH-ES",
  "kid": "ascp:cert:<uuid>"
}

```

- `kty`: Key type (Elliptic Curve)
- `crv`: Curve (NIST P-256)
- `x`, `y`: Coordinates of the EC public key
- `alg`: Suggested algorithm (ECDH-ES)
- `kid`: Key identifier used in JWE and JWS headers and directory lookup

These keys MUST correspond to the identity keys referenced by ascp:cert:\<uuid> in JWS and JWE operations.

**Note:** The `use` field is intentionally omitted. This allows the same key to be used for both:

- Signature verification in JWS (e.g., validating signed Artipoints)
- Key agreement in JWE (e.g., encrypting channel key envelopes)

This is compliant with RFC 7517 §4.2, which permits keys to omit the use field to avoid unnecessary restriction.

### **9.2.4 Key Envelope Attribute**

Each Keyframe contains a set of **channel-key-envelope** objects—one per recipient—encoded as typed Layer-2 Artipoint attributes:

```asciidoc
envelope::user@example.com := json:{ <channel-key-envelope> }
```

Each \<channel-key-envelope> holds:

- `aes_key_jwe` (wrapped JWK encoded AES key)
- `auth_key_jwe` (wrapped JWK encoded CAK private key)
- other metadata including validity and optional rotation interval

The envelope structure is **not** parsed by Layer-1. Instead, Layer-3 extracts keys and provisions them to Layer-1.

The complete normative specification of the \<channel-key-envelope> JSON structure is defined in §9.6.

### **9.2.5 Prohibition on Layer-1 Parsing**

Layer-1 MUST NOT parse Keyframe Artipoints or any of their attributes.

Layer-1 MUST receive all cryptographic material in a pre-interpreted form via provisioning.

## **9.3 KID Versioning and Selection Rules**

Keyframes provide stable, deterministic identifiers for the cryptographic state a Channel uses. The Channel Attribute `kid` points to the active Keyframe, enabling both authors and recipients to select the correct Keyframe version without ambiguity.

### **9.3.1 Keyframe as Key Version**

Each Channel Artipoint MUST have a `keyframe::kid` Attribute pointing to the current Keyframe associated with the channel:

```asciidoc
keyframe::kid := "ascp:keyframe:<uuid>"
```

This kid identifies the AES and CAK keys in use for the Keyframe’s lifetime.

### **9.3.2 Sender Behavior**

When constructing a JWE envelope:

- The Sender MUST use the **current Keyframe** kid.
- A Sender MUST NOT use a kid from a superseded Keyframe.
- A Sender MUST apply the kid to the JWE protected header.

### **9.3.3 Receiver Behavior**

When receiving an envelope:

- Layer-1 MUST use the JWE header `kid` to select the correct AES key.
- If the referenced kid is unknown or inactive, the envelope MUST NOT be decrypted.
- Layer-0 MAY retain the envelope unmodified in the log (Layer-0 responsibility).

### **9.3.4 Example Keyframe Artipoint (Required Syntax)**

```bnf
[uuid, source, timestamp,
  channel-uuid .
  (
    keyframe::kid := "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002",
  )
]
```

This mapping is critical for rotation, described next.

## **9.4 Keyframe-Driven Rotation Events**

Keyframes enable controlled rotation of the Channel's cryptographic state, including the AES Channel Key, the CAK replication keypair, and the set of recipient envelopes.

Rotation MAY be performed to achieve forward secrecy, accommodate membership changes, respond to device compromise, or fulfill scheduled key-lifecycle policies.

### **9.4.1 Rotation of the AES Channel Key**

Implementers at Layer-2 and Layer-3 **SHOULD** follow these steps to rotate the AES Channel Key:

1. Generate a new 256-bit AES key.
2. Produce new channel-key-envelopes for each recipient using **ECDH-ES + A256KW**.
3. Issue a new Keyframe Artipoint with a new Keyframe UUID.
4. Articulate a new `keyframe:kid` attribute on the Channel Artipoint to point to the new Keyframe UUID and provision the new key material into Layer-1
5. Senders now allowed to begin using the new Keyframe kid.
6. Recipients retain access to old AES keys for historical decryption.

### **9.4.2 Rotation of the CAK (Ed25519)**

Same process as AES rotation but impacts on Layer-0 are:

- A new CAK MUST be distributed in envelopes.
- Replicas MUST begin using the new CAK for authentication.
- Prior CAKs MAY remain valid for a grace period.

### **9.4.3 Recipient Identity Key Rotation**

If a recipient rotates their EC identity key:

- Only that user’s key envelope must be re-issued via a new `envelope::<user>` attribute on the active Keyframe.
- No Channel-wide rotation is required.
- The Keyframe number does not need to change unless intended.

### **9.4.4 Time-Based Validity and Policies**

`valid_from` and `rotation_interval_days`:

- do not affect Layer-1 logic directly,
- but MUST be enforced by higher layers for governance-based policy compliance.

### **9.4.5 Rotation Does Not Rewrite History**

Keyframes MUST NOT modify previously encrypted or signed envelopes.

Historical decryptability MUST be preserved unless explicitly superseded by governance decisions outside this specification.

## **9.5 Keyframe Invalidity or Supersession**

A Keyframe becomes superseded when:

- a new Keyframe Artipoint is created with a higher sequence or timestamp and the `keyframe:kid` attribute of the Channel Artipoint is pointed at it,
- governance revokes access for some recipients (higher-layer decision),
- or rotation is triggered by policy.

Layer-1 MUST follow these rules:

- Only the **latest Keyframe** may be used for new encryption.
- All historical Keyframes remain valid for decrypting historical envelopes.
- If a Keyframe’s cryptographic material becomes unavailable, envelopes referencing that Keyframe MAY become undecryptable, but MUST remain stored unchanged.

Governance MUST resolve semantic revocation; Layer-1 MUST NOT.

## 9.6 Channel Key Envelope Structure

The following structure defines the JSON format for \<channel-key-envelopes> that appear as the value of an envelope attribute targeted for a specific user in Keyframe artipoints. It MUST contain these fields:

```
{
  "type": "channel-key-envelope",
  "version": "1.0",
  "recipient_cert": "ascp:cert:<uuid>",
  "aes_key_jwe": { ...JWE object with ECDH-ES+A256KW... },
  "auth_key_jwe": { ...JWE object with ECDH-ES+A256KW... },
  "created": "2025-07-26T21:13:00Z",
  "enc": "A256GCM",
  "alg": "ECDH-ES+A256KW",
  "valid_from": "2025-07-26T21:13:00Z",
  "replaces": "ascp:keyframe:<uuid>"
  "rotation_interval_days": 365
}
```

- `type`: Identifies this as a channel-key-envelope structure
- `version`: Version of this envelope and key bundle
- `recipient_cert`: The \`kid\` of the certificate used to encode the keys in this envelope
- `aes_key_jwe`: JWE object containing the encrypted AES-256 channel encryption key
- `auth_key_jwe`: JWE object containing the encrypted Ed25519 channel authentication key
- `created`: Timestamp when this envelope was created
- `enc`: Encryption algorithm used (A256GCM)
- `alg`: Key wrapping algorithm used (ECDH-ES+A256KW)
- `valid_from`: Start time for when this key should be used (optional)
- `replaces`: References the prior Keyframe if this envelope is specifically replacing a different prior Keyframe envelope after a rotation of one or both keys. This is non-functional and informational only for each back tracing. (optional)
- `rotation_interval_days`: Recommended rotation interval in days (optional)

Each key in the envelope (`aes_key_jwe` and `auth_key_jwe`) is a standard **flattened JWE JSON object**, where the payload is a serialized **JWK** representing the symmetric AES key or the Ed25519 private key, respectively. These are encrypted to the intended recipient using their EC public identity key additionally pointed to by the `recipient_cert` within the envelope. This key reference MUST be the same as the `kid` used for wrapping both of the contained keys.

### Example JWE protected header (used in each wrapped key):

```
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "typ": "jwk",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}

```

### Example flattened JWE envelope:

```
{
  "protected": "<base64url-encoded protected header JSON>",
  "iv": "<base64url-encoded IV>",
  "ciphertext": "<base64url-encoded encrypted JWK>",
  "tag": "<base64url-encoded authentication tag>"
}

```

The payloads in these encrypted envelopes are JWK objects like the ones defined above. The typ: "jwk" in the protected header makes this explicit, aiding validation and tooling.

## **9.7 Layer Separation: Keyframes as Cryptographic Configuration Only**

Keyframes do not determine membership, authorship permissions, role semantics, visibility rules, or policy enforcement. All semantic meaning belongs to Layer-2 and Layer-3. Layer-1 MUST treat Keyframes strictly as opaque cryptographic configuration—versioned key containers that are externally provisioned and do not carry governance semantics.

This strict separation enables Channels to maintain long-term cryptographic stability even as governance models, semantic structures, and higher-layer logic evolve independently.

# **10. Grammar Integration: Channel-Relevant Artipoints**

ASCP Channels operate at Layer-1 and therefore do not interpret or parse Artipoint Grammar. However, Channels rely on the **existence and structure** of several Layer-2 Artipoint types whose semantics are evaluated by Layer-3 and then provisioned into Layer-1.

This section defines:

- the Layer-2 *representation* of Channel-related Artipoints
- the Artipoint-level attributes that influence Layer-1 cryptographic behavior
- the precise separation between Grammar semantics (Layer-2) and Layer-1 execution
- the mapping from typed attributes in Keyframes to Layer-1 configuration
- examples demonstrating canonical encoding patterns

Layer-1 observes these Artipoints only indirectly, through the configuration supplied by higher layers; it never parses or interprets Grammar content directly.

## 10.1 Keyframe Artipoints

A Keyframe is a Layer-2 **bookmark Artipoint** of type `keyframe`, linked to a Channel using the `supports` operator. Keyframes define cryptographic configuration, including encryption algorithms, signing algorithms, and (through attributes) per-recipient wrapped keys.

### **10.1.1 Canonical Form**

```clike
[uuid, source, timestamp,
  ["keyframe", "HiringTeam:v1", "urn:keyframe:@HiringTeam:v1"].
  (
    version := 1,
    payload_cipher := "AES256",
    message_signing := "ECDSA-P256",
    channel_access_alg := "Ed25519", 
  )
] supports {channel-uuid}

```

The `supports` operator attaches the Keyframe as a child of the Channel in the DAG. Keyframes contain the attributes needed for Layer-3 to generate wrapped key envelopes, which are then provisioned to Layer-1. Layer-1 never reads or interprets these fields directly; it only uses the resulting key material and configuration computed by Layer-3.

Layer-1 cares about Keyframes only insofar as:

- they result in the provisioned AES key,
- they determine the active Keyframe’s kid,
- they provide per-recipient wrapped key envelopes.

### **10.1.2 Keyframe Required Attributes**

- **version**: Keyframe format version number. MUST be `1` for this document version.
- **payload\_cipher**: The symmetric cipher algorithm used for payload encryption. SHOULD match the Channel declaration, though Keyframes MAY specify different algorithm choices for future versions.
- **message\_signing**: The signature algorithm used for message signing. SHOULD match the Channel declaration, though Keyframes MAY specify different algorithm choices for future versions.
- **channel\_access\_alg**: The signature algorithm used for the Channel Access Key (CAK). SHOULD match the Channel declaration, though Keyframes MAY specify different algorithm choices for future versions.

### 10.1.3 Keyframe Envelope Attribute

Keyframes distribute encrypted key material to authorized recipients through typed `envelope::` attributes. Each attribute contains a JSON-encoded `<channel-key-envelope>` (as defined in §9.6) that wraps both the Channel's AES encryption key and the Ed25519 CAK private key for that specific recipient.

**Attribute Format:**

```asciidoc
envelope::user := json:{ <channel-key-envelope> }
```

**Example with multiple recipients:**

```clike
[uuid, source, timestamp,
  keyframe-uuid .
  (
    envelope::user1@reframe.systems := json:{ <channel-key-envelope> },
    envelope::agent42@reframe.agents := json:{ <channel-key-envelope> }
  )
];
```

Each `<channel-key-envelope>` allows the corresponding recipient to decrypt:

- The Channel's symmetric AES key used for Layer-1 message encryption
- The Ed25519 private key used to sign Layer-0 `sync_request` access proofs (defined in the ALSP specification)

The envelopes are embedded directly in the Keyframe Artipoint, avoiding external indirection. Only recipients possessing the corresponding private keys can decrypt their respective envelopes. Layer-1 never parses these attributes; Layer-3 extracts the keys and provisions them to Layer-1 externally via an implementation specific API.

## **10.2 Channel Artipoints**

A Channel is a Layer-2 **bookmark Artipoint** of type "channel" that declares the cryptographic and operational parameters for a Layer-1 Channel, including encryption algorithms, signing requirements, and channel access controls.

### **10.2.1 Canonical Form**

```bnf
[uuid, source, timestamp,
  ["channel", "Hiring Team", "@HiringTeam"] .
  (
    payload_cipher := "A256GCM",
    message_signing := "ECDSA-P256",
    channel_access_alg := "Ed25519",
    bootstrap := false,
    keyframe::kid := "ascp:keyframe:<uuid>" 
  )
]

```

Layer-1 never reads these attributes directly; Layer-3 processes them and generates a provisioning configuration for Layer-1.

### **10.2.2 Required Attributes**

- `payload_cipher`: The symmetric cipher algorithm used for payload encryption. Determines whether Layer-1 applies JWE encryption to envelopes. Use `"none"` to explicitly disable encryption and operate the Channel in cleartext. Valid values include `"A256GCM"` or `"none"`. Note: In current implementations, a bootstrap Channel MUST use `"none"` as there is no defined mechanism for passing the symmetric key out-of-band.
- `message_signing`: The signature algorithm required for Artipoint signatures in this Channel. Determines the allowed JWS algorithms for message signing. All articulations MUST be signed using a secure algorithm. Typically `"ECDSA-P256"` for full interoperability with JOSE JWS standards-based elliptic curve key-pair systems.
- `channel_access_alg`: The signature algorithm used for the Channel Access Key (CAK) credentials in the Layer-0 storage and synchronization protocol. Determines the CAK algorithm used at Layer-0. `"Ed25519"` SHOULD be used, or `"none"` to disable channel access authentication.

#### **10.2.3 Optional Attributes**

- `bootstrap`: Boolean flag indicating if this is the organizational bootstrapping Channel. Special case for Channels without Keyframes (initial system Channel). Current implementations do not support encryption for a bootstrap Channel as there is no defined mechanism for passing the symmetric key out-of-band of any channel.
- `keyframe::kid`: The identifier for the currently active symmetric key (see Keyframes §9).

## **10.3 Layer-3 to Layer-1 Provisioning**

Layer-3 interprets Channel and Keyframe Grammar attributes (as defined in §10.1 and §10.2) to derive the cryptographic configuration required by Layer-1. This includes determining encryption algorithms, signature requirements, active key material, and historical keys needed for decryption of prior envelopes.

Layer-3 MUST provision this cryptographic material to Layer-1 through an implementation-specific interface. The provisioned material MUST include:

- All keys in JWK format (as defined in §9.2)
- Keys indexed by their respective `kid` values for lookup
- The active AES key and any historical AES keys required for decryption
- The Channel Access Key (CAK) for Layer-0 operations

Layer-1 receives **only cryptographic state**—no Grammar fields, attribute names, or semantic information. Layer-1 performs all cryptographic operations by selecting keys based solely on the `kid` field in JOSE headers, without parsing or interpreting Grammar content. The Layer-2 UUID's, via `kid` values (such as `ascp:cert:<uuid>` and `ascp:keyframe:<uuid>`), are the only identifiers that cross the Layer-3 to Layer-1 boundary, as they appear directly in JOSE envelope headers and provide the sole reference point between Grammar-level identities and Layer-1 cryptographic operations.

This strict separation preserves long-term stability and ensures that semantic evolution at Layer-2 does not require changes to cryptographic implementation. As governance models, membership semantics, and higher-layer logic evolve, Layer-1 remains unchanged—operating purely on provisioned key material and JOSE envelope structure.

## 10.4 Summary of Patterns (Informational)

This table presents a brief summary of the different types of Articulation patterns associated with Channels. Note: This is not fully inclusive and is specific to Layer-2 and Layer-3, so this is informational only:

| Purpose                 | Pattern                                              | Notes                                                            |
| ----------------------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| Declare a Channel       | bookmark("channel")                                  | Includes cipher and signing algorithm choices                    |
| Activate Keyframe       | annotation(keyframe::kid :=)                         | Applied to Chanel UUID. Specifies the active Keyframe.           |
| Manage Membership       | annotation(member +)                                 | Applied to Channel UUID.                                         |
| Declare Keyframe        | bookmark("keyframe") supports {channel}              | Supporting child of Channel                                      |
| Distribute Key Envelope | annotation(envelope::user := \<channel-key-evelope>) | Typed attribute with direct JSON payload as the attribute value. |

# **11. Message Processing Model**

This section defines the operational sender and receiver workflows for ASCP Channels. It describes how Layer-1 constructs and validates envelopes, how they are handed off to and received from Layer-0 (ALSP), and when cleartext payloads are emitted upward to Layer-2.

Layer-1 MUST treat all payloads as opaque UTF-8 strings and MUST NOT alter, canonicalize, or interpret them. Layer-0 owns log storage and replication; Layer-1 owns cryptographic protection and validation. Message processing is strictly envelope-by-envelope. Implementations MUST NOT assume the presence, order, or completeness of surrounding log entries.

## **11.1 Producer Workflow**

This workflow describes how a Sender constructs a Channel envelope prior to handing it to Layer-0 for append.

### **11.1.1 Construct Payload (Layer-2 → Layer-1 Input)**

Layer-2 produces an Articulation Sequence as a UTF-8 string. Layer-1 receives this string unmodified. Layer-1 MUST NOT adjust whitespace, ordering, or grammar structure.

### **11.1.2 Sign with Identity (JWS)**

Layer-1 MUST create a JWS Compact Serialization envelope (`<header>.<payload>.<signature>`) as defined in RFC 7515 §3.1.

**Requirements:**

- `alg` MUST be compatible with the Author's certificate (`purpose::assert`).
- `kid` MUST reference the Author's certificate (`ascp:cert:<uuid>`).
- `typ` MUST be `"ascp+jws"`.
- The payload MUST be the exact UTF-8 Articulation Sequence, base64url-encoded.

If JWS construction fails, the Sender MUST NOT continue to encryption and MUST NOT hand the envelope to Layer-0.

### **11.1.3 Optionally Encrypt (JWE)**

If the Channel's `payload_cipher` indicates encryption is enabled, Layer-1 MUST encrypt the JWS output using JWE Compact Serialization (RFC 7516 §3.1).

**Requirements:**

- JWS MUST be constructed first (sign-then-encrypt).
- Protected header MUST include:
  - `alg`: `"dir"` (direct symmetric encryption)
  - `enc`: `"A256GCM"` (required content-encryption algorithm)
  - `kid`: `"ascp:keyframe:<uuid>"` (active Keyframe identifier)
  - `zip`: `"DEF"` (optional, if compression is applied)
- A fresh 96-bit minimum IV MUST be generated.
- Encryption MUST use the active Channel AES Key.

Compression-only operation still uses authenticated encryption; ASCP does **not** support "unencrypted JWE."

### **11.1.4 Append to ALSP Replica (Layer-1 → Layer-0)**

Upon successful JWS (and optional JWE) construction:

- Layer-1 hands the envelope downward to Layer-0.
- Layer-1 MUST NOT write directly into any log.
- Layer-0 appends the envelope verbatim and handles all replication.

## **11.2 Consumer Workflow**

This workflow describes how envelopes delivered by Layer-0 are processed by Layer-1 before being passed upward to Layer-2.

### **11.2.1 Receive Envelope from ALSP (Layer-0 → Layer-1)**

Layer-0 delivers envelopes in log order. Layer-1 MUST accept each envelope for cryptographic processing, even if malformed.

### **11.2.2 If Encrypted, Decrypt with Channel AES Key**

For an `"ascp+jws+jwe"` envelope:

1. Parse the JWE Compact Serialization form.
2. Inspect the protected header:
   - `kid` MUST reference a provisioned Keyframe.
   - `alg` and `enc` MUST match supported algorithms.
3. Select the AES key associated with the Keyframe `kid`.
4. Validate IV size and structure.
5. Attempt AES-GCM decryption.

**If decryption fails:**

- Layer-1 MUST NOT release any cleartext to Layer-2.
- Layer-1 MAY log diagnostics.
- The envelope remains in the log (Layer-0 responsibility).

**If decryption succeeds,** the resulting plaintext MUST be a valid JWS Compact Serialization string.

### **11.2.3 Verify Signature (Always Required)**

Layer-1 MUST validate JWS signatures for all envelopes, whether encrypted or cleartext.

**Requirements:**

- `kid` MUST reference a known Author certificate.
- `alg` MUST be supported and permitted by that certificate.
- Signature MUST verify over protected header + payload.

If signature verification fails, cleartext MUST NOT be emitted to Layer-2.

### **11.2.4 Emit Cleartext to Grammar Layer (Layer-1 → Layer-2)**

If and only if:

1. (For encrypted envelopes) JWE decryption succeeds, ***and***
2. JWS signature verification succeeds,

Then Layer-1 MUST emit the cleartext Articulation Sequence to Layer-2. Layer-1 MUST NOT normalize, modify, or reserialize the message.

### **11.2.5 No Modification or Rewriting Allowed**

Layer-1 MUST NOT:

- rewrite JOSE headers,
- rewrite payloads,
- canonicalize whitespace or grammar,
- reorder statements,
- perform any kind of semantic evaluation.

Layer-1 is purely a cryptographic envelope processor.

## **11.3 Local-First Replication and Convergence**

Layer-1 processes each envelope atomically and independently. ALSP ensures proper ordering such that key material (Keyframes) arrives before envelopes that depend on it. Beyond retaining previously provisioned AES keys to support historical decryptability (Section 9), Layer-1 imposes no special requirements for replication scenarios—partial histories, out-of-order arrival, intermittent connectivity, or multi-writer convergence do not affect envelope processing.

## **11.4 Error Handling Summary**

This section summarizes the operational consequences of errors encountered during message processing. Normative definitions of cryptographic and JOSE constraints appear in Sections 6 and 8; Section 11 defines runtime behavior during the processing pipeline.

### **11.4.1 Conditions Requiring Rejection for Semantic Handoff**

Layer-1 MUST reject an envelope for delivery to Layer-2 when:

- JWE decryption fails
- The JWE or JWS `kid` is unknown or inactive
- JWS signature verification fails
- Required JOSE fields (`alg`, `enc`, `kid`, etc.) are missing
- Compact Serialization is structurally invalid (wrong part count, invalid base64url)
- The envelope `typ` is inconsistent with the message structure (`"ascp+jws"` vs `"ascp+jws+jwe"`)

**In all such cases:**

- Cleartext MUST NOT be provided to Layer-2.
- The envelope MUST remain in the log unmodified (Layer-0 immutability).

### **11.4.2 Conditions That Are Non-Fatal**

Layer-1 SHOULD continue processing when:

- Optional JOSE header fields are unrecognized,
- Additional header parameters do not violate JOSE rules,
- Compression metadata (`zip`) is missing or mismatched,
- Reserved header fields are present but ignorable.

If the envelope remains cryptographically valid, Layer-1 SHOULD proceed.

### **11.4.3 Logging and Diagnostics**

Layer-1 SHOULD log:

- signature verification errors,
- decryption failures,
- unknown or inactive `kid` values,
- unsupported algorithms,
- malformed Compact Serialization segments.

Diagnostic logs SHOULD include the envelope's UUID or sequencing metadata for auditability.

### **11.4.4 Notes on Immutability**

Layer-1 MUST NOT request deletion, rewriting, or removal of invalid or undecryptable envelopes. Layer-0 MUST preserve all envelopes exactly as appended.

# **12. Channel Membership Resolution**

Channel membership is a **semantic construct** defined entirely by the Governance and Identity layers of ASCP (Layers 2 and 3). Channels at Layer-1 do **not** compute membership, evaluate roles, process access policies, or determine which identities are authorized to read or write.

Instead, Layer-1 receives only the **cryptographic consequences** of membership evaluation, provisioned by Layer-3:

- the active and historical Channel AES keys for decrypting JWE envelopes,
- the current Channel Access Keypair (CAK) used by Layer-0 for replication, and
- Author identity certificates for JWS verification.

Layer-1 performs **no per-recipient processing** and MUST NOT parse any membership-related attributes in the Layer-2 grammar. The term “recipient” does not appear in Layer-1 logic; decryption success depends solely on whether a provisioned AES key matches the kid referenced in an envelope.

Membership changes (such as adding or removing participants) are reflected only through updated Keyframes and provisioning updates from Layer-3. Layer-1 treats such updates as external configuration and applies them without interpreting their semantic meaning.

This section is informative and defines no normative requirements.

# **13. Security Considerations**

ASCP Channels rely on a layered security model in which cryptographic protections, immutable replication, and governance-driven provisioning work together to ensure the confidentiality and integrity of Artipoint payloads. The Channel layer (Layer-1) is deliberately narrow in scope: it applies JOSE-based protections, verifies authorship, and decrypts content when permitted. It does not enforce or interpret semantic authorization, does not determine who should read or write, and does not evaluate governance rules. This section describes the security properties of Layer-1, the assumptions it makes about the rest of the ASCP stack, and the limitations that naturally follow from this design.

### **13.1 Confidentiality**

Confidentiality in a Channel arises when the Channel Declaration and its active Keyframe specify encryption. In this mode, payloads are wrapped using JWE with AES-GCM, making the contents of each envelope opaque to any party lacking the appropriate symmetric key. Layer-1 enforces confidentiality strictly: encrypted envelopes cannot be inspected, parsed, or validated until decryption succeeds. Since all legitimate recipients share the same symmetric key for a given Keyframe, confidentiality is exercised at the Keyframe boundary rather than per recipient. The decision about *which* identities receive the wrapped AES key is a governance-layer concern; Layer-1 simply applies whatever key material has been provisioned.

### **13.2 Authenticity**

Every Channel envelope is signed by the Author using JWS Compact Serialization. The resulting signature binds the payload and the protected header to the Author’s identity certificate, enabling strong authenticity and integrity guarantees. Layer-1 validates the signature of every envelope it processes, regardless of whether it was encrypted. Forged messages cannot be introduced into the Channel without compromising an Author’s private key or the governance rules that determine which certificates are valid. The authenticity model assumes that identity certificates and Keyframes are correctly governed and that key material is provisioned securely to the Sender.

### **13.3 Integrity and Replay**

Integrity comes from a combination of JWS signatures and AES-GCM authentication tags. Any mutation of protected headers or payload bytes invalidates these cryptographic protections. Layer-1 rejects messages with invalid signatures or authentication failures and never emits cleartext for them. Replay resistance primarily stems from the immutable log semantics of ALSP: envelopes, once appended, cannot be replaced, altered, or excised. A replica receiving a replayed message will simply recognize it as part of the existing log. Because Channels do not enforce semantic authorization or uniqueness of contributions, governance is responsible for interpreting whether a repeated action is meaningful or problematic. Layer-1’s obligation is strictly to ensure that any envelope it accepts is genuine and unmodified.

### **13.4 Keyframe Injection and Key Compromise**

Keyframes are the mechanism through which new Channel keys are introduced. A forged Keyframe could attempt to substitute unauthorized AES keys or inject invalid wrapped keys for recipients. Layer-1 is largely insulated from such attacks: it never parses Keyframes and accepts only the cryptographic state that Layer-3 has provisioned. A successful Keyframe forgery therefore requires a compromise at the governance or identity layers, not in Channels themselves. More subtle is the case of key compromise. If an AES key, CAK, or Author signing key is compromised, the security properties of the Channel degrade accordingly. Rotation through new Keyframes mitigates exposure, but cannot retroactively revoke access to messages already decryptable under older keys.

### **13.5 Governance Errors and Their Security Consequences**

Layer-1 treats the key material it receives as authoritative. If Layer-3 miscomputes membership or distributes wrapped AES keys to unauthorized identities, Layer-1 will naturally accept and use these keys. These errors fall squarely within governance and identity evaluation rather than Channel operations. Channels do not enforce semantic policy; they only enforce cryptographic correctness. The system-wide security implications of misconfigured membership therefore depend on correct governance evaluation, correct Keyframe issuance, and careful operational hygiene around device and identity management.

### **13.6 Channel Access Key (CAK) Considerations**

The CAK authenticates replication at Layer-0 and affects how envelopes travel between replicas, but it does not control semantic authorization and does not grant the ability to decrypt or author envelopes. Compromise of the CAK enables illegitimate replication—an attacker might join the sync mesh and receive all envelopes—but encrypted payloads remain protected unless the attacker also acquires the AES key for the active Keyframe. CAKs should still be rotated regularly or upon suspicion of compromise to limit passive metadata exposure and unauthorized log replication.

### **13.7 Side-Channel Risks**

As with any cryptographic system, implementations should avoid side channels in signature verification, decryption, and key lookup. Timing or behavioral differences between successful and failed decryption attempts may reveal information about key presence or message validity. Diagnostic logging must also be done with care: logs should never include decrypted payloads, raw private keys, or unredacted JWE components that could aid recovery attacks or expose sensitive metadata. Correct IV generation and secure randomness for AES-GCM are critical operational requirements.

### **13.8 Non-Goals and Limitations**

Certain security properties fall outside the scope of Layer-1 by design. Channels do not provide per-recipient encryption, sender anonymity, encrypted metadata, or message unlinkability. Forward secrecy occurs only at Keyframe boundaries, not at per-message intervals. Layer-1 does not retroactively revoke historical access nor enforce semantic policies about who can read or write. These capabilities belong to governance, identity infrastructure, or potential future protocol extensions.

### **13.9 Trust Boundaries and Assumptions**

Layer-1 assumes that all governance evaluation, membership resolution, and key distribution performed by Layer-3 is correct. It assumes that identity certificates are valid and uncompromised, and that the provisioning mechanism faithfully transmits the correct keys. It also assumes that Layer-0 provides an immutable, consistently replicated log. Layer-1’s guarantees—confidentiality, authenticity, and integrity—depend on these upstream and downstream layers operating correctly, but Layer-1 does not verify their correctness or enforce their semantics.

# **14. Implementation Considerations**

This section provides non-normative implementation guidance for ASCP Channels. It highlights practical concerns that complement the normative requirements defined in previous sections, with emphasis on correct layering, key lifecycle handling, observability, and forward compatibility.

## **14.1 Layer Separation and Provisioning Boundaries**

Implementations should maintain a strict separation between the responsibilities of Layer-1 (Channels) and those of the surrounding layers. Layer-1 is responsible only for constructing and validating JWS/JWE envelopes. It must not parse Layer-2 Grammar, evaluate governance semantics, resolve membership, or infer policy.

All cryptographic state—identity certificates, the active and historical AES keys, the CAK, permitted algorithms, and Keyframe identifiers—must be supplied externally through a provisioning interface implemented by Layer-3. Channel implementers should design this interface carefully to avoid embedding any governance or Grammar logic inside Layer-1. This avoids tight coupling between layers and ensures long-term maintainability.

## **14.3 Key Table Management**

Because Channels rely on multiple concurrent keys—current and historical AES keys, identity certificates, and the CAK—implementations should maintain a structured key table indexed by JOSE kid values. Updates to this table should be atomic to avoid inconsistent states during Keyframe rotation.

Channel implementations must expect that historical AES keys remain in use indefinitely for decrypting older envelopes, even after rotation. Keys should therefore not be overwritten or removed unless explicitly directed by higher layers. Proper key table management ensures that historical decryptability remains stable and predictable across replicas.

## **14.5 Diagnostics and Observability**

Operational debugging often depends on clear diagnostics from the Channel layer. Implementations should emit structured logs for conditions such as unknown kid values, failed signature verification, JWE decryption errors, or malformed envelopes—without revealing sensitive data such as decrypted payloads, private keys, or IVs.

Channels should not assume that Layer-0 or Layer-3 will provide sufficient context for every failure. Logging minimal but precise information (e.g., envelope identifiers, failure class, algorithm mismatch) helps operators diagnose provisioning issues, governance misconfigurations, and interoperability problems across diverse environments.

## **14.7 Forward Compatibility**

To ensure interoperability across evolving ASCP deployments, implementations should be tolerant of extensions and unrecognized fields. In particular, they should ignore unknown JOSE header parameters, avoid hardcoding algorithm lists, and support flexible provisioning that can accommodate future Keyframe attributes or new encryption/signature schemes.

This forward-compatible posture allows Channels to evolve without requiring simultaneous upgrades across all replicas and helps maintain long-term stability as the ASCP ecosystem grows.

# **15. IANA Considerations**

This document defines no new IANA registries and makes no requests of IANA at this time.

Future versions of ASCP may define media types, JOSE header extensions, or algorithm identifiers that require IANA registration.

## 15.1 Media Types

If future media types are introduced (e.g., "application/ascp+jws" or "application/ascp+jws+jwe"), they will be registered through the standard IANA Media Type registration procedures and documented in subsequent revisions of this specification.

## 15.2 JOSE Header Parameters

ASCP leverages established internet standards to ensure broad compatibility.

The following IETF registrations likely will be reserved for ASCP-specific JOSE usage:

- **JOSE JWS Header**: `{"typ": "ascp+jws"}` with MIME type: `application/ascp+jws`
- **JOSE JWE Header**: `{"typ": "ascp+jws+jwe"}` with MIME type: `application/ascp+jws+jwe`

These registrations would ensure that ASCP cryptographic envelopes are properly identified and handled by standards-compliant JOSE implementations while maintaining interoperability with existing cryptographic infrastructure.

# **16. References**

## **16.1 Normative References**

**RFC 7515**

Jones, M., Bradley, J., and N. Sakimura, *JSON Web Signature (JWS)*, RFC 7515, May 2015.

**RFC 7516**

Jones, M., Rescorla, E., *JSON Web Encryption (JWE)*, RFC 7516, May 2015.

**RFC 7518**

Jones, M., *JSON Web Algorithms (JWA)*, RFC 7518, May 2015.

**RFC 7517**

Jones, M., *JSON Web Key (JWK)*, RFC 7517, May 2015.

**RFC 4648**

Josefsson, S., *The Base16, Base32, and Base64 Data Encodings*, RFC 4648, October 2006.

**ALSP Specification**

Reframe Systems, *ASCP Log-Sync Protocol (ALSP)*, current version.

(Referenced for Layer-0 replication semantics and immutability model.)

**ASCP Trust and Identity Architecture**

Reframe Systems, *ASCP Trust & Identity Specification*, current version.

(Referenced for identity certificates and governance semantics.)

**ASCP Governance and Access Control**

Reframe Systems, *ASCP Governance Specification*, current version.

(Referenced for membership evaluation and policy semantics.)

**ASCP Grammar Specification**

Reframe Systems, *ASCP Artipoint Grammar & Structure*, current version.

(Referenced for Channel, Keyframe, and envelope attribute types.)

# **Appendix A — Examples (Informative)**

This example walks through the complete lifecycle of an ASCP articulation as it flows through **Layer 1: Channels** — from creation to secure distribution, following the updated specification.

## **A.1. Channel Declaration**

BNF Grammar Form:

```bnf
[11111111-2222-3333-4444-555555555555, jeff@reframe.systems, 2025-08-04T10:00:00.000Z,
  ["channel", "Hiring Team", "@HiringTeam"] .
  (
    payload_cipher := "AES256",
    message_signing := "ECDSA-P256",
    channel_access_alg := "Ed25519",
    bootstrap := false
  )
]

```

This declares a new secure channel @HiringTeam using:

- **AES-256-GCM** for payload encryption
- **ECDSA P-256** for signing
- **Ed25519** for replication authentication

## **A.2. Add Members to Channel**

```bnf
[22222222-3333-4444-5555-666666666666, jeff@reframe.systems, 2025-08-04T10:05:00.000Z,
  11111111-2222-3333-4444-555555555555 .
  (
    member + "user1@reframe.systems",
    member + "agent42@reframe.agents"
  )
]

```

Adds a human (<user1@reframe.systems>) and an AI agent (<agent42@reframe.agents>) to the Channel.

## **A.3. Create Initial Keyframe**

BNF Grammar Form:

```bnf
[33333333-4444-5555-6666-777777777777, jeff@reframe.systems, 2025-08-04T10:10:00.000Z,
  ["keyframe", "HiringTeam:v1", "urn:keyframe:@HiringTeam:v1"].
  (
    key_id := "v1",
    payload_cipher := "AES256",
    message_signing := "ECDSA-P256",
    channel_access_alg := "Ed25519"
  )
] supports {11111111-2222-3333-4444-555555555555}

```

This establishes **Keyframe v1** for the Channel.

## **A.4. Distribute Key Envelopes**

BNF Grammar Form:

```bnf
[44444444-5555-6666-7777-888888888888, jeff@reframe.systems, 2025-08-04T10:15:00.000Z,
  33333333-4444-5555-6666-777777777777 .
  (
    envelope::user1@reframe.systems := json:{ <channel-key-envelope-for-user1> },
    envelope::agent42@reframe.agents := json:{ <channel-key-envelope-for-agent42> }
  )
]

```

Each channel-key-envelope contains:

- AES-256 Channel Encryption Key (encrypted for the recipient)
- Ed25519 Channel Access Key (encrypted for the recipient)

## **A.5. Create an Articulation Sequence**

Now that the Channel infrastructure is established (Channel declared, members added, keys distributed), we have the actual *content* to communicate. This Articulation Sequence represents a meaningful cognitive action—in this case, bookmarking a project plan document. This is what will be signed, encrypted, and distributed through the secure Channel in the following steps.

BNF Grammar Form:

```bnf
[55555555-6666-7777-8888-999999999999, user1@reframe.systems, 2025-08-04T11:00:00.000Z,
  ["document", "Project Plan v1", uri:"https://docs.reframe.com/project-plan-v1.pdf"]
];

```

## **A.6. Sign the Articulation Sequence**

JWS Protected Header:

```json
{
  "alg": "ES256",
  "kid": "ascp:cert:aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "typ": "ascp+jws"
}

```

Flattened JWS JSON Example:

```json
{
  "protected": "<b64url-protected-header>",
  "payload": "<b64url-artipoint>",
  "signature": "<b64url-signature>"
}

```

## **A.7. Encrypt the Signed Articulation Sequence**

JWE Protected Header:

```json
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:33333333-4444-5555-6666-777777777777"
}

```

Flattened JWE JSON Example:

```json
{
  "protected": "<b64url-protected-header>",
  "iv": "<b64url-iv>",
  "ciphertext": "<b64url-ciphertext>",
  "tag": "<b64url-tag>"
}

```

## **A.8. Distribute via ALSP**

1. **Append**: Add the JWE object to the Channel's append-only log
2. **Distribute**: ALSP delivers the encrypted message to all authorized Channel members
3. **Decrypt Key**: Each member uses their channel-key-envelope to decrypt the AES and CAK keys using their private identity key.
4. **Decrypt Message**: Use the CAK for sync access to the channel at Layer 0 and use the AES key to decrypt the JWE payload and recover the signed Articulation Sequence at Layer 1.
5. **Verify Signature**: Validate the Articulation Sequence signature using the signer's EC public identity key and then the Articulation Sequence is passed to Layer 2.
6. **Parse**: The Articulation Sequence is parsed in Layer 2 and then bubbled up for inclusion into the Layer 3 DAG for view purposes.

## **A.9. Rotate Keys if Needed**

When membership changes or periodically:

```bnf
[66666666-7777-8888-9999-aaaaaaaaaaaa, jeff@reframe.systems, 2025-08-10T09:00:00.000Z,
  11111111-2222-3333-4444-555555555555 .
  (
    keyframe::kid := "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
  )
]

```

New key envelopes for this new keyframe are issued for remaining members. This only is required when we want forward secrecy, otherwise the keys don't need to be changed, we just add/remove membership from an administrative perspective.

## **A.10. End-to-End Flow Summary**

1. **Channel Creation** → bookmark("channel")
2. **Membership Additions** → annotation(member +)
3. **Keyframe Creation** → bookmark("keyframe") supports {channel}
4. **Key Distribution** → annotation(envelope::user := json:{...})
5. **Articulation Sequence Creation** → Content to be signed and placed in the channel
6. **Signing** → JWS with ECDSA P-256 using EC identity key
7. **Encryption** → JWE with AES-256-GCM using direct key mode
8. **Distribution** → Append to ALSP log
9. **Rotation** → New keyframe, re-wrapped envelopes

This MWE provides a reference for implementing the secure distribution path in ASCP Channels, demonstrating grammar-to-crypto mapping for the entire lifecycle.

# **Appendix B — Rationale & Design Notes**

This informative only appendix provides additional context on several architectural choices made in the ASCP Channels design. These notes are non-normative and are intended to aid reviewers, implementers, and designers in understanding the motivations behind the Layer-1 protocol structure.

## **B.1 Why Channels Do Not Parse Grammar**

Layer-1 intentionally treats all payloads as opaque and refrains from parsing the ASCP Artipoint Grammar. This separation is essential for stability, forward compatibility, and security. Grammar definitions evolve over time as ASCP’s semantic layer matures, and tying cryptographic envelope handling to any particular grammar structure would bind Channel implementations to specific syntax, schema, or semantic conventions.

Furthermore, Grammar-level constructs (e.g., membership declarations, Keyframe attributes, RACI roles, or rich semantic annotations) are evaluated by Layer-3 governance logic. Channels must not interpret these constructs, and attempting to do so would risk embedding policy or authorization logic into the cryptographic transport layer. By restricting Layer-1 to JOSE envelope handling and externally provisioned key material, Channels remain independent of semantic evolution, promote interoperability across heterogeneous deployments, and keep security reasoning centered around JOSE, identity certificates, and Keyframes.

## **B.2 Why Keyframes Are Expressed in Grammar**

Keyframes are expressed as Layer-2 Artipoints rather than as Layer-1 protocol messages for two primary reasons: **auditability** and **composability**.

First, Keyframes must be durable, inspectable objects in the shared DAG of Artipoints. Treating them as Grammar-level constructs ensures that key lifecycle events (initialization, rotation, supersession) are recorded in the same immutable, link-addressable structure as all other collaborative context. This fits the Cortex Layer’s design goal of treating all contributions—human or agentic—as addressable knowledge objects.

Second, expressing Keyframes at the Grammar layer allows them to be composed with other Artipoint types: governance policy, membership changes, Channel declarations, and other semantic structures. Layer-3 evaluators can then compute the effective cryptographic configuration by walking the DAG and determining the active Keyframe in context. From this evaluation, Layer-3 provisions Layer-1 with the key table, active Keyframe kid, and permitted algorithms. This separation ensures that cryptographic actions remain driven by semantic meaning, while Layer-1 continues to operate purely on JOSE payloads and provisioned keys.

## **B.3 Notes on Future Extensions**

The Channels layer is designed to accommodate future cryptographic and structural enhancements without requiring a redesign of the protocol. Possible areas of extension include:

### **HPKE and Per-Recipient Encryption**

Today, Channels use symmetric AES keys per Keyframe, resulting in group-wide confidentiality boundaries. Future revisions may adopt HPKE or hybrid modes that allow per-recipient encryption while maintaining compatibility with the Channel and Keyframe model. Such extensions would belong at Layer-1 and would require enriched Keyframe representations.

### **Alternative JOSE Serializations**

While Channels currently mandate Compact Serialization for operational envelopes, future deployments may benefit from JSON or JWE General Serialization for cases requiring explicit multi-recipient support, debug-friendly structures, or richer JOSE metadata. Any such evolution must preserve backward compatibility with the Compact format.

### **Algorithm Agility and PQC Transition**

The ASCP stack anticipates long-term upgrades to post-quantum signature and key establishment schemes. Keyframes, with their versioned UUID structure and logical placement in the Grammar, are an ideal mechanism for introducing new algorithm families without altering Envelope Format or Channel processing semantics.

### **Multi-Key or Multi-Layer Channels**

Some use cases may require layered or nested encryption, dual-signature models, or specialized Channels for compliance-driven workflows. These can be added as additional Channel types or envelope profiles while retaining the core Layer-1 machinery.

## **B.4 Comparison to Related Protocols**

ASCP Channels share design DNA with several existing secure messaging and transport systems but differ in key architectural dimensions:

### **TLS / DTLS**

Unlike TLS, Channels provide no session concept, no negotiation handshake, and no in-band identity exchange. Channels secure discrete artifacts in an immutable log rather than streams of transport-layer data.

### **MLS**

MLS provides tree-structured group keying with strong forward secrecy and per-epoch cryptographic state; Channels provide simpler Keyframe-scoped symmetric keys aligned to the shared DAG. MLS couples membership and key schedule tightly, while ASCP deliberately separates membership semantics from cryptographic state.

### **DIDComm**

DIDComm supports per-message encryption to specific recipients but lacks a shared immutable log as the coordination substrate. Channels invert this relationship: the immutable DAG is primary, and envelopes secure contributions into that DAG.

### **ActivityPub / Other DAG-like Systems**

ASCP differs by treating all shared state as cryptographically verifiable and by binding contributions to identity certificates and Keyframe-driven cryptographic state, rather than relying on server-level assurance.

These comparisons highlight ASCP’s unique goals: to provide cryptographically verifiable shared cognition across humans and agents, anchored in an immutable coordination substrate rather than a messaging protocol or transport stream.