# ASCP Terminology Primer

Version: 0.50 — Informational  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# Status of This Document

This document is an **informative, non-normative primer** intended to clarify the terminology used across the Agents Shared Cognition Protocol (ASCP) specification suite. It provides conceptual grounding, definitional precision, and naming rationale for core ASCP terms. Its purpose is twofold: first, to educate readers encountering ASCP for the first time; and second, to serve as a **terminological reference** for authors, reviewers, and implementers when drafting, reviewing, or maintaining ASCP specifications.

This document does not define protocol requirements and does not introduce new semantics beyond those already present in the normative specifications. Where key words such as MUST or SHOULD appear, they are used descriptively rather than normatively.

# Articulation and Articulation Work

ASCP is grounded in the concept of **articulation work** as developed in the Computer-Supported Cooperative Work (CSCW) literature. Articulation work refers to the meta-level activity through which cooperative work is made possible: the ongoing effort to partition tasks, align understanding, negotiate meaning, establish dependencies, and reintegrate results. This work is structural rather than procedural, and contextual rather than content-bearing. It concerns *how* work is organized and related, not *what* the work product itself contains.

In most contemporary systems, articulation work is implicit, informal, and transient. It is scattered across conversations, documents, and tool-specific state, and is rarely preserved in a durable or interoperable form. ASCP addresses this gap by treating articulation work as **first-class, persistent data**, expressed through a formal grammar and preserved in an immutable coordination history.

The term **Articulation** in ASCP therefore refers to an *act of coordination*: a discrete authored move that expresses or modifies shared understanding. Articulation is inherently active and temporal. It is something that is *performed* by a human or agent at a moment in time.

# Artipoint

An **Artipoint** is a conceptual, immutable unit of articulated meaning within the ASCP coordination model. The term is a coined contraction of *point of articulation*, emphasizing that an Artipoint represents a precise and atomic outcome of articulation work.

An Artipoint is a semantic construct, not a data structure. It names a stable unit of meaning that can be referenced, related, extended, or superseded over time. Examples include the articulation of an identity, the declaration of relevance between artifacts, the establishment of a dependency, or the introduction of a collaborative structure such as a Stream or Space.

Artipoints are **atomic with respect to coordination semantics**. Each Artipoint has a single author, a single articulation time, and a single semantic intent. Larger collaborative structures do not constitute larger Artipoints; instead, they emerge from graphs of many Artipoints related through further articulation.

An Artipoint is independent of any particular encoding, serialization, cryptographic signature, or transport mechanism. It exists at the semantic layer of ASCP and is the foundational unit of the shared cognition graph.

Unless otherwise qualified, the term *Artipoint* refers to this semantic concept.

# Artipoint Expression

An **Artipoint Expression** is the Layer-2 grammatical representation of an Artipoint, as defined by the ASCP Artipoint Grammar. It is the syntactic form through which an Artipoint’s semantic content is made explicit and machine-parseable.

An Artipoint Expression encodes the structural fields necessary to represent an Artipoint, including its identifier, author reference, timestamp, and articulation expression. It is a representation *of* an Artipoint, not an act of articulation itself.

Artipoint Expressions are purely structural artifacts. They do not carry cryptographic guarantees, imply trust or authority, or define visibility or distribution. Those concerns are addressed by lower protocol layers. The role of the Artipoint Expression is to provide a deterministic, canonical form suitable for signing, encryption, replication, and interpretation.

# Articulation Statement

An **Articulation Statement** is a single, atomic act of articulation expressed using the ASCP grammar. Each Articulation Statement corresponds to exactly one coordination move performed by an author at a specific time.

While an Artipoint names a unit of meaning, an Articulation Statement names the *act* that introduces, relates, annotates, or constructs that meaning within the shared coordination graph. This distinction is intentional and reflects the CSCW view that articulation work consists of discrete, authored coordination actions.

An Articulation Statement is therefore act-oriented rather than object-oriented. It is not itself an Artipoint, but rather the grammatical vehicle through which Artipoints come into being or are related to one another.

# Articulation Sequence

An **Articulation Sequence** is an ordered collection of one or more Articulation Statements authored together by the same identity. It represents a compound episode of articulation work.

The Articulation Sequence is the unit passed from the articulation layer to the secure distribution layer. All statements in a sequence share authorship and are jointly secured during channel processing. Conceptually, the sequence captures the fact that coordination work often occurs in small clusters of related actions rather than in isolation.

An Articulation Sequence is not a semantic object in its own right and does not introduce additional meaning beyond the statements it contains. Its purpose is organizational and operational, not semantic.

# Artipoint Record

An **Artipoint Record** is the durable, cryptographically secured materialization of an Articulation Sequence within an ASCP Channel log. It is the artifact over which trust, provenance, visibility, and replication semantics operate.

An Artipoint Record encapsulates a serialized Articulation Sequence within a Channel Envelope, applies cryptographic signing to establish authorship integrity, and may apply encryption to enforce visibility scope. Once appended to a Channel log, an Artipoint Record is immutable and addressable.

The use of the term *Artipoint* in this context reflects that the record exists to preserve articulated meaning, not merely to log events. While the articulation act has passed, the resulting record serves as the durable carrier of the Artipoints introduced or affected by that act.

# Channel Log

A **Channel Log** is the concrete, protocol-level append-only log maintained for a specific ASCP Channel.

Each Channel Log contains an ordered sequence of **Artipoint Records**, where each record encapsulates a signed and optionally encrypted **Articulation Sequence** distributed within that Channel. The Channel Log is the unit of durable storage, replication, and convergence for articulated coordination scoped to a particular visibility and trust domain.

A Channel Log has the following defining properties:

- It is **append-only**: records are never modified or removed once written.
- It is **cryptographically scoped**: only participants holding the Channel’s keys can decrypt and interpret its contents.
- It is **replica-synchronized**: multiple replicas converge on the same ordered history via the ASCP LogSync Protocol (ALSP).
- It is **semantically opaque to lower layers**: transport and synchronization treat records as uninterpreted payloads.

A Channel Log does not itself define meaning, authority, or governance. It merely preserves and distributes articulated coordination within the Channel’s scope. Interpretation of the recorded Artipoints, evaluation of governance rules, and materialization of collaborative structures occur at higher layers.

Each Channel in ASCP maintains exactly one Channel Log. There is no global or cross-Channel log.

# Coordination Log

The **Coordination Log** is the architectural abstraction representing the durable, append-only history through which articulated coordination is preserved, replayed, and audited within ASCP.

Conceptually, the Coordination Log is not a semantic object and does not itself constitute meaning. Meaning arises from **Artipoints** introduced and related through articulation. The Coordination Log preserves the *temporal evolution* of those articulations—recording *when*, *by whom*, and *in what order* coordination was expressed—without collapsing that history into a single authoritative state.

From an architectural perspective, the Coordination Log fulfills several essential roles:

- It provides a **complete and auditable memory of articulation**, preserving authorship independently of later authorization or governance decisions.
- It enables **deterministic reconstruction of shared context** by replaying articulated history in order.
- It supports **local-first replication and convergence**, allowing distributed participants to synchronize context without central authority.
- It anchors **trust and provenance**, enabling verification of authorship, signatures, and governance semantics against immutable history.

The Coordination Log is **realized concretely as Channel Logs**. Each ASCP Channel maintains its own append-only Channel Log, scoped by cryptographic visibility boundaries and synchronized via the ASCP LogSync Protocol (ALSP). There is no global Coordination Log spanning multiple Channels.

In summary, the Coordination Log names the *architectural role* played by Channel Logs: it is the **immutable memory of articulation** through which shared meaning can persist, be interpreted, and evolve over time.

# Terminological Rationale

The coexistence of the terms *Artipoint* and *Articulation* across ASCP is deliberate and reflects a fundamental conceptual distinction. **Artipoint** names the semantic object: the stable unit of meaning within the coordination graph. **Articulation** names the act: the authored coordination work through which meaning is introduced, modified, or related.

This distinction mirrors long-standing CSCW theory, in which articulation work is understood as an active, ongoing process, while the structures it produces endure beyond any single act. By preserving both terms and assigning them consistent grammatical roles, ASCP is able to model both coordination dynamics and durable shared understanding without conflating the two.

In practical terms, ASCP uses *Articulation* to name act-level constructs and syntactic groupings, such as Statements and Sequences, and uses *Artipoint* to name semantic entities and their durable representations, such as Expressions and Records. This naming scheme avoids ambiguity, preserves theoretical fidelity, and provides a clear guide for consistent usage across specifications.

# Intended Use

This primer is intended to be read alongside the ASCP architectural overview and normative specifications. It may be cited informatively, used as a checklist when reviewing specification language, or consulted as a reference to ensure terminological consistency across documents and implementations.

By making the underlying conceptual model explicit, this document aims to reduce ambiguity, improve readability, and support the long-term coherence of the ASCP specification suite.

# Table 1: Artipoint vs. Articulation

| **Conceptual Role** | **Correct Term**       |
| ------------------- | ---------------------- |
| Semantic unit       | Artipoint              |
| Grammar form        | Artipoint Expression   |
| Act                 | Articulation Statement |
| Batch of acts       | Articulation Sequence  |
| Logged artifact     | Artipoint Record       |

# Table 2: Coordination Log vs. Channel Log

| **Concept**          | **Scope**     | **Nature**              |
| -------------------- | ------------- | ----------------------- |
| **Coordination Log** | Architectural | Abstract role           |
| **Channel Log**      | Per-Channel   | Concrete data structure |

