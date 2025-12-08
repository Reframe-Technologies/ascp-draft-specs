# **ASCP Trust and Identity Architecture**

**A Log-Anchored approach to Distributed Trust**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.43 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the ASCP specification suite and defines the **Trust and Identity Architecture** that establishes cryptographic provenance, durable identity, and verifiable trust relationships for all participants—both human users and autonomous agents. This foundational layer enables every ASCP operation, from channel log authentication to cross-instance trust establishment. It is published at this time to gather community feedback on the cryptographic architecture, channel management semantics, and interoperability of the secure distribution layer.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# **2. Abstract**

This document defines the Trust and Identity Architecture for the Agents Shared Cognition Protocol (ASCP), establishing how participants—human or agent—are identified, authenticated, and cryptographically bound to their contributions. This document normatively defines the identity, certificate, endorsement, and trust-anchor constructs used throughout ASCP. ASCP employs a log-anchored trust model in which identities, certificates, endorsements, and key-binding events are recorded as immutable, signed entries rather than evaluated through real-time certificate verification. The architecture specifies procedures for secure identity bootstrap using self-sovereign key generation, key rotation workflows that preserve identity continuity, optional mechanisms for anchoring ASCP trust into external PKI ecosystems, and recovery workflows that maintain user control of private keys. Together, these mechanisms provide durable provenance, verifiable authorship, and interoperable trust relationships that underpin all ASCP collaborative and cryptographic operations.

# **3. Introduction**

This document specifies the Trust and Identity Architecture for the Agents Shared Cognition Protocol (ASCP). It defines how participants establish durable cryptographic identities, authenticate authorship, and form verifiable trust relationships across ASCP Channels. All trust decisions in ASCP derive from immutable, signed log entries — a log-anchored model that ensures historical verifiability independent of external infrastructure.

## **3.1 Design Intent**

The architecture defined here provides:

- **Durable identity** — stable, self-sovereign identifiers tied to replaceable signing keys.
- **Cryptographic provenance** — authorship and key-binding events recorded immutably in ASCP logs.
- **Interoperable trust** — optional anchoring to external PKI ecosystems for cross-domain verification.
- **Extensible verification** — endorsements that support PKI, OIDC, DID, TSA, and future mechanisms.
- **User-controlled recovery** — workflows that preserve private key confidentiality while enabling device migration and rotation.

These mechanisms provide the foundational trust substrate on which all ASCP collaborative and cryptographic operations depend.

### **Note on Transparency Architectures**

Certificate Transparency (CT) and related Verifiable Data Structure (VDS) systems establish trust through public auditability and globally comparable append-only logs. ASCP Channels operate under a different trust model in which all participants are authenticated collaborators with verifiable authorship and scoped access. Confidentiality, selective visibility, key rotation, and forward secrecy are first-class requirements. As a result, CT/VDS-style transparency mechanisms are neither required nor appropriate inside ASCP Channels.

## **3.2 Scope of this Specification**

This document defines:

- The conceptual **log-anchored trust model** used across ASCP.
- The normative **Security Construct Artipoints** for identities, certificates, trust anchors, and endorsements.
- Mechanisms for **identity bootstrap**, **certificate validation**, **key rotation**, and **recovery**.
- Optional procedures for **PKI anchoring** and external attestation.
- The verification rules required for interoperable implementations.

All normative requirements in this document apply to ASCP-compliant implementations unless otherwise stated.

## **3.3 Out of Scope**

This document does **not** define:

- Governance semantics, authorship authorization, membership, or role evaluation  
  → See *ASCP Governance and Access Control*.
- Distribution-layer cryptographic behaviors (channel signing & encryption)  
  → See *ASCP Channels: Secure Distribution Layer*.
- Transport, replication, Lamport ordering, or peer authentication  
  → See *ASCP LogSync Protocol (ALSP)*.
- Application-, UI-, or workflow-level behaviors beyond cryptographic identity concerns.

## **3.4 Relationship to Other ASCP Specifications**

The Trust and Identity Architecture supplies the cryptographic foundation on which the other protocol layers depend:

- Layer-1 Channels rely on certificate purpose bindings to verify authorship and establish encrypted delivery.
- Layer-0 ALSP relies on identity-bound credentials for session authentication and log replication authorization.
- Governance relies on Identity and Certificate Artipoints for semantic participant resolution.

Together, these specifications form a coherent architecture for secure, composable, and durable shared cognition.

# **4. Terminology**

This section defines terms used throughout this specification.

All definitions in this section are **non-normative** unless explicitly marked otherwise.

### **ASCP (Agents Shared Cognition Protocol)**

A distributed protocol that enables humans and autonomous agents to collaborate using cryptographically verifiable, immutable logs and structured coordination constructs.

### **Artipoint**

A signed, immutable entry in an ASCP channel log representing a discrete, addressable statement of record. Artipoints are the foundational primitive for identity, trust, and coordination.

### **Security Construct Artipoint**

A class of Artipoints defined in this specification that represent cryptographic identities, certificates, trust anchors, endorsements, and related security artifacts.

### **Identity Artipoint**

A Security Construct Artipoint declaring a participant (human, agent, or system) and referencing the participant’s active signing certificate via certificate::kid.

### **Certificate Artipoint**

A Security Construct Artipoint that publishes a public key in JWK format and serves as the authoritative anchor for endorsements, purpose bindings, and recovery material.

### **RootCA Artipoint**

A designated Certificate Artipoint serving as the root trust anchor for an ASCP instance, established in the instance’s bootstrap channel.

### **Log-Anchored Trust**

The ASCP trust model in which all trust decisions derive from immutable, signed log entries—rather than real-time certificate validation—ensuring historical verifiability and decentralized operation.

### **Bootstrap Channel (or Bootstrap Log)**

A special ASCP channel containing the genesis Artipoint and the RootCA Artipoint, forming the immutable foundation of trust for the repository.

### **Endorsement**

An annotation on a Certificate or Identity Artipoint containing structured, verifiable evidence issued by an external authority (e.g., PKI CA, OIDC provider, DID controller, TSA) that strengthens provenance or identity binding.

### **Purpose Attribute**

A first-party declaration attached to a Certificate Artipoint that specifies the functional role of a key (e.g., purpose::assert, purpose::auth, purpose::keyAgreement) within ASCP’s cryptographic model.

### **Recovery Envelope**

A double-encrypted payload attached to a Certificate Artipoint containing encrypted private key material used for secure recovery or device migration.

### **JWK / JWS / JWT**

Standardized JOSE structures:

- **JWK**: JSON Web Key, for encoding public keys;
- **JWS**: JSON Web Signature, for signed payloads;
- **JWT**: JSON Web Token, for structured claims and identity assertions.

### **PKI (Public Key Infrastructure)**

An external certificate trust system (e.g., public or enterprise CA hierarchy) that MAY be used to anchor ASCP trust through detached signatures and chain validation.

### **TSA (Time-Stamping Authority)**

An RFC 3161-compliant service that issues Time-Stamp Tokens (TSTs) providing independent temporal proof of when a hash or certificate existed.

# **5. ASCP Trust Architecture Overview**

ASCP establishes trust through immutable, signed log entries rather than online certificate validation. Every cryptographic decision—identity binding, authorship verification, key rotation, endorsement evaluation—is grounded in the durable provenance of the channel logs. This section provides an architectural overview of this **log-anchored trust model**, the boundaries within which it operates, and its relationship to external trust ecosystems.

## **5.1 Log-Anchored Trust Model**

ASCP’s trust model is based on three core principles:

### **1. Immutable provenance**

All trust material—identities, certificates, endorsements, key rotations—are expressed as **signed Artipoints**. Verification is performed against the state of the log at the time an event occurred (“**log-time trust**”). This avoids reliance on mutable external infrastructure or online revocation systems.

### **2. Self-contained validation**

Each replica maintains all information required to validate any historical authorship or certificate binding, enabling **offline, autonomous verification**. External systems may strengthen trust but are never required for correctness.

### **3. Deterministic authorship verification**

Signed statements typically reference a key identifier via a `kid`, which resolves—through log state—to a Certificate Artipoint containing the corresponding public key. Because identities, certificates, and their bindings are fully captured in the log, verifiers can reproduce historical trust decisions with accuracy and auditability.

This model contrasts with transparency-oriented systems (e.g., Certificate Transparency) that assume untrusted operators or global adversarial visibility. ASCP Channels operate within authenticated collaboration boundaries, and therefore require **durable provenance**, not global consistency proofs.

## **5.2 Trust Domains and Boundaries**

ASCP separates trust into three layers, each with clear responsibilities:

### **Layer-3: Semantic Identity & Trust (this document)**

Defines the **Security Construct Artipoints**—Identity, Certificate, RootCA, endorsement, and recovery structures. This layer establishes:

- how participants are identified
- how keys are bound to identities
- how endorsements strengthen provenance
- how trust anchors are introduced and rotated

### **Layer-2: Artipoint Grammer**

The Layer-2 ASCP Grammar defines the encoding and decoding rules for the Collaborative Constructs such as the Security Construct Artipoint defined by this specification, but does **not** define transport, replication, or governance semantics of these Constructs, which all happens at Layer-3.

### **Layer-1: Channels (Secure Distribution Layer)**

Channels provide authenticated, append-only communication among participants. Their trust boundary:

- assumes all members are known and authenticated
- requires correct validation of signatures and key purposes
- enforces confidentiality and selective visibility

Channels rely on the trust material defined in Section 7 (Security Construct Artipoints) but do not interpret governance attributes or identity semantics.

### **Layer-0: ALSP (Log Synchronization Protocol)**

ALSP provides:

- peer authentication
- bidirectional replication
- convergence detection and divergence handling

ALSP does **not** evaluate identity provenance or certificate semantics. It only ensures authenticated sessions and consistent log replication.

This separation of concerns ensures trust remains **local, composable, and durable**, without requiring a global ledger or a single shared trust root.

## **5.3 Relationship to External PKI / DID Systems**

ASCP can operate entirely without external trust infrastructure. However, external ecosystems can **strengthen** trust through endorsements:

### **Public or Enterprise PKI**

PKI-issued detached signatures (e.g., using x5c chains) can endorse:

- the RootCA (for repository-level anchoring)
- individual certificates (for cross-organizational identity)

ASCP does **not** rely on PKI revocation or certificate validity at verification time; instead, PKI endorsements provide **point-in-time provenance**.

### **OpenID Connect (OIDC)**

OIDC identity providers can supply portable, third-party identity claims. These appear as endorsements binding a certificate to an external account identifier.

### **Decentralized Identifiers (DIDs)**

DID methods can issue JWS endorsements that bind a DID controller’s signature to a certificate fingerprint. ASCP does not embed DID documents; it treats them as external, mechanism-specific endorsement evidence.

### **Time-Stamping Authorities (TSAs)**

RFC 3161 TSTs add third-party **temporal attestation**, proving that a certificate or RootCA existed at or before a specific time.

ASCP does not merge or reconcile external trust graphs. Instead, all external references are captured as **immutable endorsements** attached to Certificate or Identity Artipoints, enabling verifiers to interpret trust relationships independently at any later time.

## **5.4 Merkle Trees as Layer-0 Optimization**

ASCP Channels do **not** rely on Merkle consistency proofs or transparency-style global auditability. However, ALSP implementations may use Merkle trees, hash summaries, or incremental hash chains to:

- accelerate replica convergence
- detect divergence among authenticated peers
- optimize state synchronization

These optimizations:

- occur strictly **below** the Channel trust boundary
- do **not** affect authorship semantics
- do **not** replace or modify the log-anchored trust model
- do **not** provide public transparency

Their purpose is purely performance and replication efficiency. All security-relevant trust decisions remain grounded in the signed Artipoints themselves.

## **5.5 Transparency Guarantees and Replay Resistance**

Although ASCP does not implement global transparency logs, its architecture provides several important security guarantees:

### **1. Verifiable authorship history**

Because every Artipoint is individually signed and immutably recorded, verifiers can reconstruct:

- which identity authored each statement
- which certificate was active at that time
- the provenance of endorsements and trust anchors

### **2. Replay resistance**

Replay attacks are mitigated by:

- explicit key purposes that constrain key use
- immutable authorship timestamps
- nonce-bound identity bindng flows (Section 9)
- historical validation using log-time trust

Events cannot be reinterpreted under a different trust state.

### **3. Consistent trust evaluation**

Independent replicas evaluating the same log will reach the same trust conclusions, because all evidence is:

- included in the log
- verifiable offline
- bound to fixed certificate fingerprints
- interpreted against historical state, not mutable present policy

### **4. Optional external proofs without dependency**

PKI and TSA endorsements provide strong third-party evidence, but ASCP’s correctness does **not** depend on their continued availability. Once recorded, they become part of the immutable provenance chain.

# **6. ASCP Identity Model Overview**

ASCP represents identities, certificates, and trust anchors as immutable, composable Artipoints recorded in append-only channel logs. This section provides a conceptual overview of how identities operate within the ASCP coordination fabric, how keys and certificates relate to those identities, and why the protocol uses log-anchored Artipoints as the foundation for durable provenance.

## **6.1 Identity as an Addressing Construct**

In ASCP, an *Identity* represents a participant—human, agent, or system component—that can author Artipoints and participate in collaborative structures. Identities function as **addressing constructs**:

- They provide a stable, long-lived identifier for the participant.
- They bind the participant to cryptographic certificates used for authorship, authentication, and channel key agreement.
- They serve as the semantic target for governance rules and role assignment (defined in the Governance specification).

An Identity Artipoint is not a credential and does not itself contain a public key. Instead, it is able to *reference* a currently active certificate via a certificate::kid attribute. This indirection enables long-term identity continuity even as keys rotate over time.

Identities are appended to the log like any other Artipoint, ensuring they are:

- immutable
- auditable
- verifiable against historical trust state
- discoverable across replicas through normal log replication

## **6.2 Separation of Identity, Keys, Certificates, Trust Anchors**

ASCP explicitly separates these components, each serving a distinct role in the trust architecture:

### **Identity**

A durable, addressing-level concept. It names the participant and links to their active certificate. It carries **no cryptographic material**.

### **Certificate**

A Security Construct Artipoint containing a public key (JWK) and attributes declaring the key’s purposes and associated recovery information. Certificates carry the **operational cryptography** used for authorship, authentication, and key agreement.

### **Trust Anchors (RootCA)**

A distinguished Certificate Artipoint introduced at bootstrapping time as the root of all trust. It anchors:

- **Repository-level trust** — All certificate chains and trust decisions within the instance trace back to the RootCA as the ultimate trust source.
- **Onboarding verification** — New participant certificates are validated as legitimate by verifying they are signed by or chain back to the RootCA.
- **Cross-instance trust** — When ASCP instances establish trust relationships, the RootCA can be endorsed by external systems (enterprise PKI, public CAs) to bridge independent trust domains.

### **Endorsements / Attestations**

Optional evidence—PKI chains, OIDC claims, DID signatures, TSA tokens—that strengthen the provenance of certificates or identities. They enhance trust but do not alter identity semantics or governance.

**Separation is intentional.**

By keeping these layers distinct, ASCP supports:

- key rotation without identity disruption
- parallel use of multiple certificates per identity
- independent evaluation of endorsements
- verifiable historical trust decisions based on log state

## **6.3 Why Artipoint-Based Identities**

Representing identities, certificates, endorsements, and trust anchors as Artipoints offers several architectural benefits:

### **1. Immutability by construction**

Each identity, certificate, or endorsement is an append-only record. Nothing is overwritten; new facts extend old ones. This produces a complete, auditable provenance chain.

### **2. Local, replayable trust**

All information needed for verification—keys, endorsements, rotation history—is captured in the log. A replica can validate any historical statement without external calls or mutable third-party services.

### **3. First-class composability**

Identities participate in the same Graph of Meaning as work artifacts (Spaces, Streams, Groups), allowing governance, distribution, and identity systems to interoperate without special-case mechanisms.

### **4. Decoupled lifecycle events**

Because identity and key material are distinct records:

- A participant can rotate keys simply by publishing a new certificate and updating certificate::kid.
- Endorsements can be added or replaced without modifying identity data.
- Recovery material can be attached or revoked cleanly.

### **5. Symmetry across human and agent identities**

Both humans and autonomous agents appear the same to the trust layer: durable identifiers with cryptographic authoring keys and optional external endorsements.

This enables multi-agent and human–AI collaboration without requiring separate identity systems.

## **6.4 Certificate Lifecycle (High-Level Model)**

ASCP defines a simple, robust key and certificate lifecycle grounded in the log-anchored trust model.

### **Step 1 — Key generation**

A participant generates a private/public keypair locally or in secure hardware.

The public key becomes the payload of a Certificate Artipoint.

### **Step 2 — Certificate publication**

The participant publishes the Certificate Artipoint, optionally with:

- purpose declarations (purpose::assert, purpose::auth, purpose::keyAgreement),
- recovery envelope material,
- external endorsements.

This certificate immediately becomes verifiable by all replicas.

### **Step 3 — Identity binding**

The participant (or an onboarding authority) updates the Identity Artipoint via `certificate::kid` to reference the active certificate. Log provenance ensures the historical binding is verifiable indefinitely.

### **Step 4 — Rotation**

When a key must be replaced:

1. A new certificate is published.
2. The identity’s `certificate::kid` is updated to reference the new certificate.
3. Historical statements remain valid because verification uses log-time trust.

### **Step 5 — Recovery (optional)**

If recovery envelopes were attached, a participant can recover their key using:

- the recovery certificate, and
- a password-derived encryption key,

without ever exposing the private key to an ASCP server.

### **Step 6 — External anchoring (optional)**

Participants or organizations may add PKI, OIDC, DID, or TSA endorsements at any time. These do not replace certificates—rather, they provide additional evidence that strengthens trust.

# **7. Security Construct Artipoints**

## **7.0 Overview**

Security Construct Artipoints define the immutable, cryptographically verifiable records that represent identities, certificates, trust anchors, and channel-level security states in ASCP. These constructs appear at **Layer-2** of the ASCP stack, where they provide the typed and structured artifacts that upper layers (Channels, Governance, ALSP) rely upon for authorship validation, certificate resolution, key agreement, and trust anchoring.

Security Construct Artipoints share the following normative properties:

1. **They MUST be immutable Artipoints** conforming to the ASCP grammar.
2. **They MUST contain sufficient attributes** for a verifier to reconstruct authorship and trust state at any historical time (“log-time trust”).
3. **They MUST be self-contained**, meaning that all cryptographic material required for validation is either embedded in the Artipoint or discoverable via referenced UUIDs within the same log.
4. Security Construct Artipoints **MUST NOT** contain governance-related information such as roles, permissions, or authorization policies in any payload or other field. The sole exception is Governance Attributes explicitly defined in the **ASCP Governance & Access Control specification**.

The remainder of this section defines each Security Construct type and the normative requirements for its structure and validation.

## **7.1 Identity Artipoint (Normative)**

An **Identity Artipoint** declares the existence of a participant—human, agent, or system component—and binds that participant to their currently active signing certificates.

The UUID of an Identity Artipoint MUST be used to identify a participant in a stable, durable way when referencing the Identity from other Artipoints or Attributes.

Identity Artipoints MUST NOT contain embedded cryptographic key material. Key material is represented exclusively by Certificate Artipoints as described in Section 7.2. Identity Artipoints MAY be created without an initial certificate binding. Certificates are bound or updated using the process outlined in Section 6.4.

When an Identity Artipoint is added to the log for the first time, it MUST NOT be self-authored or self-signed. Instead, it MUST be authored and signed by either:

- The private key associated with the RootCA using the UUID of the RootCA to populate the author field of the Artipoint, or
- An identity and key associated with another Identity Artipoint that MUST be traceable back to the RootCA private key through a valid provenance chain.

After an Identity Artipoint is bound to a valid Certificate Artipoint for signing, future annotations onto the Identity MAY BE self-authored and self-signed with the Identity Key associated with the Identity.

### **7.1.1 Canonical Form**

An Identity Artipoint MUST be articulated with the identity type in the primary payload:

```c
[uuid, author, timestamp,
  ["identity", <label>, <optional-uri>
  ]
]
```

### **7.1.2 Field Requirements (Normative)**

An Identity Artipoint **MUST** conform to the Artipoint Grammar specification. The following requirements apply specifically to Identity instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be "identity".
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be empty.
  - The label **SHOULD** identify a display name or description of the entity associated with the identity.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MAY** be a quoted string or typed block with a uri: prefix
  - The payload SHOULD contain an email address in mailto:user\@domain" form or a URI pointing to the agent process the identity assoicates with.

### **7.1.2 Required Attributes for Bound Identities**

- `certificate::kid` - When certificate binding(s) exist, the `certificate::kid` attribute of the Identity Artipoints MUST reference the currently active key material (Certificate Artipoints) thereby bound to the identity. The value of this attribute MUST be key identifiers in the form `ascp:cert:<uuid>`**** where `<uuid>` is the UUID of the Certificate Artipoint representing the active certificates. The referenced certificate MUST itself be valid at the time of the Identity Artipoint’s articulation.

### **7.1.3 Optional Attributes**

- `type` - string - with possible values of "human" | "agent" | "system"
- `org` - string -Organizational Description

Such attributes are **informational only** and MUST NOT be interpreted as governance permissions or access privileges.

## **7.2 Certificate Artipoint (Normative)**

A **Certificate Artipoint** publishes a public key and defines the cryptographic characteristics, purposes, and trust evidence associated with that key. Certificates are the canonical objects used for verifying signatures, authentication operations, and key agreement in ASCP.

### **7.2.1 Canonical Form**

A Certificate Artipoint MUST use the certificate primary payload:

```c
[uuid, author, timestamp,
  ["certificate", <label>, json:{ <JWK public key> }
  ]
]
```

### **7.2.2 Field Requirements (Normative)**

A Certificate Artipoint **MUST** conform to the Artipoint Grammar specification. The following requirements apply specifically to Certificate instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be "certificate".
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be empty.
  - The label **SHOULD** contain a friendly description of the key associated with the certificate like how generated or what it is for.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MUST** be a JSON payload containing an embedded JWK for the key to associated with the certificate.
  - The JWK MUST conform to RFC 7517 and include all required parameters for the key type
  - The key in the JWK **MUST** match up with the private key used to sign any Artipoint referencing this certificate

### **7.2.4 JOSE kid Usage**

A verifier MUST treat the kid value in JOSE headers as an **opaque reference** that resolves to the Certificate Artipoint UUID.

Implementations MUST NOT attempt semantic interpretation of the string beyond resolving it to the Artipoint.

### **7.2.4 Representing JWK / X.509 Certificates**

A Certificate Artipoint MUST contain a JWK.

It MAY additionally include:

- x5c certificate chains
- PKI metadata
- Alternative encodings of the same key

These additional encodings MUST NOT contradict the JWK fingerprint (RFC 7638).

### **7.2.5 Endorsements (PKI, OIDC, DID, TSA)**

Certificates MAY have any number of endorsement attributes attached via annotation.

Each endorsement MUST:

- Bind to the certificate’s RFC-7638 fingerprint
- Contain an issuer mechanism and evidence consistent with Section 8
- Include timestamps enabling temporal validation

Endorsements strengthen provenance but MUST NOT alter core certificate semantics.

### **7.2.6 Certificate Purpose Semantics**

A Certificate Artipoint MAY declare one or more purpose attributes:

- `purpose::assert` — allowed for authorship signing
- `purpose::auth` — allowed for authentication handshakes
- `purpose::keyAgreement` — allowed for cryptographic key agreement

A Certificate Artipoint MUST declare at least:

- `purpose::assert` to be valid for Artipoint authorship
- `purpose::keyAgreement` to be valid for channel membership encryption or for the `recovery_cert` referenced from within a `recovery_envelope` attribute value.

A certificate MUST NOT be used for a purpose not explicitly declared.

### **7.2.7 Recovery Envelope**

A Certificate Artipoint MAY include one or more recovery\_envelope attributes.

Each recovery envelope MUST:

- Contain a double-encrypted private key bundle
- Conform to the format in Section 8
- Not leak private key material without both required decryption factors

Presence of a recovery envelope does not confer authority; it only enables recovery.

## **7.3 Organizational RootCA Construct (Normative)**

A **RootCA Artipoint** is a Certificate Artipoint with special status: it acts as the instance-wide trust anchor for onboarding, certificate acceptance, and cross-instance trust.

### **7.3.1 RootCA Canonical Form**

A RootCA Artipoint MUST use:

```c
[uuid, author, timestamp,
  ["rootca", <org-label>, json:{ <root JWK> }
  ]
]
```

The root certificate MUST be self-signed and MUST appear in the **Bootstrap Channel** per the requirements of **ASCP Bootstrap Process and Channel Discovery** specification

### **7.3.2 Field Requirements (Normative)**

A RootCA Artipoint **MUST** conform to the Artipoint Grammar specification. The following requirements apply specifically to BootCA instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be "rootca".
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MUST** be a string of "Root Certificate".
- **payload**
  - A payload field **MUST** be present.
  - The payload **MUST** be a JSON payload containing an embedded JWK of the Root Certificate public key.
  - The JWK MUST conform to RFC 7517 and include all required parameters for the key type
  - The key in the JWK **MUST** match up with the private key used to sign any Artipoint referencing this certificate.

### **7.3.3 Required Attributes**

- `purpose::assert` - Must contain the JWK fingerprint of the RootCA key.

### **7.3.3 Provenance Requirements**

A RootCA Artipoint SHOULD include one or more of the following:

- PKI endorsements (detached signatures + certificate chains)
- TSA attestation tokens
- Optional DID or OIDC evidence

These strengthen verifiability but are not required for correctness.

### **7.3.3 Cross-Signing and Out-of-Band Trust Anchors**

A RootCA MAY be:

- Cross-signed by another ASCP instance
- Endorsed by a PKI or DID issuer
- Anchored in external transparency mechanisms

These endorsements MUST be recorded as attributes on the RootCA Artipoint.

### **7.3.4 Rotation and Revocation**

If a RootCA must be rotated:

1. A new RootCA MUST be introduced in a future bootstrap epoch.
2. The new RootCA MUST endorse the old RootCA for continuity, or vice-versa.
3. Verifiers MUST treat the old RootCA as authoritative for all historical log entries prior to the rotation point.

ASCP does not support destructive revocation of RootCAs; supersession MUST be explicit and additive.

## **7.4 Keyframe / Security State Constructs (Normative)**

Keyframes are not identity constructs, but they are **security constructs** because they bind channel-level cryptographic material.

### **7.4.1 Keyframe Role in Trust**

A Keyframe Artipoint:

- Defines the active cryptographic state for a Channel
- Contains encrypted channel keys wrapped for each participant
- Is referenced by Channels using keyframe::kid

### **7.4.2 certificate::kid vs keyframe::kid**

- **Identity Artipoints** use certificate::kid → points to a Certificate Artipoint (public key).
- **Channel Artipoints** use keyframe::kid → points to a Keyframe Artipoint (private channel key bundles).

The distinction is normative and MUST be preserved.

### **7.4.3 Certificate Usage for User-Key-Envelopes**

User-key-envelopes in Channels MUST use a certificate that declares purpose::keyAgreement.

Identity-bound authorship certificates (purpose::assert) MUST NOT be used for key agreement.

### **7.4.4 Validation Rules**

A verifier MUST:

1. Resolve keyframe::kid to the referenced Keyframe Artipoint.
2. Confirm that all encrypted envelopes in the Keyframe correspond to recipients with valid purpose::keyAgreement certificates.
3. Reject any keyframe referencing certificates not suitable for key agreement.

# **8. Certificate-Related Attributes**

Certificate-related attributes extend the semantics of a Certificate Artipoint by attaching external evidence, first-party key-usage declarations, or encrypted recovery material. All attributes defined in this section MUST be expressed as Artipoint annotations referencing the UUID of the Certificate Artipoint to which they apply. Attributes MUST NOT mutate the underlying Certificate Artipoint.

This section defines three normative attribute families:

1. **endorsement::**\* — external attestations that strengthen provenance
2. **purpose::**\* — first-party declarations of intended key usage
3. **recovery\_envelope** — encrypted key-recovery material

All attributes defined here are **Layer-3 semantic constructs**, interpreted only by Trust and Identity implementations and Channel clients. Channels MUST NOT infer governance semantics from these attributes.

## **8.1 Endorsement Attribute Family**

### **8.1.1 Overview**

An **endorsement attribute** binds external, independently verifiable **attestations** to a Certificate or Identity Artipoint. Endorsements strengthen provenance by recording evidence from external systems such as PKI, DID, OIDC, or TSA services.

Endorsements are *additive*: they cannot revoke or modify certificate semantics, nor can they confer governance permissions.

Endorsements MUST be attached as:

```c
( endorsement::<mechanism> + json:{ ...evidence... } )
```

Where \<mechanism> identifies the external verification method.

### **8.1.2 Attestation vs. Endorsement (Normative Distinction)**

ASCP differentiates between:

- **Attestation** — the factual claim (e.g., key existed at time T; key maps to id X)
- **Endorsement** — the issuer-signed binding of that attestation

An endorsement, therefore, MUST contain:

- A **fingerprint** binding the attestation to the certificate’s RFC-7638 hash
- An **issuer.mechanism** describing how to verify the attestation
- **Evidence** appropriate to the mechanism
- An **issued\_at** timestamp

This distinction prevents semantic overload common in DID Documents and PKI and ensures that provenance remains verifiable and composable.

### **8.1.3 Endorsement Schema**

All endorsements MUST follow this schema:

```json
{
  "schema": "ascp.endorsement.v1",
  "subject": "cert" | "identity",
  "attestation": "<attestation-type>",
  "fingerprint": "sha256:<hex>",
  "issuer": {
    "mechanism": "<method>",
    "hint": "<display-string>"
  },
  "evidence": { ...mechanism-specific fields... },
  "issued_at": "<ISO-8601 UTC timestamp>",
  "tsa": { 
    "tst": "<base64-der-TST>", 
    "tsa_name": "<string>" 
  }
}
```

#### **Required fields**

- schema
- attestation
- fingerprint
- issuer.mechanism
- issued\_at
- evidence

#### **Optional fields**

- subject
- issuer.hint
- tsa block

#### **Fingerprint Requirement (Normative)**

The fingerprint MUST equal the RFC-7638 JWK thumbprint of the Certificate Artipoint being endorsed.

### **8.1.4 Attestation Types (Normative)**

| **Type**      | **Meaning**                                           | **Mechanisms**            |
| ------------- | ----------------------------------------------------- | ------------------------- |
| fingerprint   | Endorser vouches for correctness of key fingerprint   | jws-x5c, jws-kid, did-jws |
| issuance-time | Endorser vouches key existed at or before a timestamp | tsa-rfc3161               |
| id-binding    | Endorser binds key to an account or identifier        | oidc-icb, did-jws         |

### **8.1.5 Supported Endorsement Mechanisms**

#### **a. jws-x5c (PKI Certificate Endorsement)**

MUST contain a JWS signed with a PKI-issued key plus an x5c chain.

#### **b. jws-kid (ASCP-internal Endorser)**

Endorser is another ASCP identity; the JWS MUST reference its certificate via JOSE kid.

#### **c. oidc-icb (OIDC IdentityClaimBundle)**

Evidence MUST contain a structured OIDC claim binding the certificate’s fingerprint to an account identifier.

#### **d. did-jws (Decentralized Identifier Endorsement)**

Evidence MUST contain a DID reference and a JWS signed using a DID verification method.

#### **e. tsa-rfc3161 (Time-Stamping Authority)**

Evidence MUST contain a DER-encoded Time-Stamp Token verifying issuance-time.

### **8.1.6 Endorsement Validation Rules (Normative)**

A verifier MUST:

1. Verify that fingerprint == certificate RFC-7638 thumbprint
2. Validate mechanism-specific evidence:
   - **jws-x5c**: Verify JWS → Validate x5c chain
   - **jws-kid**: Resolve endorser certificate → Verify JWS
   - **oidc-icb**: Validate JWT signature → Verify nonce binding
   - **did-jws**: Resolve DID document → Verify JWS
   - **tsa-rfc3161**: Validate TSA signature → Confirm timestamp
3. Apply log-time trust: evaluate endorsement using the trust state at the time of issuance
4. Record per-endorsement validation result: valid / invalid / indeterminate

Endorsements MUST NOT alter the meaning of Identity or Certificate Artipoints.

## **8.2 Purpose Attribute Family**

### **8.2.1 Overview**

Purpose attributes declare **first-party key intent**. They describe what a key MAY be used for within ASCP and MUST be attached to Certificate Artipoints only.

Each purpose attribute is expressed as:

```asciidoc
( purpose::<type> + "jwk#<thumbprint>" )
```

Where \<thumbprint> MUST be the RFC-7638 JWK thumbprint of the same certificate.

**Purpose attributes are required for authorization of key use.**

Verifiers MUST confirm that a cryptographic operation uses a certificate whose thumbprint appears under the correct purpose attribute.

### **8.2.2 Defined Purpose Types**

| **Purpose**           | **Meaning**                                 | **Required For**                       |
| --------------------- | ------------------------------------------- | -------------------------------------- |
| purpose::assert       | Artipoint authorship, endorsements          | All signed Artipoints                  |
| purpose::auth         | Session authentication, proof-of-possession | ALSP handshake, channel authentication |
| purpose::keyAgreement | ECDH or similar key agreement               | Keyframe and Recovery Envelopes.       |

#### **Normative Requirements**

- Certificates MUST declare purpose::assert to be valid for Artipoint authorship.
- Certificates used for channel key agreement MUST declare purpose::keyAgreement.
- A key MUST NOT be used for a purpose not declared on its certificate.

### **8.2.3 Verification Rules**

When validating any cryptographic operation, verifiers MUST:

1. Resolve JOSE kid → Certificate Artipoint
2. Compute certificate’s RFC-7638 thumbprint
3. Confirm thumbprint appears in the appropriate purpose attribute
4. Reject operations where purpose binding is missing

Channels MUST rely exclusively on purpose attributes; they MUST NOT interpret identity metadata or governance attributes.

### **8.2.4 Design Considerations**

- Purpose attributes are **first-party declarations**, not claims of external trust.
- Purposes rotate naturally when certificates rotate.
- Purposes MUST NOT be Boolean flags; they MUST reference JWK thumbprints.
- Using separate keys for assertion and key agreement is RECOMMENDED.

## **8.3 recovery\_envelope Attribute**

### **8.3.1 Overview**

A recovery\_envelope provides a double-encrypted backup of the certificate’s private key, enabling secure recovery without exposing secret material to any ASCP server or peer.

A recovery\_envelope MUST:

- Be attached as an annotation on the Certificate Artipoint
- Contain a JWE-encrypted JWE (dual-layer)
- Bind to a valid recovery certificate (recovery\_cert)
- Require both a password and a recovery key to decrypt

### **8.3.2 Schema**

A JSON structure as follows:

```json
{
  "type": "recovery-envelope",
  "version": "1.0",
  "recovery_cert": "ascp:cert:<uuid>",
  "user_key_jwe": "<user-key-envelope>",
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "protection": ["recovery-key", "password"],
  "recovery_purpose": "device-migration | local-backup",
  "rotation_id": "v1",
  "created": "<ISO-8601 UTC>"
}
```

- `type` — Identifies this as a recovery envelope
- `version` — Schema version for forward compatibility
- `recovery_cert` — Reference to the recovery public key (Key B) used for outer encryption. See section 11.
- `user_key_jwe` — The value string holds the  `user-key-envelope` per section 11 which is the double-encrypted identity private key written in JWE compact serialization form.
- `alg` — Encryption algorithm used for the outer JWE layer
- `enc` — Content encryption algorithm for authenticated encryption
- `protection` — Array indicating the protection factors required for decryption
- `rotation_id` — Human-readable identifier for tracking key rotations
- `recovery_purpose` (optional) — Intended use case for this recovery envelope
- `created` — Timestamp when the recovery envelope was generated

See Section 11 for all details around contructing the recovery envelope and in particular the contruction and decoding process of the `user-key-envelope`populating the `user_key_jwe` JSON field.

## **8.4 Example Attribute Annotations (Informative)**

### **8.4.1 PKI Endorsement of Certificate Fingerprint**

```
[ endorse-uuid, author-uuid, 2025-08-13T14:00:00Z,
    cert-uuid .
    ( endorsement::jws-x5c + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"fingerprint",
        "fingerprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"jws-x5c","hint":"ca.example.org"},
        "evidence":{
            "jws":"<compact JWS>",
            "chain":["<leaf>","<int>","<root>"] },
        "issued_at":"2025-08-13T14:01:22Z" }
    )
];
```

### **8.4.2 OIDC Identity Binding**

```
[ verify-uuid, author-uuid, 2025-08-08T14:30:25Z,
    cert-uuid .
    ( endorsement::oidc-icb + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"id-binding",
        "fingerprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"oidc-icb","hint":"accounts.google.com"},
        "evidence":{ "icb":"<IdentityClaimBundle JWS>" },
        "issued_at":"2025-08-08T14:30:25Z" }
    )
];
```

### **8.4.3 DID-based Endorsement**

```
[ did-endorse-uuid, author-uuid, 2025-08-20T10:00:00Z,
    cert-uuid .
    ( endorsement::did-jws + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"fingerprint",
        "fingerprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"did-jws","hint":"did:key:z6Mk..."},
        "evidence":{"did":"did:key:z6Mk...","jws":"<compact JWS>"},
        "issued_at":"2025-08-20T10:00:00Z" }
    )
];
```

### **8.4.4 TSA Time Attestation**

```
[ tsa-uuid, author-uuid, 2025-08-13T14:02:00Z,
    cert-uuid .
    ( endorsement::tsa-rfc3161 + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"issuance-time",
        "fingerprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"tsa-rfc3161","hint":"tsa.example.org"},
        "evidence":{"tst":"<base64-der-TST>"},
        "issued_at":"2025-08-13T14:02:00Z" }
    )
];
```



# **9. Identity Claim Bundles (ICB)** 

This section defines how ASCP participants—human users or autonomous agents—establish verifiable identity by combining (1) self-generated signing keys and (2) authentication by an external Identity Provider (IdP). The resulting **Identity Claim Bundle (ICB)** is a compact, portable JWS structure that provides durable, cryptographically self-contained evidence binding an ASCP public key to an externally authenticated identity.

The design is intentionally patterned after the architectural principles used by **WebAuthn attestation**: a *proof-of-possession* step tied to a caller-supplied nonce establishes the authenticity of the key material, while a subsequent external attestation binds this key to a real-world identity. ASCP generalizes this pattern into a reusable, self-contained endorsement object suitable for persistent log-anchored verification.

The ICB is the normative evidence used to construct **endorsement::oidc-icb** attributes on Certificate Artipoints (Section 8). Although OIDC will be the most common mechanism, the ICB design is IdP-agnostic and supports any JWT-based identity assertion that satisfies the normative requirements below.

## **9.1 Design Assumptions (Informative)**

The ASCP Identity Bootstrap mechanism is designed to satisfy the following goals:

- **Self-Sovereign Key Control** — Participants generate and control their own identity signing keys. No IdP or ASCP service sees private key material.
- **WebAuthn-Style Fresh PoP** — The client must provide *immediate* proof of possession of its ASCP keypair before invoking the IdP.
- **Nonce-Anchored Bindings** — A single, explicit **ICB Nonce** binds all cryptographic layers—the Signed Key Package, the IdP-issued JWT, and the outer ICB JWS—into a coherent verification structure.
- **Offline Verifiability** — ICBs must be durable and self-contained. Any replica must be able to validate an ICB using only log-included material and cached IdP keys.
- **IdP Agnosticism** — Any IdP that issues a signed JWT with a carried-through nonce may be used. OIDC is *RECOMMENDED* but not required.
- **Replay Resistance** — Freshness constraints ensure that a previously used ICB or Signed Key Package cannot be replayed.

## **9.2 IdentityClaimBundle Structure (Normative)**

An **IdentityClaimBundle (ICB)** is a compact JWS whose payload contains a structured representation of:

1. A **JSON Key Package**
2. A **Signed Key Package** (inner JWS)
3. A **JWT Identity Token** from the IdP
4. A timestamp
5. A version field

A single **ICB Nonce** binds all components:

- It MUST appear in the JOSE protected header of the Signed Key Package.
- It MUST appear in the JOSE protected header of the outer ICB JWS.
- The IdP-issued JWT MUST contain a nonce equal to the **Proof-of-Possession Hash (pop\_hash)** derived from the Signed Key Package.

### **9.2.1 JSON Key Package (Normative)**

The Key Package is a minimal JSON object containing only the public key and freshness timestamp:

```json
{
  "v": 1, 
  "k": { /* JWK public key */ },
  "ts": "<ISO-8601 UTC timestamp>"
}
```

**Requirements:**

- **v** — MUST be 1.
- **k** — MUST contain the participant’s JWK-formatted public key (RFC 7517).
- **ts** — MUST record the client’s local timestamp at PoP creation.
- The Key Package MUST NOT contain nonce fields or private key material.

### **9.2.2 Signed Key Package (Inner JWS, Normative)**

The Signed Key Package is:

```asciidoc
JWS_protected_header(nonce=ICB-Nonce).
<base64url(JSON Key Package)>.
<signature>
```

**Requirements:**

- The JWS MUST be signed using the private key corresponding to the JWK in k.
- The JOSE protected header MUST contain nonce = \<ICB Nonce>.
- The JOSE protected header MAY include other parameters (e.g., alg).
- Any JOSE kid header is **ignored** for ICB validation; the key is taken *only* from the Signed Key Package payload.

### **9.2.3 Proof-of-Possession Hash (pop\_hash)**

The client MUST compute:

```asciidoc
pop_hash = base64url( SHA-256( <Signed Key Package compact serialization> ) )
```

This pop\_hash MUST be supplied as the **nonce parameter** to the IdP during authentication. The IdP-issued JWT MUST echo this exact value in its nonce claim.

### **9.2.4 ID Token Package (Normative)**

```json
{
  "v": 1,
  "ts": "<ISO-8601 UTC timestamp>",
  "signed_key_package": "<compact-JWS>",
  "id_token": "<JWT>"
}
```

**Requirements:**

- **signed\_key\_package** MUST be the exact compact JWS string from 9.2.2.
- **id\_token** MUST be a signed JWT whose nonce claim equals pop\_hash.
- **iss**, **sub**, **iat**, **exp**, and **nonce** MUST follow the rules in Section 9.7.

### **9.2.5 IdentityClaimBundle (Outer JWS, Normative)**

The final ICB is:

```json
ICB = JWS_protected_header(nonce=ICB-Nonce) .
      <base64url(ID Token Package)> .
      <signature>
```

**Requirements:**

- The outer JWS MUST be signed with the same private key used in the Signed Key Package.
- The JOSE protected header MUST contain the SAME **ICB Nonce**.
- Any JOSE kid header is ignored for validation.

## **9.3 Constructing the IdentityClaimBundle (Normative)**

### **Step 1 — Generate or Load Signing Keypair**

Client generates or loads its ASCP identity keypair (JWK). No private key material leaves the device.

### **Step 2 — Create JSON Key Package**

Client constructs the minimal Key Package JSON with v, k, and ts.

### **Step 3 — Choose ICB Nonce**

Client selects a fresh random nonce value (“ICB Nonce”).

This nonce will be placed **only** in JOSE protected headers.

### **Step 4 — Produce Signed Key Package**

Client signs the Key Package using its private key, embedding the ICB Nonce in the JOSE protected header.

### **Step 5 — Compute Proof-of-Possession Hash**

Client computes pop\_hash = base64url(SHA-256(SignedKeyPackage)).

### **Step 6 — Perform IdP Authentication**

Client authenticates using an IdP (OIDC RECOMMENDED) and MUST pass pop\_hash as the IdP nonce value.

The resulting JWT MUST contain a nonce matching pop\_hash exactly.

### **Step 7 — Construct ID Token Package**

Client assembles the structure defined in Section 9.2.4.

### **Step 8 — Sign the ICB**

Client produces the final IdentityClaimBundle JWS, again embedding the **same ICB Nonce** in the protected header.

## **9.4 Required Bindings and Cryptographic Couplings (Normative)**

Implementations MUST preserve the following bindings:

1. **Nonce Binding Across All Layers**
   - Signed Key Package header nonce == outer ICB header nonce == ICB Nonce.
2. **PoP Binding**
   - Signed Key Package MUST be signed by key corresponding to KeyPackage.k.
3. **IdP Binding**
   - id\_token.nonce MUST equal pop\_hash derived from the Signed Key Package.
4. **Outer-JWS Binding**
   - The outer ICB MUST be signed with the same key.
5. **Endorsement Binding**
   - Any endorsement::oidc-icb MUST reference the RFC 7638 JWK fingerprint of the Certificate containing KeyPackage.k.

These bindings ensure integrity, replay resistance, and deterministic offline validation.

## **9.5 Freshness Requirements (Normative)**

Verifiers MUST apply the following temporal constraints:

1. **Key Package Timestamp Freshness**
   - now − KeyPackage.ts MUST NOT exceed implementation-defined Δ₁ (e.g., 2 minutes).
2. **IdP Token Freshness**
   - abs(id\_token.iat − KeyPackage.ts) MUST NOT exceed Δ₂.
3. **ICB Submission Freshness**
   - now − id\_token.iat MUST NOT exceed Δ₃.
4. **Replay Protection**
   - A Signed Key Package MUST NOT be reused.
   - An id\_token nonce (pop\_hash) MUST NOT be reused.

Deployments MAY enforce stricter policies.

## **9.6 Validation Requirements (Normative)**

Upon receiving an ICB, a verifier MUST:

### **1. Parse ICB**

Extract ID Token Package, Signed Key Package, JWT identity token.

### **2. Validate Signed Key Package**

- Verify JWS using the JWK in KeyPackage.k.
- Reject if JOSE header nonce is missing or malformed.
- Confirm KeyPackage.ts satisfies freshness requirements.

### **3. Validate Outer ICB JWS**

- Verify signature using the same public key.
- Confirm outer JOSE header nonce equals inner nonce.

### **4. Recompute pop\_hash**

- Compute pop\_hash = base64url(SHA-256(signed\_key\_package)).
- Confirm id\_token.nonce == pop\_hash.

### **5. Validate JWT Identity Token**

- Validate its JWS signature using IdP-published JWKS.
- Validate iss, sub, aud (if present), iat, exp, and nonce bindings.
- Apply freshness rules in Section 9.5.

### **6. Accept or Reject**

If all checks succeed, the verifier MAY produce an endorsement::oidc-icb attribute referencing the certificate fingerprint.

## **9.7 Identity Token Requirements (Normative)**

### **9.7.1 Purpose**

The Identity Token binds an external authentication event to the participant’s ASCP key. It does *not* authorize ASCP actions; it proves identity and contributes evidence to the endorsement.

### **9.7.2 Required Format**

- MUST be a signed JWT (RFC 7519).
- MUST NOT be an OAuth access token.
- MUST contain:
  - iss — IdP identifier
  - sub — stable subject identifier
  - iat — recent issuance time
  - exp — expiration time
  - nonce — MUST equal pop\_hash

### **9.7.3 Signature Requirements**

- MUST use a JOSE asymmetric signature algorithm providing ≥128-bit security (e.g., ES256, ES384, PS256).
- HMAC-based algorithms (e.g., HS256) MUST NOT be accepted.
- Signature MUST validate against IdP JWKS material.

### **9.7.4 Non-OIDC Tokens**

A non-OIDC IdP MUST still issue a JWT with:

- A verifiable asymmetric signature.
- A nonce claim equal to the pop\_hash.
- Sufficient issuer metadata for trust evaluation.

### **9.7.5 Token Acquisition**

- When available, OIDC Authorization Code + PKCE MUST be used.
- ID Tokens MUST NOT be transmitted via front-channel redirection.
- Clients MUST prove possession of the PKCE code\_verifier.

## **9.8 Worked Example (Informative)**

Example showing an Identity Artipoint, Certificate Artipoint, and resulting endorsement:

```json
[identity-uuid, author-uuid, 2025-08-08T14:30:22Z,
  ["identity", "Jeff Szczepanski", "mailto:jeff@example.com"] .
  ( type := "human",
    certificate::kid := "ascp:cert:cert-uuid"
  )
];

[cert-uuid, author-uuid, 2025-08-08T14:30:22Z,
  ["certificate", "Jeff ES256 Identity Key", json:{ <JWK> }] .
  ( fingerprint := "sha256:abcd...",
    issued := "2025-08-08T14:30:22Z" )
];

[verify-uuid, author-uuid, 2025-08-08T14:30:25Z,
  cert-uuid .
  ( endorsement::oidc-icb + json:{
      "schema":"ascp.endorsement.v1",
      "attestation":"id-binding",
      "fingerprint":"sha256:abcd...",
      "issuer":{"mechanism":"oidc-icb","hint":"accounts.google.com"},
      "evidence":{
        "icb":"<IdentityClaimBundle JWS>"
      },
      "issued_at":"2025-08-08T14:30:25Z"
    }
  )
];
```

# 10. Identity: Certificate Binding & Rotation

In ASCP, every user or agent that contributes immutable entries to the articulation log must have its own cryptographic key pair. The private key is used to sign all Artipoints authored by that identity. The public key is published in the log to establish verifiable provenance.

## **10.1 Default: Privacy‑First Local Key Generation**

- **Generation**: Key pairs are generated client‑side, within the end‑user’s or agent’s own secure environment.
- **Storage**: Private key remains under sole control of the owner:
  - Desktop/mobile → OS secure enclave or keychain.
  - Browser → WebCrypto API, secure context, IndexedDB.
  - Agent → TPM, HSM, secure file system.
- **Publication**: Public key MUST be self‑signed by the private key holder and submitted during an authenticated ASCP session.
- **RootCA linkage**: A user or agent MAY also sign the public key with the repository's RootCA to bind it into the bootstrap‑anchored trust chain.

Use `certificate::kid` on the identity to set the current active signing key.

## **10.2 Set or Rotate the Active Key:**

```bnf
[rot-uuid, alice@example.com, 2025-08-12T09:00:00Z,
  identity-uuid .
  ( certificate::kid := "ascp:cert:<new-cert-uuid>" )
];

```

Layer-3 clients are responsible for selecting which Certificate is active for authorship and interpreting governance state to determine whether authorship is semantically permitted. Identity & Trust does not perform or define this evaluation.

## **10.3 Why this is right**

- Simple, explicit, and mirrors channel keyframe activation.
- Old keys remain verifiable for historical signatures; no extra bookkeeping.

# **11. Key Escrow and Recovery Strategy**

ASCP provides a secure and auditable approach to private key recovery that builds on the log-anchored trust foundation. The strategy enables users to recover their **identity private key** without ever exposing it in plaintext to any third party, supporting device migration, key backup, and enterprise compliance scenarios.

## **11.1 Core Design Principles**

- **Zero-trust storage**: Recovery materials remain confidential even if the log is compromised
- **Self-contained recovery**: No external key management dependencies
- **Dual recovery modes**: User-controlled privacy or enterprise escrow options
- **User-controlled decryption**: Only authorized parties can decrypt recovery bundles

## 11.2 user-key-envelope

The `user-key-envelope` is a **double-encrypted JWE object** used to protect and store a user identity private key for recovery purposes. It is constructed through two layers of JWE encryption, both using **compact serialization**:

### **Construction (inner to outer)**

1. **Inner JWE** (compact serialization): The identity private key (Key A) is formatted as a JWK and encrypted using a password-derived AES key (Key C)
   - Algorithm: `alg: "dir"`, `enc: "A256GCM"`
   - Payload: JWK-formatted EC private key (Key A)
   - Output: A compact JWE string (five base64url segments)
2. **Outer JWE** (compact serialization): The inner JWE compact string is encrypted using the recovery public key (Key B)
   - Algorithm: `alg: "ECDH-ES+A256KW"`, `enc: "A256GCM"`
   - Payload: The complete inner JWE compact string
   - Output: A compact JWE string (five base64url segments)

**The** `user-key-envelope` **is the compact serialization string of the outer JWE.**

### **Security properties**

This double-encryption strategy ensures:

- Only a party with the recovery key **and** the password can recover the private key
- Even if one factor is compromised, the identity key remains secure
- All encryption uses authenticated encryption (AES-GCM) via standard JOSE mechanisms
- Both layers use compact serialization, not JSON serialization

## **11.3 Normative Procedures**

### **11.3.1 Recovery Envelope Construction**

This procedure defines the normative steps for constructing a `recovery_envelope` attribute during identity bootstrap or key rotation. The resulting attribute structure is defined in Section 8.3.

1. **Generate Identity Keypair (Key A)**
   - An implementation MUST generate an EC keypair (P-256 or P-384) to serve as the original identity signing key
   - The private key MUST be used to sign Artipoints and authenticate sessions
   - The public key MUST be published in a Certificate Artipoint
2. **Establish Recovery Keypair (Key B)**
   - An implementation MUST generate or register a separate EC keypair for recovery purposes
   - The recovery private key MUST be stored in a secure location distinct from the identity key (e.g., second device, hardware token, secure backup)
   - The recovery public key MUST be published in a separate Certificate Artipoint
3. **Derive Password-Based Encryption Key (Key C)**
   - An implementation MUST derive an AES-256 key from a user-provided passphrase
   - The derivation MUST use PBKDF2 (minimum 600,000 iterations) or Argon2id
   - The salt and KDF parameters SHOULD be stored in the `recovery_envelope` attribute for interoperability.
4. **Construct user-key-envelope (Double Encryption)**
   - An implementation MUST construct the `user-key-envelope` as defined in Section 11.2:
     - **Inner JWE**: Encrypt the identity private key (Key A, formatted as JWK) using Key C with `alg: "dir"` and `enc: "A256GCM"`, producing a compact JWE string
     - **Outer JWE**: Encrypt the inner JWE compact string using the recovery public key (Key B) with `alg: "ECDH-ES+A256KW"` and `enc: "A256GCM"`, producing a compact JWE string
   - The resulting outer JWE compact string IS the `user-key-envelope`
5. **Construct recovery\_envelope Attribute**
   - An implementation MUST construct a JSON object conforming to the `recovery_envelope` schema defined in Section 8.3
   - The `user_key_jwe` field MUST contain the `user-key-envelope` string from step 4
   - The `recovery_cert` field MUST reference the UUID of the Certificate Artipoint containing the recovery public key (Key B)
   - The `kdf_params` field SHOULD contain the salt and parameters used in step 3
   - The `created` field MUST contain an ISO-8601 UTC timestamp
6. **Attach to Certificate Artipoint**
   - An implementation MUST attach the `recovery_envelope` JSON structure as an attribute to the Certificate Artipoint containing the identity public key (Key A)
   - The attribute MUST use the `recovery_envelope` attribute name
   - The annotated Certificate Artipoint MUST be published to the appropriate channel for replication

### **11.3.2 Key Recovery Procedure**

This procedure defines the normative steps for recovering an identity private key from a `recovery_envelope` attribute. This procedure is executed on a new device or after loss of the original private key.

1. **Locate Certificate and recovery\_envelope Attribute**
   - An implementation MUST locate the user's Certificate Artipoint containing the identity public key (Key A)
   - The implementation MUST retrieve the `recovery_envelope` attribute attached to that Certificate Artipoint
2. **Extract recovery\_envelope Structure**
   - An implementation MUST parse the `recovery_envelope` attribute value as JSON conforming to the schema defined in Section 8.3
   - The implementation MUST extract the `user_key_jwe` field value (the `user-key-envelope` string)
   - The implementation MUST extract the `recovery_cert` field value (UUID of the recovery Certificate Artipoint)
   - The implementation SHOULD extract the `kdf_params` field for password derivation
3. **Decrypt Outer JWE Layer**
   - An implementation MUST locate the Certificate Artipoint referenced by `recovery_cert` and retrieve the recovery private key (Key B)
   - The implementation MUST decrypt the outer JWE layer of the `user-key-envelope` using Key B
   - The result MUST be a compact JWE string (the inner JWE)
4. **Derive Password-Based Key**
   - An implementation MUST prompt the user for the recovery passphrase
   - The implementation MUST derive Key C using the KDF parameters from step 2 (or from `kdf_params` if available)
5. **Decrypt Inner JWE Layer**
   - An implementation MUST decrypt the inner JWE using Key C
   - The result MUST be a JWK-formatted EC private key (Key A)
6. **Import Identity Private Key**
   - An implementation MUST import Key A into secure storage (e.g., Secure Enclave, TPM, platform keychain)
   - The implementation SHOULD verify that the imported key corresponds to the public key in the Certificate Artipoint
7. **Resume Normal Operations**
   - The implementation MAY now use Key A to:
     - Sign new Artipoints
     - Decrypt channel key envelopes (Channel AES and CAK)
     - Authenticate ALSP replication sessions
     - Verify authorship of historical Artipoints

## **11.4 Security Considerations**

- **Passive log exposure**: Since the envelope is doubly encrypted and stored in the clear, even full replication of the log does not compromise user secrets
- **Key B storage**: Recovery key material must be protected via a second device, hardware token, or exportable file/QR code with strong local security
- **Key C (password) complexity**: The passphrase should meet entropy guidelines and use a strong KDF like Argon2id
- **No org access**: The organization cannot decrypt or recover a user's key unless explicitly granted access to Key B or Key C by the user

## **11.5 Future Enhancements**

- **Multi-party Shamir recovery**: Split Key B into multiple shares for multi-admin or user+device recovery models
- **QR-code recovery bootstrapping**: Encode the recovery JWE for scanning from air-gapped backups
- **Hardware key support**: Use FIDO2/WebAuthn for Key B storage and authentication during decryption
- **Rotation tracking**: Allow rotation of Key A while preserving old envelopes in the log for historical decryptability

## **11.6 Summary**

ASCP supports secure, privacy-preserving key recovery by leveraging layered encryption, log-anchored publication, and optional recovery keypairs. This ensures that users can regain access to their identity keys without trusting any intermediary, while maintaining the protocol's cryptographic integrity and auditability.

# **12. Provenance of the ASCP RootCA**

This section defines the full provenance chain for the **ASCP Root Certificate Authority (RootCA)**, including:

- Internal self-signing and articulation
- Optional anchoring to a Public or Enterprise PKI certificate
- Optional independent attestation via a Time-Stamping Authority (TSA)

The goal is to ensure that **any verifier, at any time in the future**, can:

1. Establish the origin and legitimacy of the RootCA.
2. Confirm when it was created and anchored.
3. See its relationship to a trusted PKI at the time of anchoring.
4. Validate the integrity of this provenance against immutable and independent records.

## **12.1 RootCA Generation and Self-Signing**

1. **Generate RootCA Key Pair**
   - Algorithm: ECDSA P-384 (default) or equivalent.
   - Created in a secure environment by the ASCP repository operator.
   - Private key remains under sole control of the repository owner organization.
2. **Create RootCA Self-Signed Certificate**
   - Contains the public key.
   - Acts as the trust anchor for all ASCP participants and components in the repository.
3. **Embed in Bootstrap Artipoint**
   - The RootCA public key and self-signed certificate are embedded in the **first immutable Artipoint** of the repository’s @Bootstrap channel.
   - This Artipoint is cryptographically signed by the RootCA private key.

## **12.2 Optional Anchoring to a Public or Enterprise PKI**

To provide **portable, externally verifiable trust**, the RootCA can be endorsed by a PKI certificate from:

- **Public PKI**: e.g., Let’s Encrypt, DigiCert.
- **Enterprise PKI**: e.g., corporate CA.

**Process:**

1. Obtain a valid TLS/identity certificate from the chosen PKI.
2. Using the PKI certificate's private key, create a **detached signature** over the RootCA public key or self-signed certificate.
3. Capture the **full validated certificate chain** for the PKI certificate at the time of signing.
4. Embed both the detached signature and certificate chain in the same bootstrap Artipoint alongside the RootCA.

**Result:**

Any verifier can:

- Validate the PKI certificate chain against trusted PKI roots.
- Verify the detached signature over the rootCA key.
- Confirm that the RootCA was endorsed by the PKI identity at the recorded point in time.

## **12.3 Optional Time-Stamping Authority (TSA) Signature**

To provide **independent temporal attestation**, the bootstrap Artipoint can include an RFC 3161-compliant TSA token:

**Process:**

1. Create a hash of the RootCA provenance payload (including the PKI signature and certificate chain if present).
2. Submit this hash to a trusted Time-Stamping Authority.
3. Receive a signed **Time-Stamp Token (TST)** from the TSA, which:
   - Proves the payload existed at or before the TSA’s timestamp.
   - Is signed by the TSA’s own PKI-trusted certificate.
4. Embed the TST in the bootstrap Artipoint.

**Result:**

- Verifiers gain third-party assurance of when the provenance payload was created.
- Even if the ASCP repository’s own time records are disputed, the TSA attestation stands as independent proof.

## **12.4 Provenance Chain Validation Steps for a Verifier**

When a verifier examines a repository’s bootstrap channel, they:

1. **Extract the RootCA certificate** from the first Artipoint.
2. **Verify RootCA self-signature** (internal trust anchor).
3. If present, **validate PKI anchor**:
   - Check certificate chain validity (at time of signing).
   - Verify detached signature over RootCA key.
4. If present, **validate TSA attestation**:
   - Verify TSA token signature against TSA public key.
   - Confirm timestamp matches repository creation timeframe.
5. **Confirm immutability**:
   - Ensure bootstrap Artipoint is intact in all replicas.
   - Validate signatures back to RootCA.

## **12.5 Key Points**

- The provenance is **permanently preserved** in the immutable bootstrap channel.
- Even if the PKI certificate later expires or is revoked, verifiers can confirm it was valid when the repository was anchored.
- Re-anchoring is possible at any later date to provide renewed PKI endorsements without replacing the RootCA.
- Optional TSA signatures and/or publication in a transparency log provide **independent temporal proof** for maximum audit assurance.

# 13. Overall Summary

ASCP's log-anchored trust architecture solves the fundamental challenges of distributed identity by anchoring all trust decisions in immutable log entries rather than real-time validation. This provides durable identity, complete cryptographic provenance, and autonomous operation without external dependencies.

The system enables both privacy-first and enterprise deployments through self-sovereign key generation, optional PKI anchoring, and secure recovery mechanisms. Most critically, it establishes verifiable trust relationships between humans and autonomous agents with identical cryptographic guarantees.

ASCP delivers a trust infrastructure that integrates with existing PKI systems today while providing the foundation for scalable human-AI coordination.