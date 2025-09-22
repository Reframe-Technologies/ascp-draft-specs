# ASCP Channels: Secure Distribution Layer Specification

**Layer 1 of the ASCP Protocol Stack**

Version 0.1 - August 2025

## Overview

This document specifies Layer 1 of the ASCP protocol stack: **ASCP Channels**—the secure distribution and encryption infrastructure that enables collaborative cognition between humans and AI agents.

ASCP Channels unify cryptographic security with human-centric collaboration semantics through a foundational principle: **everything is an Artipoint**. All forms of structure—identity, membership, policies, and content—are expressed as atomic, signed statements distributed through secure logical groups. This approach creates group chat-like distribution mechanisms, but for structured, computable forms of context rather than simple messages.

Layer 1 operates between the Articulations Sequence Grammar (Layer 2) above and the distributed, local-first storage and transport mechanisms (Layer 0) below, providing the cryptographic envelope and distribution logic that transforms individual articulations into secure, collaborative shared cognition.

### Unified Cryptographic Model

Each Channel operates as a logical distribution group with its own membership, encryption keys, and access control policies. All articulations are cryptographically signed for authenticity and integrity using JOSE standards (JWS), with optional encryption for privacy (JWE). Channel membership directly controls who receives copies of articulation statements and can decrypt their payloads.

The encryption and signing model draws inspiration from S/MIME, adapting its core principles—per-recipient symmetric key encryption, public key based identity, and durable message signing—for ASCP's append-only articulation log structure. However, it uses modern JOSE (JWS+JWE) constructs for standards compliance and simplified implementation. Note: Neither S/MIME nor X.509 is actually used within the ASCP protocol—these systems provided design inspiration only.

This unified approach creates a simple yet powerful messaging model where cryptographic primitives align naturally with human-centric collaboration patterns, ultimately unifying structure, policy, identity, and privacy under a single model—structured articulations distributed through secure, append-only Channels.

## **Glossary**

- **ALSP (ASCP LogSync Protocol)**: The Layer 0 protocol that handles distributed storage and synchronization of Channel logs across a network clients/peers.
- **Artipoint**: The fundamental building block of the ASCP grammar - a single, immutable, addressable statement that captures one complete cognitive decision or piece of structured thinking. Think of it as a "cognitive atom." Every Artipoint has a UUID, Source, Timestamp and an optional Expression that can represent anything from bookmarks to permissions to policy declarations.
- **Articulation Sequence**: The formal encoding of one or more Articulation Statements bundled into an immutable, atomic, timestamped, and cryptographically signed sequence. This is what actually gets persisted and distributed via a Channel.
- **Articulation Statement** is the formal encoding of one or more cognitive actions using Artipoints. It represents what gets recorded when someone "articulates something" using a structured expression that captures coordination decisions using one the grammar's four patterns: Instantiation, Connection, Construction or an Annotation.
- **ASCP (Agent Shared Cognition Protocol)**: This very layer of the protocol stack that enables secure collaboration between humans and AI agents through structured articulations.
- **Bootstrapping Channel**: The initial Channel used to create and configure the root of trust for a Reframe organization. This Channel SHOULD operate encrypted to enable initial key distribution.
- **Channel**: A named group communication pathway used for distribution and optional encryption of Articulation Sequences. Each Channel has a universally unique identifier (UUID) for clear identification and also a friendly name for user selection (e.g., @Org, @HiringTeam).
- **Channel Access Key (CAK)**: A shared cryptographic credential used by Channel members to authenticate replication requests in the Layer 0 storage protocol. While Ed25519 is the current default implementation, the protocol supports flexible key types through JWS mechanisms to accommodate future cryptographic requirements.
- **Control Artipoint**: A class of Artipoint types used to manage governance, membership, key rotations, or other administrative actions within a Channel. Syntactically they are no different than any other type of Artipoint.
- **Identity Key**: The JWK-encoded elliptic curve key pair used by ASCP participants (humans and AI agents) for cryptographic identity, signing, and key agreement. Unlike X.509 certificates with centralized CAs, ASCP identity keys are typically self-generated and exchanged in JOSE-compliant JWK format, with key binding proven through JWT-based identity claim bundles that establish the connection between identity (like an email address or agent URN) and the public key.
- **JWE (JSON Web Encryption)**: The JOSE standard used by ASCP to encrypt Artipoint payloads for privacy within Channels.
- **JWK (JSON Web Key)**: The JOSE standard format used to represent cryptographic keys, including Channel symmetric keys and participant certificates.
- **JWS (JSON Web Signature)**: The JOSE standard used by ASCP to cryptographically sign all Artipoints for authenticity and integrity.
- **JOSE (JavaScript Object Signing and Encryption)**: The family of internet standards (JWS, JWE, JWK) that ASCP leverages for all cryptographic operations.
- **Key Envelope**: A structure within a Keyframe that contains both the Channel's symmetric encryption key and Channel Access Key (CAK) encrypted individually to each authorized participant's public key.
- **Keyframe**: A type of Artipoint used to introduce or rotate a Channel's cryptographic keys and manage Channel membership through encrypted key envelopes.
- **Log-Anchored Trust Model**: ASCP's approach where all identity certificates, Keyframes, and trust relationships are recorded in the immutable Channel log, ensuring signatures remain verifiable in perpetuity.
- **Symmetric Channel Key**: The AES-256 (or other symmetric cipher) key used to encrypt payloads for all Artipoints distributed within a specific Channel.

## **Core Design Principles**

1. **Composability**: Cryptographic policies should align with ASCP's semantically structured objects (Artipoints, Streams, Piles, Spaces) without enforcing rigid hierarchies.
2. **Orthogonality**: Distribution and access are decoupled from structural containment. Permissioning is driven by addressability (via Channels) rather than solely by hierarchy. Channels themselves are completely orthogonal such that they operate independently regardless of which Channel was used to create them, with no required inheritance of properties or permissions between Channels.
3. **Durability over Ephemerality**: Unlike messaging systems that optimize for forward secrecy, ASCP defaults to durable, shareable cognition. Forward secrecy is supported through key rotation, but this can usually be avoided.
4. **Human-Centric Identity**: Identity and access are centered on JWK based identity keys, which may be self-signed, rooted in local organizational CAs, or part of publically verifiable chain of trust.
5. **Agent Parity**: Agents are authenticated identically to humans, requiring their own JWT proven cryptographic identities to participate in shared cognition.
6. **Standards Interoperability**: ASCP leverages established internet standards for transport, cryptographic primitives, and identity management to ensure broad compatibility and avoid reinventing foundational protocols. This typically includes standard TLS for transport security, widely-adopted signature algorithms like ECDSA-P256, and JOSE JWT and JWK systems for seamless integration with existing identity provider infrastructure if and as needed.

These principles directly shape the technical architecture: standards interoperability drives our use of JOSE (JSON Web Signature and Encryption) for cryptographic operations; human-centric identity shapes how email addresses bind to underlying JWK-encoded key pairs; durability influences our approach to key management and message persistence; and orthogonality ensures Channels operate as independent distribution mechanisms.

## **ASCP Channels: Secure Distribution Architecture**

Channels are the primary unit of distribution in ASCP, functioning as secure, append-only communication pathways that bridge human-centric collaboration with robust cryptographic protection. Each Channel operates as a logical distribution group—analogous to secure group messaging protocols like MLS, but specifically designed for ASCP's immutable articulation model.

**Channels Handle Distribution, Not Structure**

Channels are fundamentally about *who gets what*—secure distribution and privacy scopes—rather than *how work is organized*. This orthogonal design separates two critical concerns:

- **Distribution scope**: Which participants receive copies of Artipoints (controlled by Channel membership)
- **Work structure**: How Artipoints relate semantically within Streams, Spaces, and organizational hierarchies (defined in Layer 2)

Articulation Sequences can be distributed across multiple Channels while belonging to one Stream, or multiple Artipoints from different Streams can share the same Channel for distribution. This decoupling enables flexible collaboration patterns without conflating security boundaries with organizational structure.

**Channel Identity and Naming**

Every Channel is identified by both a UUID for system-level uniqueness and a friendly user name (such as @Bootstrap, @Org, or @HiringTeam) for human usability at the UI/UX level. These friendly names are scoped to the Channel in which they were declared, allowing organizational flexibility without requiring global namespace coordination.

**What Channels Are NOT:**

- Channels are not hierarchical containers like folders
- Channels do not inherit permissions from Streams or Spaces
- Channel membership does not determine structural relationships between Artipoints

### **Cryptographic Foundation: Identity and Trust**

The security model begins with identity. All Articulation Sequences entering a Channel are cryptographically signed by their originator using a long-term private key associated with a JWK encoded identity key. This approach supports flexible trust models—where keys are anchored to a root of trust for the ASCP organization, but then that may be self-signed for informal collaboration, issued by local organizational authorities, or anchored in traditional PKI chains via established certificate authorities. This design ensures that both humans and AI agents can participate as first-class cryptographic identities within the same trust framework.

### **Privacy Through Channel-Level Encryption**

While signing ensures authenticity and integrity, privacy is achieved through Channel-level encryption. Each Channel maintains an optional symmetric encryption key—typically AES-based, though the architecture supports other cipher algorithms as specified in the Channel's control metadata. When present, this key encrypts all Artipoint payloads before distribution, ensuring that only authorized Channel members can decrypt and read the content.

The symmetric key itself is secured through public key cryptography: it is encrypted individually to each authorized participant's public key, creating personalized "key envelopes" that enable secure key distribution without requiring pre-shared secrets or complex key agreement protocols.

### **Channel Membership and Cryptographic Keyframes**

Channel membership is an attribute of the Channel itself, establishing the persistent communications floor for all authorized participants. Keyframes are special Artipoints that provide the cryptographic materials needed to participate in that membership—serving as the dynamic encoding layer that makes secure Channel participation possible.

These governance messages serve multiple critical functions: they introduce or rotate a Channel's symmetric encryption key, create individualized key envelopes for each authorized participant, and establish versioned key policies (such as key\_id: v3) that enable both forward secrecy and historical access control.

Keyframes are themselves signed Artipoints, ensuring that cryptographic changes are authenticated and auditable. While Channel membership provides stability and continuity for ongoing collaboration, Keyframes can be rotated and updated as the dynamic element controlling how that collaboration is cryptographically encoded—typically retaining access to previous keys for historical decryptability, but enabling forward secrecy by rotating to new keys when sensitive membership changes occur.

### **Complete Message Lifecycle**

The complete lifecycle of an Articulation Sequence through ASCP Channels demonstrates how these cryptographic layers work together

1. **Authoring**: A user or agent creates an immutable Articulation Sequence containing one or more Articulation Statements built from Artipoints
2. **Signing**: The Articulation Sequence is cryptographically signed using the author's private key, establishing authenticity and integrity
3. **Encryption**: If the target Channel has encryption enabled, the signed Articulation Sequence is encrypted using the Channel's current symmetric key, ensuring privacy
4. **Distribution**: The resulting cryptographic envelope—containing the signed and optionally encrypted Articulation Sequence—is appended to the Channel's dedicated ALSP (ASCP LogSync Protocol) log for replication to all authorized participants

This multi-layered approach creates a secure, auditable, and human-centric messaging system where cryptographic envelopes are distributed through Layer 0's ALSP protocol, while trust validation operates through an immutable log-anchored model detailed below.

## **Trust Model: Immutable Log-Time Validation**

ASCP uses a **log-anchored trust model** rather than live PKI validation:

- All identity certs, keyframes, and delegation structures are **articulated into the channel log**.
- Their trustworthiness is **determined at the time of articulation**, based on:
  - Who signed them
  - What other Artipoints reference them
  - Whether they trace up to a trusted org-level root

This approach ensures that:

- **Expired or revoked certs do not retroactively invalidate past signatures**
- All signatures remain verifiable *in perpetuity* as long as their trust chain exists in the log
- Trust is local, explicit, auditable, and not subject to out-of-band revocation

> In other words, ASCP secures meaning *as it was understood and validated at the time*—not according to mutable or external live state.

This model supports long-term verification and archival of Artipoints—even if certificates later expire or are rotated—while maintaining full auditability and cryptographic integrity within the immutable log structure.

Building on this log-anchored trust foundation, ASCP implements Channel governance through the same articulation-based approach, ensuring that all administrative operations are as transparent and auditable as the content they manage.

## **Channel Management and Membership Governance**

Channel management operates through articulation itself—membership changes, key rotations, and policy updates are expressed as ordinary signed Artipoints rather than special control messages. This approach ensures full auditability and maintains consistency with ASCP's core principle that all meaning derives from structured articulations.

### **Bootstrapping and Trust Establishment**

Every organization begins with a bootstrapping Channel containing articulations signed by the founding organizational certificate. This Channel operates unencrypted to enable initial key distribution, but all articulations remain cryptographically signed. The bootstrap signer can delegate authority to others or articulate new Spaces and Streams that become the basis for further organizational structure.

### **Channel Creation and Independence**

While the bootstrapping Channel conventionally defines other organizational Channels (such as @org), any Channel can architecturally articulate new Channels into existence. This choice is purely a matter of organizational governance preference.

Channels are completely orthogonal in operation—they inherit no properties, permissions, or policies from their creating Channel. Each maintains independent membership, encryption keys, and governance through its own articulated control statements.

### **Common Patterns**

Organizations typically establish private Channels for individual users to hold their personal articulations. Administratively, organizations may choose to vault these "private" channel keys by encrypting the symmetric key to both the user and the organization's public key during channel creation.

This unified approach aligns with ASCP's articulation-based model while enabling the single append-only log per Channel that ALSP requires, where governance and content coexist seamlessly.

## **Cryptographic Implementation**

ASCP uses JOSE (JSON Object Signing and Encryption) standards—specifically JWS (JSON Web Signature), JWE (JSON Web Encryption), and JWK (JSON Web Key)—to secure all data across its lifecycle. This implementation accomplishes five core security functions:

- **Sign immutable Artipoint statements** using JWS
- **Encrypt messages in ASCP Channels** using JWE
- **Encode channel-level secret keys** (AES + Ed25519) using JWK format
- **Distribute keys securely to recipients** using JWE envelopes with JWK EC public identity keys
- **Move all keys and certificates out-of-band** via articulations (user identity keys) and Keyframes (channel keyframes) for encoding efficiency and key vaulting

The design emphasizes standards compliance, long-term durability, agent compatibility, and separation of message vs. key state.

All signed and encrypted message formats in ASCP **MUST use Compact Serialization** of JWS and JWE as specified in [RFC 7515 §7.1](https://datatracker.ietf.org/doc/html/rfc7515#section-7.1) and [RFC 7516 §7.1](https://datatracker.ietf.org/doc/html/rfc7516#section-7.1).

### Channel Level Signing of Articulation Sequences

Articulation Sequences going into the Channel's log are first signed:

- Protocol: **JWS** (JSON Web Signature)
- Payload: Canonical UTF-8 Articulation Sequence string
- Format: JWS **Compact Serialization**
- Signing Algorithm: **ECDSA P-256** (validated using JWK encoded identity keys)

The message signing key comes from the signer's own private identity key. The public identity key needed for validation is accessed via the JWK representation of the signer's EC public key, which is referenced in the `kid` field of the JWS header and distributed **out-of-band** via articulation statements related to the signer's identity. This simplifies message JWS headers and reduces communication and log size overhead. This approach also facilitates key vaulting and other mechanisms for long-term storage and validation of messages, even if the sender's key expires at some point in the future. Both the JWK for validation and the message itself are part of the articulation log, but are maintained independently.

The following is the **JWS protected header**:

```
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "ascp+jws"
}
```

Complete JOSE **JWS compact serialization** example:

```
<header>.<payload>.<signature>
```

This compact format consists of three base64url-encoded components separated by periods:

- **Header**: Base64url-encoded JWS protected header JSON
- **Payload**: Base64url-encoded Articulation Sequence UTF-8 string
- **Signature**: Base64url-encoded ECDSA P-256 signature

We use the **JWS compact serialization format** (RFC 7515 §7.1) because:

- It provides maximum space efficiency for ALSP log storage
- It aligns with standard JWT implementations and tooling
- It maintains consistency with space-constrained environments where ASCP may be deployed

### Channel-Level Encryption of Articulation Sequences

After the required JWS signing, then things are *optionally* encrypted:

- Protocol: **JWE** (JSON Web Encryption)
- Payload: A complete **JWS compact string** (signed Articulation Sequence)
- Content Encryption: **AES-256-GCM**
- Key Management Mode: **Direct (alg: dir)**

This Channel AES key is generated and distributed **out-of-band** via encrypted key envelopes established via the Keyframe articulation statements related to the channel's own definition in the log. No key agreement or wrapping is performed at the message level and only the key-id (kid) is needed to identify the proper Keyframe for decoding. This simplifies message JWE headers and reduces communication and log size overhead. This approach also facilitates key vaulting and other mechanisms for key recovery.

**Format: JWE Compact Serialization**

Example JWE protected header:

```
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
}
```

Complete JOSE **JWE compact serialization** example:

```
<header>.<key>.<init-vector>.<ciphertext>.<auth-tag>
```

This compact format consists of five base64url-encoded components separated by periods:

- **Header**: Base64url-encoded JWE protected header JSON
- **Key**: Encryption Key is always Empty (omitted due to direct encryption mode)
- **init-vector**: Base64url-encoded initialization vector for AES-GCM
- **Ciphertext**: Base64url-encoded encrypted JWS payload
- **Auth-Tag**: Base64url-encoded AES-GCM authentication tag

**Note:** When using direct encryption (`alg: "dir"`), the second component (JWE Encrypted Key) is an empty string, resulting in two consecutive periods in the compact serialization. This is correct per RFC 7516 for ASCP's direct encryption approach where the symmetric key is distributed out-of-band via Keyframes.

We use the **JWE compact serialization format** (RFC 7516 §7.1) because:

- It provides maximum space efficiency for encrypted message storage
- It is URL-safe and suitable for constrained transmission environments
- It aligns with the JWS compact format for consistency
- It maintains compatibility with standard JOSE libraries and tooling

However, when implementing this encryption approach, proper IV handling is absolutely critical.

### Critical: Initialization Vector (IV) Requirements for AES-GCM

When encrypting ASCP Channel messages using **AES-256-GCM**, the init-vector (Initialization Vector) parameter is critical to the **confidentiality and integrity** of the encrypted payload. ASCP use of JWE enforces the following requirements for IV handling:

- **A new IV MUST be generated for each encryption operation**. This is especially important in ASCP as the AES key is durable and encrypted payloads remain at rest in the ASCP channel logs.
- IVs MUST be **cryptographically random** using a secure entropy source and **at least 96 bits (12 bytes)** in length, as recommended by [NIST SP 800-38D](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf) and [RFC 7516 §5.3](https://datatracker.ietf.org/doc/html/rfc7516#section-5.3).
- Reuse of an IV under the same AES key can lead to catastrophic compromise of message security, including plaintext recovery and authentication bypass.

This strict IV management protects all encrypted Articulation Sequences against nonce reuse vulnerabilities that could expose both message content and authentication materials.

### **Compression of JWS Compact Serialization via JWE**

The plaintext input to JWE in this application is the JWS Compact Serialization string (header.payload.signature). JOSE defines an optional DEFLATE compression step ("zip":"DEF") that may be applied to the plaintext before encryption.

Implementations **SHOULD** apply DEFLATE compression to the JWS Compact Serialization string whenever the uncompressed string length exceeds **200 bytes**.

This requirement is justified by the following considerations:

- **Payload characteristics**: Artipoint payloads are UTF-8 encoded Unicode text drawn from the ASCP Grammar. The grammar is highly ASCII text centric, with recurring structural elements such as UUIDs, timestamps, keywords, and bracketed constructs.
- **Serialization form**: Signing first produces a JWS Compact Serialization string, which base64url-encodes both header and payload and appends a base64url signature. This introduces a 33% expansion relative to the raw payload plus fixed delimiter overhead.
- **Compression behavior**: Empirical analysis demonstrates that DEFLATE with dynamic Huffman coding reliably recovers not only the base64url overhead but also achieves significant additional reduction by exploiting repeated ASCII patterns. Compression consistently outperforms the raw JWS string once the full JWS exceeds \~150 bytes (≈32 bytes of raw Articulation Statement payload).
- **Trip point selection**: To provide a conservative safety margin across different JOSE headers and signature algorithms, the normative threshold is set at **200 bytes total JWS Compact string length**. Below this size, compression overhead can slightly outweigh savings; above it, compression consistently yields net reductions, reaching 40–60% of raw ASCP grammar as we reach into multiple KiB payloads.

Therefore, when encapsulating a JWS Compact Serialization in JWE, implementations **SHOULD** set the "zip":"DEF" parameter and apply DEFLATE compression whenever the plaintext JWS string length exceeds 200 bytes.

## Channel Key Management: Out-of-Band Wrapping

Each ASCP channel has two shared keys distributed via keyframes:

- **Channel AES Key** — used for symmetric encryption of signed Articulation Sequences.
- **Channel Access Key (CAK, Ed25519)** — used by all channel participants to authenticate replication requests to other nodes holding channel logs.

> **Note:** This Ed25519 key is **not** used to sign Artipoints. Artipoints are always signed by the originator’s personal identity key (typically ECDSA P-256 private key). The replication key functions as a shared *token* to authorize log access, not authorship.

These are **NOT included in messages**. Instead, they are:

- Encoded as **JWKs**
- Encrypted (wrapped) using **recipient's EC public key** via **JWE (ECDH-ES)**
- Delivered or vaulted in a separate object: `channel-key-envelope` as part of aritculated Keyframe statments in the grammar.

### AES Key JWK Format:

```
{
  "kty": "oct",
  "k": "<base64url AES-256 key>",
  "alg": "A256GCM",
  "use": "enc"
}

```

### Ed25519 Channel Access Key (CAK) JWK Format:

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

### EC Public Identity Key in JWK Format:

This is the JWK representation of a recipient's EC public key. It is stored in the directory of public keys and referenced via the associated `kid`.

```
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
- `kid`: Key identifier used in JWE and JWS headers and directory lookup

**Note:** The `use` field is intentionally omitted. This allows the same key to be used for both:

- Signature verification in JWS (e.g., validating signed Artipoints)
- Key agreement in JWE (e.g., encrypting channel key envelopes)

This is compliant with RFC 7517 §4.2, which permits keys to omit the use field to avoid unnecessary restriction.

## Channel Key Envelope Structure

The following structure defines the JSON format for channel key envelopes that appear as the value of an envelope attribute targeted at a specific user in Keyframe artipoints:

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

Each key in the envelope (aes\_key\_jwe and auth\_key\_jwe) is a standard **flattened JWE JSON object**, where the payload is a serialized **JWK** representing the symmetric AES key or the Ed25519 private key, respectively. These are encrypted to the intended recipient using their EC public identity key additionally pointed to by the `recipient_cert` within the envelope. This key reference MUST be the same as the `kid` used for wrapping both of the contained keys.

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

### Future-Proofing for Post-Quantum Cryptography

To prepare for the eventual need for post-quantum cryptographic robustness, ASCP will likely evolve its key-wrapping strategy to use **Hybrid Public Key Encryption (HPKE)** for the JWE envelope securing the Channel Keys. HPKE offers modern encryption primitives compatible with post-quantum algorithms and is an emerging IETF standard. Though not yet part of JOSE, future implementations may use headers like `alg: hpke-kem+X25519+HKDF-SHA256` where KEM (Key Encapsulation Mechanism) provides the post-quantum key establishment. ASCP envelope formats should remain forward-compatible to enable this transition without structural disruption.

Similarly, the explicit algorithm selection inherent in JOSE's JWS and JWK formats provides natural migration paths for both Artipoint message signing and Ed25519 channel authentication keys. Future implementations can seamlessly transition from `ES256` to post-quantum signature algorithms like `ML-DSA-65` (derived from CRYSTALS-Dilithium) by updating the `alg` field in JWS headers and corresponding JWK representations. This algorithm agility ensures that ASCP remains cryptographically robust across all dimensions—key wrapping, message signing, and channel authentication—without requiring structural changes to the protocol's envelope and referencing mechanisms.

## Summary of JOSE Usage in ASCP

| Use Case                        | Protocol | Format             | Algorithm(s)        | Key Material Source                                                       |
| ------------------------------- | -------- | ------------------ | ------------------- | ------------------------------------------------------------------------- |
| Sign Artipoints                 | JWS      | JSON Serialization | ES256 (ECDSA P-256) | EC public identity key pair (public key referenced by kid for validation) |
| Encrypt Artipoint Messages      | JWE      | JSON Serialization | AES-256-GCM         | Channel AES key (out-of-band referenced via kid)                          |
| Distribute Channel AES Key      | JWE      | JSON Serialization | ECDH-ES + A256KW    | EC public identity key (referenced by kid)                                |
| Distribute Ed25519 Auth Key     | JWE      | JSON Serialization | ECDH-ES + A256KW    | EC public identity key (referenced by kid)                                |
| Identity for Signing / Wrapping | JWK      | JSON               | ES256 / Ed25519     | Embedded or linked via kid                                                |

## ASCP Key Identifier (kid) Format

ASCP uses a structured `kid` format to reference cryptographic material stored out-of-band in articulation statements. Each `kid` acts as a pointer to a certificate or keyframe within the channel's log, allowing JOSE operations to resolve the appropriate material for message validation, decryption, or key recovery.

While `kid` is optional in JOSE specifications, ASCP **MUST** include `kid` explicitly in all message headers to ensure determinism during key rotation, avoiding ambiguity when multiple writers generate messages while updated key material is still propagating through the network.

### Format Specification

```plaintext
ascp:<type>:<uuid>
```

Where:

- `ascp` identifies this as an ASCP-specific key identifier
- `<type>` indicates the kind of cryptographic material being referenced
- `<uuid>` is the UUID of the articulation statement containing the key material

### Supported Types

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
- The index indicates which JWK encoded key was used for this Bootstrap channel message. The index is the offset, starting with zero, into the array of keys provided as part of the ALSP hello exchange in Layer 0. As such, these keys move out of band of the ASCP articulated database of Keyframes as there would be no other channel to store this in. See ALSP Layer 0 channel references and ASCP Bootstrap process documents for more details.
- The referenced index must always be the highest known index at the time of the message is being encoded into the Bootstrap channel.

> **Note:** The JOSE protected headers, even in JWE are ***not*** **encrypted**, but *are* **integrity-protected** as part of the AEAD encryption process. This means the `kid` is visible to the recipient prior to decryption, enabling proper key selection, but cannot be tampered with without causing decryption failure. This design ensures that key identifiers are safely discoverable while still cryptographically bound to the encrypted payload.

### Usage Examples

**JWS Signing (Artipoint validation):**

```json
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "ascp+jws"
}
```

**JWE Message Encryption:**

```json
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
}
```

**JWE Key Wrapping (in channel-key-envelope):**

```json
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "typ": "jwk",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}
```

### Lookup Process

1. **Parse the kid** to extract the type and UUID
2. **Locate the articulation statement** with the specified UUID in the channel logs
3. **Extract the cryptographic material** based on the type:
   - For `cert:` - retrieve the JWK encoded EC public identity key from the articulation statement
   - For `keyframe:` - locate the appropriate channel-key-envelope for the current recipient
4. **Use the material** for the intended cryptographic operation (signing, verification, encryption, or decryption)

This approach ensures that all cryptographic operations can be independently verified and audited by examining the articulation logs, while keeping individual messages compact and efficient.

## Channel Key Rotation and Re-Wrapping Policy

Channel keys in ASCP—specifically the AES-256 encryption key and the Ed25519 channel authentication key—are long-lived and must be rotatable without compromising the immutable log structure or requiring re-encryption of past messages.

### Rotation Scenarios

1. **Channel AES Key Rotation**
   - Generate a new AES-256 key
   - Issue a new channel-key-envelope with the updated key
   - Begin encrypting future Artipoints using this new key
   - Maintain access to previous key for historical decryption
2. **Channel Ed25519 Auth Key Rotation**
   - Generate a new keypair for future challenge/response use
   - Wrap and deliver in an updated channel-key-envelope
   - Maintain the old key if needed for signature validation (unless deprecating)
3. **Recipient EC Identity Key Pair Rotation**
   - When a recipient’s public key changes, issue a new channel-key-envelope for that recipient only
   - Wrap the current channel AES and Ed25519 keys using their new EC public key

### Operational Notes

- Channel history remains untouched—old messages remain decryptable with their original keys
- Multiple envelopes may coexist for different recipients or key generations
- Recipient key rotation does **not** trigger re-encryption of the channel—just new envelopes
- Channels may record a policy attribute that specifies how often keys should rotate via the rotation\_interval\_days key in the channel-key-envelope
- Revoking a recipient means **not issuing future envelopes** to that identity; past data remains as-is

Recipients MUST retain the ability to decrypt historical messages using past `channel-key-envelopes`, but SHOULD enforce `valid_from` and `rotation_interval_days` metadata for acceptance of newly encrypted payloads. This ensures expired or deprecated keys are not reused for forward-secrecy violations, and encourages consistent key lifecycle enforcement across participants.

## Security Analysis

### **Architectural Security Properties**

ASCP Channels achieve security through a unified architectural approach. By reducing all elements—Spaces, Streams, Piles, Policies, Membership Changes—to signed Artipoints published into Channels, we eliminate the need for a separate access control substrate while retaining human-centric semantics backed by robust cryptographic primitives. This uniformity supports distributed permissioning, control, and composability without introducing complex access control hierarchies that could create security vulnerabilities.

### **Distribution and Visibility Control**

ASCP separates distribution from visibility through its Channel model. While Artipoints can be distributed to multiple Channels (with UUID-based deduplication by recipients via Layer 2 to Layer 3 hand-off), actual visibility is controlled through encryption keys and channel access permissions. Each Channel maintains its own ALSP log, ensuring that access control decisions are cryptographically enforced rather than relying solely on application-level permissions.

### **Selective Historical Access and Forward Secrecy**

The Keyframe versioning system enables sophisticated access control patterns. Upon Channel creation, an initial Keyframe (v1) defines the symmetric encryption key and creates envelopes for founding members. New participants can be granted selective access: historical access by receiving original Keyframes encrypted to their certificate's public key, or forward-only access through new Keyframes (v2+) that establish fresh encryption keys. Since Keyframes are themselves signed, versioned Artipoints in the Channel log, all access control decisions are auditable and composable, supporting fine-grained privacy policies.

### **CAK Security Considerations**

The **Channel Access Key (CAK)** serves as a lightweight mechanism for **replication gating** at the ASCP LogSync Protocol (ALSP) layer. Possession of the CAK enables a peer to establish replication sessions for a given channel, but **does not grant access to the underlying channel content**. Content authenticity and confidentiality are governed separately through the channel’s signing and encryption keys (distributed in Keyframes and per-recipient envelopes).

**Threats**

The primary security concerns involve unauthorized replay or reuse of CAKs, where adversaries who obtain a CAK may attempt to replay it to initiate replication with peers. Long-lived CAK exposure presents another risk, as CAKs typically remain stable across membership events, making a leaked CAK a standing exposure concern. Additionally, membership revocation edge cases can occur when users are administratively removed from a specific channel but remain authorized elsewhere in the repository, potentially allowing continued replication attempts while still in possession of the CAK.

**Mitigations**

- **Opaque Replication:** Replicating with a CAK alone yields only encrypted ciphertext. Without the channel’s encryption keys, no useful content is exposed.
- **Repository Authorization Required:** Peers must also present valid repository-level identity credentials. A revoked or non-member identity cannot establish ALSP sessions, regardless of CAK possession.
- **Separate Cryptographic Domains:** The CAK is independent from channel signing and encryption keys. Content confidentiality and authenticity remain intact even if a CAK is leaked.
- **Optional Key Rotation:** Implementations MAY rotate CAKs or channel encryption keys if stricter separation of membership is required (e.g., in the rare case of channel-only revocation).
- **Operational Controls:** Replication services SHOULD apply rate-limiting, anomaly detection, and logging to detect and contain abuse of replayed or leaked CAKs.

**Residual Risks**

- A leaked CAK may allow an adversary who is also a valid repository member to waste bandwidth by requesting encrypted blobs from peers. This does not compromise content confidentiality but may incur resource cost.
- In the rare case where a user remains system-authorized but is removed from a single channel, enforcement requires proactive CAK and/or channel encryption key rotation. This case is uncommon in practice, given that removal from a channel typically coincides with system-level revocation.

### **Attack Resistance**

ASCP's model provides inherent resilience against common distributed system attacks:

- **Replay Protection**: Artipoints are uniquely identified via UUIDs and signed by originators, making replayed or duplicated messages easily detectable and traceable through append-only ALSP logs.
- **Fork and Equivocation Resistance**: Signed, deterministically ordered logs per Channel ensure that attempts to introduce conflicting control Artipoints to different participants are detectable and reconcilable. The protocol assumes full eventual visibility within Channels and treats transparency, auditability, and deterministic ordering as first-class security properties.
- **Long-term Integrity**: The log-anchored trust model ensures that expired or revoked certificates do not retroactively invalidate past signatures, while maintaining full cryptographic verification capabilities within the immutable log structure.

While explicit snapshot commitments (such as Merkle roots) can be added for additional assurances in adversarial environments, the base protocol already satisfies these integrity requirements for most collaborative contexts.

## Implementation Considerations

### **Articulation Statement Formatting**

Unlike many cryptographic protocols that require canonical serialization for signature validation, **ASCP does not require canonicalization of Artipoint strings for cryptographic correctness**. Artipoints are signed once at the source and never reconstructed—each has a universally unique identifier (UUID) and is treated as an immutable, read-only unit. Recipients verify signatures against the original signed bytes rather than re-serialized versions, and the ASCP DAG is built via reference (by UUID) rather than regeneration.

However, consistent formatting provides significant operational benefits for ASCP implementations, including improved storage efficiency, reliable diffing and auditing tools, and consistent behavior across different implementations and platforms.

**Recommended Articulation Statement formatting:**

- UTF-8 encoding with no BOM (Byte Order Mark)
- Unix-style line endings (LF without CR)
- No trailing whitespace
- Single space after commas and semicolons, except when at end of line
- Attribute lists ordered alphabetically by key
- Timestamp fields in strict ISO 8601 with millisecond precision (Z suffix enforced)

**JSON payload formatting** within the grammar (e.g., json: blocks or attribute values) should follow best practices from [RFC 8785 – JSON Canonicalization Scheme (JCS)](https://datatracker.ietf.org/doc/html/rfc8785), unless the structure being encoded requires a different format:

- UTF-8 encoding (strict)
- All keys sorted lexicographically (by UTF-16 codepoint)
- No insignificant whitespace
- Strings using double quotes (") with proper JSON escaping
- Numbers in exact format (no trailing .0, scientific notation, or inconsistent formatting)

These formatting choices are **not required for signature correctness**, but are highly encouraged to promote consistent encoding, diffability, and tooling interoperability across ASCP implementations.

### **Standards Compliance and Registration**

ASCP leverages established internet standards to ensure broad compatibility. The following IETF registrations define ASCP-specific JOSE usage:

- **JOSE JWS Header**: `{"typ": "ascp+jws"}` with MIME type: `application/ascp+jws`
- **JOSE JWE Header**: `{"typ": "ascp+jws+jwe"}` with MIME type: `application/ascp+jws+jwe`

These registrations ensure that ASCP cryptographic envelopes are properly identified and handled by standards-compliant JOSE implementations while maintaining interoperability with existing cryptographic infrastructure.

## Grammar Specification: Channel Articulation in ASCP

This appendix defines how ASCP Channels and related cryptographic governance mechanisms are expressed using the Artipoint Grammar, building on the foundational principles of declarative articulation, cryptographically scoped collaboration, and structured shared cognition.

### Channel Declaration Artipoint

Channels are declared using a bookmark Artipoint with the type "channel". The declaration is self-contained, minimal, and includes cryptographic metadata:

```bnf
[uuid, source, timestamp,
  ["channel", "Hiring Team", "@HiringTeam"] .
  (
    payload_cipher := "AES256",
    message_signing := "ECDSA-P256",
    channel_access_alg := "Ed25519",
    bootstrap := false
  )
]

```

Required Attributes:

- payload\_cipher := The symmetric cipher used for payload encryption, if any, as channels may be operated in the clear, if desired. Use **"none"** to explicitly disable encryption. Note: In current implementations, a bootstrap Channel must always be in the clear (see below).
- message\_signing := The signing algorithm required for Artipoint signatures in this Channel. All articulations must be signed using a secure algorithm. Typically we expect this to be **ECDSA-P256** for full interoperability with JOSE JWS standards-based elliptic curve based key-pair systems.
- channel\_access\_alg := The signature algorithm used for the Channel Access Key (CAK) credentials in the Layer 0 storage and synchronization protocol. **Ed25519** SHOULD be used, or **"none"** to disable channel access authentication.

Optional Attributes:

- bootstrap := Boolean flag indicating if this is the organizational bootstrapping Channel. Current implementations do not support a cipher for a bootstrap channel as there is no defined mechanism for passing the symmetric key to accomplish this out-of-band of any channel.
- key\_id := The identifier for the currently active symmetric key (see Keyframes)

### Channel Keyframe Establishment or Rotation (via Annotation pattern)

Channel Keyframe establishment or rotations are articulated as annotation Artipoints applied to the Channel UUID. The correct full-form expression is:

```bnf
[uuid, source, timestamp,
  channel-uuid .
  (
    keyframe::kid := "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002",
  )
]
```

Required Attributes:

- kid := The keyframe identifier for the currently active channels keys (see Keyframes)

### Membership Management (via Annotation pattern)

Membership changes are articulated as annotation Artipoints applied to the Channel UUID. The correct full-form expression is:

```clike
[uuid, source, timestamp,
  channel-uuid .
  (
    member + "user1@reframe.systems",
    member + "agent42@reframe.agents"
  )
]
```

### Notes:

- Each member + adds a recipient
- Removal is done with member - user\@domain
- All membership changes are traceable and auditable

### Keyframe Declaration (via Construction pattern)

Keyframes are declared as bookmark Artipoints with the type "keyframe", and are constructed as children of the corresponding Channel using the `supports` operator.

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

Notes:

- version := the protocol encoding version of the keyframe artipoint. This must be a value of 1 for this version of the protocol.
- payload\_cipher := should match what appears in the Channel declaration, but in principle it is possible to change the keying mechanism for a given Keyframe through this mechanism.
- message\_signing := should match what appears in the Channel declaration, but in principle it is possible to change the keying mechanism for a given Keyframe through this mechanism.
- channel\_access\_alg := should match what appears in the Channel declaration, but in principle it is possible to change the keying mechanism for a given Keyframe through this mechanism.
- The `supports` operator links the Keyframe as a child of the Channel. 

### 5. Channel Key Envelope Distribution (via Attributes pattern)

The encrypted  key material is distributed via envelope:: typed attributes attached to the Keyframe Artipoint. These follow the grammar supporting typed keys where the attribute value includes the JSON formatted `channel-key-envelope` itself.

```clike
[uuid, source, timestamp,
  keyframe-uuid .
  (
    envelope::user1@reframe.systems := json:{ <channel-key-envelope> },
    envelope::agent42@reframe.agents := json: { <channel-key-envelope> }
  )
]

```

This allows each authorized participant to decrypt both:

- The Channel's symmetric key used for Layer 1 message encryption
- The Ed25519 private key used to sign Layer 0 sync\_request access proofs. The system is defined more fully in the Layer 0 ASLP specification as this is where it is used.

Notes:

- These are embedded directly in the Keyframe, avoiding external indirection
- Only recipients with corresponding private keys can decrypt

### Summary of Patterns

| Purpose                 | Pattern                                              | Notes                                                            |
| ----------------------- | ---------------------------------------------------- | ---------------------------------------------------------------- |
| Declare a Channel       | bookmark("channel")                                  | Includes cipher and signing algorithm choices                    |
| Activate Keyframe       | annotation(key\_id :=)                               | Specifies the active Keyframe via version and UUID               |
| Manage Membership       | annotation(member +)                                 | Applied to Channel UUID                                          |
| Declare Keyframe        | bookmark("keyframe") supports {channel}              | Supporting child of Channel                                      |
| Distribute Key Envelope | annotation(envelope::user := \<channel-key-evelope>) | Typed attribute with direct JSON payload as the attribute value. |

## ASCP Channels Minimal Working Example (MWE)

This example walks through the complete lifecycle of an ASCP articulation as it flows through **Layer 1: Channels** — from creation to secure distribution, following the updated specification.

### **1. Channel Declaration**

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

### **2. Add Members to Channel**

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

### **3. Create Initial Keyframe**

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

### **4. Distribute Key Envelopes**

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

### **5. Create an Articulation Sequence**

BNF Grammar Form:

```bnf
[55555555-6666-7777-8888-999999999999, user1@reframe.systems, 2025-08-04T11:00:00.000Z,
  ["document", "Project Plan v1", uri:"https://docs.reframe.com/project-plan-v1.pdf"]
];

```

### **6. Sign the Articulation Sequence**

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

### **7. Encrypt the Signed Articulation Sequence**

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

### **8. Distribute via ALSP**

1. **Append**: Add the JWE object to the Channel's append-only log
2. **Distribute**: ALSP delivers the encrypted message to all authorized Channel members
3. **Decrypt Key**: Each member uses their channel-key-envelope to decrypt the AES and CAK keys using their private identity key.
4. **Decrypt Message**: Use the CAK for sync access to the channel at Layer 0 and use the AES key to decrypt the JWE payload and recover the signed Articulation Sequence at Layer 1.
5. **Verify Signature**: Validate the Articulation Sequence signature using the signer's EC public identity key and then the Articulation Sequence is passed to Layer 2.
6. **Parse**: The Articulation Sequence is parsed in Layer 2 and then bubbled up for inclusion into the Layer 3 DAG for view purposes.

### **9. Rotate Keys if Needed**

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

### **End-to-End Flow** 

1. **Channel Creation** → bookmark("channel")
2. **Membership Additions** → annotation(member +)
3. **Keyframe Creation** → bookmark("keyframe") supports {channel}
4. **Key Distribution** → annotation(envelope::user := json:{...})
5. **Articulation Sequence Creation** → Grammar instantiation
6. **Signing** → JWS with ECDSA P-256 using EC identity key
7. **Encryption** → JWE with AES-256-GCM using direct key mode
8. **Distribution** → Append to ALSP log
9. **Rotation** → New keyframe, re-wrapped envelopes

This MWE provides a reference for implementing the secure distribution path in ASCP Channels, demonstrating grammar-to-crypto mapping for the entire lifecycle.

## Summary

ASCP Channels realize the foundational principle that "everything is an Artipoint" by unifying cryptographic security with human-centric collaboration semantics. Through secure, append-only distribution pathways, individual articulations are transformed into collaborative shared cognition between humans and AI agents.

This specification demonstrates how Layer 1 of the ASCP protocol stack bridges the structured grammar above with distributed storage below, creating group chat-like distribution mechanisms for computable context rather than simple messages. All components—identity, membership, policies, key material, and governance—are expressed as signed, auditable Artipoints within Channels, ensuring that cryptographic primitives align naturally with collaborative patterns while maintaining the transparency and immutability essential to trustworthy agent-human interaction.

The result is a secure distribution architecture that preserves the semantic richness of structured articulation while providing the cryptographic assurances necessary for multi-party collaboration across organizational boundaries.