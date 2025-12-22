# **ASCP Terminology Primer**

**Terminology and Architectural Layering Reference**

Version: 0.61 — Informational  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is an **informative, non-normative primer** intended to define and stabilize the terminology used across the Agents Shared Cognition Protocol (ASCP) specification suite.

It serves as the **authoritative terminological reference** for ASCP, covering:

- Core coordination primitives (Artipoints, Articulation, Channels, Logs)
- The conceptual distinction between acts, semantic units, and durable records
- The architectural layering model used throughout ASCP specifications

This document does **not** define protocol requirements and does not introduce new semantics beyond those present in the normative ASCP specifications. Where key words such as MUST or SHOULD appear, they are used descriptively rather than normatively.

# **2. Articulation and Articulation Work**

ASCP is grounded in the concept of **articulation work** as developed in the Computer-Supported Cooperative Work (CSCW) literature. Articulation work refers to the meta-level activity through which cooperative work is made possible: the ongoing effort to partition tasks, align understanding, negotiate meaning, establish dependencies, and reintegrate results.

This work is structural rather than procedural, and contextual rather than content-bearing. It concerns *how* work is organized and related, not *what* the work product itself contains.

In most contemporary systems, articulation work is implicit, informal, and transient. It is scattered across conversations, documents, and tool-specific state, and is rarely preserved in a durable or interoperable form.

ASCP addresses this gap by treating articulation work as **first-class, persistent data**, expressed through a formal grammar and preserved in an immutable coordination history.

The term **Articulation** in ASCP therefore refers to an *act of coordination*: a discrete authored move that expresses or modifies shared understanding. Articulation is inherently active and temporal. It is something that is *performed* by a human or agent at a moment in time.

# **3. Artipoint**

An **Artipoint** is a conceptual, immutable unit of articulated meaning within the ASCP coordination model. The term is a coined contraction of *point of articulation*, emphasizing that an Artipoint represents a precise and atomic outcome of articulation work.

An Artipoint is a **semantic construct**, not a data structure. It names a stable unit of meaning that can be referenced, related, extended, or superseded over time.

Examples include:

- the articulation of an identity
- the declaration of relevance between artifacts
- the establishment of a dependency
- the introduction of a collaborative structure such as a Stream or Space

Artipoints are **atomic with respect to coordination semantics**. Each Artipoint has a single author, a single articulation time, and a single semantic intent. Larger collaborative structures do not constitute larger Artipoints; instead, they emerge from graphs of many Artipoints related through further articulation.

An Artipoint exists at the semantic layer of ASCP and is independent of any particular encoding, serialization, cryptographic signature, or transport mechanism.

Unless otherwise qualified, the term *Artipoint* refers to this semantic concept.

# **4. Artipoint Expression**

An **Artipoint Expression** is the Layer-2 grammatical representation of an Artipoint, as defined by the ASCP Artipoint Grammar.

An Artipoint Expression encodes the structural fields necessary to represent an Artipoint, including its identifier, author reference, timestamp, and articulation expression. It is a representation *of* an Artipoint, not an act of articulation itself.

Artipoint Expressions are purely structural artifacts. They do not carry cryptographic guarantees, imply trust or authority, or define visibility or distribution. Those concerns are addressed by lower protocol layers.

The role of the Artipoint Expression is to provide a deterministic, canonical form suitable for signing, encryption, replication, and semantic interpretation.

# **5. Articulation Statement**

An **Articulation Statement** is a single, atomic act of articulation expressed using the ASCP grammar. Each Articulation Statement corresponds to exactly one coordination move performed by an author at a specific time.

While an Artipoint names a unit of meaning, an Articulation Statement names the *act* that introduces, relates, annotates, or constructs that meaning within the shared coordination graph.

This distinction is intentional and reflects the CSCW view that articulation work consists of discrete, authored coordination actions.

An Articulation Statement is therefore act-oriented rather than object-oriented. It is not itself an Artipoint, but rather the grammatical vehicle through which Artipoints come into being or are related to one another.

# **6. Articulation Sequence**

An **Articulation Sequence** is an ordered collection of one or more Articulation Statements authored together by the same identity. It represents a compound episode of articulation work.

The Articulation Sequence is the unit passed from the articulation layer to the secure distribution layer. All statements in a sequence share authorship and are jointly secured during Channel processing.

An Articulation Sequence is not a semantic object in its own right and does not introduce additional meaning beyond the statements it contains. Its purpose is organizational and operational, not semantic.

# **7. Artipoint Record**

An **Artipoint Record** is the durable, cryptographically secured materialization of an Articulation Sequence within an ASCP Channel Log.

An Artipoint Record encapsulates a serialized Articulation Sequence within a Channel envelope, applies cryptographic signing to establish authorship integrity, and may apply encryption to enforce visibility scope.

Once appended to a Channel Log, an Artipoint Record is immutable and addressable. While the articulation act has passed, the resulting record serves as the durable carrier of the Artipoints introduced or affected by that act.

# **8. Channel Log**

A **Channel Log** is the concrete, protocol-level append-only log maintained for a specific ASCP Channel.

Each Channel Log contains an ordered sequence of Artipoint Records distributed within that Channel. Channel Logs are:

- append-only
- cryptographically scoped
- replica-synchronized
- semantically opaque to lower layers

A Channel Log preserves and distributes articulated coordination within a defined visibility and trust boundary. It does not itself define meaning, authority, or governance.

# **9. Coordination Log**

The **Coordination Log** is the architectural abstraction representing the durable, append-only history through which articulated coordination is preserved, replayed, and audited within ASCP.

Meaning does not reside in the Coordination Log itself, but arises from Artipoints introduced and related through articulation. The Coordination Log preserves *when*, *by whom*, and *in what order* articulation occurred.

The Coordination Log is realized concretely as per-Channel Channel Logs. There is no global Coordination Log spanning multiple Channels.

# **10. Architectural Layering Model**

ASCP is specified as a **strictly layered protocol suite**, in which architectural layers define responsibility boundaries rather than implementation units.

Layers:

- are conceptual, not physical
- define *what* responsibility is handled, not *how*
- prevent semantic leakage between concerns

The layering model ensures that meaning, authority, enforcement, and transport remain clearly separated, supporting evolvability, interoperability, and long-term auditability.

Layers in ASCP are **not**:

- software modules or libraries
- deployment units or processes
- APIs or service boundaries
- trust boundaries
- network hops or transport layers

A single concrete implementation may collapse multiple layers into a single executable or service, or distribute them across multiple components. The layering model exists to define **conceptual responsibility boundaries**, not to prescribe implementation structure.

# **11. Syntax vs. Semantics**

A foundational distinction in ASCP is between syntax and semantics:

- **Syntax** concerns structure and form
- **Semantics** concerns meaning and interpretation

**Layering invariants:**

- **Layer-0** is responsible for durable, ordered replication of opaque log entries.
- **Layer-1** realizes semantic constraints through cryptographic encoding and decoding.
- **Layer-2** defines the syntax of Artipoint Expressions.
- **Layer-3** defines the semantics of Artipoints and their relationships.

# **12. Representation vs. Interpretation**

The distinction between **representation** and **interpretation** is fundamental to ASCP’s layering model and is closely related to — but distinct from — the syntax–semantics boundary.

- **Representation** refers to how articulated coordination is encoded into deterministic, machine-parseable form.
- **Interpretation** refers to how meaning, authority, and consequence are derived from articulated history over time.

In ASCP:

- **Layer-2 (Artipoint Grammar)** is responsible for *representation*. It defines the canonical structure of **Artipoint Expressions** and **Articulation Statements**, ensuring that articulated coordination can be parsed, validated, signed, encrypted, replicated, and replayed.
- **Layer-3 (Semantic Evaluation)** is responsible for *interpretation*. It evaluates Artipoints, their relationships, and their accumulated history to derive meaning, governance state, trust decisions, and application-specific views.

A construct may be fully represented at Layer-2 without Layer-2 having *any knowledge* of its meaning. Conversely, all Layer-3 interpretation operates exclusively over representations produced and validated by Layer-2.

This separation ensures that **syntax never implies semantics**, and that meaning can evolve without requiring changes to the grammar or transport layers.

# **13. Semantic Constructs and Coordination Constructs**

## **13.1 Semantic Constructs**

A **Semantic Construct** is an architectural classification for concepts whose meaning exists *only* through **Layer-3 semantic interpretation**.

Semantic Constructs:

- are **represented** using Layer-2 Artipoint Expressions,
- are **authored** via Articulation Statements,
- are **recorded** durably as Artipoint Records,
- but are **never defined, interpreted, or evaluated by Layer-2 itself**.

The term *Semantic Construct* describes **where meaning lives in the architecture**, not how something is encoded or transported.

Semantic Constructs include both:

- **First-class coordination elements** (explicitly articulated Artipoints), and
- **Derived semantic structures** that emerge from interpreting articulated history over time.

## **13.2 Coordination Constructs**

A **Coordination Construct** is a specific class of Semantic Construct that plays a **first-class role in the ASCP coordination graph**.

Formally, a Coordination Construct is an **Artipoint** whose semantic role is to organize, scope, address, distribute, or secure articulated coordination.

Coordination Constructs:

- are authored as semantic acts (Articulation Statements),
- are represented using the Layer-2 grammar,
- are interpreted and evaluated at Layer-3,
- participate explicitly as nodes in the coordination graph.

All Coordination Constructs are Semantic Constructs.

Not all Semantic Constructs are Coordination Constructs.

## **13.3 Classes of Coordination Constructs**

### **13.3.1 Contextual Constructs**

Contextual Constructs organize Artipoints into meaningful scopes of work.

Examples include:

- **Spaces** — top-level accountability and organizational domains
- **Streams** — coherent threads of work
- **Piles** — thematic groupings

They define *what the work is about*, independent of visibility, cryptographic distribution, or enforcement.

### **13.3.2 Addressing Constructs**

Addressing Constructs identify *who* participates in coordination.

Examples include:

- **Identity Artipoints**
- **Group Artipoints**

They define addressable participants, while governance semantics (roles, delegation, authority) are expressed through articulated attributes and interpreted at Layer-3.

### **13.3.3 Distribution Constructs (Channels)**

A **Channel** is a Distribution Construct defined **semantically at Layer-3**.

A Channel defines:

- the intended audience for articulated context,
- the semantic trust domain under which interpretation occurs,
- the scope within which articulation is shared.

A Channel is **not** a transport, encryption scheme, or encoding format.

Its semantics are *realized* by Layer-1 through cryptographic encoding and decoding of Articulation Sequences into Artipoint Records.

The distinction between Channel semantics and their mechanical realization is central to ASCP's Architecture.

#### Channel Encoder, Channel Decoder, and “Codec” Terminology

While a **Channel** is defined semantically at Layer-3, its semantics are *realized* mechanically at Layer-1 through two concrete functions:

- the **Channel Encoder**
- the **Channel Decoder**

The **Channel Encoder** accepts a validated Articulation Sequence and, using parameters derived from the Channel’s semantic definition, produces a channel-admissible Artipoint Record. This process may include cryptographic signing, encryption, and the application of visibility or admission constraints.

The **Channel Decoder** accepts a received Artipoint Record from a Channel Log, verifies its admissibility and authenticity, and decodes it back into an Articulation Sequence suitable for semantic interpretation at Layer-3.

Together, the Channel Encoder and Channel Decoder are informally referred to as the **Channel Codec**. The term *Codec* is used here in its classical communications sense (encoder/decoder pair) and **does not** imply audio, video, or media compression semantics. In ASCP, a Channel Codec operates over **Articulation Sequences**, not media streams.

Critically, the Channel Encoder and Decoder are **parameterized by Channel semantics defined at Layer-3**, but they do **not** interpret, evaluate, or modify those semantics. All meaning, authority, and governance remain the responsibility of Layer-3.

### **13.3.4 Security Constructs**

Security Constructs establish cryptographic material or security-relevant configuration.

Examples include:

- **Keyframe Artipoints**
- certificate or endorsement Artipoints

They specify *what cryptographic material exists*, while trust relationships and authorization outcomes are evaluated semantically at Layer-3.

# **14. Derived Semantic Structures**

Not all meaning in ASCP corresponds to explicitly articulated Artipoints.

**Derived Semantic Structures** are semantic states assembled by Layer-3 through interpretation of articulated history over time.

Examples include:

- the coordination DAG,
- effective membership sets,
- active Keyframe state,
- governance and authorization outcomes,
- materialized application views.

Derived Semantic Structures:

- **are not Artipoints**,
- **are not represented at Layer-2**,
- **are not enforced by Layer-1**,
- **exist only as interpretive results** at Layer-3.

This distinction ensures that ASCP preserves a complete, immutable history of articulation while allowing flexible, context-dependent interpretation.

# **15. Definition vs. Enforcement**

ASCP strictly separates **definition** from **enforcement** as a core architectural invariant.

- **Definition** establishes meaning, authority, or consequence.
- **Enforcement** mechanically applies consequences derived elsewhere.

In ASCP:

- **Layer-3 defines meaning:** identity semantics, trust evaluation, governance rules, membership, Channel semantics.
- **Layer-1 realizes defined semantics:** by signing, encrypting, validating, and admitting Articulation Sequences as Artipoint Records.
- **Layer-0 enforces durability and convergence:** by replicating Channel Logs without interpreting their contents.

Lower layers never decide *what something means* or *whether it is authorized*. They only enforce consequences derived from Layer-3 interpretation.

# **16. Layer Responsibilities**

| **Layer** | **Name**                    | **Primary Responsibility**                                                                    |
| --------- | --------------------------- | --------------------------------------------------------------------------------------------- |
| 3         | Semantic Evaluation & Views | Interpretation of Artipoints; governance, trust, membership, derived state, application views |
| 2         | Artipoint Grammar           | Canonical syntax for Artipoint Expressions and Articulation Statements                        |
| 1         | Channel Encoding / Decoding | Cryptographic realization of Channel semantics; creation and validation of Artipoint Records  |
| 0         | Log Synchronization         | Durable, ordered, local-first replication of Channel Logs                                     |

# **17. Common Category Errors**

The following errors are explicitly avoided by the ASCP layering model:

- ❌ *Identity is a Layer-2 construct*  
  ✅ Identity is a **Layer-3 Semantic Construct** represented using Layer-2 grammar
- ❌ *Channels define membership*  
  ✅ Membership is defined by **Layer-3 interpretation**; Layer-1 enforces visibility cryptographically
- ❌ *The coordination DAG exists at Layer-2*  
  ✅ The DAG is a **Derived Semantic Structure** assembled at Layer-3
- ❌ *Trust is enforced by Layer-1*  
  ✅ Trust is **evaluated at Layer-3**; Layer-1 enforces cryptographic consequences

# **18. ASCP Layering Terminology Table**

| **Layer** | **Concept**                 | **Architectural Role**                                    | **ASCP-Specific Artifact(s)**                  |
| --------- | --------------------------- | --------------------------------------------------------- | ---------------------------------------------- |
| 3         | Semantic Construct          | Meaning and interpretation                                | Artipoints                                     |
| 3         | Coordination Construct      | First-class coordination semantics                        | Spaces, Streams, Piles, Channels, Identity     |
| 3         | Derived Semantic Structure  | Interpreted state derived from articulated history        | DAGs, membership sets, governance outcomes     |
| 2         | Representation              | Canonical syntactic form                                  | Artipoint Expressions, Articulation Statements |
| 2         | Grammar Validation          | Structural correctness                                    | ABNF grammar                                   |
| 1         | Channel Encoding / Decoding | Cryptographic realization of Channel semantics            | Artipoint Records                              |
| 1         | Visibility Enforcement      | Confidentiality and admission                             | Channel envelopes                              |
| 0         | Log Synchronization         | Durable, ordered replication                              | Channel Logs                                   |
| Meta      | Coordination Log            | Architectural abstraction of durable articulation history | Realized via Channel Logs                      |

# **19. Terminological Tables**

## **Table 1: Artipoint vs. Articulation**

| **Conceptual Role** | **Term**               |
| ------------------- | ---------------------- |
| Semantic unit       | Artipoint              |
| Grammar form        | Artipoint Expression   |
| Act                 | Articulation Statement |
| Batch of acts       | Articulation Sequence  |
| Logged artifact     | Artipoint Record       |

## **Table 2: Coordination Log vs. Channel Log**

| **Concept**      | **Scope**     | **Nature** |
| ---------------- | ------------- | ---------- |
| Coordination Log | Architectural | Abstract   |
| Channel Log      | Per-Channel   | Concrete   |

# **20. Intended Use**

This document is the **governing terminology reference** for the ASCP specification suite. It should be used to:

- interpret normative specifications
- guide consistent language across drafts
- prevent semantic drift
- support rigorous external review

By unifying coordination vocabulary and architectural layering in a single primer, ASCP establishes a durable conceptual spine for shared cognition infrastructure.