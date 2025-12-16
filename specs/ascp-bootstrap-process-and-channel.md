# **ASCP Bootstrap Process and Channel Discovery**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.30 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the **Agents Shared Cognition Protocol (ASCP)** specification suite and defines the **bootstrap process and channel discovery procedures** used to initialize an ASCP organizational instance and to onboard new replicas into an existing instance.

Specifically, this document describes how an ASCP repository establishes its initial root of trust, how mandatory bootstrap channels are created and interpreted, and how replicas deterministically discover available channels before entering normal ASCP operation. The procedures defined herein coordinate the ASCP Trust & Identity Architecture and the ASCP LogSync Protocol (ALSP) during first contact and initialization.

This document is **not** an Internet Standards Track specification. It has not undergone IETF review and has no formal standing within the IETF standards process. It is published for **community review, experimentation, and implementation feedback**. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use reflects provisional interoperability requirements and is subject to change in future revisions.

Feedback from implementers, distributed systems designers, protocol engineers, and security reviewers is explicitly requested to guide refinement toward a future Internet-Draft.

# **2. Abstract**

This document specifies the **bootstrap process and channel discovery mechanisms** for the Agents Shared Cognition Protocol (ASCP).

It defines how a new ASCP organizational instance establishes an initial cryptographic root of trust, how mandatory bootstrap channels are created and interpreted, and how replicas deterministically discover available channels before normal coordination begins. The bootstrap process addresses the fundamental challenge of validating trust anchors and channel structure in a system where all trust semantics are derived from immutable, log-anchored history.

The procedures defined herein apply both to **genesis bootstrap**, in which a new ASCP instance is created from an empty state, and to **join bootstrap**, in which a new replica connects to an existing instance.

# **3. Introduction**

The Agents Shared Cognition Protocol (ASCP) is designed to support durable, auditable, and decentralized collaboration between humans and autonomous agents. Unlike systems that rely on mutable configuration, centralized directories, or real-time trust evaluation, ASCP derives all trust and coordination semantics from immutable, signed history.

This design choice provides long-term verifiability and autonomy, but it also introduces a fundamental initialization problem: a replica cannot validate trust material until a trust graph exists, yet the trust graph itself must be constructed from signed log entries. Similarly, channel discovery cannot be safely performed until trust is established, but trust establishment requires access to specific channels.

This document addresses that problem by specifying the **bootstrap process and channel discovery mechanisms** that allow an ASCP repository to be initialized and new replicas to be onboarded in a deterministic and auditable manner.

## **3.1 Motivation**

In many distributed systems, bootstrap is treated as an implicit or operational concern: trust is assumed based on server identity, configuration files, or external infrastructure. Such approaches are insufficient for ASCP.

ASCP requires that:

- trust decisions be **reproducible at any point in the future**,
- replicas be able to **operate autonomously once initialized**,
- humans and autonomous agents participate under a **uniform trust model**, and
- coordination history retain its meaning despite key rotation, device loss, or infrastructure changes.

These requirements imply that **trust establishment and channel discovery must themselves be part of the immutable coordination substrate**, rather than external prerequisites.

Without a standardized bootstrap process, different implementations would make incompatible assumptions about initial trust, channel enumeration, and readiness, undermining interoperability and weakening ASCP’s guarantees of durable shared meaning.

The purpose of this document is therefore to make bootstrap **explicit, deterministic, and verifiable**, ensuring that all ASCP implementations initialize trust and discovery in a consistent manner.

## **3.2 Scope of This Document**

This document specifies the procedures and invariants required to transition an ASCP replica from an uninitialized state into normal operation.

Specifically, it defines:

- how an ASCP organizational instance establishes an initial root of trust,
- the mandatory bootstrap channels required for safe initialization,
- how replicas deterministically discover available channels, and
- the conditions under which a replica is considered operationally ready.

This document covers both:

- **genesis bootstrap**, where a new ASCP organizational instance is created, and
- **join bootstrap**, where a new replica connects to an existing instance.

This document does **not** define cryptographic primitives, identity semantics, governance policy, channel encryption mechanisms, or log synchronization protocols. Those concerns are defined in companion ASCP specifications and are referenced where required.

The role of this document is to define **ordering, dependencies, and readiness criteria**, not to redefine the behavior of lower ASCP layers.

## **3.3 End-to-End Bootstrap and Safe Participation Model**

Safe participation in the Agents Shared Cognition Protocol (ASCP) is achieved only through the coordinated operation of three distinct and complementary components: authenticated log replication, log-anchored trust evaluation, and ordered acquisition of bootstrap and discovery history.

First, **authenticated log replication**, as defined by the ASCP Log Synchronization Protocol (ALSP), ensures that a replica can obtain an append-only, integrity-protected history of channel events from authorized peers. ALSP provides transport-level authentication, replay protection, and deterministic convergence of replicated logs, but it is intentionally agnostic to the semantics of identity, trust, and governance.

Second, **log-anchored trust evaluation**, as defined in the ASCP Trust and Identity Architecture, derives identity validity, certificate binding, delegation, and trust relationships exclusively from immutable, signed history contained within ASCP channels. Trust is not inferred from transport-level authentication or configuration state, but from verifiable statements recorded in the shared log.

Third, **ordered acquisition of bootstrap and discovery history**, as defined in this document, specifies the sequence by which a replica transitions from initial trust input to operational readiness. This model assumes an Initial Trust Input (ITI) supplied out-of-band; **thereafter**, all safety-relevant conclusions are derived exclusively from immutable, verifiable history. This includes the mandatory retrieval and validation of bootstrap channels, establishment of authoritative trust roots, and deterministic discovery of additional channels prior to normal replication.

None of these components is sufficient in isolation. Authenticated replication without trust evaluation does not establish authority; trust evaluation without ordered history acquisition lacks a verified context; and bootstrap procedures without authenticated replication or log-anchored trust cannot be validated or audited. Safe participation in ASCP therefore requires the coordinated execution of all three components, with this document defining the ordering and readiness conditions under which normal ASCP operation may begin.

## **3.4 Repository Identification (Non-Normative)**

An ASCP repository represents a single organizational trust domain and is identified to replicas through **out-of-band context** provided at connection time. This specification does not mandate a global naming scheme for repositories.

Implementations MAY derive repository identity from deployment-specific identifiers, including but not limited to:

- the DNS name or network endpoint used to establish the initial ALSP connection,
- a cryptographic fingerprint of the organization’s bootstrap trust anchor, or
- an explicit repository identifier conveyed through provisioning or invitation mechanisms.

Repository identification is treated as an **initial context input** rather than a property derived from articulated history. Once bootstrap validation completes, all subsequent trust, discovery, and coordination semantics are derived exclusively from immutable, verifiable ASCP history as defined in Section 3.3.

# **4. Terminology and Conventions**

This section defines terminology and conventions used throughout this document. Terms defined here are intended to be used consistently across all sections. Where possible, definitions align with terminology used in other ASCP specifications.

Unless otherwise noted, all terms in this section are **normative**.

## **4.1 Requirements Language**

The key words **“MUST”**, **“MUST NOT”**, **“REQUIRED”**, **“SHALL”**, **“SHALL NOT”**, **“SHOULD”**, **“SHOULD NOT”**, **“RECOMMENDED”**, **“NOT RECOMMENDED”**, **“MAY”**, and **“OPTIONAL”** in this document are to be interpreted as described in **RFC 2119** and **RFC 8174**.

## **4.2 Defined Terms**

### **ASCP (Agents Shared Cognition Protocol)**

A distributed protocol suite that enables humans and autonomous agents to collaborate using immutable, signed logs of structured coordination artifacts.

### **Organizational Instance**

An ASCP repository representing a single organizational trust domain. An organizational instance defines the scope within which trust anchors, identities, channels, and coordination artifacts are interpreted.

### **Repository**

The complete set of ASCP channel logs associated with a single organizational instance. A repository is the unit of bootstrap and replication.

### **Replica**

A local copy of some or all of an ASCP repository maintained by a node. A replica may act as a client, a server, or both, and may participate in replication with other replicas using ALSP.

### **Bootstrap Process**

The ordered procedures by which an ASCP organizational instance is initialized or a replica joins an existing instance, establishing trust anchors, discovering channels, and transitioning into normal operation.

### **Genesis Bootstrap**

The bootstrap process in which a new ASCP organizational instance is created from an empty state. Genesis bootstrap includes creation of the organizational trust anchor and mandatory bootstrap channels.

### **Join Bootstrap**

The bootstrap process in which a new replica connects to an existing ASCP organizational instance and initializes itself by acquiring and validating bootstrap material.

### **Bootstrap Trust Anchor (RootCA)**

The cryptographic root of trust for an ASCP organizational instance. The RootCA is introduced during genesis bootstrap and serves as the ultimate trust anchor for validating identity and channel-related trust material.

### **Bootstrap Channel (@bootstrap)**

A mandatory ASCP channel that anchors the organizational trust root and contains the minimum information required for a replica to begin safe initialization. The bootstrap channel is designed to be readable prior to full authorization in order to resolve initial trust dependencies.

### **References Channel (@references)**

A mandatory ASCP channel that serves as the authoritative registry for channel discovery. The references channel contains signed records that enumerate available channels and provide sufficient information for replicas to locate and replicate them.

### **Bootstrap Key Package (BKP)**

A **Bootstrap Key Package (BKP)** is a bootstrap-scoped cryptographic package delivered to a joining replica during join bootstrap that enables initial decryption of the @references channel for deterministic discovery.

### Bootstrap-Serving Replica

A **Bootstrap-Serving Replica** is a replica that possesses sufficient local state to (a) authenticate as an identity traceable to the RootCA and (b) deliver a valid BKP enabling decryption of the @references channel.

### **Channel**

A named, append-only communication log used to distribute Artipoints among authorized participants. Each channel has a globally unique identifier (UUID) and a human-readable name.

### **Channel Log**

The persistent, ordered log of Artipoints associated with a channel. Channel logs are replicated between replicas using ALSP.

### **Artipoint**

An immutable, signed, and addressable statement recorded in an ASCP channel log. Artipoints are the foundational coordination primitive of ASCP and may represent trust material, identity bindings, channel declarations, or other coordination artifacts.

### **Channel Reference (Channel-Ref Artipoint)**

A signed Artipoint recorded in the @references channel that declares the existence of a channel and provides metadata required for discovery and replication, such as channel UUID and log location.

### **Initial Trust Input (ITI)**

Information obtained out-of-band and supplied to a replica prior to bootstrap, used to establish initial trust in the bootstrap process. Examples include pinned trust anchors, invitation artifacts, or trust-on-first-use assumptions.

### **Operationally Ready**

A state in which a replica has successfully completed bootstrap, validated required trust material, discovered available channels, and is able to participate in normal ASCP operations.

### **Local State**

Implementation-specific, mutable state maintained by a replica that is not replicated across peers. Local state includes, but is not limited to, channel encryption keys e.g., Channel Access Keys (CAKs), Channel Symmetric Keys (CSKs), keyframe material, and any key material delivered via Bootstrap Key Packages (BKPs).

Local state does not contribute to ASCP’s shared trust or coordination semantics. Loss of required local state MAY require a replica to re-enter join bootstrap in order to regain operational capability.

## **4.3 Conventions**

- Channel names prefixed with “@” (for example, @bootstrap, @references) denote **well-known or conventionally significant channels**.
- UUIDs are assumed to be globally unique identifiers unless otherwise stated.
- References to other ASCP specifications are informational unless explicitly stated as normative dependencies.

# **5. Architectural Context**

This section situates the ASCP bootstrap process within the broader ASCP architecture and clarifies its relationship to other ASCP specifications. It describes where bootstrap fits in the layer model, how responsibilities are divided across specifications, and the design principles that govern bootstrap behavior.

Bootstrap is not a standalone protocol. It is an **architectural orchestration concern** that coordinates existing ASCP layers during initialization while preserving their separation of responsibilities.

## **5.1 Position in the ASCP Layer Model**

ASCP is structured as a layered architecture, with each layer responsible for a distinct class of concerns. The bootstrap process operates **logically at Layer 3**, but necessarily invokes capabilities from all lower layers to resolve first-contact dependencies.

The relevant layers are:

- **Layer 0 — LogSync Protocol (ALSP) -** Provides authenticated session establishment, append-only log replication, ordering, and convergence detection. ALSP treats log payloads as opaque and does not interpret trust or coordination semantics.
- **Layer 1 — Channels (Secure Distribution Layer) -** Defines channel membership, confidentiality boundaries, and encryption semantics for log content once trust relationships exist.
- **Layer 2 — Artipoint Grammar -** Defines the structural encoding of Artipoints and their attributes, independent of transport or trust evaluation.
- **Layer 3 — Semantic Trust and Coordination -** Interprets Artipoints to construct trust graphs, identity bindings, and higher-level coordination structures.

Bootstrap is a **Layer 3 concern** because it establishes the initial semantic conditions under which trust and coordination can be evaluated. However, bootstrap cannot function without invoking Layer 0 for replication and Layer 1 for subsequent secure distribution.

This document specifies how those layers are composed during initialization without altering their normative behavior.

## **5.2 Relationship to Other ASCP Specifications**

This document is one of several specifications that together define ASCP. Each specification has a narrowly defined scope.

### **5.2.1 Relationship to ASCP Trust and Identity Architecture**

The **ASCP Trust and Identity Architecture** defines:

- cryptographic identity constructs,
- certificates and trust anchors,
- endorsement and provenance mechanisms, and
- rules for validating trust material using log-anchored history.

This bootstrap specification does **not** define identity formats, certificate semantics, or validation algorithms. Instead, it defines:

- **when** trust anchors and identity material are introduced,
- **where** they are recorded during initialization, and
- **how** they are obtained and validated during bootstrap.

Bootstrap relies on Trust & Identity to interpret trust material but does not modify its trust model.

### **5.2.2 Relationship to ASCP LogSync Protocol (ALSP)**

The **ASCP LogSync Protocol (ALSP)** defines:

- peer authentication,
- session establishment,
- log replication,
- ordering and convergence behavior.

This bootstrap specification does **not** define ALSP messages, cryptographic handshakes, or replication algorithms. Instead, it defines:

- the **required ordering** of replication during bootstrap,
- which channels MUST be replicated first, and
- the conditions under which a replica may transition to normal synchronization.

ALSP remains transport- and semantics-agnostic. Bootstrap supplies the semantic ordering that ALSP intentionally avoids.

### **5.2.3 Relationship to ASCP Channels**

The **ASCP Channels** specification defines:

- channel membership,
- encryption and key distribution,
- confidentiality and visibility constraints.

This document defines the **bootstrap-specific exceptions and sequencing** required before normal channel semantics apply, but does not redefine channel behavior once trust is established.

## **5.3 Design Principles**

The bootstrap architecture is governed by the following principles.

### **5.3.1 Explicit Trust Establishment**

Bootstrap makes trust establishment explicit and auditable. No trust assumptions are implicit, hidden in configuration, or delegated to mutable external systems. All trust anchors and discovery mechanisms are expressed as signed, immutable log entries.

### **5.3.2 Log-Anchored Initialization**

Bootstrap artifacts are subject to the same log-anchored trust model as all other ASCP coordination data. Initialization does not rely on external directories, live validation services, or mutable state.

Any temporary assumptions made during bootstrap exist only to enable construction of a minimal trust graph and are subsequently validated using standard ASCP verification rules.

### **5.3.3 Log Integrity vs. Content Visibility**

During bootstrap, ASCP distinguishes between **verification of log integrity** and **visibility of encrypted content**.

A replica MAY authenticate, order, and verify the integrity and authorship of all ASCP bootstrap-related log entries without possession of any channel encryption keys. However, when the @references channel is encrypted, possession of appropriate bootstrap-scoped key material is REQUIRED in order to decrypt and validate its contents for deterministic channel discovery (see Section 10.7).

This distinction applies only during bootstrap and discovery initialization and does not alter normal ASCP channel authorization, visibility, or key provisioning semantics once bootstrap has completed.

### **5.3.4 Deterministic Discovery**

Channel discovery is deterministic and auditable. Given the same bootstrap material, independent replicas MUST discover the same set of channels and resolve them consistently.

This property is essential for shared cognition: participants must agree not only on the content of coordination artifacts, but on **what coordination structures exist at all**.

### **5.3.5 Separation of Concerns**

Bootstrap coordinates multiple layers without collapsing them. Cryptography, replication, grammar, and trust semantics remain defined in their respective specifications.

This separation ensures that:

- bootstrap remains minimal,
- implementations remain interoperable, and
- future evolution of individual layers does not invalidate bootstrap guarantees.

### **5.3.6 Symmetry Between Humans and Agents**

Bootstrap procedures apply uniformly to human users and autonomous agents. There are no special-case initialization paths based on participant type.

This symmetry ensures that agents participate in ASCP under the same trust and provenance guarantees as humans, preserving consistent semantics across the system.

## **5.4 Architectural Summary**

Bootstrap is the mechanism by which ASCP transitions from **absence of context** to **shared, verifiable context**.

Architecturally, it:

- anchors organizational trust,
- establishes deterministic discovery,
- coordinates layered protocols during first contact, and
- produces a well-defined operational readiness state.

The following sections build on this context by defining the concrete concepts, channels, and procedures that realize this architecture.

# **6. Bootstrap Concepts and Invariants**

This section defines the core concepts and invariants that govern the ASCP bootstrap process. These concepts apply uniformly to both genesis bootstrap and join bootstrap and are independent of specific deployment topologies.

The invariants defined in this section MUST hold for all ASCP-compliant implementations.

## **6.1 ASCP Organizational Instance**

An **ASCP organizational instance** is the fundamental unit of trust and coordination in ASCP. It defines the scope within which identities, channels, and coordination artifacts are interpreted.

An organizational instance is characterized by:

- a single **repository** containing all associated channel logs,
- a single **bootstrap trust anchor** that serves as the root of trust, and
- a fixed set of **mandatory bootstrap channels**.

All trust decisions within an organizational instance ultimately derive from artifacts recorded in its repository. Replicas participating in the same organizational instance MUST evaluate trust and discovery using the same bootstrap invariants.

An ASCP organizational instance is created exactly once via **genesis bootstrap**. Subsequent replicas join the instance via **join bootstrap** but do not redefine its trust boundary.

## **6.2 Bootstrap Trust Anchor (RootCA)**

Each ASCP organizational instance MUST define exactly one **Bootstrap Trust Anchor (RootCA)**.

The RootCA is the cryptographic root of trust for the organizational instance and serves as the ultimate authority for validating trust material introduced during bootstrap.

### **6.2.1 Role of the RootCA**

The RootCA:

- anchors trust for identity and certificate material introduced during bootstrap,
- enables deterministic validation of bootstrap artifacts, and
- establishes a stable trust root against which all subsequent trust relationships are evaluated.

All trust material recorded during bootstrap MUST be verifiable, directly or indirectly, against the RootCA.

### **6.2.2 Scope and Lifetime**

The RootCA is scoped to a single organizational instance and remains authoritative for the lifetime of that instance.

ASCP does not support destructive replacement of the RootCA. If trust anchoring must evolve, such evolution MUST occur through additive mechanisms that preserve historical verifiability.

### **6.2.3 Relationship to External Trust Systems**

The RootCA MAY be optionally endorsed or attested by external trust systems (e.g., PKI, identity providers). Such endorsements strengthen provenance but do not replace the RootCA’s role as the internal trust root.

Evaluation of external endorsements is defined in the ASCP Trust and Identity Architecture and is outside the scope of this document.

## **6.3 Mandatory Bootstrap Channels**

Every ASCP organizational instance MUST contain the following mandatory channels:

1. the **Bootstrap Channel** (@bootstrap), and
2. the **References Channel** (@references).

These channels are required for deterministic initialization and discovery and are treated as architectural invariants.

### **6.3.1 Bootstrap Channel Invariant**

The @bootstrap channel:

- MUST exist in every organizational instance,
- MUST anchor the RootCA,
- MUST contain sufficient information to enable a replica to begin safe initialization, and
- MUST be discoverable by any replica attempting to bootstrap.

The @bootstrap channel represents the minimal, authoritative entry point into the organizational instance.

### **6.3.2 References Channel Invariant**

The @references channel:

- MUST exist in every organizational instance,
- MUST serve as the authoritative registry for channel discovery, and
- MUST enumerate all channels that replicas may subsequently replicate, subject to authorization.

The @references channel is the sole normative source of truth for channel discovery. Replicas MUST NOT rely on out-of-band or inferred channel listings for discovery.

### **6.3.3 Separation of Responsibilities**

The separation between @bootstrap and @references is intentional and invariant:

- @bootstrap establishes **trust** and enables transition out of the uninitialized state.
- @references establishes **discovery** once trust exists.

This separation ensures that trust establishment remains minimal and auditable while allowing channel discovery to evolve independently under normal ASCP channel semantics.

## **6.4 Bootstrap Readiness Invariant**

A replica MUST NOT be considered **operationally ready** until all of the following conditions are satisfied:

1. The @bootstrap channel has been acquired and validated.
2. The RootCA has been successfully established as the trust root.
3. The @references channel has been acquired and validated.
4. Channel discovery has completed deterministically.

Only after these conditions are met MAY a replica transition into normal ASCP operation and replicate additional channels.

## **6.5 Invariant Summary**

The following invariants define the minimum conditions for ASCP bootstrap correctness:

- Each organizational instance has exactly one RootCA.
- Each organizational instance has exactly one @bootstrap channel.
- Each organizational instance has exactly one @references channel.
- All replicas bootstrap trust before discovery.
- All replicas discover channels deterministically via @references.

Violations of these invariants indicate a non-compliant or corrupted bootstrap state.

## **6.6 Forward References**

The subsequent sections of this document define:

- the semantics and required contents of the @bootstrap channel (Section 7),
- the semantics and required contents of the @references channel (Section 8),
- the procedures for genesis bootstrap (Section 9),
- the procedures for join bootstrap (Section 10), and
- the coordination of bootstrap with ALSP (Section 11).

# **7. The @bootstrap Channel**

The @bootstrap channel is the foundational entry point into an ASCP organizational instance. It exists to resolve the initial trust and discovery dependency that arises when no prior shared context exists.

This section defines the normative semantics, required contents, and exceptional properties of the @bootstrap channel.

## **7.1 Purpose and Semantics**

The @bootstrap channel serves three primary purposes:

1. **Trust Anchoring:** It introduces and anchors the Bootstrap Trust Anchor (RootCA), establishing the root of trust for the organizational instance.
2. **Initialization Enablement:** It provides the minimum information required for a replica to transition from an uninitialized state into a state where normal ASCP verification rules can be applied.
3. **Discovery Bridging:** It enables a replica to safely locate and acquire the @references channel, after which normal channel discovery may proceed.

The @bootstrap channel is intentionally minimal. It is not a general coordination channel and MUST NOT be used to convey ordinary application, governance, or workflow data.

Visibility seeding for encrypted discovery artifacts, where required, is performed via bootstrap session mechanisms and not via content recorded in the @bootstrap channel.

## **7.2 Required Contents**

The @bootstrap channel MUST contain sufficient information to enable deterministic trust establishment and transition to discovery.

This information is limited to **trust anchoring and discovery identification** and MUST NOT include channel encryption keys, per-replica key envelopes, or other visibility-granting material.

At a minimum, the @bootstrap channel MUST include:

1. **Bootstrap Trust Anchor Declaration**
   - A signed Artipoint that introduces the RootCA and binds it to the organizational instance.
   - This Artipoint serves as the cryptographic root for validating subsequent bootstrap artifacts.
2. **Mandatory Channel Declarations**
   - A signed reference identifying the @references channel as the authoritative discovery registry.
   - This reference MUST be sufficient for a replica to locate and replicate the @references channel.
3. **Bootstrap Manifests or Metadata (Optional but RECOMMENDED)**
   - Structured metadata that assists replicas in interpreting bootstrap state (e.g., versioning, creation timestamps).
   - Such metadata MUST be immutable and subject to the same validation rules as other bootstrap content.
4. **Bootstrap Identity References (Optional)**
   - One or more Identity Reference Artipoints identifying identities explicitly permitted to serve join bootstrap, providing minimal, RootCA-traceable public key material sufficient for bootstrap-time peer authentication.

All Artipoints in the @bootstrap channel MUST be verifiable, directly or indirectly, against the RootCA once validation completes.

## **7.3 Identity Reference Artipoint**

An **Identity Reference Artipoint** (“identity-ref”) is a signed, immutable declaration recorded in the **@bootstrap** channel that enables a joining replica to authenticate a bootstrap-serving peer **prior to full identity resolution**.

Identity Reference Artipoints exist to support **join bootstrap via replicas that do not possess the RootCA private key**, while preserving ASCP’s log-anchored trust model and keeping the @bootstrap channel minimal.

An identity-ref provides a **bootstrap-time binding** between:

- a declared ASCP identity,
- a specific authentication certificate identifier, and
- the public key material required to verify ALSP peer authentication,

without requiring access to the full identity or certificate history at bootstrap time.

### **7.3.1 Purpose and Scope**

The identity-ref mechanism serves a narrowly scoped purpose:

- to allow a joining replica to validate that a peer’s authenticated ALSP session identity is **traceable to the Bootstrap Trust Anchor (RootCA)** using only:
  - Initial Trust Input (ITI),
  - content in the @bootstrap channel, and
  - ALSP session authentication.

Identity Reference Artipoints:

- **MUST NOT** function as a general identity directory,
- **MUST NOT** enumerate organizational membership,
- **MUST NOT** grant authorization, governance roles, or channel visibility, and
- **MUST** be interpreted solely as a **bootstrap-serving allowlist**.

Only identities explicitly recorded via identity-ref are eligible to act as **join bootstrap peers**.

### **7.3.2 Canonical Form**

An Identity Reference Artipoint MUST be articulated using the following primary payload form:

```asciidoc
[ uuid, author, timestamp,
  ["identity-ref", <label>, <identity-uuid>]
]
```

Where:

- \<identity-uuid> is the UUID of the ASCP Identity Artipoint that the peer asserts during ALSP session establishment.
- \<label> is informational and has no semantic effect.

### **7.3.3 Required Attributes**

An identity-ref Artipoint MUST include the following required attributes:

- **cert\_kid:** A stable identifier for the authentication certificate associated with the identity, of the form `ascp:cert:<cert-uuid>`
- **cert\_public\_jwk:** A JSON Web Key (JWK), as defined by RFC 7517, representing the **public key** corresponding to \<cert-uuid>. The JWK MUST:
  - be sufficient to verify signatures or authentication proofs presented during ALSP session establishment, and
  - exactly match the public key used by the peer to authenticate the ALSP session.

### **7.3.4 Normative Semantics**

The following semantics apply to all Identity Reference Artipoints:

- identity-ref entries are **additive** and MUST NOT be removed or overwritten.
- identity-ref entries declare **bootstrap-time identity resolution metadata only**.
- identity-ref entries do not replace or supersede full Identity or Certificate Artipoints recorded elsewhere in the repository.
- A joining replica MUST treat identity-ref entries as **temporary resolution shims** valid only during bootstrap.

Once normal identity and certificate resolution becomes available, identity-ref entries MUST be treated as informational and MUST NOT be used in preference to canonical ASCP trust material.

### **7.3.5 Validation Requirements**

A replica validating an Identity Reference Artipoint MUST:

1. Verify the Artipoint signature under normal ASCP rules.
2. Confirm that the author identity is traceable, directly or indirectly, to the Bootstrap Trust Anchor (RootCA).
3. Confirm that cert\_kid is well-formed.
4. Confirm that cert\_public\_jwk is syntactically valid and usable for cryptographic verification.

Failure to validate an identity-ref MUST prevent the referenced identity from being treated as eligible to serve join bootstrap.

## **7.4 Encryption and Visibility Requirements**

The @bootstrap channel is subject to exceptional visibility requirements and:

- MUST NOT function as a channel key distribution mechanism, and
- MUST NOT carry material that directly enables decryption of any channel, including @references.

### **7.4.1 Readability Prior to Full Authorization**

The @bootstrap channel MUST be readable by a replica prior to full identity-based authorization. This exception exists solely to allow initial trust establishment and MUST NOT be generalized to other channels.

### **7.4.2 Encryption Constraints**

The @bootstrap channel MUST NOT require pre-established channel encryption keys in order to be read. The @bootstrap channel MUST NOT be encrypted at the channel layer.

This does not imply the absence of security protections. During first contact, integrity, authenticity, and replay resistance for @bootstrap acquisition are provided by:

- authenticated session establishment,
- message authentication, and
- replay prevention mechanisms

as defined by ALSP.

## **7.5 Validation Semantics**

Bootstrap validation follows a bounded two-phase model:

1. **Assumed Validity Phase:** A replica MAY temporarily assume the validity of @bootstrap content in order to construct a minimal trust graph.
2. **Verification Phase:** Once the RootCA is established, all bootstrap content MUST be validated using normal ASCP trust evaluation rules.

If validation fails, the replica MUST treat the bootstrap attempt as unsuccessful and MUST NOT proceed to normal operation.

During the Assumed Validity Phase, a replica MUST NOT derive authorization, discovery, or governance conclusions from any content other than the @bootstrap channel.

## **7.6 Immutability and Evolution**

The @bootstrap channel is immutable in the same sense as all ASCP channels: entries are append-only and never overwritten.

Updates to bootstrap-relevant information MUST preserve historical verifiability. Bootstrap evolution MAY occur through additive mechanisms, but MUST NOT invalidate previously valid trust decisions.

Destructive modification or replacement of bootstrap history is not permitted.

## **7.7 Security Considerations (Bootstrap-Specific)**

Because the @bootstrap channel is readable prior to full authorization, it represents a high-value security boundary.

Implementations MUST ensure that:

- all bootstrap content is cryptographically authenticated,
- all signatures are validated during the verification phase,
- replay attacks are mitigated during acquisition, and
- bootstrap artifacts are scoped strictly to initialization purposes.

The @bootstrap channel MUST NOT be used as a substitute for secure coordination channels once trust is established.

## **7.8 Section Summary**

The @bootstrap channel provides the minimal, authoritative foundation required to initialize an ASCP organizational instance and onboard replicas safely.

It:

- anchors the organizational root of trust,
- enables deterministic transition to discovery,
- operates under narrowly scoped exceptional visibility rules, and
- preserves ASCP’s log-anchored trust guarantees through explicit validation.

Section 8 defines the semantics and required contents of the @references channel, which completes the bootstrap transition by enabling deterministic channel discovery once trust is established.

# **8. The @references Channel**

The @references channel is the authoritative channel discovery registry for an ASCP organizational instance. Once trust has been established via the @bootstrap channel, the @references channel provides the sole normative mechanism by which replicas learn which channels exist and how they may be located for replication.

This section defines the semantics, required contents, and discovery guarantees of the @references channel, including the normative definition of the **Channel Reference Artipoint**.

## **8.1 Purpose and Semantics**

The @references channel serves a single, well-defined purpose: **deterministic channel discovery**.

Specifically, the @references channel:

- enumerates all channels defined within the organizational instance,
- provides sufficient metadata to allow replicas to locate and replicate channel logs, and
- ensures that independent replicas discover the same channel universe when operating over the same validated history.

The @references channel does not itself grant authorization to access channels. It defines *what exists*, not *who may participate*. Authorization decisions are made by other ASCP layers once discovery has occurred.

### **8.1.1 Role of @references in Layer-0 Provisioning**

In addition to deterministic discovery, the **@references channel serves as the authoritative export of Layer-0 replication provisioning metadata** for an ASCP organizational instance.

Specifically, validated Channel Reference Artipoints in @references provide replicas with the information required to:

- identify channels as replication objects,
- locate channel logs via ALSP, and
- perform **Layer-0 channel access authorization**, including verification of Channel Access Proofs using Channel Access Key (CAK) public keys.

A replica MAY rely on @references to provision Layer-0 replication capability for channels **even when the replica is not authorized to decrypt or interpret the channel’s payloads**. Possession of CAK public keys enables authorization checks at Layer-0 but does not grant visibility into encrypted channel content or confer membership, governance, or application-level semantics.

This separation allows replicas to participate as relays, mirrors, or backup participants for channels while preserving strict confidentiality and governance enforcement at higher ASCP layers.

## **8.2 Relationship Between @bootstrap and @references**

The @bootstrap and @references channels serve complementary but distinct roles.

- The @bootstrap channel establishes **trust** and enables transition out of the uninitialized state.
- The @references channel establishes **discovery** under normal ASCP trust semantics.

A replica MUST NOT attempt to interpret or rely on @references content until:

1. the @bootstrap channel has been acquired, and
2. the Bootstrap Trust Anchor (RootCA) has been established and validated.

Once these conditions are met, the @references channel becomes the authoritative source for channel discovery.

This separation ensures that discovery operates only after trust exists, while avoiding the need for @bootstrap to carry evolving discovery state.

## **8.3 Channel Reference Artipoint (Normative)**

A **Channel Reference Artipoint** is a signed, immutable declaration recorded in the @references channel that defines the existence and discoverability of an ASCP channel.

Channel Reference Artipoints are the **sole normative mechanism** by which channels are introduced into the organizational discovery space.

### **8.3.1 Canonical Form**

A Channel Reference Artipoint MUST be articulated using the following primary payload form:

```asciidoc
[ uuid, author, timestamp,
  ["channel-ref", <label>, <channel-uuid>]
]
```

Where:

- \<label> is a human-readable channel name, and
- \<channel-uuid> is the UUID that uniquely identifies the channel.

The \<channel-uuid> value is authoritative for channel identity. The label is informational and MAY change over time through additive declarations.

## **8.3.2 Required Attributes**

A Channel Reference Artipoint MUST include the following required attributes:

- **cak\_kid -** Identifies the Channel Access Key (CAK) associated with the channel. The value MUST be a stable key identifier of the form:

```asciidoc
ascp:cak:<channel-uuid>
```

- **cak\_public\_keys -** One or more Ed25519 public keys corresponding to the referenced CAK.
  - At least one public key MUST be designated as **active**.
  - Additional public keys MAY be included to support transitional overlap during CAK rotation.

These attributes MUST be sufficient for a replica to locate the channel log and to perform **Layer-0 channel access authorization** using ALSP.

## **8.3.3 Optional Attributes**

A Channel Reference Artipoint MAY include additional attributes, including but not limited to:

- **declaredIn** — UUID of the Artipoint or channel in which this channel was declared
- **implementation-specific routing hints**

Optional attributes MUST NOT alter the semantic meaning of channel discovery or authorization and MUST NOT be required for interoperability.

## **8.3.4 Normative Semantics**

The following semantics apply to all Channel Reference Artipoints:

- Each Channel Reference Artipoint declares exactly one channel.
- Channel discovery state is **additive**; references are never removed or overwritten.
- Multiple Channel Reference Artipoints referring to the same channel UUID MAY exist and MUST be interpreted according to ordering and supersession (replaces) rules defined by higher ASCP layers.
- Channel Reference Artipoints declare **existence, location, and Layer-0 authorization metadata only**; they do not confer channel membership, payload visibility, or governance semantics.
- Possession of CAK public keys enables verification of Channel Access Proofs but **does not imply authorization to access encrypted channel payloads**.

## **8.3.5 Validation Requirements**

A replica validating a Channel Reference Artipoint MUST:

1. Verify the Artipoint signature under normal ASCP trust rules.
2. Confirm that the author identity is traceable to the organizational trust anchor.
3. Confirm that all required attributes are present and well-formed.
4. Confirm that at least one CAK public key is designated as active.

Failure to validate a Channel Reference Artipoint MUST prevent discovery and replication authorization of the referenced channel.

## **8.4 Discovery Authority and Determinism**

The @references channel is the **sole authoritative source** of channel discovery.

Replicas MUST derive the set of discoverable channels exclusively from validated Channel Reference Artipoints in the @references channel. Replicas MUST NOT infer channels from other logs, local configuration, or out-of-band sources.

Given the same validated @references history, independent replicas MUST discover the same set of channels and resolve them consistently.

## **8.5 Channel Lifecycle Representation**

Channel creation, evolution, and retirement are represented additively in the @references channel.

- New channels are introduced by appending new Channel Reference Artipoints.
- Channel updates MAY be represented by additional Channel Reference Artipoints that supersede or annotate earlier entries.
- Channel retirement or deprecation MUST be expressed explicitly and MUST preserve historical discoverability.

Discovery state MUST NOT be removed or rewritten.

## **8.6 Encryption and Visibility**

The @references channel operates under **normal ASCP channel semantics**.

Accordingly, the @references channel:

- MAY be encrypted,
- MAY enforce membership and access controls, and
- MAY restrict visibility based on authorization.

A replica that cannot decrypt or validate the @references channel MUST treat discovery as incomplete and MUST NOT proceed to replicate additional channels beyond those explicitly permitted.

Decryption capability for the @references channel depends on possession of appropriate local state (e.g., CAKs, CSKs, or key material delivered via BKP). Such key material is not reconstructible from ASCP logs.

## **8.7 Failure and Inconsistency Handling**

If a replica encounters invalid, unverifiable, or conflicting Channel Reference Artipoints, it MUST:

- halt discovery processing for affected entries, and
- preserve all previously validated discovery state.

Replicas MUST NOT attempt to reconcile or repair discovery inconsistencies locally. Resolution occurs only through additional Artipoints recorded in the @references channel.

## **8.8 Section Summary**

The @references channel completes the bootstrap transition by providing a deterministic, auditable channel discovery mechanism.

It:

- defines the universe of channels within an organizational instance,
- specifies the Channel Reference Artipoint as the normative discovery primitive,
- ensures consistent discovery across replicas, and
- preserves historical discovery state through immutable logs.

Sections 9 and 10 define how the @bootstrap and @references channels are used during **genesis bootstrap** and **join bootstrap**, respectively. Section 11 defines how bootstrap discovery is orchestrated over ALSP.

# **9. Genesis Bootstrap (New Organizational Instance)**

Genesis bootstrap is the process by which a **new ASCP organizational instance** is created from an empty state. It establishes the organizational trust boundary, introduces mandatory bootstrap channels, and produces a repository that is ready to accept additional replicas via join bootstrap.

Genesis bootstrap occurs exactly once per organizational instance.

## **9.1 Overview**

Genesis bootstrap creates the initial, immutable foundation upon which all subsequent ASCP coordination depends. Its purpose is to:

- establish the **Bootstrap Trust Anchor (RootCA)**,
- create the mandatory bootstrap channels, and
- record sufficient trust and discovery information to allow future replicas to initialize deterministically.

At the conclusion of genesis bootstrap, the organizational instance MUST satisfy all bootstrap invariants defined in Section 6 and MUST be capable of supporting join bootstrap without further special handling.

## **9.2 RootCA Creation**

During genesis bootstrap, the organizational operator (human or system acting in that role) MUST generate the **Bootstrap Trust Anchor (RootCA)**.

The RootCA:

- MUST be generated prior to authoring any bootstrap Artipoints,
- MUST be scoped exclusively to the organizational instance, and
- MUST be introduced as the cryptographic root of trust for the repository.

The RootCA MUST be articulated into the @bootstrap channel as a signed Artipoint in accordance with the ASCP Trust and Identity Architecture. No other trust anchor MAY precede it.

Optional endorsements or attestations (e.g., PKI, external identity providers) MAY be associated with the RootCA at genesis time, but such endorsements are not required for correctness.

## **9.3 Initial Channel Creation**

Genesis bootstrap MUST create the following channels:

1. **@bootstrap**
2. **@references**

### **9.3.1 Bootstrap Channel Initialization**

The @bootstrap channel MUST be created first.

Its initial contents MUST include:

- the Artipoint introducing the RootCA, and
- a signed declaration identifying the @references channel as the authoritative discovery registry.

No other channels MAY be referenced prior to the creation of @references.

### **9.3.2 References Channel Initialization**

The @references channel MUST be created immediately following the @bootstrap channel.

Its initial contents MUST include:

- a Channel Reference Artipoint declaring the @bootstrap channel, and
- a Channel Reference Artipoint declaring the @references channel itself.

These self-descriptive references establish a closed, discoverable core and ensure that future replicas can deterministically discover the mandatory bootstrap channels.

## **9.4 Genesis Completion Criteria**

Genesis bootstrap is considered complete when all of the following conditions are met:

1. A RootCA has been generated and recorded in the @bootstrap channel.
2. The @bootstrap channel exists and contains valid bootstrap Artipoints.
3. The @references channel exists and contains valid Channel Reference Artipoints for:
   - @bootstrap, and
   - @references.
4. All bootstrap Artipoints are verifiable against the RootCA.

Once these conditions are satisfied, the organizational instance is **initialized** and MAY accept new replicas via join bootstrap.

No replica other than the genesis author SHOULD attempt to perform genesis bootstrap for an existing organizational instance.

## **9.5 Properties of a Genesis-Initialized Instance**

An organizational instance created via genesis bootstrap has the following properties:

- All trust decisions are traceable to a single, immutable RootCA.
- All channel discovery proceeds exclusively via the @references channel.
- All future bootstrap activity occurs through join bootstrap.
- Historical verifiability of trust and discovery state is preserved indefinitely.

These properties are invariant for the lifetime of the organizational instance.

## **9.6 Section Summary**

Genesis bootstrap establishes the **irreversible foundation** of an ASCP organizational instance.

It:

- creates the organizational trust root,
- instantiates mandatory bootstrap channels,
- records the initial discovery state, and
- produces a repository capable of deterministic replication and onboarding.

Section 10 defines **Join Bootstrap**, which describes how new replicas connect to and initialize themselves against an organizational instance created through genesis bootstrap.

# **10. Join Bootstrap (Replica Onboarding)**

Join bootstrap is the process by which a **new replica** connects to an existing ASCP organizational instance and initializes itself into normal ASCP operation.

Unlike genesis bootstrap, join bootstrap does not create trust anchors or mandatory channels. Instead, it acquires, validates, and applies existing bootstrap state in a deterministic manner.

## **10.1 Overview**

The purpose of join bootstrap is to transition a replica from an uninitialized state into an **operationally ready** state, as defined in Section 6.4.

Join bootstrap MUST:

- establish authenticated connectivity to at least one existing replica,
- acquire and validate the @bootstrap channel,
- establish the Bootstrap Trust Anchor (RootCA),
- acquire and validate the @references channel, and
- complete deterministic channel discovery.

Join bootstrap MAY be initiated by human users or autonomous agents and applies uniformly to both.

## **10.2 Initial Trust Inputs (Out-of-Band)**

Prior to initiating join bootstrap, a replica MUST possess at least one **Initial Trust Input (ITI)** sufficient to anchor trust in the bootstrap process.

Initial Trust Inputs are obtained **out of band** from ASCP and ALSP. This document does not mandate a single trust model but defines acceptable categories of inputs.

An implementation MUST clearly identify which Initial Trust Input model(s) it supports.

## **10.3 Supported Initial Trust Input Models**

The following Initial Trust Input models are recognized:

### **10.3.1 Pinned Trust Anchor**

The replica is provisioned with a trusted fingerprint or public key corresponding to the organizational RootCA.

This model provides strong security guarantees and is RECOMMENDED for enterprise and managed deployments.

### **10.3.2 Invitation Artifact**

The replica is provisioned with a signed bootstrap artifact (e.g., file, QR code, or secure bundle) containing sufficient information to authenticate the organizational instance and retrieve the @bootstrap channel.

Invitation artifacts MUST be verifiable against the organizational trust anchor once bootstrap validation completes.

### **10.3.3 Domain- or PKI-Assisted Discovery**

The replica derives trust information via an external trust system (e.g., WebPKI, DNS-based mechanisms) and uses this information to authenticate the organizational instance during bootstrap.

External trust mechanisms MAY strengthen provenance but MUST NOT replace the internal RootCA as the authoritative trust root.

### **10.3.4 Trust-On-First-Use (TOFU)**

The replica accepts the first encountered bootstrap trust anchor without prior verification and pins it for future validation.

TOFU provides weaker security guarantees and SHOULD be used only in environments where stronger initial trust inputs are impractical.

Implementations using TOFU MUST make this trust assumption explicit.

## **10.4 Join Bootstrap Acquisition Sequence**

Using the selected Initial Trust Input, the replica performs the following high-level sequence:

1. Establish authenticated connectivity to an existing replica.
2. Acquire the @bootstrap channel.
3. Establish and validate the Bootstrap Trust Anchor (RootCA).
4. Acquire the @references channel including replication and, where applicable, decryption and validation.
5. Perform deterministic channel discovery.

The detailed mechanics of session establishment, authentication, and log replication are defined by ALSP and are not redefined here.

### 10.4.1 Bootstrap Peer Authentication via identity-ref

During join bootstrap, a replica MAY connect to an existing replica that does not possess the private key of the Bootstrap Trust Anchor (RootCA).

In such cases, the joining replica MUST authenticate the peer as follows:

1\. The joining replica establishes an authenticated ALSP session with the peer.

2\. The peer authenticates using an identity and public key during ALSP session establishment.

3\. The joining replica MUST locate a corresponding **Identity Reference Artipoint** in the validated @bootstrap channel whose `<identity-uuid>` matches the peer’s asserted identity.

4\. The joining replica MUST verify that the public key presented during ALSP authentication matches the `cert_public_jwk` recorded in the matching identity-ref.

5\. The joining replica MUST verify that the identity-ref Artipoint itself is traceable to the Bootstrap Trust Anchor (RootCA).

If any of the above checks fail, the joining replica MUST treat the peer as ineligible to serve join bootstrap and MUST NOT proceed with bootstrap using that peer.

Successful validation establishes that the peer’s identity is **RootCA-traceable via log-anchored bootstrap history**, without requiring the peer to possess RootCA private key material.

## **10.5 Validation and Failure Handling**

During join bootstrap, the replica MUST validate all acquired bootstrap content according to normal ASCP trust rules once the RootCA is established.

If validation of the @bootstrap channel, RootCA, or @references channel fails, the replica:

- MUST treat the join bootstrap attempt as unsuccessful, and
- MUST NOT transition to normal ASCP operation.

Partial or speculative bootstrap state MUST NOT be reused without revalidation.

## **10.6 Completion Criteria**

Join bootstrap is considered complete when:

1. The @bootstrap channel has been acquired and validated.
2. The RootCA has been established as the trust root.
3. The @references channel has been acquired and validated.
4. Channel discovery has completed deterministically.

Once these conditions are satisfied, the replica is **operationally ready** and MAY proceed to replicate additional channels subject to authorization.

## **10.7 Visibility Seeding and @references Acquisition**

This section clarifies the join bootstrap sequence in deployments where the **@references channel is encrypted** and therefore not immediately usable by a joining replica. It defines the **minimal, joiner-scoped asymmetry** permitted during bootstrap to enable secure discovery without leaking organizational structure or identity membership.

### **10.7.1 Separation of Replication, Decryption, and Validation**

During join bootstrap, acquisition of the @references channel MUST be understood as comprising three distinct steps:

1. **Replication** — obtaining opaque @references log entries via ALSP, without interpretation.
2. **Decryption** — obtaining sufficient key material to decrypt @references entries.
3. **Validation** — verifying decrypted Channel Reference Artipoints under the established Bootstrap Trust Anchor (RootCA).

A joining replica MAY replicate encrypted @references entries prior to decryption. However, the replica MUST treat channel discovery as incomplete until decryption and validation have both succeeded.

Possession of encrypted @references data does not constitute channel disclosure.

### **10.7.2 Bootstrap Visibility Seed**

When @references is encrypted, a joining replica requires a **bootstrap visibility seed** to enable initial decryption. This seed MUST be scoped **exclusively** to @references and MUST NOT grant visibility to any other channels.

ASCP provides this capability via the **boot\_keys\_jwe field in the ALSP hello message**, which MAY be used to deliver a joiner-specific bootstrap key package containing one or more channel key envelopes sufficient to enable decryption of @references entries required for deterministic discovery:

- The boot\_keys\_jwe payload is JWE wrapped Bootstrap Key Package to the joining replica’s authenticated session keys.
- It is delivered **out-of-band of ASCP logs**, during ALSP session establishment.
- It enables decryption of @references entries sufficient to perform discovery.

This mechanism allows visibility seeding without publishing per-identity key envelopes into @bootstrap and without enumerating organizational identities.

### **10.7.3 Role of @bootstrap**

The @bootstrap channel remains intentionally minimal and serves only to:

- Establish the organizational trust anchor (RootCA).
- Identify or locate the @references channel.

@bootstrap MUST NOT function as an identity directory, channel list, or key distribution mechanism for non-bootstrap channels. Visibility to @references is unlocked via the bootstrap visibility seed, not by static publication of membership or key material in @bootstrap.

This constraint does not prohibit the presence of Identity Reference Artipoints, which serve solely as a bootstrap-time allowlist for peer authentication and do not enumerate organizational membership or confer authorization.

### **10.7.4 Transition to Normal Provisioning**

Once the joining replica has successfully decrypted and validated @references:

- Deterministic channel discovery completes.
- The replica MAY transition to the DISCOVERED state as defined in Section 12.
- Subsequent channel visibility and access MUST be granted exclusively through normal governance evaluation and Channel keyframe provisioning.

The bootstrap visibility seed, delivered as a Bootstrap Key Package (BKP), is a one-time initialization mechanism and MUST NOT be reused to provision access to additional channels.

### **10.7.5 Failure Handling**

If a joining replica is unable to decrypt or validate @references:

- The replica MUST remain in the BOOTSTRAPPED state.
- The replica MUST NOT infer channel existence from any source other than validated @references content.
- The bootstrap process MAY be retried according to local policy.

Failure to complete this step MUST prevent transition to normal operational behavior.

### **10.7.6 Bootstrap Key Package (BKP)**

When the @references channel is encrypted, the `boot_keys_jwe` field in the ALSP hello message MUST carry a JWE wrapped **Bootstrap Key Package (BKP)** sufficient to enable decryption of @references entries required for deterministic discovery.

The Bootstrap Key Package is a **joiner-scoped, out-of-band visibility artifact** and is not recorded in any ASCP channel log.

#### **10.7.6.1 Purpose and Scope**

The BKP exists solely to seed visibility of the @references channel during join bootstrap.

A BKP:

- MUST be scoped exclusively to the @references channel,
- MUST NOT grant visibility to any other channel,
- MUST NOT function as a general key distribution mechanism, and
- MUST NOT persist beyond completion of bootstrap.

The BKP does not confer authorization, governance roles, or membership semantics. It enables decryption only and does not bypass normal ASCP trust validation.

#### **10.7.6.2 Encoding and Payload Structure**

The BKP payload MUST be encoded as a **JSON object**, encrypted using JWE and delivered via `boot_keys_jwe`.

The decrypted BKP payload MUST have the following structure:

```json
{
  "type": "bootstrap-key-package",
  "version": "1.0",
  "scope": "ascp:channel:@references",

  "keys": [
    <channel-key-envelope>, <channel-key-envelope>
  ],

  "ordering": "oldest-first",
  "active_index": 0
}
```

Where:

- type MUST be "bootstrap-key-package".
- version MUST be "1.0".
- scope MUST identify the @references channel.
- keys MUST be a non-empty ordered array of plaintext JSON formatted **Channel Key Envelopes**, each conforming exactly to the **Channel Key Envelope** structure defined in *ASCP Channels, Section 9.6*.
- ordering MUST be "oldest-first".
- active\_index MUST be an integer index into the keys array indicating the most recent key that senders SHOULD use when encrypting new @references entries.

Each element of keys corresponds to one historical keyframe epoch of the @references channel.

#### **10.7.6.3 Relationship to Bootstrap Key Identifiers**

Encrypted @references entries that rely on bootstrap visibility MUST reference a bootstrap key identifier of the form:

```asciidoc
ascp:bkp:<index>
```

Where \<index> selects the corresponding element in the BKP keys array.

A replica decrypting an @references entry with kid = ascp:bkp:\<index> MUST:

1. Select keys\[index] from the decrypted BKP.
2. Extract the AES key from the selected Channel Key Envelope.
3. Use that key to decrypt the JWE-protected @references entry.

If \<index> is out of range, the replica MUST treat the entry as undecryptable and MUST NOT infer discovery state from it.

### **10.7.6.4 Key History and Rotation**

The BKP MAY contain multiple Channel Key Envelopes in order to support key rotation of the @references channel.

Implementations:

- SHOULD include only a bounded number of historical keys,
- SHOULD include the current key and any immediately preceding keys required for correct decryption of retained @references history, and
- MUST support refreshing the BKP by re-establishing an ALSP session if an older bootstrap key index is encountered.

Historical decryptability of @references entries is preserved through the inclusion of prior keys in the BKP and does not require modification of @bootstrap content.

### **10.7.6.5 Lifecycle Constraints**

The BKP:

- MUST be delivered only during join bootstrap,
- MUST NOT be persisted as a long-lived credential,
- MUST NOT be reused for provisioning access to non-bootstrap channels, and
- MUST be discarded once the replica transitions to the DISCOVERED state.

Channel key material obtained via BKP becomes part of the replica’s local state. Loss of such local state prevents decryption and interpretation of encrypted channels and requires the replica to re-enter join bootstrap in order to restore operational capability.

After discovery completes, all subsequent channel visibility and key provisioning MUST occur exclusively through normal ASCP Channel keyframe mechanisms.

## **10.8 Section Summary**

Join bootstrap enables safe, deterministic onboarding of new replicas into an existing ASCP organizational instance.

It:

- relies on explicit Initial Trust Inputs,
- validates all bootstrap artifacts against the organizational trust root,
- establishes authoritative channel discovery, and
- transitions replicas into normal ASCP operation without special-case behavior.

Section 11 defines how join bootstrap is **orchestrated over ALSP**, including session establishment, replication ordering, and bootstrap-specific sequencing constraints.

# **11. Bootstrap Over ALSP**

This section defines how the ASCP bootstrap process is **orchestrated over the ASCP LogSync Protocol (ALSP)**. It specifies the required sequencing and constraints under which bootstrap-related channel replication occurs.

This section does **not** redefine ALSP message formats, authentication mechanisms, or replication algorithms. Those are defined exclusively in the ALSP specification.

## **11.1 Role of ALSP in Bootstrap**

ALSP serves as the **transport and synchronization substrate** for bootstrap.

During bootstrap, ALSP provides:

- authenticated session establishment between replicas,
- secure retrieval of append-only channel logs, and
- ordered, replay-protected delivery of bootstrap artifacts.

ALSP treats channel payloads as opaque. All semantic interpretation of bootstrap content occurs at higher ASCP layers.

Bootstrap imposes **ordering and gating constraints** on ALSP usage during initialization but does not modify ALSP’s core behavior.

## **11.2 Initial ALSP Session Establishment**

Prior to any bootstrap channel replication, a replica MUST establish an authenticated ALSP session with at least one existing replica.

ALSP session establishment during bootstrap MUST provide:

- peer authentication sufficient to bind the session to the intended organizational instance,
- integrity protection for replicated logs, and
- replay resistance for bootstrap artifacts.

The specific mechanisms used to satisfy these requirements are defined by ALSP and MAY vary by implementation.

A replica MUST NOT replicate any non-bootstrap channels prior to successful session establishment.

## **11.3 Bootstrap Channel Replication Sequence**

Bootstrap imposes a strict replication order over ALSP.

### **11.3.1 Replication of @bootstrap**

The @bootstrap channel MUST be the **first channel replicated** during bootstrap.

During this phase:

- the replica acquires the complete @bootstrap log,
- the replica MAY temporarily assume bootstrap content validity in order to construct a minimal trust graph, and
- no other channels MAY be interpreted or relied upon.

The replica MUST establish the Bootstrap Trust Anchor (RootCA) from @bootstrap content before proceeding.

### **11.3.2 Validation Gate**

After acquisition of the @bootstrap channel:

- the replica MUST validate bootstrap content using normal ASCP trust rules once the RootCA is established, and
- failure to validate MUST abort the bootstrap process.

No further channel replication is permitted until this validation gate is satisfied.

## **11.4 Discovery Channel Replication Sequence**

Once the validation gate is satisfied, the replica MAY proceed to replicate the @references channel.

During this phase:

- the replica acquires and validates the @references channel,
- Channel Reference Artipoints are interpreted to construct the discovery set, and
- no additional channels MAY be replicated until discovery completes deterministically.

The @references channel MUST be validated under normal ASCP trust semantics. In deployments where the @references channel is encrypted, successful acquisition for discovery purposes additionally requires decryption and validation as described in Section 10.7.

## **11.5 Transition to Normal Replication**

After successful acquisition and validation of both @bootstrap and @references, the replica transitions into **normal ALSP replication mode**.

In this mode:

- channel replication proceeds according to authorization and subscription logic,
- channel encryption and membership rules apply normally, and
- no special bootstrap sequencing constraints remain in effect.

From this point forward, ALSP operates without bootstrap-specific restrictions.

## **11.6 Error Handling and Restart Semantics**

If any bootstrap step fails during ALSP-mediated replication, the replica MUST:

- treat the bootstrap attempt as unsuccessful, and
- discard or quarantine unvalidated bootstrap state.

A replica MAY retry bootstrap by re-establishing an ALSP session and re-acquiring bootstrap channels.

Implementations SHOULD ensure that partial bootstrap attempts do not leak into normal operation.

## **11.7 Section Summary**

Bootstrap over ALSP is defined by **ordering and gating**, not by new protocol mechanisms.

ALSP provides the secure transport and replication substrate, while this document specifies:

- which channels MUST be replicated first,
- when validation must occur, and
- when a replica may safely transition to normal operation.

This separation preserves ALSP’s generality while ensuring deterministic and secure bootstrap behavior.

Section 12 defines the **Bootstrap State Machine**, which formalizes bootstrap progress and failure handling independent of transport.

# **12. Bootstrap State Machine**

This section defines the **bootstrap state machine** for ASCP replicas. The state machine provides a normative model for tracking bootstrap progress, enforcing ordering constraints, and handling failure conditions during initialization.

The state machine applies uniformly to both **genesis bootstrap** and **join bootstrap**, with differences reflected only in how initial state is entered.

## **12.1 States**

An ASCP replica MUST be in exactly one of the following bootstrap states at any time.

### **12.1.1 UNINITIALIZED**

The replica has no established trust context for an ASCP organizational instance.

In this state:

- no organizational trust anchor is established,
- no bootstrap channels are validated, and
- the replica MUST NOT interpret ASCP coordination artifacts.

A replica enters this state prior to any bootstrap activity.

### **12.1.2 BOOTSTRAPPED**

The replica has successfully acquired the @bootstrap channel and established the Bootstrap Trust Anchor (RootCA).

In this state:

- the @bootstrap channel has been acquired,
- the RootCA has been established as the trust root, and
- bootstrap content has been validated or is in the process of validation.
- any bootstrap-serving peer identities used during ALSP session establishment have been validated against Identity Reference Artipoints in the @bootstrap channel.

In this state, the replica MAY acquire the @references channel but MUST NOT replicate additional channels.

### **12.1.3 DISCOVERED**

The replica has successfully acquired and validated the @references channel and completed deterministic channel discovery.

In this state:

- the discovery set is fully constructed,
- all mandatory bootstrap invariants are satisfied, and
- the replica is eligible to transition to normal operation.

No application or coordination channels are yet required to be replicated.

### **12.1.4 OPERATIONAL**

The replica has completed bootstrap and is operating under normal ASCP semantics.

In this state:

- channel replication proceeds according to authorization and subscription logic,
- no bootstrap-specific ordering constraints apply, and
- the replica participates fully in ASCP coordination.

## **12.2 State Transitions**

State transitions are monotonic and MUST follow the ordering defined below.

### **12.2.1 UNINITIALIZED → BOOTSTRAPPED**

This transition occurs when:

- an authenticated ALSP session is established, and
- the @bootstrap channel is acquired and the RootCA is established.

Failure to validate bootstrap content MUST prevent this transition.

### **12.2.2 BOOTSTRAPPED → DISCOVERED**

This transition occurs when:

- the @references channel is acquired, and
- all Channel Reference Artipoints are validated successfully.

For encrypted deployments, acquisition of @references includes successful decryption using an appropriate bootstrap visibility seed and subsequent validation of Channel Reference Artipoints under the established Bootstrap Trust Anchor (RootCA), as defined in **Section 10.7**.

If discovery fails or is incomplete, the replica MUST remain in the BOOTSTRAPPED state.

### **12.2.3 DISCOVERED → OPERATIONAL**

This transition occurs when:

- deterministic channel discovery completes, and
- all bootstrap readiness criteria defined in Section 6.4 are satisfied.

Upon entering OPERATIONAL, the replica MAY begin normal channel replication.

## **12.3 Failure Transitions**

Bootstrap failures MAY occur at any stage prior to OPERATIONAL.

If a failure occurs:

- the replica MUST NOT advance to a higher state,
- partial bootstrap state MUST NOT be treated as valid, and
- the replica MAY reattempt bootstrap by restarting from UNINITIALIZED or BOOTSTRAPPED, depending on implementation policy.

Failures MUST NOT cause the replica to enter OPERATIONAL.

## **12.4 Re-bootstrap and Recovery**

A replica MAY re-enter the bootstrap state machine under the following conditions:

- loss or invalidation of trust anchors,
- detection of corrupted bootstrap data, or
- explicit administrative reset.

Re-bootstrap MUST preserve the invariant that bootstrap state is revalidated before returning to OPERATIONAL.

## **12.5 State Machine Invariants**

The following invariants MUST hold:

- A replica MUST NOT enter OPERATIONAL without passing through BOOTSTRAPPED and DISCOVERED.
- A replica MUST NOT interpret @references content before establishing the RootCA.
- A replica MUST NOT replicate non-bootstrap channels prior to OPERATIONAL.

Violation of these invariants indicates a non-compliant implementation.

## **12.6 Section Summary**

The bootstrap state machine provides a clear, enforceable model for ASCP initialization, defining the ordering and validation gates that govern the transition to operational readiness.

# **13. Security Considerations**

This section discusses security considerations specific to the ASCP bootstrap process and channel discovery mechanisms, focusing on risks introduced by initialization, first contact, and exceptional bootstrap behavior.

## **13.1 Threat Model (High-Level)**

The bootstrap process assumes an adversarial environment in which:

- network communication may be observed, delayed, replayed, or modified,
- replicas may connect to untrusted or malicious peers,
- bootstrap artifacts may be captured and replayed out of context, and
- some Initial Trust Input models may provide weaker guarantees than others.

Bootstrap security is concerned primarily with **preventing incorrect trust establishment** and **ensuring deterministic discovery**, rather than with availability or performance.

## **13.2 Bootstrap Trust Risks**

### **13.2.1 Incorrect Trust Anchor Establishment**

If a replica accepts an incorrect Bootstrap Trust Anchor (RootCA), all subsequent trust decisions become invalid.

Mitigations include:

- explicit Initial Trust Inputs,
- post-bootstrap validation of all bootstrap artifacts, and
- pinning of validated trust anchors for future sessions.

Implementations SHOULD prefer stronger Initial Trust Input models where available.

### **13.2.2 Trust-On-First-Use (TOFU) Exposure**

Trust-on-first-use provides no protection against active man-in-the-middle attacks during initial contact.

Implementations that support TOFU MUST:

- make the trust assumption explicit to users or operators, and
- ensure that accepted trust anchors are pinned and revalidated on subsequent connections.

TOFU SHOULD NOT be used in environments where stronger trust inputs are practical.

## **13.3 Unencrypted @bootstrap Channel Exposure**

The @bootstrap channel is readable prior to full authorization and therefore represents a high-value target.

Security properties of the @bootstrap channel rely on:

- authenticated ALSP session establishment,
- cryptographic authentication of all bootstrap Artipoints, and
- strict minimization of bootstrap content.

The @bootstrap channel MUST NOT be used to convey sensitive coordination data, authorization state, or application-level information.

## **13.4 Replay and Substitution Attacks**

Bootstrap artifacts may be replayed or substituted by an attacker attempting to confuse or mislead a replica.

Mitigations include:

- ALSP-level replay prevention during bootstrap acquisition,
- signature verification of all bootstrap and discovery Artipoints, and
- validation of author identities against the organizational trust anchor.

Replicas MUST reject bootstrap artifacts that cannot be validated in context.

## **13.5 Discovery Integrity Risks**

Incorrect or inconsistent channel discovery undermines shared cognition by causing replicas to disagree about what coordination structures exist.

Discovery integrity is ensured through exclusive reliance on the @references channel, deterministic interpretation of Channel Reference Artipoints, and immutable, auditable discovery history. Replicas MUST NOT infer channel existence from any source other than validated @references content.

## **13.6 Compromised or Malicious Replicas**

A compromised replica may attempt to serve invalid bootstrap or discovery content.

Mitigations include:

- independent validation of all bootstrap artifacts,
- rejection of unverifiable or improperly authored Artipoints, and
- reliance on log-anchored history rather than live assertions.

No single replica is trusted implicitly beyond its ability to present verifiable log content.

## **13.7 Divergent Bootstrap Views**

ASCP does not attempt to automatically reconcile divergent or conflicting bootstrap histories. A replica that encounters bootstrap or discovery artifacts that cannot be validated against its established trust anchor MUST treat those artifacts as invalid and MUST NOT incorporate them into its local coordination state.

Replicas are expected to rely on **validated, log-anchored history** rather than live assertions or peer claims when determining bootstrap correctness. Resolution of divergent bootstrap views, if required, is an administrative or operational concern external to the protocol and may involve re-provisioning, explicit trust re-establishment, or replica reset.

This design avoids implicit authority resolution and ensures that bootstrap correctness remains auditable, deterministic, and consistent across replicas.

## **13.8 Recovery and Revocation Considerations**

Bootstrap does not support destructive revocation of historical trust decisions. If a trust anchor or bootstrap artifact is later determined to be compromised:

- remediation MUST occur through additive, auditable Artipoints, and
- historical bootstrap state MUST remain verifiable.

Recovery procedures are addressed in Section 14.

## **13.9 Section Summary**

Bootstrap security in ASCP is achieved through:

- explicit trust establishment,
- minimal and auditable bootstrap artifacts,
- deterministic discovery mechanisms, and
- strict separation between bootstrap exceptions and normal operation.

By constraining exceptional behavior to a narrow initialization window and subjecting all bootstrap artifacts to post-hoc validation, ASCP preserves its core guarantees of durable, verifiable shared context even in adversarial environments.

# **14. Operational Considerations (Non-Normative)**

This section provides non-normative guidance on operational and deployment considerations related to the ASCP bootstrap process. The topics discussed here are intended to assist implementers and operators in deploying ASCP systems robustly, but they do not introduce new protocol requirements.

## **14.1 Key Rotation and Re-bootstrap**

ASCP bootstrap is intentionally designed to avoid destructive changes to trust history. As a result:

- The Bootstrap Trust Anchor (RootCA) is not replaced or revoked.
- Key rotation and trust evolution occur through **additive mechanisms**, preserving historical verifiability.

Operationally, this means:

- New signing keys or certificates are introduced via additional Artipoints.
- Replicas joining after key evolution will observe the full trust history and validate it deterministically.
- Existing replicas do not need to re-bootstrap unless local trust state is lost or corrupted.

Re-bootstrap SHOULD be treated as an exceptional recovery action, not a routine operation.

## **14.2 Recovery from Partial or Failed Bootstrap**

Bootstrap attempts may fail due to network interruption, validation errors, or incorrect Initial Trust Inputs.

Recommended practices include:

- discarding or quarantining unvalidated bootstrap state,
- restarting bootstrap from a clean state when validation fails, and
- avoiding reuse of partially validated trust material without revalidation.

Because bootstrap artifacts are immutable and replayable, recovery typically consists of retrying acquisition and validation rather than reconstructing state.

## **14.3 Multi-Node and Server-Assisted Deployments**

In deployments where multiple replicas act as servers or gateways:

- any replica capable of serving validated bootstrap logs may assist in onboarding new replicas,
- no single server is inherently trusted beyond its ability to present verifiable log history, and
- replicas may switch bootstrap sources without affecting correctness.

Server-assisted deployments do not alter bootstrap semantics; they affect only availability and performance characteristics.

## **14.4 Offline and Intermittent Connectivity**

ASCP bootstrap is compatible with offline or intermittently connected environments provided that bootstrap artifacts can be transferred securely.

Operational patterns may include:

- importing bootstrap artifacts from removable media,
- using invitation artifacts or pre-provisioned trust anchors, and
- delaying full discovery until connectivity is restored.

Once bootstrap validation completes, replicas can operate autonomously using locally replicated logs.

## **14.5 Caching and Persistence**

Implementations typically cache validated bootstrap artifacts locally to:

- accelerate reconnection,
- support offline operation, and
- reduce repeated bootstrap overhead.

Cached bootstrap state SHOULD be treated as sensitive trust material and protected accordingly. Loss of cached state does not compromise correctness but may require re-bootstrap.

## **14.6 Operational Summary**

Operationally, ASCP bootstrap is intended to be:

- **rare** (performed once per replica),
- **deterministic** (replayable without ambiguity), and
- **recoverable** (restartable without special handling).

By relying on immutable, log-anchored artifacts, ASCP minimizes operational fragility while preserving long-term auditability and trust continuity.

# **15. Out of Scope**

This document deliberately excludes the following topics, which are addressed in other ASCP specifications or reserved for future work:

- **Cryptographic primitives and algorithms:** Signature schemes, hashing algorithms, encryption mechanisms, and key derivation functions are defined in the ASCP Trust and Identity Architecture and related cryptographic profiles.
- **Identity semantics and governance policy:** Rules governing identity verification, authorization, membership, delegation, and governance are defined outside this document.
- **Channel encryption and key management internals:** Channel access keys (CAKs), key rotation mechanisms, and membership enforcement are defined in the ASCP Channels specification.
- **Log synchronization mechanics and convergence guarantees:** Wire formats, message flows, replication algorithms, and convergence validation are defined exclusively by the ASCP LogSync Protocol (ALSP).
- **Federation and cross-organizational trust:** Discovery, trust negotiation, or synchronization across multiple ASCP organizational instances are not defined here.
- **Application- or domain-specific semantics:** This document does not define how applications interpret or act upon coordination artifacts once bootstrap is complete.

These exclusions are intentional. This document defines only the **bootstrap ordering, invariants, and readiness criteria** required to initialize an ASCP organizational instance and onboard replicas safely.

# **16. IANA Considerations**

This document has no IANA considerations.

It does not define new protocol numbers, registries, MIME types, or other IANA-managed namespaces.

If future revisions introduce such requirements, this section will be updated accordingly.

# **17. References**

## **17.1 Normative References**

- **RFC 2119:** *Key words for use in RFCs to Indicate Requirement Levels.*
- **RFC 8174:** *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words.*
- **ASCP Trust and Identity Architecture:** Defines cryptographic identity constructs, trust anchors, certificate semantics, and validation rules used during bootstrap.
- **ASCP LogSync Protocol (ALSP):** Defines authenticated session establishment, log replication, ordering, and replay protection mechanisms used during bootstrap.

## **17.2 Informative References**

- **ASCP Channels Specification:** Defines channel membership, encryption, and distribution semantics applied after bootstrap.
- **Agents Shared Cognition Protocol (ASCP) Core Specification:** Describes the overall ASCP architecture and layer model.

# **Appendix A - Bootstrap Design Rationale**

## **A.1 Overview**

The Agents Shared Cognition Protocol (ASCP) is designed to support durable, auditable collaboration between humans and autonomous agents over long time horizons. To achieve this, ASCP makes several deliberate architectural separations that differ from common practice in messaging systems, access-controlled data stores, and real-time collaboration platforms.

This appendix explains the rationale for these design choices in a descriptive manner, focusing on how they support ASCP’s core invariants: log-anchored trust, deterministic interpretation, privacy-preserving distribution, and future-verifiable coordination history.

## **A.2 Separation of Authentication, Authorization, and Visibility**

ASCP treats **authentication**, **authorization**, and **visibility** as independent concerns, each addressed by a distinct protocol mechanism.

- **Authentication** establishes authorship. It answers the question: *Who authored this statement?*
- Authentication is cryptographic and objective. Every Artipoint is individually signed and attributable to a specific key, enabling durable provenance and replayable verification.
- **Authorization** determines semantic effect. It answers the question: *Should this statement have effect in this context?*
- Authorization is evaluated declaratively through governance semantics applied to immutable history. Authorization may vary by scope, time, or role, and does not affect the existence or authenticity of statements.
- **Visibility** determines disclosure. It answers the question: *Who can receive and decrypt this statement?*
- Visibility is enforced cryptographically through ASCP Channels and key distribution, independent of semantic authorization.

By separating these axes, ASCP avoids conflating authorship, authority, and audience. This allows statements to exist immutably even if they are unauthorized, overridden, or visible only to a limited set of participants.

## **A.3 Immutable History and Non-Authoritative Statements**

ASCP preserves all authenticated articulations as part of an append-only log, regardless of whether they are ultimately authorized or operative.

This design supports several properties:

- **Auditability across time**: Verifiers can reconstruct not only what took effect, but what was proposed, contested, or superseded.
- **Deterministic interpretation**: Governance rules determine semantic effect without requiring deletion or mutation of history.
- **Symmetry between humans and agents**: All participants may articulate statements; authority is evaluated, not assumed.

Unauthorized or non-operative articulations are treated as a governance concern, not a transport or security failure. Resource control and abuse prevention are handled at channel admission and replication layers rather than by suppressing history.

## **A.4 Visibility as a Cryptographic Boundary**

ASCP Channels define visibility boundaries through encryption and key distribution. Possession of encrypted log material does not imply semantic disclosure.

Key properties of this model include:

- Decryption capability, not log replication, defines visibility.
- Governance semantics may inform who should receive keys, but do not themselves grant decryption capability.
- Channels act as audience and privacy boundaries, not as semantic or topical classifiers.

This approach allows replicas to participate in synchronization, mirroring, or archival roles without gaining access to plaintext content, and supports privacy-preserving collaboration across organizational and trust boundaries.

## **A.5 Bootstrap Asymmetry and Initial Discovery**

ASCP derives all trust and coordination semantics from immutable log history. This creates an inherent bootstrap dependency: trust evaluation and channel discovery require access to specific logs, while access to those logs may itself depend on trust and keys.

ASCP resolves this with a narrow, explicit bootstrap asymmetry:

- A minimal **@bootstrap** channel anchors the organizational trust root and provides pointers required to initiate discovery, **without carrying encrypted payloads or visibility-granting material**.
- Joiner-specific key material required to decrypt encrypted bootstrap artifacts (e.g., **@references**) is delivered via authenticated session mechanisms during initial connection.
- After bootstrap completes, all trust, discovery, and visibility decisions are derived exclusively from validated log history under normal ASCP rules.

This asymmetry is limited in scope, auditable, and non-recurring. Once bootstrap completes, ASCP operates as a fully symmetric, self-describing system.

This design avoids turning the bootstrap channel into an identity directory or key distribution surface, preserving privacy and minimizing exposed organizational structure.

## **A.6 Validated Discovery**

Channel discovery in ASCP is considered complete only after referenced discovery artifacts are both:

1. **Decryptable**, using properly provisioned channel keys, and
2. **Verifiable**, under normal trust evaluation rules anchored at the organizational trust root.

Replication of encrypted discovery logs prior to decryption is permitted, but does not constitute semantic discovery. Channel existence and interpretation are derived solely from decrypted and validated discovery statements.

## **A.7 Progressive Trust States**

ASCP allows identities and trust relationships to evolve over time. Authentication, recognition under a trust anchor, and external endorsement are distinct and additive states.

This enables:

- Local-first operation prior to institutional endorsement
- Later convergence on stronger trust guarantees
- Historical verification of statements under the trust state that existed at the time of authorship

Trust is evaluated at log-time, not join-time, preserving the ability to reason accurately about past coordination.

## **A.8 Architectural Summary**

The ASCP architecture prioritizes reproducibility, privacy, and durable shared meaning over immediate convenience. By maintaining strict separation between authentication, authorization, and visibility, and by treating bootstrap as an explicit phase transition rather than an implicit assumption, ASCP enables long-lived, auditable collaboration between humans and agents without reliance on centralized control or mutable state.

