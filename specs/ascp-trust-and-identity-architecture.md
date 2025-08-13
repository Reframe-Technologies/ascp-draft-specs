## **ASCP Trust and Identity Architecture**

**A Log-Anchored approach to Distributed Trust**  
Version 0.1 - August 2025

Jeff Szczepanski, Founder and CEO Reframe Technologies

## **Introduction**

This document defines the cryptographic trust and identity architecture for the Agents Shared Cognition Protocol (ASCP). It establishes how participants—both human users and autonomous agents—prove their identity, bind cryptographic keys to their identities, and establish verifiable trust relationships within and across ASCP instances.

ASCP's trust model addresses several fundamental challenges in distributed systems:

- **Durable Identity**: How do we establish and maintain verifiable identities that persist across devices, sessions, and time?
- **Cryptographic Provenance**: How do we ensure every piece of content can be traced back to its authentic author?
- **Cross-Instance Trust**: How do independent ASCP instances establish trust without requiring a single global authority?
- **Recovery and Migration**: How do users securely recover their identity keys and migrate between devices?

The architecture described here provides the foundational layer for all ASCP operations. Every channel log, every Artipoint, and every participant interaction depends on the trust relationships established through these mechanisms. Without this identity layer, ASCP would be unable to provide its core guarantees around immutable provenance, authenticated authorship, and verifiable coordination.

## **Reading Map**

This document is structured in layers, from foundational concepts to implementation details:

**Foundation & Architecture** *(Essential for all readers)*

- **Glossary** - Key terminology and concepts
- **Log-Anchored Trust Architecture** - Core trust model and design principles
- **Security and Identity Artipoints** - Data structures for cryptographic identities
- **Cross-PKI Anchoring** - Integration with existing certificate infrastructure
- **Provenance of the ASCP RootCA** - Root trust establishment and validation

**Implementation Details** *(Focus for developers and implementers)*

- **Identity Bootstrap and Verification** - New participant onboarding process
- **Key Recovery Strategy** - Backup and migration mechanisms
- **Identity Token Requirements** - JWT validation specifications

**For Security Reviewers:** Pay special attention to the trust model, key escrow mechanisms, and PKI anchoring sections.

Each ASCP instance operates as a cryptographically independent trust domain while providing optional mechanisms to establish verifiable connections to external PKI systems and other ASCP instances.

## **Glossary**

**ASCP (Agents Shared Cognition Protocol)**: A distributed protocol enabling humans and autonomous agents to collaborate through cryptographically verified, immutable communication channels.

**Artipoint**: The fundamental data structure in ASCP - a signed, immutable entry in the distributed log containing content, metadata, and cryptographic provenance.

**Bootstrap Channel/Log**: The foundational channel containing the repository's root trust anchor and initial configuration. Always public and replicated to all participants.

**Certificate Artipoint**: An Artipoint that publishes a public key in JWK format, serving as the anchor for endorsements and recovery materials.

**Identity Artipoint**: An Artipoint that declares a participant (human, agent, or system) and links to their active signing key via `certificate::kid`.

**JWK/JWS/JWT**: JOSE (JSON Object Signing and Encryption) standards - JWK for key representation, JWS for signatures, JWT for tokens.

**Log-Anchored Trust**: ASCP's core trust model where cryptographic decisions are based on immutable, signed entries in distributed logs rather than real-time validation.

**PKI (Public Key Infrastructure)**: External certificate systems (like Let's Encrypt or enterprise CAs) that can optionally anchor ASCP trust to existing certificate hierarchies.

**Recovery Envelope**: Double-encrypted backup of a private key, protected by both a recovery key and password-derived encryption.

**RootCA**: The foundational certificate authority for an ASCP instance, established in the bootstrap log and serving as the root of trust for all participants.

**TSA (Time-Stamping Authority)**: RFC 3161-compliant service providing independent temporal attestation of when cryptographic materials were created.

## **Log-Anchored Trust Architecture**

ASCP employs a **Log-Anchored Trust** model where cryptographic trust decisions are based on immutable, signed entries in a distributed log rather than real-time certificate validation. This fundamental design choice ensures that trust relationships established at any point in time remain verifiable and auditable indefinitely, even as external PKI infrastructure changes or becomes unavailable.

### **The Bootstrap Foundation**

Each ASCP instance establishes trust through a **bootstrap channel log** that serves as the repository's immutable root of trust:

- **Genesis Artipoint**: The foundational first entry contains the repository's RootCA public key and is self-signed by the corresponding private key
- **Self-Contained Trust**: This establishes the baseline internal trust chain without requiring external dependencies
- **Immutable Provenance**: Because the bootstrap log cannot be altered without detection, the trust foundation remains auditable across the entire system lifecycle

### **Optional External Anchoring**

To enable **portable, verifiable trust** beyond the ASCP instance itself, the RootCA can be optionally anchored to external PKI infrastructure:

- **Detached Signatures**: Using a publicly verifiable certificate (such as one from Let's Encrypt or an enterprise CA) to produce a detached signature over the ASCP RootCA's public key
- **PKI Compatibility**: This approach respects PKI constraints by signing arbitrary data rather than issuing subordinate certificates
- **Chain of Accountability**: Any party trusting the anchoring PKI can independently verify the RootCA endorsement
- **Temporal Proof**: The signature is bundled with the complete certificate chain as it existed at signing time, preserving the exact trust state

### **Why Log-Anchored Trust Works**

- **Historical Verification**: Trust decisions can be validated against the state that existed when relationships were established
- **Autonomous Trust**: Each ASCP replica contains a complete copy of the RootCA and verification chain, enabling independent trust establishment without external dependencies
- **Resilient Operations**: Systems remain functional even if external PKI infrastructure becomes unavailable
- **Audit Transparency**: Every trust decision is recorded as a signed, immutable entry in the distributed log
- **Enterprise Integration**: Optional key escrow mechanisms support enterprise recovery scenarios while preserving the core trust model

This architecture provides the foundation for all subsequent identity and trust operations in ASCP, from participant key binding to cross-instance coordination. A full, step-by-step procedure for RootCA anchoring and long-term provenance capture is provided in the section **Provenance of the ASCP RootCA**, after we define the Artipoint structures required to represent this trust data.”

### **Self‑Signing as the Baseline**

- **Proof of possession**: Self‑signing the public key demonstrates that the registrant controls the matching private key.
- **Authenticated provenance**: Since only authenticated sessions can write to the log, each self‑signed key is linked to the session identity that submitted it.
- **Immutable record**: The articulation log preserves the public key, the self‑signature, and the author identity permanently.

### **When to Add RootCA Signing**

RootCA signing of public keys is optional for:

- **External trust**: Cross‑organization verification using the bootstrap log’s PKI anchor.
- **Policy enforcement**: Enterprises may require all keys be “blessed” by the rootCA.
- **Escrow linkage**: Tying vault entries directly to a PKI‑anchored identity.

### **Enterprise Escrow Option**

Some deployments require recoverability of private keys for compliance or legal hold.

- **Vaulting**: Private key is encrypted with an escrow key (e.g., protected by the rootCA private key or a dedicated escrow key pair).
- **Log storage**: Encrypted private key is stored in a secure vault entry in the articulation log.
- **Recovery**: Only parties holding the escrow key material can decrypt. All recovery operations are logged immutably.
- **Security model**: Even in escrow mode, private key is never stored in plaintext; vault entries are cryptographically bound to their corresponding public keys.
- **For personal devices**: Escrow can’t be activated without a vendor‑approved client change that would generally be visible or require user consent.
- **For enterprise‑managed devices**: Escrow activation is effectively a deployment policy decision, and transparency is up to enterprise policy and platform rules. ASCP should still log escrow events for auditability, but user‑level visibility may be moot in this trust model.

The detailed operational process for creating, storing, and recovering encrypted key vaults—both personal and enterprise—is covered in the section **ASCP Key Escrow and Recovery Strategy**.

### **Recommended Policy**

- **Always** require self‑signed public keys for proof of possession.
- **Optionally** require rootCA signing for interoperability or policy reasons.
- **Default** to local key generation for maximum privacy; offer escrow vaulting as an enterprise‑available feature.

By combining self‑signed keys, optional rootCA signing, and optional encrypted vaulting, ASCP supports both maximum privacy for individual users and controlled recoverability for enterprise environments — all anchored immutably in the articulation log. This unified approach ensures cryptographic integrity for every piece of authored content while maintaining immutable provenance for all keys, signatures, and trust anchors.

The result is a system that provides privacy‑first defaults with enterprise‑grade recovery options, enabling interoperability through PKI‑anchored rootCA signatures when needed. ALSP (ASCP Log Synchronization Protocol) session authentication controls who can participate using JOSE JWT-based tokens—typically via OpenID Connect. Key provisioning ensures every participant can sign content verifiably, while log‑anchored trust ties all keys and signatures back to the immutable bootstrap trust root.

This foundational trust model enables ASCP to provide durable, verifiable trust across both open, privacy‑respecting deployments and tightly‑controlled enterprise environments. The following sections detail the specific Artipoint types and mechanisms that implement this architecture.

## Security and Identity Artipoints

ASCP implements its trust architecture through specialized **Artipoint types** that represent cryptographic identities, keys, and trust relationships in the immutable log. These Artipoints handle identity declaration, public key distribution, root certificate authority establishment, and the attachment of trust evidence such as endorsements and recovery bundles.

This design leverages the log-anchored trust model described above, ensuring that evidence remains co-located with its subject and enabling flexible routing of sensitive data across different channels.

### Core design principles

1. **Everything is an Artipoint** → all actors, keys, and evidence are first-class, signed records.
2. **Bookmark pattern** → payloads are referenced; structure and provenance live in the log.
3. **Attributes over types** (when feasible) → fewer nodes, clearer ownership, easier rotation.
4. **Log-time trust** → verify with what existed *then*, not with mutable present state.

### Artipoint Types

These are minimal, composable types. Everything else (endorsements, recovery, current key selection) is attached via attributes and annotations, which are detailed in the following section.

**Identity Artipoint**

Declares a human, agent, or system identity. Also carries the **current active signing key** via certificate::kid.

```bnf
[identity-uuid, admin@org.com, 2025-08-08T10:00:00Z,
  ["identity", "Alice Example", "mailto:alice@example.com"] .
  ( type := "human",
    org := "Example Corp",
    certificate::kid := "ascp:cert:<cert-uuid>"
  )
];

```

- Rotation is a simple **annotation** that updates certificate::kid.
- No need to track previous keys—history is visible by scanning prior annotations.
- Mirrors Channel keyframe activation (keyframe::key\_id) for symmetry with Layer 1.

**Certificate Artipoint**

Publishes a JWK-encoded public key and metadata.

```bnf
[cert-uuid, alice@example.com, 2025-08-08T10:00:00Z,
  ["certificate", "Alice ES256 key", json:{ <public JWK> }] .
  ( fingerprint := "sha256:abcd...",
    issued := "2025-08-08T10:00:00Z" )
];

```

- The certificate Artipoint is the **anchor node** for endorsements and recovery.
- Keeping evidence on the cert avoids scattering provenance across many nodes.

**RootCA Artipoint**

Declares an ASCP instance’s Root CA. This Artipoint is very similar to any other "Certificate" Artipoint, but has a special name given it's unique purpose.

```bnf
[rootca-uuid, bootstrap@org.com, 2025-08-08T10:00:00Z,
  ["rootca", "Organization Name", json:{ <rootca public JWK> }] .
  ( endorsement + json:{ <endosement_info> },
    endorsement + json:{ <endorsement_info> },
    recovery_envelope + json:{ <recovery_info> }
];

```

- **Endorsements** tie the rootCA to externally verifiable references such as Enterprise or Public PKI certificates through detached signatures and captured certificate chains, establishing trust anchors outside the ASCP system.
- **TSA support** enables independent time attestation via RFC-3161 tokens, providing cryptographic proof of when the rootCA was established.
- **Recovery options** are available for rootCA private key material using the same recovery\_envelope mechanism as other certificates.
- While none of these external references are required, **at least some form of external PKI endorsement is RECOMMENDED** to bootstrap initial trust and enable verification by parties outside the ASCP instance.

## Certificate-Related Attributes

Attach these via **annotations** to the target certificate UUID. Values can be inline json:{...} or **UUID references** to other Artipoints that carry the JSON blob (reuse without duplication).

### Endorsement

Binds the certificate to a PKI anchor using a detached signature and contemporaneous chain capture.

**Inline JSON (minimum fields):**

- sig — detached signature over a declared subject (sig\_over)
- sig\_over — one of: "cert\_jwk" | "cert\_pem" | "fingerprint"
- cert\_chain\_pem — full chain at signing time (leaf→root)
- signer — endorser identity (e.g., domain, issuer DN)
- timestamp — ISO 8601 Z
- tsa (optional) — RFC 3161 token

**Inline example:**

```bnf
[endorse-uuid, ca-ops@example.com, 2025-08-08T10:05:00Z,
  cert-uuid .
  ( endorsement + json:{
      "sig": "<b64 detached signature>",
      "sig_over": "cert_jwk",
      "cert_chain_pem": "...",
      "signer": "example.com",
      "timestamp": "2025-08-08T10:05:00Z",
      "tsa": { ... }
    }
  )
];

```

**By reference:**

```bnf
cert-uuid .
( endorsement + endorse-artipoint-uuid );

```

**Client behavior (endorsement):**

1. Collect all endorsement values on the cert (dereference UUIDs if needed).
2. Validate the certificate chain **as recorded**.
3. Verify sig against the chosen sig\_over target (JWK/PEM/fingerprint).
4. If present, verify tsa and record asserted time.

### Recovery\_Envelope

Stores a double‑encrypted private key recovery bundle (outer: ECDH‑ES to recovery\_cert, inner: password‑derived AES).

**Inline JSON (minimum fields):**

- recovery\_cert — ascp:cert:\<uuid> for the recovery public key (Key B)
- encrypted\_key — JWE-of-JWE payload (outer ECDH‑ES, inner AES‑GCM)
- protection — \["recovery-key","password"] (or future schemes)
- rotation\_id — human-friendly version tag (e.g., "v1")
- created — ISO 8601 Z
- purpose (optional) — "device-migration" | "local-backup" | ...

**Inline example:**

```bnf
[recover-uuid, alice@example.com, 2025-08-08T10:06:00Z,
  cert-uuid .
  ( recovery_envelope + json:{
      "recovery_cert": "ascp:cert:<uuid-of-B>",
      "encrypted_key": { ...JWE-of-JWE... },
      "protection": ["recovery-key","password"],
      "rotation_id": "v1",
      "created": "2025-08-08T10:06:00Z",
      "purpose": "device-migration"
    }
  )
];

```

**By reference:**

```bnf
cert-uuid .
( recovery_envelope + recovery-artipoint-uuid );

```

**Authoring & routing patterns**

- Third‑party CA posts the endorsement as an annotation in *its* channel; value can be inline or a pointer to a CA‑hosted Artipoint.
- Recovery material for an org user can be published in a **more restricted channel** than the cert itself; the attribute still cleanly targets the cert UUID.

## Identity: Certificate Binding & Rotation

In ASCP, every user and agent that contributes immutable entries to the articulation log must have its own cryptographic key pair. The private key is used to sign all Artipoints authored by that identity. The public key is published in the log to establish verifiable provenance.

### **Default: Privacy‑First Local Key Generation**

- **Generation**: Key pairs are generated client‑side, within the end‑user’s or agent’s own secure environment.
- **Storage**: Private key remains under sole control of the owner:
  - Desktop/mobile → OS secure enclave or keychain.
  - Browser → WebCrypto API, secure context, IndexedDB.
  - Agent → TPM, HSM, secure file system.
- **Publication**: Public key is self‑signed by the private key holder and submitted during an authenticated ASCP session.
- **RootCA linkage**: ASCP may also sign the public key with the repository’s rootCA to bind it into the bootstrap‑anchored trust chain.

Use certificate::kid on the identity to set the current active signing key.

### **Set or Rotate the Active Key:**

```bnf
[rot-uuid, alice@example.com, 2025-08-12T09:00:00Z,
  identity-uuid .
  ( certificate::kid := "ascp:cert:<new-cert-uuid>" )
];

```

### **Why this is right**

- Simple, explicit, and mirrors channel keyframe activation.
- Old keys remain verifiable for historical signatures; no extra bookkeeping.

## **Identity Bootstrap and Verification Design**

This section outlines how new participants—human users or autonomous agents—establish verifiable identity using self-generated cryptographic keys and portable, third-party-issued JWT attestations. The mechanism supports decentralized operation while ensuring secure, auditable identity binding.

### Core Concepts

**Identity Representation**

- **Human Identities**: Represented as resource URIs (e.g., mailto:<jeff@example.com>)
- **Agent Identities**: Represented as persistent URNs (e.g., urn:agent:reframe:cortex-coordinator)

Each identity is cryptographically bound to a **self-generated identity key pair**:

- sk\_user: Private key (kept secret by identity owner)
- pk\_user: Public key, expressed as a JOSE JWK (shared for verification and encryption)
- Also referred to as a **JWK-based EC Identity Key**, using NIST EC P-256 or stronger JOSE-native keys.

### Bootstrapping Identity: How New Users Introduce Themselves

This process occurs **once**, as the user/client joins the ASCP system for the first time. All subsequent authentication is handled using cryptographic signatures associated with the self-generated identity key bound through this process.

**Step 1: Key Generation and Nonce Retrieval**

- The user/client generates their secure NIST P-256 EC identity key pair
- The client connects to a known ASCP peer or server.
- As part of the initial TLS handshake and ASCP "hello" exchange, the peer/server ASCP node issues its **challenge nonce**.

This nonce:

- MUST be high-entropy and unpredictable
- MUST be included in the signed key binding proof to ensure freshness and replay resistance

**Step 2: Third-Party Authentication with Forced Login**

The user initiates an authentication session with a trusted third-party identity provider (e.g., Google, Microsoft, Apple, or custom OIDC provider), using the nonce from the ASCP peer.

Requirements:

- Identity provider MUST support nonce parameter inclusion in the JWT
- Authentication request SHOULD include prompt=login to force real-time reauthentication
- Resulting ID token MUST include the unmodified nonce in a signed field

### Secure IdentityClaimBundle Format

The following steps are required to generate the necessary IdentityClaimBundle to link the client identity to the public key of the self-generated key pair:

**Step 1: Create the ASCP JSON Key Package**

```json
{
  "v": 1
  "n": "<ascp_protocol_nonce>",
  "k": { /* JWK-formatted public EC P-256 key */ },
  "ts": <timestamp>
}
```

This package includes:

- The version of the ASCP JSON Key Package. This must be 1 for now.
- The full ASCP protocol issued challenge nonce
- The public key of the self-generated NIST P-256 key pair encoded in JWK form for **ECDSA-P256** signing.
- The current timestamp in strict ISO 8601 with millisecond precision (Z suffix enforced)

**Step 2: Create the Signed Key Package (Compact JWS)**

- Sign the above JSON using sk\_user, producing a JWS **compact serialization**:

```
<protected-header>.<base64url(key-package)>.<signature>

```

- Extract only the signature portion (after the second dot).
- This becomes the nonce field for inclusion in the JWT authention request with the authorization server of the identity provider.

**Step 3: Authenticate with Identity Provider**

- Use the above extracted JWS signature as the JWT nonce parameter. If necessary the signature may be truncated to the maximum length support by the specific provider, however the truncated length SHOULD be no less than 88 characters when Base64URL encoded. This is just enough to hold a full ES256 signature, for example.
- Perform identity verification (with prompt=login) to enfore a direct realtime validation of identity credentials with the identity provider. This is typically an OpenID Connect based Authorization server, but the actual interaction system and protocols used is outside the scope of ASCP.
- Receive back a signed ID\_Token in JWT format from the provider ala an OpenID Connect type token or similar.

**Step 4: Construct the ID Token Package (Structured Envelope)**

```json
{
  "v": 1,
  "ts": <timestamp>,
  "signed_key_package": "<JWS(compact)>",
  "id_token": "<JWT from provider>"
}

```

This encapsulates:

- The full signed key package (includes nonce, JWK of public key and proves key binding via the JWS signature.
- The ID token from the identity provider (proves identity) and is crytographically linked to the public key and nonce via the includes of the JWS signature as the JWT nonce.
- A version and timestamp for replay protection

**Step 5: Sign the ID Token Package**

- Use sk\_user to sign the full ID Token Package, again as a JWS compact object
- This final outer JWS signed ID Token Package is called the **IdentityClaimBundle** and can be submitted to the ASCP peer or server as the binding identity response to the original ASCP **challenge nonce.**

### Validation Process by ASCP Server/Peer

Upon receiving the JWS signed IdentityClaimBundle, the ASCP verifier must:

1. Extract the JWK public key from inside the `signed_key_package` which is inside the IdentityClaimBundle making sure this JWK is properly formatted and encoded.
2. Use this JWK public key to verify the JWS signature on ASCP JSON Key Package itself. This confirms that public key and the private key go together.
3. Verify the outer JWS signature of the IdentityClaimBundle using the extracted JWK key as well, thereby confirming both signatures were created by the same private key.
4. Validate the key package contents:
   - ASCP protocol nonce matches the originally issued challenge nonce
   - Timestamps are within policy limits
5. Verify the original JWT (OIDC id\_token):
   - Signed by a trusted identity provider
   - Identity fields match expected structure and format
   - The JWT nonce matches the signed key package JWS compact serialized signature accounting for the fact that very long signatures may be truncated to as few as the first 88-characters, but ideally is a full match. Note that all provided characters of the JWT nonce MUST match. The truncation here affords the case where a specific identity provider implementation MAY truncate the nonce to some maximum length of 88-bytes or more. We expect implementation to supper at least 128 characters or perhaps 256 characters or more.

The specific cryptographic and claim requirements for access tokens used in ASCP sessions are defined in the **Appendix: Identity Token Requirements**.

### Why This Design Can Be Trusted

- **Nonce-bound**: Proves key binding occurred in real-time, in response to challenge
- **Self-sovereign**: No third-party holds user private keys
- **Cryptographically verifiable**: All layers are independently signed and crytographically connected.
- **Composably secure**: Outer envelope makes audit and validation straightforward
- **Aligns with standards**: Uses JWK, JWS, and JWT—all JOSE-native

### Logging Identity Verification in ASCP

Once verified, the ASCP peer/server writes an Identity Artipoint:

```
[identity-uuid, jeff@example.com, 2025-08-08T14:30:22Z,
  ["identity", "Jeff Szczepanski", "mailto:jeff@example.com"] .
  ( type := "human",
    certificate::kid := "ascp:cert:cert-uuid"
  )
];

[cert-uuid, jeff@example.com, 2025-08-08T14:30:22Z,
  ["certificate", "Jeff ES256 Identity Key", json:{ <JWK public key> }] .
  ( fingerprint := "sha256:abcd...",
    issued := "2025-08-08T14:30:22Z" )
];

[verification-uuid, alice@peer.ascp, 2025-08-08T14:30:25Z,
  cert-uuid .
  ( endorsement + json:{
      "validation_method": "oidc@https://accounts.google.com",
      "evidence": "<IdentityClaimBundle JWS>",
      "verified_by": "ascp://peer/alice",
      "timestamp": "2025-08-08T14:30:25Z"
    }
  )
];
```

This Artipoint acts as durable, auditable proof of the key–identity binding.

### Minimal Client Validation Rules

1. **Indexing**: Build per‑identity views by resolving certificate::kid and associating certs authored or endorsed for that identity.
2. **Signature verification**: For any Artipoint signed by an identity, resolve the kid → certificate → JWK. Verify against the historical cert that was active at the time of the message.
3. **Endorsements**: Validate detached signatures and chains from recorded evidence; keep a verdict per endorsement (valid/invalid/indeterminate).
4. **Recovery envelopes**: Do structural checks; decryption occurs only on explicit recovery.
5. **Timestamps**: Prefer tsa time if present; else use annotation timestamp.

### Benefits of This Architecture

- **Cryptographic binding**: Complete traceability linking verified identity to EC P-256 public key through immutable log entries
- **Replay protection**: Strong resistance via challenge nonces and timestamp signatures in nested JWS structure
- **Self-sovereign keys**: Users control their private keys without requiring third-party escrow or key management services
- **Universal provider support**: Works seamlessly with consumer providers (Google, Apple) and enterprise systems (Okta, Azure AD, Keycloak)
- **Enterprise flexibility**: Supports corporate authentication flows while maintaining consistent OIDC token interface

## **ASCP Key Escrow and Recovery Strategy**

ASCP provides a secure and auditable approach to private key recovery that builds on the log-anchored trust foundation. The strategy enables users to recover their **identity private key** without ever exposing it in plaintext to any third party, supporting device migration, key backup, and enterprise compliance scenarios.

### **Core Design Principles**

- **Zero-trust storage**: Recovery materials remain confidential even if the log is compromised
- **Self-contained recovery**: No external key management dependencies
- **Dual recovery modes**: User-controlled privacy or enterprise escrow options
- **User-controlled decryption**: Only authorized parties can decrypt recovery bundles

### **High-Level Flow**

**During Key Provisioning (e.g., account setup or key rotation)**

1. Generate identity keypair (**Key A**):
   - Used to sign Artipoints and authenticate user sessions
2. Generate or register a recovery keypair (**Key B**):
   - A separate EC keypair stored in a secure location (e.g., second device, hardware token)
3. Derive a password-based encryption key (**Key C**) using PBKDF2 or Argon2:
   - Based on a user-chosen passphrase
4. Wrap Key A in two layers of encryption (**double encryption**):
   - **First**, encrypt the identity private key with Key C (password-derived AES key), using JWE with alg: "dir" and enc: "A256GCM"
   - **Then**, encrypt the resulting ciphertext with Key B (the recovery public key), using JWE with alg: "ECDH-ES+A256KW" and enc: "A256GCM"
   - This layering ensures that **both the password and the recovery key are required** to recover the identity key
5. Create a user-key-envelope Artipoint:
   - Stores the doubly encrypted private key
   - Signed by the current identity (Key A) or another trusted issuer (e.g., org CA)
   - Written to the @Bootstrap channel for universal discoverability and replication

**On a New Device or During Recovery**

1. Sync the @Bootstrap channel (public and unencrypted)
2. Locate the user’s user-key-envelope
3. Use the recovery private key (**Key B**) to decrypt the outer layer
4. Prompt the user for their password to derive Key C
5. Use Key C to decrypt the inner layer, recovering the identity private key (**Key A**)
6. Import Key A into secure storage (e.g., Secure Enclave, TPM, or keychain)
7. Proceed to access the user’s private channel using the now-available Key A:
   - Decrypt key envelopes for Channel AES and CAK
   - Authenticate ALSP replication and Artipoint authorship

### **user-key-envelope Structure**

This Artipoint is published to the @Bootstrap channel as a signed, unencrypted payload.

```json
[   
    "uuid": "...",
    "source": "user@domain",
    "timestamp": "2025-08-06T12:00:00Z",
    "type": "user-key-envelope",
    "recovery_cert": "ascp:cert:<uuid-of-recovery-key>",
    "encrypted_key": { ...JWE-wrapped encrypted JWE... },
    "alg": "ECDH-ES+A256KW",
    "enc": "A256GCM",
    "protection": ["recovery-key", "password"],
    "recovery_purpose": "device-migration | local-backup",
    "rotation_id": "v1 | v2 | ...",
    "created": "2025-08-06T12:00:00Z"
]
```

The `encrypted_key` field contains a **JWE-wrapped JWE object**:

- **Outer JWE**: encrypted to the recovery public key (Key B)
- **Inner JWE**: encrypted using a password-derived AES key (Key C)
- **Payload**: a JWK-formatted EC private key representing the identity private key (Key A)

This double-encryption strategy ensures:

- Only a party with the recovery key **and** the password can recover the private key
- Even if one factor is compromised, the identity key remains secure
- All encryption uses authenticated encryption (AES-GCM) via standard JOSE mechanisms

### **Security Considerations**

- **Passive log exposure**: Since the envelope is doubly encrypted and stored in the clear, even full replication of the @Bootstrap channel does not compromise user secrets
- **Key B storage**: Recovery key material must be protected via a second device, hardware token, or exportable file/QR code with strong local security
- **Key C (password) complexity**: The passphrase should meet entropy guidelines and use a strong KDF like Argon2id
- **No org access**: The organization cannot decrypt or recover a user’s key unless explicitly granted access to Key B or Key C by the user

### **Future Enhancements**

- **Multi-party Shamir recovery**: Split Key B into multiple shares for multi-admin or user+device recovery models
- **QR-code recovery bootstrapping**: Encode the recovery JWE for scanning from air-gapped backups
- **Hardware key support**: Use FIDO2/WebAuthn for Key B storage and authentication during decryption
- **Rotation tracking**: Allow rotation of Key A while preserving old envelopes in the log for historical decryptability

### **Summary**

ASCP supports secure, privacy-preserving key recovery by leveraging layered encryption, log-anchored publication, and optional recovery keypairs. This ensures that users can regain access to their identity keys without trusting any intermediary, while maintaining the protocol’s cryptographic integrity and auditability.

## **Provenance of the ASCP RootCA**

This section defines the full provenance chain for the **ASCP Root Certificate Authority (rootCA)**, including:

- Internal self-signing and articulation
- Optional anchoring to a Public or Enterprise PKI certificate
- Optional independent attestation via a Time-Stamping Authority (TSA)

The goal is to ensure that **any verifier, at any time in the future**, can:

1. Establish the origin and legitimacy of the rootCA.
2. Confirm when it was created and anchored.
3. See its relationship to a trusted PKI at the time of anchoring.
4. Validate the integrity of this provenance against immutable and independent records.

### **RootCA Generation and Self-Signing**

1. **Generate rootCA Key Pair**
   - Algorithm: ECDSA P-384 (default) or equivalent.
   - Created in a secure environment by the ASCP repository operator.
   - Private key remains under sole control of the repository owner organization.
2. **Create rootCA Self-Signed Certificate**
   - Contains the public key.
   - Acts as the trust anchor for all ASCP participants and components in the repository.
3. **Embed in Bootstrap Artipoint**
   - The rootCA public key and self-signed certificate are embedded in the **first immutable Artipoint** of the repository’s @Bootstrap channel.
   - This Artipoint is cryptographically signed by the rootCA private key.

### **Optional Anchoring to a Public or Enterprise PKI**

To provide **portable, externally verifiable trust**, the rootCA can be endorsed by a PKI certificate from:

- **Public PKI**: e.g., Let’s Encrypt, DigiCert.
- **Enterprise PKI**: e.g., corporate CA.

**Process:**

1. Obtain a valid TLS/identity certificate from the chosen PKI.
2. Using the PKI certificate’s private key, create a **detached signature** over the rootCA public key or self-signed certificate.
3. Capture the **full validated certificate chain** for the PKI certificate at the time of signing.
4. Embed both the detached signature and certificate chain in the same bootstrap Artipoint alongside the rootCA.

**Result:**

- Any verifier can:
  1. Validate the PKI certificate chain against trusted PKI roots.
  2. Verify the detached signature over the rootCA key.
  3. Confirm that the rootCA was endorsed by the PKI identity at the recorded point in time.

### **Optional Time-Stamping Authority (TSA) Signature**

To provide **independent temporal attestation**, the bootstrap Artipoint can include an RFC 3161-compliant TSA token:

**Process:**

1. Create a hash of the rootCA provenance payload (including the PKI signature and certificate chain if present).
2. Submit this hash to a trusted Time-Stamping Authority.
3. Receive a signed **Time-Stamp Token (TST)** from the TSA, which:
   - Proves the payload existed at or before the TSA’s timestamp.
   - Is signed by the TSA’s own PKI-trusted certificate.
4. Embed the TST in the bootstrap Artipoint.

**Result:**

- Verifiers gain third-party assurance of when the provenance payload was created.
- Even if the ASCP repository’s own time records are disputed, the TSA attestation stands as independent proof.

### **Provenance Chain Validation Steps for a Verifier**

When a verifier examines a repository’s bootstrap channel, they:

1. **Extract the rootCA certificate** from the first Artipoint.
2. **Verify rootCA self-signature** (internal trust anchor).
3. If present, **validate PKI anchor**:
   - Check certificate chain validity (at time of signing).
   - Verify detached signature over rootCA key.
4. If present, **validate TSA attestation**:
   - Verify TSA token signature against TSA public key.
   - Confirm timestamp matches repository creation timeframe.
5. **Confirm immutability**:
   - Ensure bootstrap Artipoint is intact in all replicas.
   - Validate signatures back to rootCA.

### **Key Points**

- The provenance is **permanently preserved** in the immutable bootstrap channel.
- Even if the PKI certificate later expires or is revoked, verifiers can confirm it was valid when the repository was anchored.
- Re-anchoring is possible at any later date to provide renewed PKI endorsements without replacing the rootCA.
- Optional TSA signatures and/or publication in a transparency log provide **independent temporal proof** for maximum audit assurance.

## Overall Summary

ASCP's log-anchored trust architecture solves the fundamental challenges of distributed identity by anchoring all trust decisions in immutable log entries rather than real-time validation. This provides durable identity, complete cryptographic provenance, and autonomous operation without external dependencies.

The system enables both privacy-first and enterprise deployments through self-sovereign key generation, optional PKI anchoring, and secure recovery mechanisms. Most critically, it establishes verifiable trust relationships between humans and autonomous agents with identical cryptographic guarantees.

ASCP delivers a trust infrastructure that integrates with existing PKI systems today while providing the foundation for scalable human-AI coordination.

## **Appendix: Identity Token Requirements**

ASCP is agnostic to how clients obtain access tokens. Token acquisition—whether through OAuth 2.0 flows, enterprise SSO, or other authentication mechanisms—falls outside the ASCP protocol scope. Earlier sections detail where these tokens are presented and validated during identity bootstrap and session establishment.

This section defines the JWT validation requirements for authentication.

From ASCP’s perspective:

- **A token is a token.**
- The only requirement is that it meets the validation and policy rules established by the ASCP deployment.

### **Token Presentation**

- Clients MUST present a valid OAuth 2.0 access token when initiating an ASCP session.
- Preferred token format: **JWT** per RFC 7519.
- Token may be transmitted:
  - In the Authorization: Bearer header during the WebSocket upgrade request.
  - Or as the first ASCP protocol message after the connection is established.

### **Token Validation Requirements**

On receiving a token, the ASCP server MUST:

1. **Verify signature** using the issuer’s public keys (e.g., via JWKS endpoint).
2. **Verify claims**:
   - exp (expiration) is in the future.
   - nbf (not before) is in the past.
   - aud matches the ASCP service identifier.
   - iss is in the deployment’s list of trusted issuers.
3. **Enforce scope/role policy** as defined by the deployment.
4. Optionally verify **delegation claims** (e.g., act claim showing an agent acting on behalf of a user).

### **Deployment Policy**

- The ASCP protocol does not mandate a specific token acquisition flow.
- Each deployment determines:
  - Which issuers to trust.
  - Which OAuth flows are supported.
  - How client and agent tokens are obtained.
- This separation ensures ASCP remains compatible with both public/consumer logins and enterprise/private identity systems.

**In summary:**

ASCP does not dictate *how* tokens are obtained, only that they are valid, verifiable, and authorized for the intended session. This allows maximum flexibility in identity integration while keeping ASCP’s core protocol simple, consistent, and interoperable.