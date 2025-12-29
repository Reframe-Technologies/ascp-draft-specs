# **ASCP Trust and Identity Architecture**

**A Log-Anchored approach to Distributed Trust**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.52 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the **Agents Shared Cognition Protocol (ASCP) specification suite** and defines the **Trust and Identity Architecture** for ASCP. It is published as an **Informational, pre-RFC working draft** for early technical review and experimentation.

This document has not been reviewed by the IETF and has no formal standing within the IETF standards process. Implementations based on this document **MUST be considered experimental** and subject to change. This document aims to inform future Standards Track work, though not all mechanisms defined here are guaranteed to remain normative.

The key words **"MUST"**, **"MUST NOT"**, **"SHOULD"**, **"SHOULD NOT"**, and **"MAY"** in this document are to be interpreted as described in RFC 2119 and RFC 8174. In this draft, these terms describe the authors' intended normative behavior for interoperable ASCP implementations but do not yet constitute stable, standards-track requirements.

This document serves as a basis for community and security review, a reference for early implementers, and an input to future ASCP Internet-Draft development.

This specification complements, and is intended to be read alongside, other ASCP documents, including the **ASCP Governance and Access Control Specification**, which defines authorization, role semantics, and policy evaluation independent of the identity and trust mechanisms described here.

Feedback from protocol designers, implementers, cryptographers, and distributed systems researchers is explicitly solicited to inform subsequent revisions.

# **2. Abstract**

This document specifies the **Trust and Identity Architecture** for the **Agents Shared Cognition Protocol (ASCP)**.

It defines a **log-anchored, replica-local, deterministically evaluable** trust model in which a verifier derives trust state by replaying immutable, append-only **Coordination Log** history and applying replica-local policy. In this model, identities, certificates, endorsements, key-binding events, and recovery artifacts are recorded as cryptographically verifiable log entries, enabling durable authorship verification, offline replayability, and reconstruction of historical trust state without dependence on continuous external infrastructure.

This specification normatively defines the core Layer-3 security constructs and artifacts used to represent and evaluate trust over time, including schemas and semantics for **Identity**, **Certificate**, **Keyframe**, and **RootCA** Artipoints, as well as associated distribution and evidence objects such as **Identity Claim Bundles (ICB)**, **Recovery Envelopes**, and **Channel Key Envelopes (CKE)**. It also specifies certificate publication and rotation semantics, optional anchoring to external trust systems (e.g., public-key infrastructures and identity providers), and privacy-preserving key recovery mechanisms.

This document explicitly separates **authentication and authorship verification** from **authorization, governance, and policy evaluation**, which are defined in companion ASCP specifications. It does not define governance rules, access-control decisions, distribution-layer encryption, or log synchronization behavior.

# **3. Introduction**

This document specifies the **Trust and Identity Architecture** for the **Agents Shared Cognition Protocol (ASCP)**. It defines how participants—humans, agents, and system components—are identified, and how trust is evaluated over time from immutable, append-only log history. ASCP separates **signature integrity validation** performed by the **Channel Codec (Layer-1)** from **authorship attribution and trust interpretation** performed by **Layer-3**, yielding deterministic **log-time** provenance derived from recorded evidence.

In ASCP, **identity is provenance**, not permission. Cryptographic identity establishes *who authored a statement* and *which key material was bound to that identity at the relevant log-time*. Whether a statement is authorized, accepted, or acted upon is determined by governance and policy semantics defined in companion specifications.

ASCP’s trust model is **log-anchored**: trust decisions are derived from signed log entries and their recorded relationships, rather than from real-time certificate validity checks or mutable external state. This enables deterministic, replayable evaluation that remains valid offline and does not require continuous availability of third-party infrastructure.

## **3.1 Layering Invariant**

ASCP strictly separates **representation** from **interpretation**:

- **Layer-2 (Artipoint Grammar)** defines the canonical syntax, serialization, and structural validity of ASCP statements.
- **Layer-3 (Trust and Identity)** defines the semantic meaning of trust-related constructs and the evaluation rules that derive cryptographic consequences.

All constructs defined in this document are **Layer-3 semantic constructs**. They are represented using the Layer-2 grammar and parsed according to Layer-2 rules, but are defined and interpreted only by Layer-3.

## **3.2 Design Goals**

The Trust and Identity Architecture defined in this document is intended to provide:

- **Durable identity** across key rotation and device change.
- **Cryptographic provenance** via immutable recording of authorship, bindings, and attestations in ASCP logs.
- **Deterministic evaluation** over historical log state (“log-time trust”).
- **Extensibility** through endorsement mechanisms that admit new attestation forms without changing core semantics.
- **Optional external anchoring** to ecosystems such as enterprise PKI, OIDC, DIDs, and timestamping authorities, without introducing hard runtime dependencies.
- **Recovery continuity** mechanisms supporting device migration while preserving confidentiality.

## **3.3 Scope of This Specification**

This document defines:

- Trust-relevant **Security Construct Artipoints** used to represent identity, certificates, trust anchors, endorsements, key lifecycle state, and recovery evidence.
- The Layer-3 verification and evaluation rules for **identity bootstrap**, **certificate binding**, **key rotation**, and **recovery**.
- Verification requirements needed for interoperable Trust and Identity implementations.

Normative requirements use the terms **MUST**, **SHOULD**, and **MAY** as described in RFC 2119 and RFC 8174.

## **3.4 Out of Scope**

This document does **not** define:

- Governance semantics, authorization decisions, membership rules, role evaluation, or view derivation (see ASCP Governance and Access Control specification).
- Layer-1 Channel Envelope formats or codec behavior, including distribution-layer cryptographic mechanics (see ASCP Channels).
- Transport, replication, ordering, or peer authentication semantics (see ALSP).
- Discovery and onboarding procedures for initial trust establishment (see Bootstrap and Channel Discovery).
- Application UI or workflow behavior.

This specification is strictly Layer-3: it defines **what** trust evidence exists and **how** it is evaluated to produce cryptographic consequences for use by other layers.

## **3.5 Relationship to Other ASCP Specifications**

This document provides Layer-3 trust evidence and evaluation rules that other specifications consume:

- **Channels (Layer-1)** use cryptographic consequences derived from Layer-3 evaluation (e.g., selected keys and recipient sets) to sign and/or encrypt Articulation Sequences into Artipoint Records. Channels enforce these consequences mechanically and do not interpret trust semantics.
- **ALSP (Layer-0)** replicates Channel Logs without semantic interpretation; any identity-bound session or transport credentials are defined by ALSP.
- **Governance and View Evaluation (Layer-3)** consumes Identity and Certificate constructs defined here as input evidence when deriving roles, membership sets, and application views; governance rules themselves are defined elsewhere.

# **4. Terminology**

This section defines terms used throughout this specification and this document follows the terminology and layering conventions defined in the **ASCP Terminology Primer**, which serves as the authoritative reference for distinguishing semantic constructs, representations, derived structures, and enforcement mechanisms across the ASCP specification suite.

Unless explicitly stated otherwise, all definitions in this section are **informational**. Normative requirements are defined in subsequent sections.

### **Artipoint**

A **conceptual, immutable unit of articulated meaning** in the ASCP semantic model.

An Artipoint represents a discrete contribution—such as an identity declaration, certificate publication, endorsement, or coordination construct—and serves as a first-class node in ASCP’s semantic graph. Artipoints are **Layer-3 semantic constructs**.

An Artipoint is independent of any specific serialization, signature, or transport mechanism. It is *represented* using the Layer-2 Artipoint Grammar and *materialized* as a signed record in an ASCP channel log, but the Artipoint itself is not a log entry.  
  
Convention: Subsequent sections use ‘Artipoint’ to refer to the semantic construct, unless explicitly referring to **Artipoint Expressions** or **Artipoint Records**.

### **Artipoint Expression**

A **Layer-2 syntactic representation** of an Artipoint, defined by the ASCP Artipoint Grammar.

An Artipoint Expression encodes the type, label, payload, and attributes of an Artipoint in a canonical form suitable for signing, distribution, and replication. Artipoint Expressions carry **no inherent trust, authorship, or governance semantics** until evaluated by Layer-3.

### **Articulation Statement**

A **Layer-2 syntactic statement** that asserts, introduces, or relates Artipoints using the ASCP Artipoint Grammar.

Articulation Statements have no inherent trust, governance, or authorization meaning until interpreted at Layer-3.

### **Articulation Sequence**

An **ordered sequence of Articulation Statements** that forms the payload committed to a Channel Log.

An Articulation Sequence is the unit encoded, protected, and carried by the Channel Envelope.

### **Channel Envelope**

A **Layer-1 container** that encapsulates a serialized Articulation Sequence together with the metadata and cryptographic protections required for channel distribution (e.g., signing and optional encryption).

### **Channel Log**

An **append-only log** for a specific Channel consisting of Artipoint Records.

### **Artipoint Record**

A durable, cryptographically secured materialization of an **Articulation Sequence** within an ASCP **Channel Log**. An Artipoint Record encapsulates a serialized Articulation Sequence within a **Channel Envelope**, applies signing to establish authorship integrity, and MAY apply encryption to enforce visibility scope.

### **Addressing Construct (Artipoints)**

In the ASCP terminology taxonomy, **Addressing Constructs** are Artipoints whose primary semantic role is to identify *who* participates in coordination and to provide stable, durable references to participants over time.

Addressing Constructs are **Layer-3 semantic constructs**. They are represented using the Layer-2 Artipoint Grammar and materialized as signed Artipoint Records in ASCP Channels.

Examples include Identity Artipoints and, in other ASCP documents, Group Artipoints.

### **Security Construct (Artipoints)**

In the ASCP terminology taxonomy, **Security Constructs** are Artipoints whose primary semantic role is to establish or describe **cryptographic material, provenance evidence, trust anchors, or security-relevant lifecycle state**.

Security Constructs are **Layer-3 semantic constructs**. They are represented using the Layer-2 Artipoint Grammar and materialized as signed Artipoint Records in ASCP Channels.

This specification defines Security Constructs that are evaluated in relationship to **Addressing Constructs** (i.e., participants). Concretely: Addressing Constructs like Identity Artipoints name participants, while Security Constructs like Certificate, RootCA, and Keyframe Artipoints define cryptographic keys, anchors, and epochs that can be bound to those participants and evaluated at log-time.

### **Identity Artipoint**

An **Addressing Construct** Artipoint that declares the existence of a participant—human, agent, or system component—and provides a **durable addressing reference** for that participant.

An Identity Artipoint does **not** contain cryptographic key material. Instead, it references one or more Certificate Artipoints via `certificate::kid` attributes to establish which keys are currently bound to the identity.

### **Certificate Artipoint**

A **Security Construct** Artipoint that publishes a JWK encoded **public cryptographic key** and declares the cryptographic purposes, endorsements, and recovery material associated with that key.

Certificate Artipoints are the canonical source of cryptographic material used for authorship signing, authentication, and key agreement in ASCP.

### **RootCA Artipoint**

A distinguished Certificate Artipoint that serves as the **root trust anchor** for an ASCP instance.

The RootCA Artipoint is introduced during bootstrap and anchors certificate acceptance, onboarding provenance, and optional cross-instance trust. RootCA Artipoints are immutable and may be endorsed by external trust systems.

### **Keyframe Artipoint**

A Security Construct Artipoint that defines a **cryptographic epoch** for an ASCP Channel.

Keyframe Artipoints encapsulate Layer-3 cryptographic configuration and distribute encrypted key material to authorized participants. Their evaluated cryptographic consequences are provisioned to lower layers, but their semantic meaning is defined exclusively at Layer-3.

### **Log-Anchored Trust**

In ASCP, trust is an evaluation performed by a verifier over immutable evidence. The ASCP trust model ensures that **all trust decisions derive from immutable, signed Artipoint Records** rather than from real-time certificate validation or mutable external state.

Trust evaluation is performed against historical log state ("log-time trust") and explicitly configured local policy inputs, enabling deterministic, replayable verification independent of external infrastructure availability.

### **Bootstrap Channel (or Bootstrap Log)**

A special ASCP Channel that contains the **genesis Artipoint Records** of an instance, including the RootCA Artipoint.

The Bootstrap Channel establishes the immutable foundation of trust for the ASCP instance.

### **Endorsement**

In ASCP, endorsements provide evidence but do not themselves confer trust. An Endorsement is an **immutable annotation** attached to a Certificate Artipoint or Identity Artipoint that records **externally verifiable evidence** issued by a third-party authority.

Endorsements strengthen provenance but do **not** alter the semantic meaning of identities, certificates, or governance rules.

### **Purpose Attribute**

A **first-party declaration** attached to a Certificate Artipoint that specifies the allowed cryptographic use of the key (e.g., `purpose::assert`, `purpose::auth`, `purpose::keyAgreement`).

Purpose attributes constrain key usage but do not confer trust or governance authority.

### **Author**

The **Author** is the **Layer-3** participant identity to which an Articulation Sequence is attributed.

Authorship is evidenced by **signature material** (e.g., a JWS) produced using a private key that is bound—at the relevant **log-time**—to the Author’s Identity Artipoint via a Certificate Artipoint.

**Layer-1** validates signature integrity as part of Channel Envelope processing. **Layer-3** attributes authorship by resolving the signing key reference (e.g., `kid`) to the applicable Certificate and Identity history and evaluating log-time binding and endorsement evidence.

### **Sender**

The **Sender** is the operational actor that constructs and commits a Channel Envelope into the Channel log.

The Sender:

- renders the Articulation Sequence payload into a JWS-signed object using the **Author’s** signing key,
- optionally encrypts the signed payload for a Recipient Set under the active Keyframe, and
- submits the resulting envelope for replication via Layer-0 (ALSP).

The Sender is often the Author, but may be a delegated agent or service acting on the Author’s behalf. The Sender’s transport identity is not treated as a substitute for authorship; authorship is determined only by the Author-bound signature evidence evaluated under this specification.

### **Author / Sender / Recipient Roles**

ASCP separates:

- **Author** (semantic authorship of payload),
- **Sender** (operational submission of an envelope), and
- **Recipient** (membership in the set for whom decryptable key material is issued).

This separation is intentional: the Recipient Set is a cryptographic distribution decision, while authorship is a provenance claim. These roles are not required to be symmetric.

### **Recipient (Recipient Set)**

A **Recipient** is an identity included in the evaluated **Recipient Set** for which the Keyframe consequences include a decryptable JWE channel-key-envelope (i.e., wrapped key material enabling payload decryption for the associated key version).

Recipient Sets define confidentiality scope for a given key version. Replication authorization (e.g., CAK use at Layer-0) is defined independently.

### **Recovery Envelope**

A **double-encrypted payload** attached to a Certificate Artipoint that contains encrypted private key material for secure recovery or device migration.

Recovery Envelopes enable key recovery without exposing private keys to ASCP services or peers.

### **Channel**

A named ASCP construct that defines **scoped distribution and replication** of Artipoint Records among authorized participants. Channels provide append-only ordering, confidentiality, and authenticated participation but do not define identity semantics or governance rules.

A Channel defines a scoped trust domain for replicated coordination history.

Within a Channel:

- all trust conclusions are derived from the immutable, append-only log contents plus local policy inputs,
- confidentiality is enforced via Recipient Sets and Keyframe-distributed key material, and
- replication authorization is enforced via CAK-based mechanisms at Layer-0.

No participant is required to trust external availability of third-party infrastructure at runtime. Trust evaluation is replica-local and derivable from log evidence (and permitted external attestations) as defined by this specification.

### **Channel Access Key (CAK)**

The **Channel Access Key (CAK)** is a replication credential used to authenticate access to a Channel’s Layer-0 synchronization domain.

Properties:

- The CAK authorizes **Layer-0 replication only** (e.g., production of Channel Access Proofs).
- The CAK has **no semantic meaning** at Layer-1 or Layer-2.
- The CAK **does not grant decryption rights** for payload confidentiality.
- The CAK **does not grant authorship rights** and is not used to infer authorship.
- CAK key material is declared and distributed as part of Keyframe consequences, but is **enforced only** by Layer-0 replication procedures.

Layer-1 and Layer-2 treat the CAK as opaque and do not attempt to validate, interpret, or derive meaning from CAK material beyond what is provisioned by Layer-3 evaluations.

### **ALSP (ASCP Log Synchronization Protocol)**

The Layer-0 protocol responsible for **authenticated replication, convergence, and synchronization** of ASCP channel logs across peers.

ALSP does not interpret Artipoint semantics, trust state, or governance meaning.

### **JOSE Structures (JWE / JWK / JWS / JWT)**

Standardized JSON-based cryptographic structures defined by the JOSE specifications:

- **JWE** — JSON Web Encryption
- **JWK** — JSON Web Key
- **JWS** — JSON Web Signature
- **JWT** — JSON Web Token

ASCP uses JOSE structures for key representation, signing, encryption, and external attestations.

### **Public Key Infrastructure (PKI)**

An external certificate trust system (e.g., public or enterprise CA hierarchy) that **MAY** be used to provide endorsements or anchoring evidence for ASCP certificates.

PKI validity is evaluated at log-time and does not impose real-time dependencies.

### **Time-Stamping Authority (TSA)**

An RFC 3161-compliant service that issues Time-Stamp Tokens (TSTs), providing independent temporal attestation that specific cryptographic material existed at or before a given time.

# **5. ASCP Trust Architecture Overview**

ASCP establishes trust using **durable, signed log records** rather than mutable external state or real-time certificate validation. Identity binding, authorship verification, endorsement evaluation, and key lifecycle interpretation are derived from the **history articulated in the Channel log** (“log-time trust”).

This section is **informative**. It provides architectural context and scope boundaries for the normative requirements in Sections 6–9.

## **5.1 Log-Anchored Trust Model (Informative)**

ASCP’s trust model is grounded in three architectural principles:

1. **Immutable provenance -** Trust-relevant events (e.g., identity declarations, certificate bindings, endorsements, and key lifecycle events) are expressed as signed artifacts and committed to an append-only history. Verifiers evaluate evidence **as recorded**, not as reinterpreted under present-day external conditions.
2. **Replica-local validation -** A replica can validate authorship and trust evidence from log contents plus explicitly configured local policy inputs. External systems may strengthen provenance, but are **not required** for correctness or availability.
3. **Deterministic evaluation over articulated history -** Trust conclusions are derived by deterministic Layer-3 interpretation of the log state. Cryptographic verification provides evidence; trust is the semantic interpretation of that evidence under the rules defined by this document.

## **5.2 Scope and Layer Boundaries (Informative)**

ASCP separates responsibilities across layers. This document specifies **Layer-3 (Trust and Identity)** semantics and their evaluation rules.

- **Layer-3 (this document):** defines the meaning and evaluation of trust-related constructs (e.g., Identity, Certificate, RootCA, Keyframe, endorsements, recovery). It specifies how trust state is derived from log history and local policy inputs.
- **Layer-2 (Artipoint Grammar):** defines the syntactic structure and encoding of statements. It does not define trust meaning or evaluation.
- **Layer-1 (Channels / Channel Envelope Codec):** defines envelope admissibility and codec behavior. It applies cryptographic protections and verifies cryptographic integrity **using Layer-3-derived consequences**, but does not define trust semantics, governance, or key agreement procedures.
- **Layer-0 (ALSP):** replicates the log and provides authenticated synchronization. It does not interpret trust, identity, or authorship semantics.

This separation is intentional: **Layer-3 defines meaning; Layer-1 carries and validates bytes; Layer-0 synchronizes history.**

## **5.3 Relationship to External Trust Ecosystems (Informative)**

ASCP can operate without external trust infrastructure. When external systems are used (e.g., enterprise PKI, identity providers, DIDs, timestamping), they contribute **immutable evidence** captured in the log as endorsements or attestations.

ASCP does not require online revocation checks or runtime dependency on third-party availability for correctness. External evidence may strengthen provenance, but the trust model remains **log-anchored and replica-local**.

(Concrete mechanisms and normative requirements for endorsements and external evidence are specified in later sections.)

## **5.4 Replica-Local Evaluation Invariants (Normative)**

Implementations MUST support local-first operation in which envelope arrival order, partial replication state, and multi-writer convergence behavior do not change the interpretation rules defined by Layer-3.

- Layer-1 envelope codec processing MUST be atomic with respect to each envelope and MUST NOT depend on global ordering, causal delivery, or replica convergence state.
- Layer-3 trust evaluation MUST be a deterministic function of the articulated history available to the replica (and explicitly configured local policy inputs).
- During partial replication, conclusions are necessarily provisional; however, once replicas possess identical articulated history for the relevant scope, they MUST derive equivalent trust results under equivalent policy inputs.

## **5.5 Non-Goals and Related Mechanisms (Informative)**

ASCP does not require global transparency logs or Merkle consistency proofs to define trust or authorship semantics. Implementations MAY use hash chains, Merkle trees, or similar structures **within Layer-0** to optimize synchronization and detect divergence among authenticated peers, but such mechanisms:

- are performance- and convergence-oriented, and
- MUST NOT be interpreted as providing identity, authorship, or trust semantics.

All trust decisions remain grounded in signed log evidence and Layer-3 evaluation.

## **5.6 Security Properties at a Glance (Informative)**

The log-anchored model provides the following high-level properties:

- **Verifiable provenance:** a replica can reconstruct the evidence trail supporting identity bindings, certificate use, endorsements, and key lifecycle interpretation from immutable log history.
- **Replay robustness:** trust-relevant events are interpreted against historical log state rather than mutable external conditions, reducing the opportunity to reinterpret evidence under a different trust state.
- **Offline-capable verification:** replicas can evaluate trust without requiring continuous connectivity to external services.

These properties are achieved through the normative requirements specified in the remainder of this document.

## **5.7 Summary (Informative)**

ASCP’s trust architecture replaces online validation and mutable authority with **durable, replayable trust grounded in immutable history**. This document defines the Layer-3 constructs and deterministic evaluation rules that produce replica-local trust conclusions from Channel log evidence, while preserving strict separation from Channel codec behavior (Layer-1) and log synchronization (Layer-0).

# **6. ASCP Identity Model Overview**

This section provides a **conceptual overview** of the ASCP identity model. It explains how identities relate to certificates, cryptographic keys, and trust anchors, and why ASCP represents these elements as immutable, log-anchored Artipoints.

This section is **informational**. It establishes the mental model required to interpret the **normative** Trust and Identity constructs defined in Section 7 and related lifecycle mechanisms defined later in this document.

## **6.1 Identity as an Addressing Construct**

In ASCP, an *Identity* represents a participant—human, autonomous agent, or system component—that may author Artipoints and participate in collaborative structures. Identity functions as a **semantic addressing construct** rather than a credential, account, or permission bearer.

An identity provides:

- A **stable, long-lived identifier** for a participant
- A semantic anchor for authorship attribution
- A reference target for governance and role evaluation (defined in the Governance specification)

An Identity Artipoint does **not** contain cryptographic key material and does **not** grant authorization. Identity presence alone does not confer authorship permission, channel membership, or governance authority. All such determinations are made by governance and channel semantics outside the trust layer.

Instead, an Identity Artipoint *references* one or more Certificate Artipoints via a `certificate::kid` attribute, indicating which certificates are currently bound to the identity for cryptographic operations. This indirection allows identities to remain stable while keys rotate over time.

Because Identity Artipoints are immutable and recorded in append-only logs, identity bindings are:

- auditable,
- replayable,
- verifiable against historical trust state, and
- consistently interpretable across replicas.

## **6.2 Separation of Identity, Keys, Certificates, and Trust Anchors**

ASCP intentionally separates identity concerns into distinct constructs, each with a well-defined role.

### **Identity**

A durable, semantic reference that names a participant. Identity Artipoints contain **no cryptographic material** and serve as stable anchors for authorship and governance.

### **Certificate**

A Security Construct Artipoint that publishes a public key and declares its intended cryptographic purposes. Certificates carry the **operational cryptography** used for signing, authentication, and key agreement.

### **Trust Anchors (RootCA)**

A distinguished Certificate Artipoint introduced during bootstrap that anchors Replica-level trust. The RootCA provides a stable reference point for onboarding verification and optional cross-instance trust relationships.

### **Endorsements and Attestations**

Optional, immutable annotations that attach external evidence—such as PKI signatures, OIDC claims, DID assertions, or TSA timestamps—to identities or certificates. Endorsements strengthen provenance but do not alter identity semantics or governance meaning.

This separation enables:

- key rotation without identity disruption,
- parallel certificates bound to a single identity,
- independent evaluation of external evidence, and
- deterministic reconstruction of historical trust decisions.

### **6.2.1 Where Trust and Identity Material Lives (Quick Mapping)**

To reduce ambiguity about “where a verifier should look” for specific classes of trust and identity information, the following table summarizes the primary Layer-3 constructs and what each is responsible for carrying.

| **Construct**             | **Role in the model**                                                            | **Primary contents / bindings**                                                                                                            | **How other constructs reference it**                                                                                                                   |
| ------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Identity Artipoint**    | Durable addressing reference for a participant (human/agent/system)              | **No key material.** Indicates active key bindings by referencing one or more Certificate Artipoints via `certificate::kid`.               | Referenced as the **Author** of Artipoint Records; referenced by Keyframes via `envelope::<recipient>` attribute naming (recipient identity UUID).      |
| **Certificate Artipoint** | Publishes operational cryptography and cryptographic intent                      | **JWK public key** (payload), declared `purpose::*`, optional `endorsement::*`, optional `recovery_envelope`.                              | Referenced by Identity Artipoints (`certificate::kid`); referenced by JOSE headers (e.g., `kid`) when used for signing or key agreement.                |
| **Keyframe Artipoint**    | Defines a channel cryptographic epoch and distributes per-recipient key material | Carries per-recipient **Channel Key Envelopes (CKEs)** as `envelope::<recipient-identity-uuid> := "<JWE>"` attributes (one per recipient). | Referenced by Channels (e.g., via `keyframe::kid`, as defined elsewhere); evaluated at Layer-3 to provision cryptographic consequences to lower layers. |
| **RootCA Artipoint**      | Instance trust anchor                                                            | Distinguished certificate-like Security Construct that anchors certificate acceptance and onboarding provenance for the ASCP instance.     | Introduced via bootstrap history; used as the trust root for instance-level verification and optional external anchoring evidence.                      |

## **6.3 Why Artipoint-Based Identities**

Representing identities, certificates, and trust evidence as Artipoints provides several architectural advantages.

### **1. Immutability by construction**

All identity-related facts are append-only. New statements extend history rather than overwriting it, producing a complete and auditable provenance chain.

### **2. Log-anchored, replayable trust**

All information required for verification—keys, bindings, endorsements, rotation history—is captured in the log. Any replica can evaluate trust without relying on real-time external services.

### **3. First-class composability**

Identities participate in the same directed acyclic graph as other ASCP constructs, allowing identity, governance, and collaborative structures to interoperate without special-case mechanisms.

### **4. Decoupled lifecycle events**

Because identities and keys are distinct:

- keys can rotate independently,
- endorsements can be added or replaced without rewriting identity data, and
- recovery material can be attached or updated without altering authorship history.

### **5. Symmetry across humans and agents**

Human users and autonomous agents are represented identically at the trust layer: durable identifiers bound to cryptographic keys with optional external evidence. This symmetry enables uniform trust evaluation across mixed human–AI collaboration.

## **6.4 Certificate Lifecycle (Conceptual Model)**

ASCP employs a simple certificate lifecycle grounded in immutable history. The following describes the *shape* of that lifecycle; normative rules are defined in Section 7.

1. **Key generation:** A participant generates a cryptographic keypair locally or in secure hardware.
2. **Certificate publication:** The public key is published as a Certificate Artipoint, optionally including purpose declarations, recovery material, and endorsements.
3. **Identity binding:** The Identity Artipoint references the active certificate via `certificate::kid`, establishing a verifiable binding.
4. **Rotation:** When a key is replaced, a new Certificate Artipoint is published and the identity’s binding is updated. Historical signatures remain valid because verification uses log-time trust.
5. **Recovery (optional):** If recovery material is present, a participant may restore access to a private key without exposing it to ASCP services.
6. **External anchoring (optional):** Endorsements from PKI, OIDC, DID, or TSA systems may be added at any time to strengthen provenance.

These steps are **illustrative**, not procedural mandates.

## **6.5 Common Implementation Model (Informative)**

A common—but not universal—deployment model uses **locally generated keys** under participant control.

In this model, each participant generates a keypair, publishes the public key in a (typically self-signed) Certificate Artipoint, and uses the corresponding private key to produce signature evidence on authored payloads. Because certificate bindings and related evidence are committed to immutable log history, historical signatures can be evaluated using **log-time** state without requiring online revocation services.

Implementations typically store private keys using platform-appropriate secure storage. Key storage and device protection mechanisms are deployment-specific and out of scope for this specification.

Deployments MAY additionally introduce stronger anchoring by having participant certificates endorsed or signed under a RootCA policy. Other deployments MAY rely on self-signing plus external endorsements. This specification supports both patterns.

Additional informational context regarding deployment models and their relationship to the ASCP trust model is provided in Appendix B.

## **6.6 Relationship to Normative Sections**

This section provides a **conceptual overview** of ASCP identities, certificates, and trust anchors, including their architectural rationale, lifecycle concepts, and relationship to log-anchored trust. Section 6 establishes the conceptual foundation necessary for understanding the normative requirements defined in subsequent sections.

The **normative structure, validation rules, and lifecycle constraints** for Identity Artipoints, Certificate Artipoints, RootCA Artipoints, endorsements, and recovery mechanisms are specified in:

- **Section 7** — Security Construct Artipoints
- **Section 8** — Security Construct Attributes
- **Section 9** — Identity Claim Bundles
- **Section 10** — Keyframe Rotation and Cryptographic Continuity
- **Section 11** — Key Escrow and Recovery Strategy
- **Section 12** — Channel Key Envelopes
- **Section 13** — Provenance of the ASCP RootCA

# 7. Security Construct Artipoints

This section defines the **normative Security Construct Artipoints** used by ASCP to represent identity, cryptographic keys, trust anchors, and channel cryptographic state. All constructs defined in this section are **Layer-3 semantic constructs**, represented using the Layer-2 Artipoint Grammar and materialized as immutable, signed Artipoint Records in ASCP logs.

Each Security Construct Artipoint defined below is a **peer construct**. All follow the same structural pattern and are evaluated under the same log-anchored trust model.

All Security Construct Artipoints share the following common normative requirements:

1. They **MUST** be immutable Artipoints conforming to the ASCP Artipoint Grammar.
2. They **MUST** be sufficient for deterministic reconstruction of trust state at any historical time (“log-time trust”).
3. They **MUST** be self-contained, with all cryptographic material either embedded or resolvable via immutable references within the same log.
4. They **MUST NOT** encode governance semantics, authorization rules, or policy decisions, except where explicitly defined by the ASCP Governance & Access Control specification.

## 7.1 Identity Artipoint (Normative)

### 7.1.1 Definition and Purpose

An **Identity Artipoint** declares the existence of a participant—human, autonomous agent, or system component—and provides a durable addressing reference for that participant.

An Identity Artipoint:

- Represents *who* may author Artipoints
- Contains **no cryptographic key material**
- Binds to one or more Certificate Artipoints via indirection

The UUID of an Identity Artipoint is the stable identifier used throughout ASCP to reference that participant.

Identity Artipoints establish authorship provenance but do not grant authorization or permissions.

### 7.1.2 Canonical Form

```c
[uuid, author, timestamp,
  ["identity", <label>, <payload>]
]
```

### 7.1.3 Field Requirements

- **type**: MUST be `"identity"`
- **label**: MAY be empty; SHOULD be human-readable
- **payload**: MUST be present; MAY be a URI or descriptive string

### 7.1.4 Required Attributes

- `certificate::kid` — references the UUID(s) of active Certificate Artipoints bound to this identity

### 7.1.5 Optional Attributes

- `type`: `"human" | "agent" | "system"`
- `org`: informational organization string

These attributes are informational only.

### 7.1.6 Key Binding and Rotation

Identity Artipoints bind active signing keys by updating `certificate::kid`. Historical bindings remain valid for log-time verification.

#### Proof of Possession for Third-Party Published Certificates

If a Certificate Artipoint is authored by an identity other than the Identity Artipoint that binds to it via `certificate::kid`, then verifiers **MUST NOT** treat that certificate binding as valid unless the binding Identity demonstrates **Proof of Possession (PoP)** of the corresponding private key at **log-time**.

A binding Identity minimally demonstrates PoP by authoring at least one **PoP Assertion** that:

1. is an **Artipoint annotation expression** targeting the Certificate Artipoint (i.e., an annotation *onto* that Certificate Artipoint), and
2. is signed using the private key corresponding to the Certificate Artipoint’s published JWK (e.g., via the JOSE `kid` / JWK thumbprint profile), and
3. unambiguously binds the signing act to that specific Certificate Artipoint (the annotation target reference satisfies this requirement; implementations MAY additionally include an explicit `kid` / thumbprint reference for defense-in-depth).

Verifiers **MUST** treat the `certificate::kid` binding as effective only from the log position at which a valid PoP Assertion exists (i.e., the binding is **pending** until PoP is observed).

ASCP instance policy **MAY** require stronger PoP, including but not limited to: (a) a self-signed assertion over the certificate thumbprint, or (b) PoP as part of an **Identity Claim Bundle (ICB)**.

## 7.2 Certificate Artipoint (Normative)

### 7.2.1 Definition and Purpose

A **Certificate Artipoint** publishes a public cryptographic key and declares its cryptographic purposes, endorsements, and optional recovery material.

Certificates are the canonical source of public keys used for:

- Artipoint authorship
- Authentication
- Key agreement

### 7.2.2 Canonical Form

```c
[uuid, author, timestamp,
  ["certificate", <label>, json:{ <JWK> }]
]
```

### 7.2.3 Field Requirements

- **type**: MUST be `"certificate"`
- **label**: MAY be empty
- **payload**: MUST contain a valid JWK (RFC 7517)

The `author` of a Certificate Artipoint MUST be the UUID of the RootCA or another Identity Artipoint with traceable provenance back to the RootCA.

If a Certificate Artipoint is authored by an identity other than the Identity Artipoint that binds to it via certificate::kid, verifiers MUST apply the Proof of Possession (PoP) requirements in **Section 7.1.6** before treating that binding as valid.

### 7.2.4 Purpose Semantics

Certificates MAY declare one or more `purpose::*` attributes. A key MUST NOT be used for a purpose not declared.

### 7.2.5 Endorsements

Certificates MAY carry endorsement attributes providing external attestations. Endorsements strengthen provenance but do not alter certificate semantics.

### 7.2.6 Recovery Envelopes

Certificates MAY include `recovery_envelope` attributes enabling secure key recovery.

## 7.3 RootCA Artipoint (Normative)

### 7.3.1 Definition and Purpose

A **RootCA Artipoint** is a distinguished Security Construct that serves as the trust anchor for an ASCP instance.

A RootCA:

- Acts as a certificate (contains key material)
- Acts as an author (may sign Artipoints directly)
- Does **not** require an Identity Artipoint pointing to it

The RootCA represents an implied, unnamed **semantic authority** anchoring the instance. It is a trust anchor for verification and provenance, not a participant identity and not a governance actor unless explicitly referenced by articulated governance rules.

### 7.3.2 Canonical Form

```c
[uuid, author, timestamp,
  ["rootca", "Root Certificate", json:{ <JWK> }]
]
```

### 7.3.3 Field Requirements

- **type**: MUST be `"rootca"`
- **label**: MUST be `"Root Certificate"`
- **payload**: MUST contain a valid JWK

### 7.3.4 Provenance and Endorsements

RootCA Artipoints SHOULD include endorsements (PKI, TSA, DID) to strengthen external verifiability.

### 7.3.5 Rotation Semantics

RootCAs are never revoked; instead, supersession occurs by introducing a new RootCA Artipoint with explicit cross-endorsement from the previous root. This mechanism handles both compromise and deprecation through governance interpretation of the supersession chain, without requiring retroactive invalidation of historical signatures.

## 7.4 Keyframe Artipoint (Normative)

### 7.4.1 Definition and Purpose

A **Keyframe Artipoint** defines the cryptographic epoch for an ASCP Channel.

Keyframes:

- Encapsulate cryptographic configuration
- Distribute encrypted channel key material
- Are referenced by Channels via `keyframe::kid`

Keyframes define *what cryptographic state exists*, not how it evolves over time.

Keyframe semantics are evaluated at Layer-3; key material contained in envelope formats are provisioned opaquely into lower layers

### 7.4.2 Canonical Form

```c
[uuid, author, timestamp,
  ["keyframe", <label>, <urn>]
] supports {channel-uuid}
```

### 7.4.3 Required Attributes

- `version`
- `payload_cipher`
- `message_signing`
- `channel_access_alg`

### 7.4.4 Envelope Attributes

Keyframes MUST distribute encrypted key material using `envelope::*` attributes, one per recipient.

### 7.4.5 Validation Rules

Verifiers MUST ensure:

1. Envelopes target valid recipients
2. Recipient certificates declare `purpose::keyAgreement`
3. Keyframe integrity is preserved

Verifiers MUST ensure that Keyframe Artipoints are evaluated according to the lifecycle and non-retroactivity semantics defined in Section 10.

# **8. Security Construct Attributes**

This section defines normative attribute families that extend the semantic meaning of Security Construct Artipoints defined in Section 7. Attributes are immutable, additive, and non-destructive: they attach additional semantic information to an Artipoint without mutating or invalidating its original meaning.

All attributes defined herein are evaluated at Layer-3 and expressed using the Layer-2 grammar. This section provides normative definitions for attribute names, value encodings, and schemas. Construction, decoding, and validation procedures for Recovery Envelopes and Channel Key Envelopes are specified in Sections 11 and 12, respectively.

This specification defines Layer-0 and Layer-1 mechanisms only where necessary to ensure interoperable generation and decoding of Layer-3 security material.

The following attribute families are defined:

1. `endorsement::*`
2. `purpose::*`
3. `envelope::*`
4. `recovery_envelope`
5. `certificate::*`

## **8.1 Endorsement Attribute Family**

### **8.1.1 Overview**

An **endorsement attribute** binds externally verifiable evidence to a **Certificate Artipoint** or **Identity Artipoint**. Endorsements strengthen provenance by recording evidence issued by external systems such as PKI, OIDC identity providers, decentralized identifier controllers, or time-stamping authorities.

Endorsements are **additive annotations**. They:

- MUST NOT revoke or modify certificate semantics
- MUST NOT grant governance authority
- MUST NOT alter identity meaning

An endorsement records *evidence*, not trust.

Endorsements are attached as:

```json
( endorsement::<mechanism> + json:{ ... } )
```

### **8.1.2 Attestation vs. Endorsement (Normative Distinction)**

ASCP explicitly distinguishes between:

- **Attestation** — a factual claim (e.g., “this key existed before time T” or “this key corresponds to account X”)
- **Endorsement** — a cryptographically verifiable binding of that attestation to a specific Artipoint at a specific time

ASCP records **endorsements**, not attestations.

This distinction prevents semantic overload common in PKI and DID systems and ensures that trust evaluation remains **explicit, composable, and log-anchored**. Endorsements do not assert truth; they provide verifiable evidence that verifiers may evaluate independently.

### **8.1.3 Endorsement Schema**

All endorsement values **MUST** conform to the following JSON structure. While inspired by the W3C Verifiable Credentials data model, ASCP endorsements use simple JSON with JOSE-based cryptographic evidence and do **not** rely on JSON-LD or VC processing rules.

```json
{
  "schema": "ascp.endorsement.v1",
  "subject": "cert" | "identity",
  "attestation": "<attestation-type>",
  "thumbprint": "sha256:<hex>",
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

**Required fields**

- schema
- attestation
- thumbprint
- issuer.mechanism
- issued\_at
- evidence

**Optional fields**

- subject
- issuer.hint
- tsa

### **Thumbprint Requirement (Normative)**

The thumbprint value **MUST** equal the RFC 7638 JWK thumbprint of the key material being endorsed.

### **8.1.4 Attestation Types (Normative)**

| **Type**      | **Meaning**                                         | **Mechanisms**            |
| ------------- | --------------------------------------------------- | ------------------------- |
| fingerprint   | Endorser vouches for correctness of key fingerprint | jws-x5c, jws-kid, did-jws |
| issuance-time | Endorser vouches key existed at or before time T    | tsa-rfc3161               |
| id-binding    | Endorser binds key to an external identifier        | oidc-icb, did-jws         |

### **8.1.5 Supported Endorsement Mechanisms**

- **jws-x5c** — PKI certificate endorsement using X.509 chains
- **jws-kid** — ASCP-internal endorsement using another certificate
- **oidc-icb** — Identity Claim Bundle binding an external IdP account
- **did-jws** — DID controller signature
- **tsa-rfc3161** — RFC 3161 Time-Stamp Token

Unknown mechanisms **MUST be preserved** but **MAY be ignored** by verifiers.

### **8.1.6 Endorsement Validation Rules (Normative)**

A verifier MUST:

1. Verify thumbprint equals the endorsed key’s RFC 7638 thumbprint
2. Validate mechanism-specific evidence
3. Apply **log-time trust**, evaluating evidence using the trust state at issued\_at
4. Record the endorsement result as *valid*, *invalid*, or *indeterminate*

Endorsements **MUST NOT** alter the semantic meaning of Identity or Certificate Artipoints.

## **8.2 Purpose Attribute Family**

### **8.2.1 Overview**

Purpose attributes declare **first-party cryptographic intent**. They specify which cryptographic operations a certificate’s key **MAY** be used for.

Purpose attributes authorize **cryptographic use only**; they do **not** grant governance authority or access rights.

Each purpose attribute is expressed as:

```json
( purpose::<type> + "jwk#<thumbprint>" )
```

### **8.2.2 Defined Purpose Types (Normative)**

| **Purpose**           | **Meaning**                           | **Required For**                |
| --------------------- | ------------------------------------- | ------------------------------- |
| purpose::assert       | Artipoint authorship and endorsements | All signed Artipoints           |
| purpose::auth         | Session authentication                | ALSP and channel authentication |
| purpose::keyAgreement | Key agreement and encryption          | Keyframe and recovery envelopes |

## **8.2.3 Validation Rules (Normative)**

A key **MUST NOT** be used for any cryptographic operation unless the corresponding purpose attribute is present and references the correct thumbprint.

Channel Codec operations (Layer-1 signing/encryption) **MUST rely exclusively** on purpose attributes of the active **Certificate Artipoint(s)** selected by Layer-3 trust evaluation, and **MUST NOT** infer cryptographic intent from identity metadata, governance attributes, labels, or other non-purpose fields.

Layer-1 MUST NOT interpret Keyframe or Certificate semantics directly; it MUST consume only the cryptographic consequences provisioned by Layer-3 evaluation (e.g., selected keys, recipient sets, and epoch state).

## **8.3 Envelope Attribute Family**

### **8.3.1 Overview**

Envelope attributes carry **encrypted cryptographic material** associated with **Keyframe Artipoints**. They define payload schemas carried by attributes; they do **not** define lifecycle or rotation semantics.

Envelope attributes are expressed as:

```json
envelope::<recipient> := "<JWE compact serialization>"
```

where the value of the attribute MUST be a JOSE JWE in compact serialization form as specified in Section 12.

The JWE payload MUST be encrypted for the intended recipient using the recipient’s active **Certificate Artipoint** public key that is authorized for key agreement (i.e., purpose::keyAgreement is present and valid under Layer-3 trust evaluation).

The JWE kid header field MUST identify the specific recipient Certificate Artipoint key being targeted (e.g., by certificate thumbprint or certificate identifier as defined for purpose::keyAgreement).

### **8.3.2 Channel Key Envelope (CKE) Schema (Normative)**

This schema defines the initial reference envelope format; future versions MAY introduce additional envelope types. The decrypted JWE payload **MUST** contain the following JSON structure:

```json
{
  "type": "channel-key-envelope",
  "version": "1.0",
  "aes_key_jwk": { ...JWK object holding Channel Symmetric Key... },
  "auth_key_jwk": { ...JWK object holding CAK Private key... },
  "created": "2025-07-26T21:13:00Z",
  "valid_from": "2025-07-26T21:13:00Z",
  "replaces": "ascp:keyframe:<uuid>",
  "rotation_interval_days": 365
}
```

- `type`: Identifies this CKE structure
- `version`: Version of this envelope and key bundle
- `aes_key_jwk`: JWK object containing the channel encryption key
- `auth_key_jwk`: JWK object containing the CAK private key
- `created`: Timestamp when this envelope was created
- `valid_from`: Start time for when this key should be used (optional). When specified, this field is informational only and MUST NOT be used to determine Keyframe activation; activation semantics are defined exclusively by articulated state as specified in Section 10.
- `replaces`: References the UUID of a prior Keyframe being superseded by this envelope (optional). This field aids provenance tracing but is informational only and MUST NOT be interpreted as defining rotation order, activation, or lifecycle behavior.
- `rotation_interval_days`: Recommended rotation interval in days (optional)

Each key in the envelope (`aes_key_jwk` and `auth_key_jwk`) is a JWK containing the channel symmetric encryption key and the Channel Access Key (CAK) private key, respectively.

Layer-3 provisions extracted key material to lower layers. Lower layers **MUST NOT** parse or interpret this structure directly.

Key material lifecycle semantics are defined in Section 10 while full JOSE JWE encoding and validation details are specified in Section 12.

## **8.4 recovery\_envelope Attribute**

### **8.4.1 Overview**

The recovery\_envelope attribute enables secure recovery of a certificate’s private key using **double encryption**, without exposing secret material to ASCP services.

### **8.4.2 Schema (Normative)**

The value of the `recovery_envelope` attribute is a JSON structure as follows:

```json
{
  "type": "recovery-envelope",
  "version": "1.0",
  "recovery_cert": "ascp:cert:<uuid>",
  "user_key_jwe": "<user-key-envelope>",
  "protection": ["recovery-key", "password"],
  "recovery_purpose": "device-migration | local-backup",
  "rotation_id": "v1",
  "created": "<ISO-8601 UTC>"
}
```

- `type` — Identifies this as a recovery envelope
- `version` — Schema version for forward compatibility
- `recovery_cert` — Reference to the recovery public key (Key B) used for outer encryption. See Section 11.
- `user_key_jwe` — The value string holds the  `user-key-envelope` per Section 11 which is the double-encrypted identity private key written in JWE compact serialization form.
- `protection` — Array indicating the protection factors required for decryption
- `rotation_id` — Human-readable identifier for tracking key rotations
- `recovery_purpose` (optional) — Intended use case for this recovery envelope
- `created` — Timestamp when the recovery envelope was generated

See Section 11 for all details around constructing the recovery envelope and in particular the construction and decoding process of the `user-key-envelope` populating the `user_key_jwe` JSON field.

## 8.5 Certificate Attributes

### 8.5.1 Purpose and Scope

The `certificate::*` attribute family provides semantic modifiers specific to **certificate binding and interpretation**.

This family is intentionally extensible. At present, only one attribute is defined.

### 8.5.2 Value Encoding

The `certificate::kid` attribute declares the key identifier associated with a Certificate Artipoint.

Its semantics are modeled directly after JOSE `kid` values and are interpreted consistently with **ASCP Channels §6.4**. This cross-reference is strictly for interoperability of JOSE header `kid` values across ASCP documents; it does not delegate any Trust & Identity evaluation semantics, key generation, or key-wrapping behavior to Layer-1.

### 8.5.3 Semantics

The `certificate::kid` value:

- MUST uniquely identify the certificate within the ASCP instance,
- MUST be stable for the lifetime of the certificate,
- MUST align with JOSE header usage for signing and encryption.

### 8.5.4 Validation Rules

Verifiers MUST resolve `certificate::kid` deterministically against historical log state.

## **8.6 Key Usage Evaluation Summary (Normative)**

This section defines attribute families used to evaluate whether a given Certificate Artipoint is acceptable for a specific cryptographic use at a given **log-time**. Implementations MUST evaluate key usage deterministically from log evidence plus explicitly configured local policy inputs.

Given:

- `cert_kid`: the UUID of a candidate Certificate Artipoint
- `required_purpose`: one of `purpose::assert`, `purpose::auth`, `purpose::keyAgreement`, etc.
- `t`: a log-time instant (i.e., evaluation point within an append-only history)
- `policy`: replica-local trust policy (including optional endorsement requirements)

A verifier MUST evaluate key usage as follows:

1. **Verify certificate validity at log-time**
   - Resolve `cert_kid` to a Certificate Artipoint present in the Coordination Log at or before `t`.
   - Apply any lifecycle constraints relevant to certificate applicability at `t` (e.g., non-retroactivity, supersession handling, or instance policy constraints defined elsewhere in this document).
   - If the certificate is not applicable at `t`, the verifier MUST reject it for the candidate use.
2. **Verify purpose match**
   - The Certificate Artipoint MUST declare `required_purpose` as a `purpose::*` attribute.
   - If `required_purpose` is not declared, the verifier MUST reject the certificate for that use.
3. **Optionally evaluate endorsement strength**
   - If `policy` requires endorsements for `required_purpose`, the verifier MUST evaluate endorsement presence and validity at `t` (including any external attestation verification procedures required by `policy`).
   - If endorsement requirements are not met, the verifier MUST reject the certificate for that use.

If all required checks succeed, the verifier MAY treat the certificate as an acceptable public key for `required_purpose` at log-time `t`.

## **8.7 Example Attribute Annotations (Informative)**

### **8.7.1 PKI Endorsement of Certificate Fingerprint**

```
[ endorse-uuid, author-uuid, 2025-08-13T14:00:00Z,
    cert-uuid .
    ( endorsement::jws-x5c + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"fingerprint",
        "thumbprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"jws-x5c","hint":"ca.example.org"},
        "evidence":{
            "jws":"<compact JWS>",
            "chain":["<leaf>","<int>","<root>"] },
        "issued_at":"2025-08-13T14:01:22Z" }
    )
];
```

### **8.7.2 OIDC Identity Binding**

```
[ verify-uuid, author-uuid, 2025-08-08T14:30:25Z,
    cert-uuid .
    ( endorsement::oidc-icb + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"id-binding",
        "thumbprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"oidc-icb","hint":"accounts.google.com"},
        "evidence":{ "icb":"<IdentityClaimBundle JWS>" },
        "issued_at":"2025-08-08T14:30:25Z" }
    )
];
```

### **8.7.3 DID-based Endorsement**

```
[ did-endorse-uuid, author-uuid, 2025-08-20T10:00:00Z,
    cert-uuid .
    ( endorsement::did-jws + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"fingerprint",
        "thumbprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"did-jws","hint":"did:key:z6Mk..."},
        "evidence":{"did":"did:key:z6Mk...","jws":"<compact JWS>"},
        "issued_at":"2025-08-20T10:00:00Z" }
    )
];
```

### **8.7.4 TSA Time Attestation**

```
[ tsa-uuid, author-uuid, 2025-08-13T14:02:00Z,
    cert-uuid .
    ( endorsement::tsa-rfc3161 + json:{
        "schema":"ascp.endorsement.v1",
        "attestation":"issuance-time",
        "thumbprint":"sha256:abcd1234...",
        "issuer":{"mechanism":"tsa-rfc3161","hint":"tsa.example.org"},
        "evidence":{"tst":"<base64-der-TST>"},
        "issued_at":"2025-08-13T14:02:00Z" }
    )
];
```

# **9. Identity Claim Bundles (ICB)**

This section specifies **Identity Claim Bundles (ICB)**, which provide verifiable evidence binding an ASCP public key to an externally authenticated identity. An ICB combines: (1) a nonce-bound proof that the claimant controls the corresponding private key, and (2) an identity assertion issued by an external Identity Provider (IdP). The result is a compact, portable JWS object suitable for durable, log-anchored verification.

ICBs are used as normative evidence for constructing **endorsement::oidc-icb** attributes on Certificate Artipoints (Section 8). While OpenID Connect (OIDC) is expected to be a common mechanism, the ICB design is IdP-agnostic and may be used with any JWT-based identity assertion that satisfies the requirements in this section.

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
   - A Signed Key Package nonce MUST NOT be reused.
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

## **9.7.5 Token Acquisition**

This specification does not define how external ID Tokens are acquired. It defines only how externally issued claims MAY be bound to ASCP identities and certificates via endorsement evidence.

Deployments that use OIDC **SHOULD** use Authorization Code + PKCE and **SHOULD** avoid front-channel delivery of tokens; however, token acquisition and refresh mechanisms are out of scope for ASCP Trust & Identity and are specified by the relevant external identity provider and OIDC profiles.

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
      "thumbprint":"sha256:abcd...",
      "issuer":{"mechanism":"oidc-icb","hint":"accounts.google.com"},
      "evidence":{
        "icb":"<IdentityClaimBundle JWS>"
      },
      "issued_at":"2025-08-08T14:30:25Z"
    }
  )
];
```

# **10. Keyframe Rotation and Cryptographic Continuity**

## **10.1 Purpose and Scope**

This section defines the **Layer-3 semantic model** for Keyframe rotation and cryptographic continuity in ASCP.

Keyframe rotation establishes **forward-only cryptographic epochs** for a Channel. Rotation is expressed as **articulated state**, not mutation. It defines *when* new cryptographic material becomes active, *how* historical material remains valid, and *what* cryptographic consequences MUST be provisioned to lower layers.

This section defines **semantic requirements and invariants** only. It does **not** prescribe cryptographic algorithms, JOSE serialization formats, envelope schemas, transport behavior, or provisioning APIs. Those concerns are defined by lower layers and consume the evaluated results of Layer-3 semantics.

Keyframe rotation is a **semantic transition**, not a procedural command. All rotation events are expressed as immutable Artipoint Records, evaluated deterministically at Layer-3 using log-anchored trust.

## **10.2 Keyframes as Cryptographic Epochs**

Each **Keyframe Artipoint** defines a discrete cryptographic epoch for an ASCP Channel.

The following invariants apply:

1. **Forward-only progression**  
   Keyframes progress monotonically through articulated history.
2. **Supersession for future use only**  
   A new Keyframe supersedes prior Keyframes for **future encryption only** and MUST NOT retroactively alter previously produced artifacts.
3. **Non-retroactivity**  
   No rotation event invalidates historical signatures, envelopes, or trust conclusions.
4. **Historical validity**  
   Envelopes created under an earlier Keyframe remain valid, verifiable, and (where applicable) decryptable using the cryptographic material active at the time of their creation.
5. **Deterministic evaluation**  
   Given identical articulated history and identical local policy inputs, all conforming Layer-3 evaluators MUST derive the same active Keyframe and historical Keyframe set.
6. **Semantic ownership**  
   The meaning, activation, and consequences of Keyframes are defined exclusively at Layer-3. Lower layers consume only evaluated cryptographic results.

## **10.3 Rotation as an Articulated State Transition**

Keyframe rotation occurs through **articulated state transitions**, not in-place mutation.

A rotation event consists of:

1. Creating a **new Keyframe Artipoint**, and
2. Articulating its activation for future use.

The designation of an active Keyframe MUST be derivable **solely from immutable articulated state recorded in the Channel log** and MUST NOT rely on out-of-band configuration, mutable local state, or procedural triggers.

Layer-3 evaluation MUST treat rotation as **supersession**, not replacement or deletion of prior state.

Historical artifacts remain valid. At no point does rotation invalidate existing signatures, envelopes, or log records.

## **10.4 Cryptographic Continuity Guarantees**

Layer-3 evaluation MUST ensure the following continuity guarantees across Keyframe rotations.

### **10.4.1 Signature Verification Continuity**

Rotation of Channel cryptographic material MUST NOT affect the ability to verify historical signatures. All Artipoint Records remain verifiable as long as the corresponding certificate material is available.

### **10.4.2 Decryption Continuity**

For encrypted payloads:

- Envelopes encrypted under a given Keyframe MUST remain decryptable using the symmetric key material associated with that Keyframe.
- Rotation MUST NOT require re-encryption of historical envelopes.
- Loss of historical key material MAY affect decryptability but MUST NOT affect log integrity or signature validity.

### **10.4.3 No Retroactive Revocation**

Keyframe rotation does **not** retroactively revoke access to previously decryptable content.

Governance-level revocation or access decisions MAY restrict future use but do not rewrite cryptographic history or invalidate log-time trust conclusions.

## **10.5 Classes of Rotation Events**

The following rotation scenarios are distinguished for semantic clarity. Implementations MAY support additional rotation patterns provided all invariants in Sections 10.2–10.4 are preserved.

### **10.5.1 Channel-Wide Cryptographic Rotation**

A Channel-wide rotation introduces a new cryptographic epoch by:

1. Generating new symmetric Channel encryption key material,
2. Generating new replication authentication material if required,
3. Issuing a new Keyframe Artipoint,
4. Distributing new wrapped key material to authorized recipients, and
5. Activating the new Keyframe for future encryption.

After activation, only the new Keyframe MAY be used for encrypting new payloads.

### **10.5.2 Replication Authentication Rotation**

Rotation of replication authentication credentials follows the same semantic model as Channel-wide rotation but affects only replication authentication material.

Historical credentials MAY remain valid for a bounded transition period, subject to governance policy.

### **10.5.3 Recipient Identity Key Rotation**

If an individual participant rotates their identity key:

- Only that participant’s wrapped key material MUST be re-issued.
- A Channel-wide Keyframe rotation is NOT required unless explicitly intended.
- The active Keyframe identifier MAY remain unchanged, provided cryptographic continuity and recipient-specific envelope correctness are preserved.

### **10.5.4 Policy-Driven or Time-Based Rotation**

Rotation triggers derived from validity windows, rotation intervals, or governance policy are evaluated exclusively at Layer-3. They MAY require issuance of new Keyframes but MUST NOT introduce procedural dependencies in lower layers.

## **10.6 Active Keyframe Determination**

At any point in the evaluated history, Layer-3 MUST deterministically derive:

- The **active Keyframe** for new encryption operations, and
- The **set of historical Keyframes** required for verification and decryption.

Layer-3 implementations MUST NOT select an active Keyframe based on receipt order, wall-clock time, or local mutable state. Selection MUST be based solely on articulated history and semantic evaluation.

## **10.7 Layer-3 to Layer-1 Provisioning Contract**

Layer-3 evaluation determines whether certificate purpose constraints are satisfied; Layer-1 enforces those constraints mechanically using provisioned cryptographic material.

Following evaluation, Layer-3 MUST provision Layer-1 with **cryptographic consequences only**, including:

- The active symmetric encryption key indexed by its Keyframe identifier,
- All historical symmetric keys required for decryption,
- Replication authentication material, and
- Stable key identifiers (`kid`) corresponding to evaluated Keyframes.

Provisioned outputs are **pure data artifacts**, not executable instructions.

Layer-1 MUST NOT:

- Parse Keyframe Artipoints,
- Interpret Keyframe semantics,
- Infer activation or rotation rules,
- Decide rotation timing, or
- Apply governance logic.

Layer-1 operates solely by selecting cryptographic material based on provisioned identifiers.

## **10.8 Failure and Edge Conditions**

If cryptographic material for a historical Keyframe becomes unavailable:

- Envelopes referencing that Keyframe MAY become undecryptable.
- The immutable log remains valid and MUST NOT be altered.
- Signature verification MUST continue independently of decryption success.

If multiple rotation events are observed out of order due to replication delay, Layer-3 evaluation MUST resolve them deterministically based on articulated history, not arrival order.

## **10.9 Summary**

Keyframe rotation in ASCP:

- Establishes **forward-only cryptographic epochs**,
- Is **semantic**, not procedural,
- Preserves **cryptographic continuity**,
- Is evaluated entirely at **Layer-3**, and
- Provisions **Layer-1 deterministically** without semantic interpretation.

This model ensures long-term auditability, stable cryptographic execution, and strict separation of responsibility across the ASCP stack.

# **11. Key Escrow and Recovery Strategy**

ASCP provides a secure and auditable approach to private key recovery that builds on the log-anchored trust foundation. This mechanism enables users to recover their **identity private key** without ever exposing it in plaintext to any third party, supporting device migration, key backup, and enterprise compliance scenarios.

The recovery strategy is designed to accommodate both personal recovery workflows—where users control their own recovery credentials—and enterprise-managed recovery policies. Recovery materials remain under the control of designated parties and are never accessible to ASCP infrastructure or log observers; the mechanism is **not escrow by default**.

Key escrow and recovery is an **optional** capability. It does not affect trust evaluation of authorship except insofar as it enables key continuity across devices or recovery events.

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

This procedure defines the normative steps for constructing a `recovery_envelope` attribute during identity bootstrap or key rotation. The resulting attribute structure is defined in Section 8.4.

1. **Generate Identity Keypair (Key A)**
   - An implementation MUST generate an EC keypair (P-256 or P-384) to serve as the original identity signing key
   - The private key MUST be used to sign Artipoints and authenticate sessions
   - The public key MUST be published in a Certificate Artipoint
2. **Establish Recovery Keypair (Key B)**
   - An implementation MUST generate or register a separate EC keypair for recovery purposes
   - The recovery private key MUST be stored in a secure location distinct from the identity key (e.g., second device, hardware token, secure backup)
   - The recovery public key MUST be published in a separate Certificate Artipoint
   - The Certificate Artipoint MUST include the attribute `purpose::keyAgreement`.
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
   - An implementation MUST construct a JSON object conforming to the `recovery_envelope` schema defined in Section 8.4.
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
   - An implementation MUST parse the `recovery_envelope` attribute value as JSON conforming to the schema defined in Section 8.4.
   - The implementation MUST extract the `user_key_jwe` field value (the `user-key-envelope` string)
   - The implementation MUST extract the `recovery_cert` field value (UUID of the recovery Certificate Artipoint) which MUST be confirmed to already include the attribute `purpose::keyAgreement`.
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

# **12. Channel Key Envelopes (CKE)**

This section defines the normative construction, encoding, and decoding rules for **Channel Key Envelopes (CKEs)**.

A Channel Key Envelope is the mechanism by which **Keyframe Artipoints** distribute per-recipient wrapped key material, including:

- the Channel symmetric encryption key used for payload confidentiality, and
- the Channel Access Key (CAK) private key used for Layer-0 channel access proofs.

CKEs are **Layer-3 constructs**:

- they are generated and interpreted exclusively by Layer-3 Trust & Identity evaluation,
- they are represented using Layer-2 grammar as `envelope::*` attributes,
- and they are carried opaquely by Layer-1 as part of the Articulation Sequence / Channel Envelope codec path.

## **12.1 Scope and Non-Goals**

This section specifies:

- how a CKE plaintext is constructed (schema and required fields),
- how that plaintext is wrapped using **JWE Compact Serialization**,
- the required JOSE protected header profile for CKEs,
- permitted algorithms, and
- recipient-side decoding, validation, and provisioning.

This section does **not** specify:

- how Layer-1 encodes/decodes Channel Envelopes for Artipoint Records,
- how ALSP synchronizes or authenticates replication sessions,
- Keyframe activation/rotation semantics (see Section 10),
- authorization / membership semantics (see Governance).

## **12.2 CKE Placement and Addressing**

A CKE is carried as the value of an `envelope::<recipient>` attribute attached to a **Keyframe Artipoint**.

- The attribute name suffix `<recipient>` MUST be the UUID of the intended recipient’s **Identity Artipoint**.
- The attribute value MUST be a **JWE Compact Serialization** string containing a decryptable plaintext conforming to the schema in Section 8.3.2.

The recipient key used for JWE encryption is selected via the JWE protected header `kid`, which MUST identify the recipient’s **Certificate Artipoint** key authorized for `purpose::keyAgreement`.

## **12.3 Plaintext Schema (Normative)**

The decrypted plaintext of a Channel Key Envelope MUST be a UTF-8 encoded JSON object conforming to the **Channel Key Envelope (CKE) Schema** defined in Section 8.3.2.

For clarity, the required semantic fields are:

- `type` and `version` — schema identification
- `aes_key_jwk` — a JWK representing the Channel symmetric encryption key
- `auth_key_jwk` — a JWK representing the Channel Access Key (CAK) *private* key
- `created` — creation timestamp

Any additional fields (e.g., `valid_from`, `replaces`, `rotation_interval_days`) are informational and MUST NOT be used to determine Keyframe activation or lifecycle semantics, which are defined exclusively by articulated state (Section 10).

## **12.4 JWE Wrapping and JOSE Header Profile (Normative)**

CKEs MUST be encrypted using **JWE Compact Serialization**.

A conforming CKE JWE MUST contain exactly five base64url segments:

```asciidoc
BASE64URL(ProtectedHeader) .
BASE64URL(EncryptedKey) .
BASE64URL(IV) .
BASE64URL(Ciphertext) .
BASE64URL(Tag)
```

The BASE64URL(EncryptedKey) segment handling is algorithm-dependent:

- If `alg` is `ECDH-ES+A256KW`, the EncryptedKey segment **MUST be present and MUST NOT be empty** (it carries the CEK wrapped under the ECDH-derived KEK).
- If `alg` is `ECDH-ES`, the EncryptedKey segment **MUST be present and MUST be empty** (the CEK is derived directly; no wrapped CEK is transported).

### **12.4.1 Protected Header Requirements**

The JWE Protected Header for a CKE MUST be valid JSON and MUST include:

- `alg` — key management / key agreement algorithm
- `enc` — content encryption algorithm
- `kid` — identifies the recipient Certificate Artipoint key
- `typ` — media type identifier for ASCP CKE envelopes

The header MUST satisfy:

- `typ` MUST be `"ascp+cke"`.
- `kid` MUST be the exact `certificate::kid` value of the recipient’s Certificate Artipoint whose public key is used for the JWE operation. The `kid` value MUST conform to the JOSE `kid` encoding profile defined in **ASCP Channels §6.4**.
- The referenced Certificate Artipoint MUST declare purpose::keyAgreement and MUST be valid under Layer-3 trust evaluation for the relevant log-time.

The header MAY include:

- `cty` — implementations MAY set "application/json".

### **12.4.2 Algorithm Requirements**

Implementations MUST NOT rely on runtime negotiation. Interoperability requires a fixed permitted algorithm set.

**Content Encryption (enc)**

- Implementations MUST support: A256GCM
- A CKE specifying any other enc MUST be rejected.

**Key Management / Agreement (alg)**

- Implementations MUST support: ECDH-ES+A256KW
- Implementations MAY support: ECDH-ES

A CKE specifying an unsupported alg MUST be rejected.

## **12.5 CKE Construction Procedure (Normative)**

To construct a CKE for a given Keyframe and recipient identity:

1. **Select the recipient Certificate**
   - Resolve the recipient Identity Artipoint.
   - Select a recipient Certificate Artipoint bound to that identity that declares purpose::keyAgreement.
   - Determine the Certificate identifier to place in kid.
2. **Construct the CKE plaintext**
   - Build the JSON object per Section 8.3.2.
   - aes\_key\_jwk MUST represent the symmetric key material required by the Keyframe’s payload cipher requirements.
   - auth\_key\_jwk MUST represent the CAK private key material required by the Keyframe’s channel access algorithm.
3. **Encrypt as JWE compact**
   - Encode the plaintext JSON as UTF-8 bytes.
   - Produce a JWE Compact Serialization object using the recipient’s selected Certificate public key, with a Protected Header satisfying Section 12.4.
4. **Attach to the Keyframe Artipoint**
   - Add the attribute envelope::\<recipient-identity-uuid> := "\<JWE-compact-string>" to the Keyframe Artipoint.

The resulting Keyframe Artipoint, once articulated as an Artipoint Record, provides log-anchored, replayable distribution of recipient-specific key material.

## **12.6 Recipient Decoding and Validation (Normative)**

Upon encountering a Keyframe with `envelope::<recipient>` attributes, a recipient implementation MUST:

1. **Select the correct envelope**
   - Identify the **envelope::\<recipient-identity-uuid>** attribute matching the recipient’s Identity Artipoint UUID.
2. **Parse the JWE compact serialization**
   - Confirm it has exactly five segments.
   - Base64url-decode the Protected Header and confirm it is valid JSON.
   - Confirm the header includes `alg`, `enc`, `kid`, and `typ`.
   - Confirm `typ == "ascp+cke"`.
   - Confirm `enc` and `alg` are permitted by Section 12.4.2.
   - Enforce `EncryptedKey` segment rules:
     - If `alg == "ECDH-ES+A256KW"`, the `EncryptedKey` segment MUST NOT be empty.
     - If `alg == "ECDH-ES"`, the `EncryptedKey` segment MUST be empty.
     - Otherwise, reject.
3. **Resolve and validate the recipient certificate**
   - Resolve kid to the referenced Certificate Artipoint.
   - Confirm the Certificate Artipoint is bound to the recipient Identity Artipoint at the relevant log-time.
   - Confirm the certificate declares purpose::keyAgreement.
   - If any of these checks fail, the recipient MUST reject the envelope.
4. **Decrypt**
   - Use the recipient’s private key corresponding to the kid Certificate Artipoint to decrypt the JWE.
   - If authenticated decryption fails, the recipient MUST reject the envelope.
5. **Validate plaintext schema**
   - Parse the decrypted plaintext as JSON.
   - Confirm it conforms to the schema in Section 8.3.2.
   - Reject if required fields are absent or malformed.
6. **Provision cryptographic consequences**
   - Extract aes\_key\_jwk and provision the symmetric key material to Layer-1 for message confidentiality and decryption for the Keyframe epoch.
   - Extract auth\_key\_jwk and provision the CAK private key material to Layer-0 for channel access proofs (as required by the Channel access algorithm).
   - Provisioning MUST be treated as pure data output of Layer-3 evaluation (see Section 10.7).

## **12.7 Security Considerations (Informative)**

- **Confidentiality**: CKEs are encrypted per-recipient; observers lacking the recipient private key cannot recover channel secrets from the log.
- **Integrity**: JWE authenticated encryption provides integrity for the CKE plaintext; the enclosing Keyframe Artipoint Record signature provides log-anchored provenance.
- **Non-retroactivity**: Rotation or membership changes do not rewrite historical CKEs; historical decryptability follows Section 10 continuity rules.
- **Key compromise**: If a recipient key agreement private key is compromised, CKEs addressed to that key may be compromised; this motivates recipient certificate rotation and re-issuance of envelopes as needed.

## **12.8 Example (Informative)**

Example Protected Header:

```json
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "typ": "ascp+cke",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}
```

Example attribute placement on a Keyframe Artipoint:

```json
envelope::550e8400-e29b-41d4-a716-446655440010 := "<jwe-compact-string>"

```

# **13. Provenance of the ASCP RootCA**

The **RootCA Artipoint** is the distinguished trust anchor for an ASCP instance. This specification defines the semantic role of the RootCA Artipoint and the evidence that verifiers MAY use to establish confidence in it including:

- Internal self-signing and articulation
- Optional anchoring to a Public or Enterprise PKI certificate
- Optional independent attestation via a Time-Stamping Authority (TSA)

The goal is to ensure that **any verifier, at any time in the future**, can:

1. Establish the origin and legitimacy of the RootCA.
2. Confirm when it was created and anchored.
3. See its relationship to a trusted PKI at the time of anchoring.
4. Validate the integrity of this provenance against immutable and independent records.

## **13.1 RootCA Generation and Self-Signing**

- An ASCP instance MUST have exactly one active RootCA trust anchor at any given time for the purpose of certificate chain acceptance.
- The associated algorithm SHOULD be ECDSA P-384 or better security level.
- The keys SHOULD be created in a secure environment by the ASCP instance operator.
- The RootCA trust anchor MUST be represented as a **RootCA Artipoint** (a distinguished Certificate Artipoint) whose public key and thumbprint are immutable once articulated.
- Verifiers MUST treat acceptance of the RootCA as a trust decision rooted in immutable evidence (log-anchored trust), not as a mutable online lookup.

## **13.2 Optional Anchoring to a Public or Enterprise PKI**

Deployments MAY publish additional evidence binding the RootCA public key to external trust systems (e.g., a detached signature using an external PKI identity, transparency log inclusion, or other third-party attestations). Such evidence MAY be referenced by endorsement attributes or auxiliary artifacts, but does not change the internal semantics of the RootCA Artipoint.

This specification defines only the *evidence semantics and validation rules*. The *transport and placement* of RootCA provenance material (e.g., which channel(s) carry it, how it is discovered, or where it is located within bootstrap history) is defined in the companion **ASCP Bootstrap and Channel Discovery** specification.

### **Evidence Format (Informational)**

A typical PKI anchoring evidence set consists of:

1. A valid PKI-issued certificate for the anchoring identity (e.g., TLS/identity certificate).
2. A **detached signature** produced using the PKI certificate’s private key over the RootCA public key material (or a canonical encoding / thumbprint thereof).
3. The **validated certificate chain** for the PKI-issued certificate at the time of signing.

### **Verifier Semantics (Normative)**

If PKI anchoring evidence is present, a verifier:

- **MUST** validate the PKI certificate chain against its configured PKI trust roots under replica-local policy.
- **MUST** verify the detached signature over the RootCA key material.
- **MUST** treat successful validation as *additional provenance evidence* and **MUST NOT** treat it as altering the RootCA Artipoint’s semantic role as the instance trust anchor.

## **13.3 Optional Time-Stamping Authority (TSA) Signature**

To provide independent temporal attestation, deployments MAY include an RFC 3161-compliant Time-Stamp Token (TST) covering a hash of the RootCA provenance payload.

This specification defines only the *evidence semantics and validation rules*. The *transport and placement* of TSA evidence is defined in the companion **ASCP Bootstrap and Channel Discovery** specification.

### **Evidence Format (Informational)**

A typical TSA evidence set consists of:

1. A hash computed over a canonical RootCA provenance payload (including PKI anchoring evidence if present).
2. An RFC 3161 **Time-Stamp Token (TST)** issued by a TSA, asserting the hash existed at or before the TSA timestamp.

### **Verifier Semantics (Normative)**

If a TSA token is present, a verifier:

- **MUST** verify the TST signature using the TSA’s certificate chain under replica-local policy.
- **MUST** verify that the TST covers the expected hash of the provenance payload.
- **MUST** treat successful validation as independent temporal evidence and **MUST NOT** treat it as altering the RootCA Artipoint’s semantic role as the instance trust anchor.

## **13.4 Provenance Evidence Validation Steps for a Verifier**

The procedures by which a verifier *discovers* the RootCA Artipoint and any associated provenance material (e.g., bootstrap channels, ordering conventions, or discovery heuristics) are defined in the companion **ASCP Bootstrap and Channel Discovery** specification.

Given a RootCA Artipoint and any associated provenance evidence objects discovered by those procedures, a verifier evaluates provenance as follows:

1. **Verify RootCA self-consistency**
   - Confirm the RootCA Artipoint is well-formed as a distinguished Certificate Artipoint.
   - Confirm the RootCA key material (or canonical thumbprint thereof) matches the expected immutable value derived from the RootCA Artipoint.
2. **Validate optional PKI anchoring evidence (if present)**
   - Validate the PKI certificate chain under replica-local policy.
   - Verify the detached signature over the RootCA key material.
3. **Validate optional TSA attestation (if present)**
   - Verify the TSA token signature under replica-local policy.
   - Confirm the token covers the expected hash of the provenance payload.
4. **Confirm log-anchored immutability**
   - Ensure that all evaluated provenance evidence objects are immutable, replayable, and bound to Coordination Log history such that independent replicas can reach the same conclusions at log-time, subject only to explicitly configured replica-local policy inputs.

## **13.5 Key Points**

- The provenance is **permanently preserved** as immutable log-anchored evidence discoverable via the bootstrap and discovery procedures.
- Even if the PKI certificate later expires or is revoked, verifiers can confirm it was valid when the ASCP instance was anchored.
- Re-anchoring is possible at any later date to provide renewed PKI endorsements without replacing the RootCA.
- Optional TSA signatures and/or publication in a transparency log provide **independent temporal proof** for maximum audit assurance.

# 14. Overall Summary

ASCP's log-anchored trust architecture solves the fundamental challenges of distributed identity by anchoring all trust decisions in immutable log entries rather than real-time validation. This provides durable identity, complete cryptographic provenance, and autonomous operation without external dependencies.

The system enables both privacy-first and enterprise deployments through self-sovereign key generation, optional PKI anchoring, and secure recovery mechanisms. Most critically, it establishes verifiable trust relationships between humans and autonomous agents with identical cryptographic guarantees.

ASCP delivers a trust infrastructure that integrates with existing PKI systems today while providing the foundation for scalable human-AI coordination.

# **Appendix A: Design Rationale**

The ASCP Trust and Identity Architecture is grounded in a set of principles that guide how identities, certificates, endorsements, and trust anchors interoperate across distributed environments. While the normative sections of this specification focus on precise requirements and behaviors, the design rationale presented here provides the conceptual framing behind those choices. Its purpose is to help reviewers understand why the protocol is structured as it is, how optional features fit into the larger system, and what trade-offs informed particular architectural decisions. These explanations are non-normative and offer additional insight into the model’s intent and the reasoning that supports it.

## **A.1 Log-Anchored Trust and Historical Verifiability**

ASCP places trust not in online authorities or ephemeral validations, but in immutable records captured at the moment coordination occurs. This design ensures that every trust decision can be reevaluated in the future based on the state of evidence that existed at the time the statement was made. The log, rather than a central authority, becomes the locus of trust.

Historical verifiability is central to this approach. Certificate chains may expire, identity providers may rotate keys, and external systems may change policies or even disappear. By ensuring that endorsements, attestations, and evidence are captured alongside the statements they support, ASCP allows participants to reconstruct trust decisions long after external systems have changed. A signature validated today must remain valid decades later, independent of the stability or availability of the systems that originally supported it.

The log-anchored model also supports autonomy. Each participant, or replica, carries a complete copy of the trust-relevant history for the channels it has joined. No participant must ask for permission to trust: verification becomes a local, deterministic operation, influenced only by immutable history and locally configured policies. This autonomy is vital in distributed environments, particularly where human agency, offline operation, or cross-organizational collaboration require independence from continuous reliance on external validation infrastructures.

## **A.2 Self-Sovereign Keys, Proof of Possession, and Self-Signing**

At the heart of each ASCP identity is a self-generated key pair. This reflects an intentional commitment to self-sovereignty: identities should not depend on any external authority for their creation or basic operation. By requiring that public keys be self-signed and introduced through authenticated sessions, the system ensures that the holder of the private key is demonstrably the same entity initiating participation.

Self-signing establishes proof of possession at the moment the key is introduced. It binds the key to a specific historical point in the log and provides permanent provenance for that binding. Even when later endorsements, attestations, or RootCA signatures strengthen a certificate’s credibility, the root of that trust remains the participant’s self-generated proof.

This approach also enables seamless migration across devices and environments. Because the key pair originates with the participant, the system avoids introducing assumptions about centralized key issuance, escrow by default, or authority-managed identity creation. Optional enterprise integrations may exist, but they build atop this self-sovereign core.

## **A.3 Why and When to Use RootCA Signing**

Many deployments will not require RootCA signing of participant keys, as self-signing and endorsements are often sufficient for establishing authorship and identity provenance within a trust domain. However, some scenarios call for additional layers of assurance that RootCA signing is specifically designed to provide.

RootCA signing becomes relevant when participants must be verifiably tied to an organization’s internal identity authority. Enterprises may enforce policies requiring all participant keys to be blessed by the organization’s RootCA in order to ensure they conform to internal security requirements. Similarly, cross-organizational collaboration may require presenting externally verifiable assurances that a particular certificate originates from a specific entity.

Anchoring participant keys to the RootCA also supports recovery workflows where enterprise-managed escrow keys facilitate restoration of corporate access. Although individual participants retain self-sovereignty over their private keys, RootCA signatures enable organizations to layer their own expectations for governance, accountability, and compliance without altering the participant-centered trust model.

In ASCP, RootCA signing is intentionally optional. It is a mechanism for strengthening confidence, not a prerequisite for authorship or participation. This flexibility allows ASCP to support both privacy-forward deployments and tightly controlled enterprise environments without compromising the integrity of the core trust model.

## **A.4 Enterprise Escrow and Deployment Considerations**

Key recovery is an essential feature for both individuals and organizations, but the motivations differ. Individual users often require recovery mechanisms to protect against device loss or failure. Enterprises, meanwhile, may be legally or operationally compelled to ensure business continuity in the face of employee turnover, device loss, or forensic requirements.

To accommodate these differing needs, ASCP introduces recovery envelopes as a flexible mechanism for encrypting private keys in a layered structure. The model preserves confidentiality through double encryption: once with a user-derived key and once with a recovery key stored separately. The log-anchored publication of the recovery envelope enables deterministic retrieval during a recovery operation while avoiding any exposure of sensitive material.

For personal deployments, recovery keys typically reside on a secondary device or hardware token, reinforcing the principle that no centralized service should hold unilateral control over the participant’s private keys. In enterprise-managed deployments, recovery keys may be controlled by the organization, allowing recovery actions under well-defined governance policies. Even in those cases, ASCP emphasizes transparency: the fact of recovery, the scope of materials involved, and the provenance of recovery actions should appear in the log for auditability whenever appropriate to the deployment’s trust model.

The architecture ensures that recovery mechanisms remain orthogonal to governance semantics. Recovering a key does not grant or revoke membership, roles, or authorship permissions; it merely restores cryptographic capability. This separation helps maintain clarity in distributed environments where cryptographic identity and governance authority often intersect but must remain conceptually distinct.

## **A.5 Orientation for Reviewers**

ASCP’s trust architecture spans foundational infrastructure, cryptographic constructs, and workflow-level mechanisms such as identity bootstrap and recovery. Reviewers approaching this specification for the first time may find it helpful to consider the document in terms of three conceptual layers.

The first layer establishes the trust primitives: identities, certificates, endorsements, and trust anchors. These represent the building blocks from which all trust decisions are made. The second layer defines how these primitives interplay within the ASCP log to provide cryptographically verifiable authorship, provenance, and trust evaluation. The third layer focuses on lifecycle processes: how participants join, how they rotate keys, how they bind external evidence to their identities, and how lost keys may be restored.

Understanding these layers clarifies the intent of the protocol. ASCP does not treat identity as a monolithic construct but instead decomposes it into composable, auditable parts that better reflect the ways in which people and systems operate across devices, organizations, and time. This layered structure also ensures that deployments can adapt to their specific requirements without altering the underlying trust model, preserving interoperability while accommodating variability in governance or operational policies.

## **A.6 Concluding Perspective**

The design of ASCP’s trust architecture reflects a commitment to durable provenance, human-centered autonomy, and long-term verifiability. By grounding trust in immutable history rather than transient systems, and by privileging participant-generated keys over externally issued identities, ASCP offers a trust substrate capable of supporting the evolving relationship between humans and intelligent agents.

These guiding principles shape every normative requirement in the main specification. While implementers need not internalize every nuance of the rationale to build compliant systems, understanding these motivations helps illuminate why ASCP prioritizes verifiable history, explicit declarations, and cryptographic self-sovereignty. It also underscores the protocol’s role as a foundational layer for richer forms of coordination and collaboration in distributed, multi-agent ecosystems.

# **Appendix B: Deployment Models**

ASCP is designed to operate across a wide spectrum of deployment environments, from lightweight personal workspaces to large-scale organizational infrastructures and cross-institution collaboration networks. Although the normative behavior of the protocol remains the same in all cases, the practical realities of deployment often introduce distinct expectations around identity onboarding, key lifecycle management, recovery, and interoperability. This appendix provides a conceptual overview of the primary deployment models for ASCP, offering non-normative guidance that helps situate the Trust and Identity Architecture within real operational contexts.

## **B.1 Personal and Local-First Deployments**

In personal or small-team environments, ASCP typically begins with a single-user bootstrap, a self-generated RootCA, and a minimal identity and certificate set. The design philosophy in this context emphasizes privacy, autonomy, and resilience. Users generate their own identity keys locally, maintain exclusive control of private key material, and often employ a second device or hardware token as the recovery key for optional key backup.

These deployments benefit from the self-sovereign nature of ASCP’s identity architecture. Because no external certificate authority or institutional identity provider is required, a user can operate an ASCP instance entirely offline or across intermittent connectivity. The log-anchored structure ensures that authorship and provenance remain verifiable even without access to external systems. Recovery workflows are similarly self-contained: a locally stored double-encrypted envelope, together with a recovery key kept on a trusted secondary device, enables secure device migration without reliance on third parties.

Personal deployments often evolve organically. Additional devices may join the trust domain over time, each establishing its own certificate and purpose bindings. Cross-device verification occurs through replication of the log and validation of historical signatures. These deployments illustrate ASCP’s capacity to support human-scale workflows where durability, sovereignty, and privacy are paramount.

## **B.2 Enterprise and Organizational Deployments**

Enterprises frequently have requirements that extend beyond the needs of individual users. These may include identity governance, continuity of operations, legal and compliance obligations, and integration with existing security infrastructures. ASCP accommodates these needs without compromising the core trust model by layering enterprise controls on top of self-sovereign identities.

In organizational deployments, identity bootstrap may use corporate identity providers such as Azure AD, Okta, or custom OIDC services. These providers supply authoritative user assertions, which ASCP binds directly to participant-generated keys through IdentityClaimBundles. Enterprises may also elect to sign participant keys using the RootCA, allowing internal systems to verify that keys have been formally recognized or authorized by organizational policy.

Recovery expectations differ in this environment as well. While ASCP’s standard envelope mechanism allows users to maintain private control of their identity keys, enterprises may require recovery keys held in a secure, organization-managed hardware module or escrow service. Such configurations provide business continuity while preserving the cryptographic and audit protections built into the envelope structure. Recovery operations can be logged, enabling transparency and traceability consistent with institutional oversight.

Governance in enterprise deployments typically relies on additional ASCP specifications that define roles, membership semantics, and authorization rules. Those governance attributes, when combined with the trust primitives defined in this specification, enable organizations to enforce structured collaboration without centralizing control of private keys or undermining the durability of the log-anchored trust model.

## **B.3 Federated and Multi-Organization Deployments**

In cross-institution collaborations—whether between companies, research groups, agencies, or distributed teams—ASCP’s architecture provides mechanisms for independently governed trust domains to interoperate without requiring a shared global identity authority. Each organization maintains its own RootCA and identity onboarding process, while ASCP enables trust relationships to form at the boundary between these domains.

Endorsements play a crucial role here. When a participant from one ASCP instance wishes to collaborate with another, their certificates may carry endorsements from trusted external identity providers, enterprise CAs, or DID-based controllers. These endorsements offer portable, verifiable evidence of identity provenance, allowing the receiving instance to make trust decisions without giving up autonomy or requiring pre-established bilateral agreements.

Federated deployments also make use of selective replication. Instead of merging trust domains, each organization retains control over its own channels and logs, replicating only those necessary for shared work. The immutability of the log ensures that participants can audit each other’s contributions independently, while the layered trust model allows each domain to evaluate evidence according to its own policies.

These deployments highlight ASCP’s ability to create trust “bridges” while preserving local identity authority. Rather than imposing a global PKI or universal naming scheme, ASCP allows trust to emerge from verifiable, composable evidence, anchored in each domain’s own immutable history.

## **B.4 Hybrid Deployments and Evolution Over Time**

Many real-world deployments do not neatly fit into a single category. An individual may begin with a personal ASCP instance and later integrate with an organizational one. A team may adopt ASCP internally before opening channels to external collaborators. Enterprises may gradually introduce ASCP into their workflows, first as a coordination tool and later as a governance substrate.

ASCP is designed for this fluidity. Because identities are self-sovereign and certificates accumulate provenance over time, the same cryptographic foundations can persist across shifting contexts. A participant who later joins an enterprise deployment can attach new endorsements without altering earlier evidence. Recovery mechanisms remain compatible across device migrations and deployment boundaries. RootCA provenance allows institutions to evolve their trust anchors without invalidating historical signatures.

This adaptability ensures that ASCP can support long-lived collaboration, even as its participants, organizations, and infrastructures change. Trust grows cumulatively: new endorsements, new certificates, and new purposes extend a participant’s history rather than replacing it. The log becomes a durable narrative of trust formation and evolution.

## **B.5 Closing Perspective on Deployment Diversity**

ASCP’s trust and identity architecture is intentionally deployment-agnostic. Its primitives—Identity Artipoints, Certificate Artipoints, endorsements, purposes, and recovery envelopes—operate consistently whether used by an individual, an enterprise, or a federation of organizations. What varies is not the protocol itself but the policies, evidence sources, and operational expectations layered atop it.

By grounding trust in immutable history and participant-controlled keys, ASCP provides a stable substrate capable of supporting a wide variety of real-world collaboration patterns. Whether deployed privately on a single machine, across a corporate boundary, or within a global ecosystem of agents and organizations, the same foundational mechanisms ensure verifiable provenance, durable identity, and trustworthy coordination. This appendix offers a conceptual guide to those possibilities, helping reviewers understand how the normative requirements of the specification translate into practical deployment scenarios.

