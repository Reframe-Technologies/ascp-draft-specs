# The Agents Shared Cognition Protocol (ASCP)

**Architectural Framework and System Model**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.55 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document defines the architectural framework and conceptual foundation for the **Agents Shared Cognition Protocol (ASCP) suite**. It describes the system model, design motivations, architectural boundaries, and cross-layer invariants that structure the companion specifications. Together, these documents define ASCP as a suite of interoperable protocols—covering the Artipoint Grammar, Channels, Log Synchronization, Identity & Trust, Governance, and Bootstrap mechanisms—that form a shared cognition substrate.

This document is **informational**. It establishes the conceptual and architectural model for ASCP but is **not itself a normative protocol specification**. All protocol requirements—syntax, cryptographic mechanisms, synchronization behavior, identity semantics, and governance evaluation rules—are defined normatively in the companion specifications referenced in the *Specification Map*.

This document is **not** an Internet Standards Track specification. It has not undergone IETF review and has no formal standing within the IETF process. The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY” are used according to RFC 2119 and RFC 8174 only to summarize requirements defined normatively in the companion documents.

This document is intended for system architects, protocol designers, implementers, researchers, and reviewers who seek a unified architectural understanding of ASCP and how its layers relate. It provides the context necessary to interpret and implement the companion specifications correctly.

# **2. Abstract**

Modern digital work lacks a protocol for **context** itself. Humans coordinate through documents and messaging systems with no persistent structure, while AI agents operate through stateless prompts that discard meaning between sessions. The result is fragmented understanding, brittle coordination, and the continual loss of shared intent.

The **Agents Shared Cognition Protocol (ASCP)** defines a protocol suite that provides a durable, structured, and addressable substrate for shared cognition. ASCP introduces a formal grammar for expressing coordination through immutable, author-attributed **Artipoints**, which compose into cryptographically signed Articulation Sequences distributed via secure Channels and synchronized through an append-only log.

Just as HTTP standardized content exchange and Git standardized versioned change, ASCP standardizes **the articulation of context**. It enables persistent shared memory, auditable reasoning, and interoperable collaboration across tools, devices, and intelligent agents. ASCP forms a distributed, private, and composable coordination graph that allows humans and agents to build, share, and evolve context together.

# **3. Introduction**

The **Agents Shared Cognition Protocol (ASCP)** is the architectural foundation for a persistent, structured, and secure substrate of shared context across humans and AI agents. ASCP addresses a fundamental gap in modern computing: **there is no protocol for representing, sharing, and evolving context itself**.

This document defines the conceptual model and system architecture for the ASCP protocol suite. It introduces the core primitives and cross-layer principles that organize the companion specifications for the coordination grammar, secure distribution layer, log synchronization protocol, identity and trust model, and governance semantics.

At the heart of ASCP is the **Artipoint**: an immutable, addressable, author-attributed unit of articulated context. Artipoints compose into **Articulation Statements**, which express discrete coordination acts and accumulate in an append-only, cryptographically verifiable log. Together, they form a distributed DAG of articulated context that both humans and agents can reference, extend, and reason over.

Applications do not expose this DAG directly. Instead, they materialize it into familiar constructs—timelines, threads, agendas, project spaces—while ASCP ensures that the underlying context remains coherent, durable, and appropriately scoped for each participant. This separation enables private working context to coexist with shared structure, resolving the long-standing tension between individual focus and collective awareness.

This architectural specification describes:

- the motivations and theoretical foundations of ASCP;
- the architectural primitives (Artipoints, Streams, Spaces, Channels) and their composition model;
- the layered protocol architecture that defines articulation, secure distribution, and log synchronization; and
- the authorship, trust, and governance framework supporting shared cognition.

It provides the **architectural overview** for the ASCP specification suite. All normative protocol requirements—including the formal grammar, channel envelope structure, synchronization behavior, identity semantics, and governance evaluation rules—are defined in the companion specifications referenced in the **Specification Map**.

# 4. Motivations for ASCP

Modern digital systems have no shared substrate for **context**. Humans coordinate through documents and chats with no persistent structure, while AI agents operate through stateless prompts. Applications maintain isolated internal models, and there is no protocol for representing or evolving the contextual relationships that give work continuity.

This gap produces three systemic limitations:

- **Ephemeral Working Context** — The rationale behind decisions, relationships between artifacts, and evolving interpretations are scattered across tools and degrade over time.
- **Fragmentation Across Applications** — Each system models tasks, data, and relationships differently; users become the manual integration layer between incompatible contexts.
- **Stateless AI Interaction** — Agents cannot rely on durable shared memory and must be re-supplied with context on every invocation, even when they authored prior contributions.

ASCP addresses these limitations by establishing a protocol for **articulated context**: structured, immutable, author-attributed statements (Artipoints) distributed via secure Channels and synchronized through an append-only log. Rather than treating coordination as an application feature, ASCP provides a shared, interoperable substrate that any tool or agent can extend.

ASCP builds on the CSCW concept of *articulation work*—the activity through which collaborators structure tasks, express relationships, and maintain shared understanding. Today this layer is informal, implicit, or lost entirely. ASCP makes it explicit and durable through a common grammar and verifiable coordination history.

Three principles motivate the protocol’s design:

1. **Context as First-Class Content** — Coordination context must be represented as structured, addressable statements, not inferred from application state.
2. **Immutable Coordination History** — A durable log of articulated decisions enables coherence, auditability, and reproducible reasoning.
3. **Shared Persistence Across Humans and Agents** — Both must operate on a common substrate of context that spans tools, devices, and organizational boundaries.

ASCP therefore provides the foundational layer needed for persistent shared cognition, enabling humans and intelligent systems to build, share, and evolve context with continuity and precision.

# **5. What This Document is and is Not**

This document defines the **architectural framework, scope, and conceptual boundaries** of the Agents Shared Cognition Protocol (ASCP). It introduces the core coordination model, structural primitives, and layered architecture that organize the ASCP protocol suite.

This document is **informational**. It does **not** define any normative protocol behavior. In particular, it does not specify:

- the formal Artipoint grammar,
- the cryptographic envelope or channel structure,
- log synchronization mechanics (ALSP),
- identity or key management semantics, or
- governance and access-control rules.

All normative requirements—including syntax, cryptographic processing, synchronization algorithms, and governance semantics—are defined in the companion specifications referenced in the **Specification Map**.

With the exception of the consolidated compliance matrix in the Appendix, this document should be read solely as the architectural and conceptual foundation for implementing ASCP.

# 6. Intended Audience

This document is intended for readers who require an architectural understanding of ASCP, including:

- **System architects** designing ASCP-compatible applications or infrastructure
- **Protocol implementers** building clients, agents, or services that participate in ASCP Channels or parse the coordination grammar
- **Researchers and theorists** studying coordination, distributed cognition, or articulation-based models
- **Technical evaluators and decision-makers** assessing ASCP for integration into organizational or multi-agent environments

It provides the conceptual material necessary to interpret the companion normative specifications and to design interoperable systems that use ASCP as a shared substrate for context.

# 7. Reading Guide

This document provides the architectural overview for ASCP and is intended to be read alongside the companion normative specifications. This section offers guidance for navigating the material based on the reader’s goals and role.

## **7.1 Document Structure**

This document is organized to move from high-level purpose to concrete architectural detail:

- **Sections 1–4** introduce ASCP’s goals, motivations, and the overall framing of the protocol suite.
- **Sections 9–12** present the conceptual model, including Artipoints, contextual structures, and the collaboration model.
- **Sections 13–16** describe the architectural layers, grammar, governance model, and end-to-end data flow.

The **Specification Map** in Section 8 identifies where all normative protocol requirements reside in the companion specifications.

## **7.2 Recommended Paths Through This Document**

- **New Readers:** Start with Sections **3–4** (Introduction and Motivations), then proceed to Sections **9–12** for the conceptual model.
- **Readers Seeking the Conceptual Understanding:** Focus on Sections **10–12**, which introduce Artipoints, contextual structures, and the collaboration model.
- **Architects and System Designers:** Read Sections **12–16**, which describe the coordination model, layered architecture, and data-flow across ASCP.
- **Implementers:** Begin with the **Specification Map** (Section 8), then follow the companion normative specifications for the grammar, Channels, ALSP, identity, and governance.
- **Security and Governance Reviewers:** Consult Sections **14–15** for the architectural summary of trust, authorship, membership, and declarative governance semantics.

## **7.3 Relationship to Companion Specifications**

This document provides conceptual and architectural context only. All normative requirements—including syntax, cryptographic processing, synchronization rules, and governance semantics—are defined in the companion specifications referenced in Section 8.

# 8. Specification Map

This section enumerates the companion specifications that comprise the ASCP protocol suite. Each document defines one layer or functional component of the architecture. This table identifies the scope of each document and the normative content it contains:

| **Layer / Scope**                       | **Document**                                               | **Purpose**                                                                                    | **Normative Contents**                                                                                                                      |
| --------------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Top-Level Architecture**              | **The Agents Shared Cognition Protocol** (this document)   | Defines the architectural model, conceptual framework, and cross-layer relationships for ASCP. | Architectural principles, protocol layering, conceptual primitives, non-normative overview; consolidated compliance matrix (informational). |
| **Layer 2 — Articulation Layer**        | **ASCP Artipoint Grammar**                                 | Specifies the formal grammar for immutable, addressable coordination statements.               | Artipoint structure, articulation expressions, operator taxonomy, ABNF grammar, serialization requirements.                                 |
| **Layer 1 — Secure Distribution Layer** | **ASCP Channels: Secure Distribution Layer Specification** | Defines the cryptographic envelope and distribution mechanism for Articulation Statements.     | JWS/JWE processing, channel membership semantics, keyframes, key rotation, envelope formats, access control inputs.                         |
| **Layer 0 — Log and Transport Layer**   | **ASCP LogSync Protocol (ALSP)**                           | Specifies append-only log synchronization across replicas.                                     | Message formats, ordering rules, pull/push models, divergence detection, channel access proofs, error handling.                             |
| **Identity & Trust**                    | **ASCP Identity & Trust**                                  | Establishes identity representation and authorship verification.                               | Key provisioning, trust-root anchoring, identity binding, signature verification rules, recovery mechanisms.                                |
| **Governance & Access Control**         | **ASCP Governance and Access Control**                     | Defines the declarative governance model for participation, access, and role semantics.        | Attribute definitions, inheritance rules, virtual groups, RACI-style roles, evaluation algorithm.                                           |
| **Bootstrap & Discovery**               | **ASCP: Bootstrap Process and Channel Discovery**          | Describes initial provisioning, trust establishment, and channel discovery procedures.         | Bootstrap workflow, trust-graph retrieval, manifest discovery, validation steps.                                                            |

## **8.1 Using This Specification Map**

The specification map provides the authoritative reference for all normative ASCP documents. Implementers should use it to determine where each protocol behavior is defined and to identify required specifications for their implementation role.

## **8.2 Recommended Reading Order for Implementers**

1. **The Agents Shared Cognition Protocol** — Architectural understanding of the ASCP suite.
2. **ASCP Artipoint Grammar** — Required for parsing and generating articulation statements.
3. **ASCP Channels** — Required for secure distribution, membership, and envelope handling.
4. **ASCP LogSync Protocol (ALSP)** — Required for replica convergence and log synchronization.
5. **ASCP Identity & Trust** — Required for authorship, signature validation, and trust anchoring.
6. **ASCP Governance and Access Control** — Defines participation rules, role semantics, evaluation logic, and access-control inheritance.
7. **ASCP Bootstrap Process and Channel Discovery** — Required for first-time setup and trust initialization.

# 9. Underpinnings of ASCP

ASCP addresses context fragmentation by establishing a protocol-level substrate for articulated context—one that both human and machine participants can reference and extend. Rather than implementing coordination as an application-layer feature, ASCP defines a foundational protocol that enables interoperability across heterogeneous systems.

This differs from existing coordination systems by introducing a common protocol layer that any application can implement, enabling persistent shared context across the digital ecosystem without requiring centralized platforms or proprietary integration.

## Articulation: The Foundation of Coordination

In Computer-Supported Cooperative Work (CSCW), **articulation work** refers to the process by which cooperating individuals partition work into units, divide it amongst themselves and, after the work is performed, reintegrate it (Schmidt and Bannon, 1992). Articulation work represents the coordination effort required to ensure that distributed activities integrate effectively toward a common objective.

ASCP formalizes articulation work through the **Articulation** primitive: an operation that writes context into a shared, durable log. An Articulation makes implicit coordination explicit by producing an immutable, addressable record that participants can reference and build upon.

The following scenarios illustrate the operational difference between systems without and with ASCP's articulation substrate:

## Without ASCP

A user bookmarks meeting notes at `https://notion.so/project-kickoff-notes`. When the user later queries an AI agent regarding a decision documented in those notes, the agent has no persistent access to the prior context. The user must re-supply the relevant information, even if the agent participated in the original interaction that produced the notes.

## With ASCP

The same meeting notes are articulated as an Artipoint within a shared Stream. The AI agent maintains access to the articulation log and can reference the documented decisions without requiring the user to re-supply context. The agent operates on a persistent shared memory substrate rather than stateless prompts.

# **10. Understanding Artipoints**

Artipoints are the foundational primitive of ASCP. Each Artipoint is an immutable, addressable, author-attributed unit of articulated context—a durable statement that can be referenced, related, extended, and reinterpreted over time. Artipoints form the basic substrate through which shared understanding emerges across human and AI collaborators.

Artipoints are deliberately minimal. They do not encode application behavior, workflow state, or user interface expectations. Instead, they provide a precise, durable language for expressing relevance, relationships, intent, and interpretation—allowing meaning to be built up through articulation rather than inferred from mutable structures.

## **10.1 Artipoints Are Structured References**

Artipoints originate from a familiar notion: a bookmark. But while a traditional bookmark merely records *where* something is, an Artipoint records *why it matters* within an evolving line of work.

Where a bookmark says:

- “this exists,”
- “save this for later,”

an Artipoint conveys a structured assessment, such as:

- *“This document is relevant to our authentication strategy.”*
- *“This research report supersedes the earlier analysis.”*

This shift—from pointing at content to articulating a contextual judgment—makes Artipoints the elementary building blocks of explicit reasoning. They allow collaborators and agents to anchor meaning directly rather than rely on incidental or implicit structure.

## **10.2 Artipoints Are Immutable Contextual Atoms**

Each Artipoint is a **Contextual Atom**: a permanent, author-attributed record of a contributor’s point of view. Rather than asserting a single shared truth, Artipoints represent *perspectives*—capturing the reality that collaborators often hold partial, evolving, or diverging understandings of work.

An Artipoint includes:

- a **UUID** (stable identifier),
- an **author** (human or agent),
- a **timestamp**,
- a **payload** (commonly a URI or structured reference), and
- an optional **expression** describing the articulated claim.

Immutability is essential. Once created, an Artipoint is never modified; new Artipoints refine, extend, supersede, or contextualize earlier ones. This allows the coordination log to record *how* shared understanding evolves rather than overwrite its history.

By preserving authored perspectives over time, ASCP enables collaborators and tools to detect emerging patterns of agreement, drift, or misalignment—critical signals that traditional systems obscure.

## **10.3 Artipoints Are Statements About Things**

Artipoints are **declarative statements**, not representations of ground truth. They express something about an artifact, idea, task, decision, or structure—for example:

- *“This research report supersedes that earlier one.”*
- *“These two documents should be considered together.”*
- *“This proposal depends on that technical decision.”*

These statements capture a contributor’s perspective at a moment in time. Over the course of a collaboration, additional articulations may extend, clarify, reinterpret, or diverge from earlier ones. ASCP does not enforce consensus; it preserves the articulated perspectives so that alignment—or misalignment—can be surfaced and resolved through further articulation.

This approach contrasts with traditional systems (e.g., CRMs, project plans, task lists) that implicitly present a single authoritative state, even when it is incomplete, out of date, or not uniformly agreed upon. ASCP instead records *what was articulated*, enabling a transparent trail of evolving understanding.

## **10.4 Artipoints Form Articulation Sequences and Statements**

Artipoints are formed and instantiated via **Articulation Sequences**. Each articulation operation produces an **Articulation Sequence**—an immutable, author-signed, timestamped bundle containing one or more **Articulation Statements**. The Sequence represents a discrete moment of coordination: *who* articulated *what* and *when*.

Each Articulation Statement expresses exactly one operation. Conceptually, ASCP supports four expression types:

- **Instantiation** — introduce a new Artipoint
- **Connection** — relate existing Artipoints
- **Construction** — create a new Artipoint and relate it in one operation
- **Annotation** — refine or augment the interpretation of an existing Artipoint

These expressions constitute the minimal vocabulary necessary to introduce, relate, and evolve context without requiring mutable shared state. The formal grammar and semantics of each statement form are defined in the companion specification **ASCP Artipoint Grammar**.

Because every Articulation Sequence is immutable and author-attributed, the coordination log retains the evolution of perspectives—not just their current resolution.

## **10.5 Artipoints Support Addressability**

Addressability enables ASCP to support differentiated visibility, participation, and attention without conflating them. Each Artipoint and Articulation Sequence has a stable identifier, allowing the protocol to distinguish between:

| **Mechanism** | **Governs**                | **Purpose**                                                       |
| ------------- | -------------------------- | ----------------------------------------------------------------- |
| **Channels**  | Visibility / Access        | Who can receive, decrypt, and store articulations                 |
| **Members**   | Structural Participation   | Who is contextually part of a Space, Stream, Pile, or Channel     |
| **Flags**     | Attention / Working Memory | Who is actively tracking or maintaining awareness of an Artipoint |

This separation allows collaborators—human and AI—to maintain shared understanding without forcing global visibility or universal notifications. It lets systems surface relevant articulations at the right time without altering the underlying durable record.

Further details of these mechanisms appear in **Section 12: Collaboration Model**.

# **11. Contextual Constructs**

Artipoints and Articulation Statements supply the atomic vocabulary of coordination. But meaningful collaboration requires more than isolated statements — it requires **organization**, a way to group, scope, and structure related work.

ASCP supports this by allowing collaborators to articulate higher-order organizational patterns using the same grammar and primitives. **Piles, Streams, and Spaces are not special system objects or predefined schema types**. They are ordinary Artipoints whose meaning emerges from how they are articulated and related to other Artipoints.

This unified model ensures:

- All structures are addressable Artipoints.
- All organization is expressed through immutable Articulation Statements.
- The same operators, governance semantics, and composition rules apply.
- Contextual structure is durable and fully captured in the coordination log.

By eliminating the distinction between “content” and “structure,” ASCP enables applications and agents to interpret and navigate collaboration context without requiring any hidden or privileged data models.

## **11.1 Piles: Thematic Groupings of Artipoints**

**Piles** are Artipoints that designate a thematic grouping of related Artipoints. They are typically used to collect items within a Stream or Space — for example, “Open Questions,” “Design Ideas,” or “Pending Decisions.”

Piles support **flat, associative grouping**. A Pile may contain Artipoints and be referenced from multiple places, but **Piles do not contain other Piles by value**, maintaining a simple organizational layer appropriate for early-stage or loosely scoped work.

A Pile inherits governance and contextual semantics from the Stream or Space into which it is articulated, unless explicitly overridden.

## **11.2 Streams: Context-Switchable Threads of Work**

**Streams** are Artipoints representing coherent lines of work — the atomic units of collaboration. A Stream collects all Artipoints relevant to a particular goal or initiative and provides a stable contextual frame that participants can enter, exit, and revisit.

Streams **may contain Piles and arbitrary Artipoints**, but they **do not contain other Streams**. This constraint maintains clarity: each Stream represents a distinct thread of work that can be independently shared, delegated, paused, or resumed.

Examples include a hiring process, a product feature, a research investigation, or a customer escalation.

When shared among collaborators, Streams allow synchronized context while preserving individual views. Streams inherit governance from their parent Space unless overridden.

## **11.3 Spaces: Accountability Bubbles**

**Spaces** are Artipoints representing top-level accountability contexts. A Space can include other Spaces and any number of Streams, organizing coordination into meaningful domains such as teams, projects, programs, or organizational units.

Each Space also functionally acts a special "*Stream Zero***"**, used for coordinating the Streams and other structures within that Space. This provides a consistent, durable scaffold for administrative and structural articulation.

Spaces form hierarchical trees that reflect organizational structure, ownership boundaries, and strategic initiatives. Like all structures in ASCP, Spaces gain their semantics through articulated relationships rather than predefined schema.

# 12. Collaboration Model

ASCP distinguishes three independent dimensions of coordination—**visibility**, **participation**, and **attention**.

These dimensions are often conflated in traditional systems (e.g., “everyone who can see this is assumed to be involved,” or “everyone involved must be notified”), but ASCP treats them as *separate articulations*.

This allows collaborators and agents to work together with far greater precision.

- **Channels** determine *who can see* articulated context.
- **Members** determine *who is contextually part of a collaborative structure*.
- **Flags** determine *who is actively attending to or elevating a particular Artipoint*.

The combination of these mechanisms supports rich forms of collaboration, ranging from tightly aligned teams to loosely coupled multi-agent workflows.

## **12.1 Channels: Visibility and Access**

**Channels define the visibility scope for articulated context.**

If an Articulation Sequence is sent to a Channel, every participant with access to that Channel can receive, decrypt, and store it. This is similar in spirit to a **cryptographically scoped distribution list**: everyone who is part of the Channel receives the same articulated context, regardless of what tools they use.

Channels provide two guarantees:

1. **Distributed Consistency** — Everyone with access sees the same articulated context.
2. **Cryptographic Privacy** — No one outside the Channel can decrypt or access that context.

A Channel therefore expresses the **audience** of an articulation. It does *not* express:

- whether the recipient is working on the topic (that’s **Members**)
- whether they are paying attention to it (that’s **Flags**)

A single collaborative Space might broadcast its coordination activity across multiple Channels—for example, a public one for general alignment and a private one for sensitive discussions—while preserving coherent context across both.

## **12.2 Members: Structural Participation**

Membership defines **who is part of a collaborative structure**, such as a Space, Stream, or Pile. Membership is an **articulated relationship**, not a cryptographic one.

Participants become Members because someone (often the Space or Stream owner) explicitly articulates that relationship.

Membership conveys:

- **who is included** in the collaborative structure
- **who has standing** to contribute or act within it
- **who inherits governance and access semantics**

But membership does *not* automatically determine what a participant can see.

Visibility still depends on Channels. A participant may be a Member of a Space but not have visibility into all Streams unless they participate in the relevant Channels.

Membership therefore expresses:

> **“Include this person or agent in the structure of this collaboration.”**

This allows collaborators to organize work at the level of *roles and responsibilities* rather than merely *who has access to data*.

## **12.3 Flags: Social Attention and Shared Working Memory**

Flagging is the mechanism through which participants express **active attention** toward an Artipoint.

A Flag is an articulation—an immutable, author-attributed statement—that declares interest, salience, or priority.

Flags serve two complementary purposes:

### **1. Personal Attention**

A participant may flag something *for themselves* to keep it in their working memory.

Systems can then surface it when related articulations appear.

### **2. Social Attention**

Because Flags are articulated and broadcast through Channels, **everyone can see everyone’s Flags** (within the visibility scope).

This enables rich collaborative behaviors:

- You can flag something **for someone else**
- You can see what others consider important
- You can see when someone removes a Flag
- The team develops **shared situational awareness**
- Agents can detect when attention diverges or converges
- Collaborative presence emerges (“I see what you’re paying attention to”)

Flags do *not* alter visibility (Channels do that) or structural participation (Members do that).

Instead, they create a **layer of social signaling** and **distributed working memory**:

> **“I’m attending to this now.”
> “Please take a look at this.”
> “We should keep an eye on this.”**

These signals support coordination without requiring interrupts, notifications, or application-level workflow mechanics. Flags simply express, explicitly and durably, what collaborators and agents consider important at a given moment.

# 13. The Grammar of Coordination

ASCP defines a minimal, declarative grammar for expressing articulated context. This grammar provides the structural backbone of the coordination graph: every Artipoint, every relationship, and every refinement is encoded as an immutable **Articulation Statement** conforming to this shared syntax.

The purpose of the grammar is architectural rather than expressive. It ensures that:

- all coordination acts are represented as **structured, machine-parsable statements**;
- collaborators and agents can operate on a **common, deterministic model** of articulation;
- no application defines its own hidden semantics for meaning, relationships, or context;
- the coordination graph remains **composable**, **extensible**, and **interoperable** across tools;
- articulation is constrained to **declarative statements**, avoiding computational or imperative complexity.

A central architectural invariant of ASCP is that **contextual meaning is externalized into the coordination graph itself**. Meaning is not embedded in payloads, inferred from application state, or encoded in hidden schemas. Instead, it arises exclusively from the immutable Artipoints and the relationships articulated between them over time.

This invariant allows context to remain durable, auditable, and interoperable across applications and agents, even as the content referenced by payloads or the views materializing the graph evolve independently.

## **Everything Is an Artipoint**

A fundamental implication of this design—visible across Sections 10, 11, and 12—is that **all collaborative constructs in ASCP are made of the same primitive**. Artipoints can represent domain objects, contextual structures (such as Piles, Streams, and Spaces), membership relations, attention signals (Flags), or any other coordination concept a system or team articulates.

There are **no reserved types**, no built-in hierarchies, and no privileged schema. All structure emerges from **how Artipoints are articulated and related**, not from predefined object classes. This architectural uniformity allows a simple grammar to support rich, evolving collaboration semantics while remaining straightforward to validate, synchronize, and reason about.

The grammar provides only the constructs necessary for introducing Artipoints, relating them, annotating them, or constructing new composite meaning. Higher-level structures—such as Piles, Streams, Spaces, and other collaboration patterns—emerge purely from articulated relationships. This minimalism supports predictable reasoning, verifiable authorship, and clear provenance within the shared articulation log.

The **formal definition** of the grammar—including its ABNF syntax, operator taxonomy, serialization rules, and validation semantics—is specified normatively in the companion document **ASCP Artipoint Grammar: A Structure for Shared Cognition**. That specification serves as the authoritative reference for implementers building parsers, encoders, or extensions to the operator set. This architectural document provides the conceptual context for understanding the grammar’s role within ASCP.

# 14. Governance and Security

ASCP separates coordination semantics, governance semantics, access control, and trust foundations into distinct layers. This avoids the traditional conflation of roles, permissions, authorship rights, and organizational accountability. In ASCP, each of these concerns is articulated explicitly and preserved as part of the immutable coordination graph.

Governance in ASCP is **declarative and authoritative**: it defines **semantic authority and permission**—who may author, steward, delegate, or hold responsibility within a given context—without embedding procedural enforcement or cryptographic control. These governance articulations express what *ought* to be true within the collaboration and are evaluated deterministically by applications and agents.

Security in ASCP is **architectural**: it arises from cryptographic identity, immutable logs, scoped visibility via Channels, and local-first replica verification. Cryptographic mechanisms enforce *who can see and receive* articulated context, while governance semantics define *who is authorized to act* upon it. These dimensions are intentionally decoupled to model real-world collaboration accurately and to prevent authority, access, and accountability from being conflated.

Normative details for governance evaluation, identity binding, and certificate semantics are specified in the companion ASCP documents. This section summarizes the architectural principles that unify them.

## **14.1 Collaborative Governance: Roles, Authority, and Accountability**

Collaborative governance in ASCP describes how people and agents relate to work rather than how they access data. These relationships are articulated directly in the coordination graph, typically reflecting RACI-style semantics. Participants may be responsible for advancing a Stream, accountable for its outcomes, consulted for their expertise, or merely informed about its evolution. These distinctions capture real-world coordination dynamics: responsibility for tasks, authority for decisions, sources of input, and expectations of awareness.

These governance attributes are expressed as annotations attached to Spaces, Streams, and Piles. They are immutable statements of perspective that may be refined or superseded over time. Importantly, collaborative governance does not confer cryptographic privileges. A participant may be accountable for a Stream yet lack permission to alter its structure, or may have authorship rights without being responsible for the work’s outcome. ASCP treats these as separate conceptual layers to more accurately model human and organizational collaboration.

## **14.2 Administrative Governance: Permissions, Delegation, and Membership**

Administrative governance expresses who may act upon articulated context — who may author, annotate, restructure, or delegate authority within a Space or Stream. These permissions are articulated declaratively and evaluated deterministically based on the accumulated context. They define participation rights, specify role inheritance across structural hierarchies, and allow delegation or restriction of authority.

Administrative membership differs fundamentally from collaborative membership. A participant may be structurally included in a Space without being responsible for its outcomes; conversely, someone may be responsible or accountable without being permitted to make structural changes. Administrative permissions determine capability, not meaning.

Visibility, however, is not governed here. A participant’s ability to see Artipoints depends entirely on the Channels through which articulations are distributed (Section 12). Governance cannot override cryptographic visibility boundaries.

## **14.3 Trust Model: Log-Anchored, Local-First, and Boundary-Scoped**

ASCP’s trust model is built on immutable history, local verification, and scoped trust domains. Every security-relevant event — identity introduction, certificate binding, key rotation, governance decision, or delegation — is expressed as a signed, immutable Artipoint. Trust, therefore, is anchored in the log rather than in mutable external systems. A replica can verify authorship, certificate validity, and governance semantics by replaying articulated history.

Verification is local-first. Each replica carries the materials necessary to evaluate trust, identity, and governance without relying on a central authority. Two replicas evaluating the same log will arrive at the same trust decisions, yielding deterministic interpretation and eliminating hidden mutable state.

Trust domains are bounded by Channels. Each Channel defines its own visibility scope, keyframes, authenticated participants, and provenance boundaries. ASCP does not require global trust infrastructure or certificate transparency; trust arises within each Channel's articulated and cryptographically enforced boundary. As explained in the ASCP Trust & Identity Architecture, this model generalizes across personal, organizational, and federated deployments without *requiring* shared PKI or global coordination—though deployments may layer these through articulated endorsements (see Section 14.5).

## **14.4 Security Properties Emergent from the Architecture**

ASCP’s security properties arise naturally from the combination of immutable articulation and cryptographic identity. Because every Artipoint is signed by its author and bound to a certificate, authorship is always verifiable, and provenance forms a complete chain. Since visibility is controlled by Channel encryption, no participant can view articulated context unless they hold the appropriate keys, and governance cannot override this boundary.

All operations occur in an append-only log, ensuring any unauthorized modification is cryptographically detectable. Timestamped, signed events ensure stable semantics when interpreting the log at any later time. Evaluating the same history yields the same outcomes, allowing systems to reason about governance, trust, and authorship deterministically.

Finally, ASCP maintains a clear separation among authority dimensions. Access rights do not imply responsibility; responsibility does not imply authorship capability; and authorship does not imply visibility. By structurally decoupling these layers, ASCP prevents many classes of governance and security failure that arise in systems where these concepts are conflated.

## **14.5 Deployment Topologies and Operational Trust**

The same architectural principles apply across personal, organizational, and federated deployments. A personal or local-first deployment may rely entirely on self-generated keys and self-contained trust, with no external identity provider. Organizational deployments may bind identities to enterprise-managed credentials, layering structured governance policies on top of articulated context. Federated deployments allow independent trust domains — each with their own roots of trust — to interoperate through articulated endorsements and selectively replicated Channels.

In all cases, the primitives remain the same: Artipoints record trust decisions, Channels define visibility boundaries, and replicas independently verify history to arrive at consistent interpretations. This enables ASCP to support diverse deployment environments while preserving transparency, resilience, and cryptographic integrity.

# **15. ASCP Architectural Layered Model**

ASCP is organized into four architectural layers that maintain strict boundaries while working in concert to form a coherent substrate for shared cognition. Each layer defines a distinct responsibility and can evolve independently. Together, they enable durable articulation, secure distribution, local-first replication, and application-specific interpretation.

## **15.1 Layer 3 — View & Governance Layer**

Layer 3 is where articulated context becomes interpretable and actionable. It materializes the coordination graph into application-specific views, reconstructing Spaces, Streams, Piles, Roles, Flags, and other structures from the immutable history produced by Layers 0–2.

**Governance semantics are evaluated and applied at this layer by applications and agents.** While Layer 2 expresses governance rules and Layer 1 scopes visibility, Layer 3 determines how articulated policies apply within a specific runtime environment. Enforcement, delegation, permissions, and workflow-specific behaviors are therefore realized through Layer-3 application logic, not through lower protocol layers.

Layer 3 also provisions the substrate beneath it: it determines which Channels exist, who receives key material, which Streams are distributed into which trust domains, and how replicas join or depart. Layers 1 and 0 treat these as configuration inputs and do not interpret their meaning.

ASCP places no constraints on presentation or workflow at Layer 3. The only architectural requirement is that changes to shared context must be made through Layer 2 articulations, not only by mutating local materializations.

## **15.2 Layer 2 — Articulation Layer**

Layer 2 defines the formal grammar and semantics through which coordination context is expressed. Artipoints and Articulation Statements form the immutable vocabulary used to describe relevance, relationships, roles, membership, Flags, governance policies, and all higher-level collaborative constructs.

Layer 2 enforces ASCP’s core invariants:

- all meaning is represented declaratively as Artipoints;
- structure emerges from articulated relationships, not predefined schemas;
- the grammar is deterministic and machine-parsable;
- any compliant implementation can interpret the coordination graph.

Layer 2 defines the **semantic substrate** for the protocol suite. It provides the meaning and structure that Layers 1 and 0 transport and store, and that Layer 3 interprets and enforces.

**Technical specification:** The complete syntax, semantics, and parsing rules are defined in the *ASCP Artipoint Grammar* specification document.

## **15.3 Layer 1 — Channels (Secure Distribution & Visibility Scoping)**

Layer 1 governs how articulated context is distributed and who can see it. Channels define cryptographic visibility scopes: an Articulation Sequence sent to a Channel is accessible only to participants who hold the relevant keys.

Layer 1 manages signing and encryption, Channel membership keys, keyframes, and the provenance boundaries of each trust domain. It ensures that all authorized participants receive a coherent view of the articulated context, while unauthorized parties cannot decrypt or store it.

Layer 1 does not interpret the meaning of articulations and does not evaluate governance. It simply provides secure, authenticated distribution.

**Technical specification:** Channel structure, encryption protocols, and membership management are detailed in the *ASCP Channels: Secure Distribution Layer Specification* document.

## **15.4 Layer 0 — Log and Transport Layer (ALSP)**

Layer 0 provides durable, append-only storage and replication of encrypted Articulation Sequences. Using the ASCP LogSync Protocol (ALSP), replicas synchronize Channel logs in a local-first, eventually consistent manner. Layer 0 ensures deterministic convergence while treating all payloads as opaque.

Layer 0 does not parse grammar, interpret relationships, or evaluate policy. Its sole responsibility is maintaining the integrity and convergence of Channel logs across distributed replicas.

**Technical specification:** Synchronization algorithms, logical log management, conflict resolution, and transport protocols are defined in the *ASCP Log Sync Protocol (ALSP)* specification document.

# 16. ASCP Data Flow

The flow of data through ASCP reflects the strict separation of responsibilities between its architectural layers. Each step contributes one part of the end-to-end pipeline—from expressing an idea, to distributing it securely, to materializing it for interpretation.

## **16.1 Authoring and Articulation (Layer 2)**

A participant or agent begins by expressing an articulation using the ASCP grammar. Layer 2 produces an **Articulation Sequence** containing one or more Articulation Statements. At this stage the sequence is a **pure semantic object**: it carries meaning, references, relationships, governance expressions, roles, Flags, and other articulated structure — **but it is not yet signed or encrypted**.

Layer 2 defines articulation semantics only. It does not dictate visibility, distribution, or authorship verification.

## **16.2 Signing, Packaging, and Encryption (Layer 1)**

The originating system passes the unsigned Articulation Sequence to Layer 1, which **signs and encrypts** it as part of constructing a **Channel envelope**.

In this step:

1. The author’s identity is bound to the articulation via a **Layer-1 signature**.
2. The articulation is encrypted according to the Channel’s key material, defining its **visibility scope**.
3. The completed envelope becomes an addressed, immutable unit suitable for logging  and replication.

Layer 1 enforces authenticated authorship and cryptographic privacy. It does *not* interpret the semantics of the articulation.

## **16.3 Logging and Replication (Layer 0)**

Layer 0 receives the encrypted Channel envelope and appends it to the Channel’s **append-only log**. Using the ASCP LogSync Protocol (ALSP), Layer 0 synchronizes Channel logs across replicas in a local-first, eventually consistent manner.

Layer 0 treats the envelope as **opaque**. It does not parse or validate the inner articulation. Its responsibilities are limited to durability, ordering, and deterministic convergence of Channel a given node is authorized to replicate.

## **16.4 Decryption and Local Reconstruction (Layers 1 → 3)**

When a replica receives an envelope, **Layer 1 decrypts** it (subject to key access) and extracts the original Layer-2 Articulation Sequence. The sequence is then handed to Layer 3 for **interpretation**.

Layer 3 reconstructs the materialized view of context — Streams, Spaces, Piles, roles, Flags, governance state — by evaluating the newly received articulation against prior articulated history. This process does not modify the underlying log; it merely updates the local, derived model.

## **16.5 Presentation and Interaction (Layer 3)**

Applications and agents present the updated coordination context through their chosen interfaces. When participants take actions that affect shared state — such as adding a dependency, changing a role, proposing a decision, or flagging an item — these actions are converted into new **Layer-2 articulations**, beginning the pipeline anew.

Layer 3 also evaluates governance policies and permissions to determine whether a participant or agent is allowed to perform the requested action. Enforcement is application-specific and never performed by Layers 0–2 except as constructs provisioned via Layer-3 operations.

# 17. Why ASCP Matters

Modern computational systems lack a protocol for shared context. Applications maintain isolated internal models, messaging systems provide ephemeral coordination, and AI systems operate through stateless prompts. ASCP addresses this gap by standardizing **the articulation of context itself** as a first-class, durable, and interoperable substrate.

ASCP’s core innovation is not the invention of new collaboration primitives, but the **formalization and distribution** of existing coordination behaviors as immutable, machine-interpretable structures. By representing context as authored, addressable, and evolvable Artipoints—distributed through secure Channels and maintained in a durable log—ASCP enables a level of continuity, auditability, and shared understanding that is not possible with today’s ad-hoc mechanisms.

This substrate unlocks several capabilities:

- **Human–human collaboration gains structure and persistence**, eliminating fragmentation across tools and enabling coherent, cross-application workflows.
- **Human–agent collaboration becomes continuous rather than prompt-driven**, as agents can reference and contribute to the same durable context as their human counterparts.
- **Multi-agent systems gain persistent working memory**, enabling richer coordination strategies, division of labor, and inference over stable context.
- **Every contribution becomes auditable and interpretable**, allowing teams and agents to understand how meaning, roles, and decisions have evolved over time.

Importantly, the underlying coordination graph remains invisible to end users. They interact through natural application views—threads, agendas, dashboards—while ASCP ensures these views remain private where needed, coherent across participants, and grounded in the same articulated context. ASCP does not mandate how work appears; it ensures that **meaning persists, synchronizes, and remains interpretable**.

By establishing a protocol for articulated context, ASCP provides the foundational layer for hybrid human–AI collaboration: a basis for shared cognition in the next era of computing.

# 18. Conclusion

ASCP provides an architectural foundation for shared cognition across humans and intelligent agents. Through Artipoints, Articulation Statements, structured grammar, cryptographically scoped Channels, and durable log synchronization, ASCP replaces the fragmented, ephemeral, and stateless defaults of modern digital systems with a secure, composable, and persistent substrate for context.

The protocol suite does not prescribe how applications should appear or behave. Instead, it defines how context is expressed, distributed, verified, and evolved. This separation allows diverse tools, agents, and organizations to interoperate while maintaining privacy, authorship integrity, and durable shared understanding.

By formalizing articulation and grounding coordination in immutable, structured history, ASCP enables collaboration models that are transparent, auditable, and robust—supporting everything from simple human workflows to sophisticated multi-agent ecosystems. ASCP is not merely a protocol; it is a conceptual and architectural shift toward computing systems that think **with** us, not merely **for** us.

# **19. References**

The following documents provide the broader context and layered exposition leading up to this ASCP whitepaper. They form a progressive narrative arc from high-level framing to technical specification:

### **Building the Missing Cortex Layer: Why Human and AI Collaboration Needs Common Persistent Context**

*By Jeffrey Szczepanski – June 2025*

This architectural explainer introduces the **Cortex Layer** as a foundational substrate missing from today’s agent ecosystems. It examines how current protocols (like MCP and A2A) fail to support structured, persistent collaboration, and positions the Agent Shared Cognition Protocol (ASCP) as the infrastructure required for shared understanding and coordination over time.

### **Introduction to the Agent Shared Cognition Protocol (ASCP)**

*By Jeffrey Szczepanski – July 2025*

This document introduces the key primitives of ASCP: **Artipoints**, **Articulation Statements**, **Streams**, and **Spaces**. It explains the rationale behind ASCP’s design, the layers of its architecture and how it supports persistent shared memory across agents and humans. Ideal for system architects and contributors looking to understand the conceptual model before diving into full specification.

# **Appendix A: Normative Compliance Specification**

The authoritative definitions of these requirements reside in the companion normative specifications. This appendix consolidates them into a single compliance reference for convenience, providing implementers with a one-stop summary of **MUST / SHOULD / MAY** requirements for ASCP compliance.

## **A.1 Compliance Levels**

### **A.1.1 Core ASCP Compliance**

An implementation MUST:

- Support Layer 2 (Artipoint Grammar) and Layer 1 (Channels) fully.
- Implement at least one Layer 0 transport binding (WebSockets/TLS REQUIRED).
- Support baseline cryptographic algorithms.

### **A.1.2 Full ASCP Compliance**

An implementation MUST:

- Meet all Core ASCP requirements.
- Implement the full ALSP sync model (push and pull modes).
- Support channel bootstrap and discovery procedures.
- Implement all required governance and key rotation mechanisms.

## **A.2 Normative Requirements by Layer**

### **A.2.1 Layer 2 – Artipoint Grammar**

**MUST:**

- Parse and generate Artipoints per ABNF grammar.
- Support all four articulation patterns: instantiation, connection, construction, annotation.
- Treat all Artipoints as immutable.
- Use UUIDv7 or equivalent RFC-4122 compliant UUIDs.
- Record timestamps in ISO 8601 UTC with mandatory Z suffix.

**SHOULD:**

- Apply canonical serialization for signed Artipoints (consistent whitespace, attribute ordering).
- Validate payload URI formats.

**MAY:**

- Extend operator taxonomy, provided new operators are prefixed or namespaced.

### **A.2.2 Layer 1 – Channels**

**MUST:**

- Sign all Artipoints with JWS (ES256 using X.509 EC certificates).
- Support optional encryption of Artipoints with JWE (AES-256-GCM, alg: dir).
- Manage membership and keys via articulated Keyframes.
- Distribute keys using per-recipient JWE key envelopes (ECDH-ES+A256KW).
- Enforce channel access control using Channel Access Keys (CAK, Ed25519).

**SHOULD:**

- Rotate AES and CAK keys periodically (policy recommended).
- Store all trust material in immutable channel logs.

**MAY:**

- Support alternative crypto algorithms if negotiated during bootstrap.

### **A.2.3 Layer 0 – ALSP**

**MUST:**

- Maintain append-only channel logs.
- Sign all messages with JWS (ES256 using X.509 EC certificates).
- Synchronize using Lamport clock ordering (lamport\_time, node\_id, message\_id).
- Authenticate peers with X.509 mutual authentication at ALSP hello.
- Authorize channel replication using CAK credentials.
- Support at least pull-mode sync.

**SHOULD:**

- Implement push-mode sync.
- Use digest hash exchange for divergence detection.
- Persist Lamport counters across restarts.

**MAY:**

- Support alternative transports (HTTP POST, gRPC).
- Implement Merkle tree summaries for advanced integrity checks.

## **A.3 Bootstrap and Discovery**

**MUST:**

- Retrieve and validate bootstrap documents.
- Discover organizational manifests including channel metadata and public keys.
- Verify bootstrap material signatures.

**SHOULD:**

- Support live bootstrap updates.

**MAY:**

- Cache bootstrap metadata locally with expiration policy.

## **A.4 Versioning and Extensibility**

**MUST:**

- Include ascp\_version in all protocol handshakes.
- Reject unsupported major versions.

**SHOULD:**

- Support minor version negotiation.

**MAY:**

- Ignore unknown attributes/operators while preserving them in logs.

## **A.5 Security Considerations**

**MUST:**

- Protect all payloads in transit with TLS 1.3 or higher.
- Enforce replay protection using nonces and timestamps.
- Validate all signatures before processing Artipoints.

**SHOULD:**

- Implement key revocation workflows.

**MAY:**

- Support post-quantum algorithms when available.

## **A.6 Conformance Testing**

An ASCP-compliant implementation SHOULD pass:

- **Grammar validation tests** (syntax parsing, serialization).
- **Channel crypto tests** (sign/verify, encrypt/decrypt, key rotation).
- **LogSync interoperability tests** (multi-peer sync scenarios).
- **Bootstrap trust chain validation**.

Official conformance test fixtures and validation tooling will be published separately.