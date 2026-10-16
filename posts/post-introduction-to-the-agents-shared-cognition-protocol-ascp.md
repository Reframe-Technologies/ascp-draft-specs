# Introduction to the Agents Shared Cognition Protocol (ASCP)

*An open protocol for work that persists across people, agents, and applications*

**Jeff Szczepanski | 10 December 2025**

## **Abstract**

ASCP provides an open foundation for ongoing work that independent humans, agents, and applications can participate in, with durable coordination history and shared interpretation. It represents coordination through authored articulations that preserve decisions, relationships, and their evolution across implementations. By treating coordination as first-class infrastructure rather than ephemeral conversation, ASCP enables persistent shared cognition across distributed teams and intelligent systems.

## **Introduction**

This is the third post in a series exploring shared cognition infrastructure. The [first post](https://blog.reframetech.com/the-pathway-to-agi-isnt-intelligence-its-shared-cognition) identified fragmentation and loss of continuity in collaboration. The [second post](https://blog.reframetech.com/building-the-missing-cortex-layer) argued for a Cortex Layer that treats coordination as persistent, structured data. This post introduces ASCP, the protocol proposed to implement that vision.

Agents and applications can retain extensive project context while the coordination of work remains tied to their individual environments. When another participant joins, the history of decisions, responsibilities, and unresolved questions must remain accessible and meaningful. ASCP proposes a common model through which participants can contribute to that ongoing work as tools and participants change.

ASCP works by representing acts of coordination as computational structures. **Articulation Statements** introduce, relate, and annotate immutable **Artipoints**—addressable units of articulated meaning. These statements capture coordination patterns: instantiating concepts, relating ideas, organizing work, and superseding outdated context. Rather than letting coordination and context dissolve into chat history, ASCP builds a persistent graph of articulated coordination that both humans and AI can navigate and extend.

The result is infrastructure for collaboration that functions like Git does for code: a shared, auditable record that supports distributed coordination and reduces the need to reconstruct prior context. Context becomes durable, addressable, and machine-readable—transforming articulation work from invisible overhead into computable infrastructure.

For example, a team records a release decision, its supporting rationale, and an unresolved concern in a shared Stream. An agent contributes an analysis and later leaves. A replacement agent, using another application, can access the authorized history, distinguish the current decision from an earlier proposal, and contribute further work without depending on the original agent's private memory.

## **From Discourse Theory to Protocol**

ASCP draws from human discourse theory, which models coordination through conversational floors, turn-taking protocols, and relevance boundaries. These principles apply to both human and AI participants. ASCP makes these discourse structures computable through core primitives that support persistent shared cognition:

- **Artipoints** are immutable, addressable units of articulated meaning.
- **Channels** define distribution and visibility scopes for articulated context.
- **Streams** provide persistent contexts for coherent threads of work.
- **Spaces** coordinate multiple streams toward broader outcomes.

ASCP transforms articulation work—the coordination that normally lives only in people's heads or dissolves into unstructured chat threads—into formal, durable infrastructure that both humans and machines can reason about. This creates a foundation for persistent shared cognition, enabling humans and AI agents to build on a common, auditable record of understanding rather than constantly re-establishing context.

Participants share an attributable coordination history while retaining their own perspectives. Different participants may have different authorized views, and disagreement remains part of the record until further articulation changes its status.

The following core concepts define the design principles and constraints that shape ASCP into viable infrastructure for enabling shared cognition.

## **Core Concepts**

### **The Building Blocks**

An articulation is an authored act through which a participant contributes to or changes the coordination of work. Its significance depends on who authored it, the relevant authority, its relationships to other articulations, and the applicable semantic profile.

**Artipoints** are immutable, addressable units of articulated meaning. Referencing a document is one use; representing a decision or a contextual construct is another. The name derives from "point of articulation." Each Artipoint is addressable, semantically typed, and cryptographically secured through its containing Articulation Sequence, functioning as a cognitive atom for structured coordination.

**Articulation Statements** introduce, relate, and annotate Artipoints through authored acts of coordination, like "this doc is relevant to this project" or "flag this item for review," capturing decisions on work organization and pointers to associated content. Applications generate these statements, but they do so based on the actions and assertions of humans or AI agents—entities with agency and point of view. Like a courtroom stenographer whose machine records testimony, the recording mechanism isn't what matters; it's who said what. Similarly, Articulation Statements attribute authorship through their containing Articulation Sequence. Reading that record does not itself confer responsibility or authority on another participant.

### **Structure, Not Content: The Bookmark Pattern**

Artipoints capture cognitive *structure* independently of the dynamic content they reference. In the bookmark pattern, an Artipoint references external content and may identify the application intended to manipulate it. For example, instead of embedding a 50-page paper directly, ASCP has a statement like "this paper is highly relevant to our project" with a URI pointing to the document and an optional indicator of the application intended to manipulate that document.

This allows that 50-page paper to change independent of ASCP. But the cognitive decision—the structural relationship between paper and project at a given point in time—remains immutable and auditable in the coordination graph.

### **Immutable by Design**

The articulated history in ASCP is *immutable*, like a ledger, recording not just current structures and relationships, but their origins. Changes to shared coordination are recorded as additional immutable statements, while the derived coordination state evolves. Supersession, not overwrite, is the model.

### **Composition Through Reference**

Given the immutable nature of ASCP, composition through reference is essential. Each Artipoint is associated with a UUID. New Articulation Statements can reference previously defined Artipoints by their UUID, from which Layer-3 interpretation derives a Directed Acyclic Graph (DAG) of coordination.

This compositional approach—without nesting—enables:

- Machine-readable and human-relatable cognitive statements
- Individual referenceability and auditability
- Incremental composition into complex structures
- Decentralized distribution via ASCP channel synchronization

The grammar includes four articulation patterns: **instantiation** (creating cognitive atoms), **annotation** (enriching them), **connection** (linking atoms), and **construction** (creating and linking simultaneously). Complex cognitive artifacts naturally emerge from simple, atomic statements over time.

### **Channels: Structured Coordination Pathways**

**Channels** define distribution and visibility scopes (like @Org or @Hiring Team) for articulated context. Each Channel has an append-only immutable log and can encrypt payloads with symmetric keys for end-to-end protection of confidential context.

This creates a durable memory for collaborative context—shared across humans and intelligent agents.

## [**ASCP Architecture**](https://github.com/Reframe-Technologies/ascp-draft-specs/blob/main/specs/the-agents-shared-cognition-protocol.md)

| Layer Number | **Layer Name** | **Description** |
| --- | --- | --- |
| 0 | [ALSP](https://github.com/Reframe-Technologies/ascp-draft-specs/blob/main/specs/ascp-logsync-protocol-alsp.md) | Provides local-first synchronization and deterministic convergence of Channel Logs across authorized replicas. |
| 1 | [Channels](https://github.com/Reframe-Technologies/ascp-draft-specs/blob/main/specs/ascp-channels-secure-distribution-layer.md) | Signs Articulation Sequences and optionally encrypts them in Channel Envelopes for distribution. |
| 2 | [**Articulation Layer**](https://github.com/Reframe-Technologies/ascp-draft-specs/blob/main/specs/ascp-artipoint-grammar-a-structure.md) | Defines canonical representation of Artipoint Expressions, Articulation Statements, and Articulation Sequences through a formal ABNF grammar. |
| 3 | **Semantic Evaluation and Views** | Interprets articulations under applicable semantic profiles, evaluates governance and trust, and derives task views, agendas, and project spaces. |

Each layer builds on the previous to enable secure, distributed collaboration infrastructure.

## **Strategic Merits**

Intended for international standardization, ASCP isn't a replacement protocol; it addresses what existing systems miss: the infrastructure gap for articulation work. Traditional tools reduce coordination to chat, tickets, and tasks, obscuring discourse structure. ASCP treats coordination as an append-only log of structured context that preserves meaning, enables traceability, and supports concurrent agency. Its architecture:

- **Articulation Work as Data**: Transforms coordination—meaning negotiation, alignment, sensemaking—into persistent, machine-readable infrastructure. Every act of coordination is addressable, timestamped, and referenceable within the grammar.
- **Composable**: Fits into any workflow or process as discrete building blocks.
- **Interoperable**: A common grammar and compatible semantic profiles let independent implementations exchange and interpret articulations; URI references connect external artifacts.
- **Secure and auditable**: Statements are immutable and cryptographically protected together in signed Articulation Sequences.
- **Human + agent native**: Designed for hybrid collaboration.

ASCP enables durable coordination across participants and applications, reducing the need to reconstruct context as tools and agents change. It is designed as a protocol layer to complement foundational internet standards like TCP/IP, DNS, and HTTP, addressing the coordination infrastructure the internet currently lacks—turning these strategic advantages into concrete capabilities for real workflows.

**Key Capabilities**

- **Durable Coordination Structures**: Artipoints encode contextual relevance, turning ephemeral coordination into durable structure facilitating shared cognition.
- **Making Context Computable**: Transforms coordination into structured, machine-readable data.
- **Cryptographic Channel Architecture**: Preserves authorship integrity and protects confidential context through signed sequences and optional Channel encryption.
- **Streams & Spaces**: Streams organize work threads. Spaces group streams into accountability structures, all as addressable Artipoints themselves.
- **Participation and Governance**: Humans and agents participate in ongoing coordination, establishing goals, contributing decisions, and revising direction within the applicable authority and semantic rules.
- **Auditable Coordination History**: Activity logs unify human and agent contributions, enhancing accountability.

Underpinning these capabilities is ASCP's cryptographic and identity foundation, which supports protected, attributable, and auditable coordination history across humans and agents.

Governance articulations express participation and authority within a coordination context. Applications evaluate those semantics when determining the effects of articulations and permitted actions. Cryptographic protection and runtime security mechanisms enforce the relevant boundaries.

## **Technical Foundation**

ASCP’s security model uses JOSE standards to protect the integrity and, when encrypted, confidentiality of coordination history. Each Articulation Sequence is **JWS-signed** at Layer 1, protecting its contained statements together and providing evidence for authorship verification. This forms a tamper-evident, log-anchored history of coordination. This log-anchored trust model means identity, structure, and governance are recorded in the same append-only history that carries the work itself—creating an attributable record that agents and applications can interpret under compatible semantic profiles, including when participants disagree.

**Channel encryption** provides privacy using symmetric keys (typically AES) shared with authorized participants, distributed through standard JWE key envelopes. Channels distribute **immutable coordination statements**, not chat messages—combining the privacy of end-to-end encryption with the reliability of append-only log sync.

Identity relies on **self-sovereign JWK key pairs**, which may be self-generated or organization-issued, with optional PKI or DID attestation. This hybrid approach avoids both centralized lock-in and crypto-ideological rigidity, supporting real-world collaboration models across individuals and enterprises.

Because governance, membership, and key rotation are themselves expressed as Articulation Statements protected in signed sequences within channel logs, the security history is **auditable and available for local evaluation** to participants with the relevant access. The same log that carries context also carries its access rules and their evolution over time.

Together, these choices support protected exchange and verifiable coordination history across implementations. The coordination grammar supplies the common representation on which semantic profiles build shared interpretation across tools and agents.

## **Formal Grammar Specification**

ASCP includes a [complete ABNF grammar](https://github.com/Reframe-Technologies/ascp-draft-specs/blob/main/specs/ascp-artipoint-grammar-a-structure.md) defining coordination patterns for deterministic parsing and a common representation. Compatible Layer-3 semantic profiles define how constructs and relationships are interpreted. Together, they allow independent implementations to preserve coordination history and derive compatible interpretations of the work.

## **Conclusion**

ASCP provides a protocol layer for shared cognition: immutable, addressable articulations that form an auditable graph of coordination. It transforms articulation work—the ongoing negotiation of meaning, decisions, and context—from invisible overhead into structured, machine-readable infrastructure that both humans and agents can reference, build upon, and synchronize.

The [draft ASCP specifications are available on GitHub](https://github.com/Reframe-Technologies/ascp-draft-specs), including the formal ABNF grammar, Channel envelope specifications, and the ALSP synchronization model. These are evolving drafts in an early-stage repository; core specifications have first drafts in place while other elements are earlier in development. For those interested in AI, agent protocols, and coordination infrastructure for hybrid human-agent systems, we welcome review and feedback as we work toward a mature set of specifications and initial FOSS reference implementations of the whole protocol stack.
