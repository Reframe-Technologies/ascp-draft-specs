# The Agents Shared Cognition Protocol (ASCP)

**Architectural Framework and System Model for the Cortex Layer**

Informational Specification — Version 0.5  
November 2025

Jeffrey Szczepanski  
Founder & CEO, Reframe Technologies, Inc.

# **Abstract**

Modern digital work has no protocol for context. Human collaborators coordinate through ad-hoc chat, documents, and links; AI agents interact through stateless prompts and forget everything between sessions. As a result, shared understanding scatters, coordination is fragile, and meaning dissolves as fast as it is created.

The **Agents Shared Cognition Protocol (ASCP)** introduces a missing layer of infrastructure: a **Cortex Layer** that provides durable, structured, and addressable context for humans and agents. ASCP defines a universal grammar for coordination statements—encoded as immutable, signed statements called **Artipoints**—and distributes these statements through secure Channels backed by an append-only synchronization layer.

Where HTTP standardized content delivery, ASCP standardizes **context articulation**. It enables persistent shared memory, auditable reasoning, and seamless collaboration across tools, devices, and intelligent agents. The result is an interoperable substrate for shared cognition: a distributed, private, and composable graph of articulated context that evolves as collaborators think and work together.

# **Introduction**

The **Agents Shared Cognition Protocol (ASCP)** is the foundational protocol for creating a persistent, structured, and secure coordination substrate shared by humans and AI agents. It addresses a critical gap in today’s computing environment: there is **no common infrastructure for representing, sharing, and evolving context itself**.

This document provides the architectural framework for ASCP, describing the conceptual model and system architecture that organize the protocol’s layered specifications.

*To ground the architectural model, we begin with ASCP’s atomic primitive: the Artipoint.*

 Artipoints are immutable, addressable units of articulated context: structured statements about relevance, relationships, intent, and decision-making. Artipoints compose into **Articulation Statements**, which capture discrete acts of coordination, and these accumulate into an append-only, cryptographically verifiable log that forms a distributed DAG of shared understanding.

End users do not interact with this DAG directly. Instead, applications materialize it through familiar constructs—timelines, threads, agendas, task lists—while ASCP ensures that the underlying context remains coherent, durable, and scoped appropriately for each participant. This allows private context to remain consistent with shared structure, resolving the long-standing tension between individual focus and collective awareness.

This architectural specification defines:

- the motivations and theoretical foundations of ASCP;
- the core architectural primitives (Artipoints, Streams, Spaces, Channels);
- the layered protocol model for grammar, security, and synchronization; and
- the authorship, trust, and governance framework underpinning ASCP’s collaborative semantics.

It is the **conceptual and architectural overview** for the ASCP specification suite. All normative protocol requirements—including the formal grammar, channel envelope structure, log synchronization mechanisms, identity and trust model, and governance semantics—are defined in the companion specifications referenced in the **Specification Map** below.

# Motivations for ASCP

Despite the explosion of tools and connectivity, shared working context remains fragmented and ephemeral. Bookmarks are personal and lack structure, while teams coordinate through brittle messaging systems with no persistent memory. Work gets siloed across incompatible applications, each operating as a black box to the others. Meanwhile, AI agents require repeated prompt injections and lack any form of shared memory, forcing users to re-explain context even when the same agent helped create that context originally.

There is no foundational layer to manage shared intent, knowledge, and responsibility—no protocol for context itself.

## The Digital Paper Problem

This fragmentation stems from a deeper issue with our current digital infrastructure: while we've successfully encoded all content into digital form with links to reference it, we're still operating under what we call the "digital paper metaphor."

Just as physical papers scattered across a desk require humans to manually organize and maintain the context connecting them, our digital artifacts exist as independent pieces that humans must manually coordinate:

- Figma designs
- Slack threads
- Local files
- Database records
- Meeting notes

When we copy and paste links between applications for collaboration, we not only lose the context of how and why these connections were made, but we also burden humans with continuously maintaining this coordination overhead.

## ASCP as the Missing Cortex Layer

The coordination context that melds independent digital artifacts into coherent work remains locked in human memory and informal communication. We have the content, but we lack any structure for the **articulation work**—the work that coordinates the work, so that distributed activities can mesh together effectively—**forming** a first-class digital entity called the Cortex Layer.

ASCP implements this missing Cortex Layer by making articulation work itself structured, persistent, and addressable. Through a grammar that both humans and agents can read and write, a shared log of coordination statements, and addressable context scoped by task, person, or goal, ASCP transforms ad-hoc coordination into durable infrastructure. This enables agents to contribute meaningfully to workflows, evaluate and adjust plans dynamically, and collaborate transparently—finally bridging the gap between stateless tools and true collaborative partners.

## Three Key Innovations

**1. Context as First-Class Content**  
Rather than treating coordination as an informal, human-only activity, ASCP makes context itself a structured, addressable digital entity. Just as HTTP made content universally accessible, ASCP makes coordination context universally readable and writable by both humans and agents.

**2. Immutable Coordination History**  
Instead of forcing consensus on a single "current state," ASCP captures what people actually articulated—when, by whom, and in what context. This creates an auditable trail of collaborative thinking that supports coordination without eliminating the nuanced human reasoning that emerges from seeing how decisions evolved.

**3. Distributed Shared Memory**  
The protocol provides persistent memory that spans across tools, sessions, and participants. When you work with an AI agent on Monday and return on Wednesday, both you and the agent have access to the same coordination context—no re-explanation required.

# **What This Document is and is Not**

This document defines the **scope, structure, and architectural boundaries** of the Agents Shared Cognition Protocol (ASCP). It describes the core concepts, design principles, and system model that underpin ASCP, but it **does not** contain the normative protocol specifications themselves.

Specifically, this document **does not** define:

- the formal Artipoint grammar,
- the cryptographic channel envelope structure,
- log synchronization mechanics (ALSP),
- identity and key management semantics, or
- governance and access-control rules.

Those topics are defined normatively in the companion ASCP specifications.

The **Specification Map** provides the authoritative reference to all such documents.

Other than the consolidated compliance matrix in the Appendix—which summarizes normative requirements specified elsewhere—this document is **informational** and should be read as the architectural and conceptual foundation of ASCP, not as a protocol definition.

# Intended Audience

This document is designed for:

- **System architects** designing ASCP-compatible systems
- **Protocol implementers** building ASCP clients and servers
- **Researchers** studying coordination protocols and shared cognition
- **Technical decision-makers** evaluating ASCP for organizational adoption

# Reading Guide

This document is the **third in a three-part sequence** introducing the ASCP ecosystem.

Readers seeking broader context may wish to begin with two higher-level whitepapers before reading this document:

1. **Building the Missing Cortex Layer** – motivation for the Cortex Layer and the gap in existing agent and collaboration infrastructure.
2. **Introduction to ASCP** – conceptual overview of Artipoints, Streams, Spaces, and Channels.

This document provides the **architectural overview** that unifies those concepts and prepares readers for the detailed specifications in the ASCP suite. The sections below help different audiences navigate this document efficiently.

### **For Readers New to ASCP**

Begin with the **Motivations for ASCP** and **Underpinnings of ASCP** sections. These introduce the coordination challenges ASCP addresses and establish the conceptual foundations that drive the design.

### **For Readers Seeking a Conceptual Model**

Read **Understanding Artipoints** and **Contextual Structures**. These sections define the cognitive building blocks of the Cortex Layer and how they compose into Piles, Streams, and Spaces.

### **For Architects and System Designers**

Proceed to the **Collaboration Model** and the **ASCP Architectural Layered Model**. These sections explain how the data model, security model, and synchronization model fit together.

### **For Implementers**

Use the **Specification Map** as the entry point. It enumerates each companion specification and indicates where the normative requirements for Artipoints, Channels, synchronization, identity, and governance are defined.

### **For Reviewers Focused on Security or Governance**

Consult the **Governance and Security** section, which provides the architectural overview of trust, authorship, membership, and declarative governance semantics.

### **For Compliance and Interoperability Assessment**

See the **Appendix: Normative Compliance Specification**, which consolidates MUST/SHOULD/MAY requirements from the normative specifications into a single reference.

# Specification Map

This section provides a complete reference to the ASCP specification suite, showing how each document fits into the layered architecture and where to look for specific implementation details.

| **Layer / Scope**                       | **Document**                                                 | **Purpose**                                                                                  | **Key Contents**                                                                                                                                                          |
| --------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Top-Level Overview**                  | **The Agents Shared Cognition Protocol**(this document)      | Architectural overview and unifying narrative for ASCP.                                      | Vision, problem statement, core concepts (Artipoints, Streams, Spaces, Channels), four-layer architecture, governance/security principles, normative compliance appendix. |
| **Layer 2 – Articulation Layer**        | **ASCP Artipoint Grammar: A Structure for Shared Cognition** | Defines the formal grammar for immutable, addressable coordination statements.               | Four articulation patterns (Instantiation, Annotation, Connection, Construction), operator taxonomy, payload formats, attributes, ABNF grammar.                           |
| **Layer 1 – Secure Distribution Layer** | **ASCP Channels: Secure Distribution Layer Specification**   | Cryptographic envelope and membership model for distributing Artipoints.                     | Signing/encryption with JOSE (JWS/JWE), channel membership governance, keyframe rotation, channel key envelope format, trust model.                                       |
| **Layer 0 – Log and Transport Layer**   | **ASCP LogSync Protocol (ALSP)**                             | Transport-agnostic append-only log synchronization across replicas.                          | Lamport ordering, push/pull sync models, message formats, channel access proofs (CAKs), deduplication rules, error handling.                                              |
| **Identity & Trust**                    | **ASCP Identity & Trust**                                    | Establishes cryptographic identity, Root anchoring, and trust verification for participants. | Key provisioning, PKI anchoring, identity binding via JWTs/JWKs, recovery strategies, certificate-related Artipoints.                                                     |
| **Governance & Access Control**         | **ASCP Governance and Access Control**                       | Specifying participation, access control, permission inheritance, and role resolution        | Governance attributes (member, writer, owner), inheritance rules, groups and virtual groups, RACI-style roles, and the evaluation algorithm.                              |
| **Bootstrap & Discovery**               | **ASCP: Bootstrap Process and Channel Discovery**            | First-time replica provisioning and channel discovery process.                               | Bootstrapping trust root, retrieving Bootstrap channels, validating trust graph, discovering channel manifests.                                                           |

### Reading Order for Implementers

1. **The Agents Shared Cognition Protocol** (This Document) – Understanding the architecture, core concepts, and layered model.
2. **ASCP Artipoint Grammar** – Implement the syntax/semantics for immutable coordination statements.
3. **ASCP Channels** – Build secure distribution and membership governance for Artipoints.
4. **ASCP LogSync Protocol (ALSP)** – Implement log replication and ordering at the transport layer.
5. **ASCP Identity & Trust** – Integrate identity provisioning and trust anchoring into your security model.
6. **ASCP: Bootstrap Process and Channel Discovery** – Implement first-time setup and channel discovery.

# Underpinnings of ASCP

ASCP addresses the context fragmentation problem through a fundamental shift: instead of building better tools for managing scattered information, we create a **shared cognitive substrate** that both humans and AI agents can read from and write to.

Unlike current approaches that bolt coordination features onto existing platforms, ASCP provides the foundational protocol that any application can implement, creating true interoperability for collaborative context across the entire digital ecosystem.

## Articulation: The Foundation of Coordination

In Computer-Supported Cooperative Work (CSCW), **articulation work** refers to the process by which cooperating individuals partition work into units, divide it amongst themselves and, after the work is performed, reintegrate it. This foundational concept, introduced by Schmidt and Bannon in 1992, captures the essential coordination challenge: articulation work is the work that makes other work exist and possible—the often invisible effort required to ensure that distributed activities mesh together effectively toward a common goal.

In ASCP, an **Articulation** is an action that results in context being written into working memory—the act of making something explicit and shareable in collaborative work.

To see how this plays out in practice, consider the difference between today's fragmented workflow and what ASCP will enable through Artipoints:

### Before ASCP

You bookmark meeting notes as `https://notion.so/project-kickoff-notes`. Three weeks later, when you ask your AI assistant "Should we add user authentication?", you must re-upload the notes, re-explain that authentication was deprioritized, and provide context the agent has no memory of—even if it helped you draft those original meeting notes.

### After ASCP

When you bookmark those same meeting notes as an Artipoint with a specific working context, your AI assistant gains direct access to the project's foundational context through the shared articulation log. Three weeks later, when you ask "Should we add user authentication?", the agent can reference those initial decisions without any re-explanation—it has persistent memory of your collaborative work together.

# **Understanding Artipoints**

Artipoints are the foundational primitive of ASCP. Each Artipoint is an immutable, addressable, and author-attributed unit of articulated context—a durable statement that can be referenced, related, and built upon over time. They form the cognitive substrate of the Cortex Layer, enabling shared understanding across humans and AI agents.

## **Artipoints Are Structured Bookmarks**

Artipoints begin from a familiar idea: the bookmark. But where a traditional bookmark says *"this exists, and I may want it later,"* an Artipoint says *why* the item matters and *how* it fits within ongoing work.

Instead of capturing only a pointer, an Artipoint captures a structured judgment such as:

- "This document is relevant to our authentication strategy."
- "This research contradicts our earlier market assumption."

This shift—from **referencing content** to **reasoning about context**—is what makes Artipoints the basic building blocks of shared cognition rather than *just* enhanced bookmarks.

## **Artipoints Are Immutable Cognitive Atoms**

Each Artipoint is a **cognitive atom**: a permanent, atomic record of a specific coordination decision. An Artipoint contains:

- a **UUID** for addressability
- an **author** (human or agent)
- a **timestamp**
- a **payload** (usually a URI referencing external content)
- an **expression** (optional) describing the cognitive statement being made

Crucially, Artipoints are **immutable**. Once articulated, they are never overwritten—only superseded or complemented by new Artipoints.

This immutability solves the "What should I see?" problem inherent in collaborative work. Instead of enforcing a single evolving state, ASCP preserves the full sequence of what each participant said and when they said it, allowing applications to derive coherent views without losing historical meaning. The result is a durable, auditable substrate of shared understanding.

## **Artipoints Are Statements About Things**

At their core, Artipoints are **declarative statements**. They assert something about an artifact, idea, person, task, or structure—expressing relevance, dependency, categorization, intent, ownership, priority, or scope.

When someone creates an Artipoint linking a document to a project stream, they are not merely organizing—it is a formal act of articulation: *"this belongs in this context."* When someone flags that Artipoint, they are making another statement: *"this matters to me; keep it in my working memory."*

Through countless small statements like these, the coordination graph emerges—not as an inferred or overwritten state, but as a transparent, explicit history of how collaborative thinking evolves.

## **Artipoints Form Articulation Sequences and Statements**

Artipoints do not exist in isolation. When participants articulate context, Artipoints are emitted as part of an **Articulation Sequence**—an immutable, timestamped, and cryptographically signed bundle that records one or more coordination acts. An Articulation Sequence is the atomic unit of authorship in ASCP: it captures *who* said *what* and *when*, and is appended to the coordination log without overwriting prior statements.

Each **Articulation Statement** within a sequence contains exactly one expression. At a conceptual level, ASCP supports four expression types:

- **Instantiation** — create a new Artipoint
- **Connection** — relate existing Artipoints
- **Construction** — create a new Artipoint while linking it to existing context
- **Annotation** — enrich the interpretation of an existing Artipoint

These expressions represent the minimal vocabulary needed to express how context is introduced, related, and evolved over time. Together they allow human collaborators and AI agents to contribute durable structure to shared work without requiring mutable global state.

The complete formal definition of Articulation Sequences and expression types—including syntax, operators, and parsing rules—is defined in the companion specification **ASCP Artipoint Grammar**.

## **Artipoints Support Addressability**

The addressability of Articulation Statements enables effective collaboration by solving a core challenge in Computer-Supported Cooperative Work: notifying everyone about everything increases awareness but reduces relevance to the point of information overload, while notifying only about critical items guarantees awareness but risks missing things people care about.

ASCP solves this through a three-layer model that separates orthogonal concerns:

| **Mechanism** | **Governs**                | **Purpose**                                                               |
| ------------- | -------------------------- | ------------------------------------------------------------------------- |
| **Channels**  | Visibility / Access        | Who can receive, decrypt, and store Artipoints (cryptographic scoping)    |
| **Members**   | Structural Inclusion       | Who is part of a Space, Stream, Channel or any other contextual structure |
| **Flags**     | Attention / Working Memory | Who is actively tracking an Artipoint (transparent social contract)       |

See the **Collaboration Model** section for more details.

# Contextual Structures

Artipoints and Articulation Statements provide the atomic mechanisms for coordination. But coordination requires organization—a way to group, scope, and structure related work. ASCP defines three types of contextual structures to organize collaboration: **Piles**, **Streams**, and **Spaces**.

Critically, these structures are not special system objects. They are themselves Artipoints, defined using the same Articulation Statement grammar as any other coordination act. This means:

- You can reference Piles, Streams and Spaces in coordination statements
- They can be flagged, shared, and organized like any other Artipoint
- The same operators and composition rules apply
- All coordination history is preserved in the same immutable log

This unified approach eliminates the artificial boundaries between "content" and "structure"—everything becomes part of the same addressable coordination grammar that both humans (through collaborative applications) and AI agents can read, write, and reason about.

## Piles: Thematic Groups of Artipoints

**Piles** are thematic groupings of Artipoints, often used to collect related items within a Stream (e.g., "Open Questions" or "Design Ideas"). Piles may be articulated into Streams or directly into Spaces. **Piles of Piles are not allowed**, reinforcing their flat, associative nature. By default, a Pile inherits from the Stream or Space it is articulated into.

## Streams: Context-Switchable Threads of Work

**Streams** represent atomic units of collaboration—shareable containers for each context-switchable thread of work. They can contain Piles and Artipoints but **cannot contain other Streams**. Unlike arbitrary buckets of bookmarks, Streams represent the full scope of Artipoints that relate to a specific goal or outcome.

For example, one might have a Stream for hiring a new IT Manager, implementing a product feature, or preparing for a tradeshow. Each Stream becomes the entire "stadium" for that work. When shared between multiple people, Streams sync Artipoints while each user maintains their own view, creating awareness and enabling collaboration. Each Stream inherits its governance context from its parent Space unless overridden.

## Spaces: Accountability Bubbles

**Spaces** serve as top-level accountability containers that may encapsulate other Spaces and Streams. They represent "Accountability Bubbles"—organizational structures that create distinctions between who is accountable for outcomes and who is responsible for tasks.

Each Space has a "Stream Zero"—a special coordination Stream for managing all other Streams within that Space. Spaces can be hierarchically organized into trees representing both organizational structure or strategic priorities, with full administrative flexibility for managing complex collaborative work.

# Collaboration Model

### Channels: Cryptographically Scoped Knowledge Base

Articulation Sequences are distributed via **Channels**, which work like group chats for coordination among humans and AI agents. Channel membership directly controls privacy—who receives a copy of the Articulation Statement and has it available in their working context.

Channels establish the **shared knowledge base** for everyone participating in a scope of work—the common context and information that all channel members can access when needed, similar to long-term memory that doesn't actively compete for attention. This scope can range from large collaborative teams to an audience of one, enabling private personal context that remains connected to nodes in the broader coordination graph.

### Members: Structural Participation

**Member** attributes define semantic inclusion in collaborative structures—who is contextually part of a Space, Stream, Pile, or Channel. This creates an important distinction: people can be members of the same Space or Stream but have different views of the coordination graph depending on which Channel(s) they belong to. Channel membership determines which Artipoints you can decrypt and access, while structural membership indicates your contextual participation in collaborative work—regardless of your specific permissions or visibility into all aspects of that work.

### Flagging: Working Memory as Social Contract

Beyond the shared knowledge base, each individual and agent needs different items in their active working memory. While everyone in a channel shares the same broader context, what belongs in each person's day-to-day focus varies based on their role and responsibilities.

**Flagging** serves as a transparent social contract mechanism for managing individual working memory within shared cognition:

- When you're **flagged** on an Artipoint, you're signaling it's relevant to you and you want to maintain awareness of changes to it
- When you **flag someone else**, you're explicitly bringing it into their working memory and requesting their awareness
- Anyone can add or remove themselves from being flagged on a particular Artipoint—this is about opting in or out of monitoring specific items
- Flagging is visible to all channel members, creating transparency about who's actively maintaining awareness of what

When you unflag yourself, items are archived from your active awareness while remaining in your contextual history—available if you're flagged again later or want to pull it back into your working memory at any time.

This model balances shared awareness with personal focus, helping teams maintain a durable memory system that supports dynamic coordination without information overload.

Flag Operations:

- flag + user\@domain — user is actively tracking this Artipoint
- flag - user\@domain — user is no longer tracking it

Scope:

- **Applies to any Artipoint**, including Streams, Piles, Spaces, and content-level statements
- Flags are visible to all Channel members and act as a social contract—indicating who is maintaining awareness
- Used to drive attention, notifications, inboxes, and working memory interfaces

# The Grammar of Coordination

Drawing from programming language design, ASCP implements a context-free grammar where **Articulation Statements** contain **Artipoint expressions** using **verb operators**, and can be combined into **compound statements**. However, we deliberately avoid Turing completeness—we're not trying to compute or iterate, just make declarative statements about coordination context.

ASCP defines a small, structured language for expressing coordination as immutable statements. This language is intentionally minimal—just expressive enough to capture how humans and agents articulate relevance, relationships, structure, and change—while remaining easy to parse deterministically across systems.

At the core of the grammar are **four articulation types**, each representing a distinct kind of coordination act:

- **Instantiation** — Creates a new Artipoint (a cognitive atom).
- **Annotation** — Enriches or modifies the interpretation of an existing Artipoint without altering its identity.
- **Connection** — Expresses a relationship between existing Artipoints.
- **Construction** — Creates a new Artipoint while simultaneously establishing one or more relationships to existing ones.

Together, these four expression types form the building blocks of the coordination graph. Every Articulation Statement encodes exactly one such expression, allowing complex structures to emerge from simple, atomic statements over time.

## **Example (Conceptual)**

A simple Instantiation may be expressed conceptually as:

```plaintext
Bookmark this <thing> as <type>
```

This corresponds to creating a new Artipoint with a typed payload, authored by a specific participant at a specific time. In practice, the formal syntax uses UUIDs, structured payloads, and verb operators—but the conceptual intent remains the same: **to articulate a discrete unit of shared context.**

## **Formal Definition**

The **complete coordination grammar**—including the precise ABNF syntax, operator vocabulary, composition rules, and formal semantics for all expression types—is defined in the companion specification **ASCP Artipoint Grammar: A Structure for Shared Cognition**, which serves as the authoritative reference for implementers building parsers, validating statements, or extending the operator taxonomy.

# **Governance and Security**

ASCP separates **coordination semantics** from **policy enforcement**, providing a durable, declarative substrate for expressing governance without embedding application-level behavior. Governance in ASCP is articulated through immutable Artipoints and governs *who participates*, *how authority flows*, and *how accountability is represented* within collaborative structures. Enforcement is left to applications and agents that interpret these statements.

This section summarizes the governance model at a high level. The complete normative semantics, attribute definitions, inheritance rules, and evaluation logic are defined in the companion specification **ASCP Governance and Access Control**.

## **Governance Principles Overview**

ASCP governance is designed to express real-world collaborative structures—membership, delegation, responsibility, and visibility—as durable, auditable statements. **Critically, governance in ASCP is purely declarative: it articulates authority relationships and participation rules but does not enforce them.** Policy enforcement is left to applications and agents that interpret these governance statements.

Five core principles define this model:

### **1. Groups as Reusable Access Blocks**

ASCP allows the definition of reusable **Groups**, which act as modular sets of participants. Groups may reference other groups and may be assigned roles or privileges within any collaborative structure. They provide a composable way to express organizational structure and reuse participant sets across Spaces, Streams, and Piles.

### **2. Access & Delegation**

Governance attributes articulate who may **participate in**, **author within**, or **administer** a given structure. These attributes include membership, authorship rights, administrative stewardship, and optional scoping or denial rules. They encode authority relationships—not runtime enforcement.

### **3. Group Membership and Virtual References**

Governance attributes may reference:

- explicitly defined Groups
- individual participants
- **virtual groups** such as @members, @owners, or role-based selectors

These virtual references enable dynamic evaluation of participant sets based on the current state of a structure’s governance attributes.

### **4. RACI-Style Roles**

ASCP supports declarative assignment of **responsibility, accountability, consultation, and notification** roles. These roles describe the human or agent participation model and provide a semantic map for applications determining who should act, approve, or be informed of changes.

### **5. Default Inheritance**

Governance attributes follow a **hierarchical inheritance model** aligned with the structural DAG:

- Streams inherit from their parent Space
- Piles inherit from their parent Stream or Space
- Groups do not participate in structural inheritance but may be composed

Explicitly articulated attributes override inherited values. This enables both broad organizational defaults and fine-grained local governance.

All governance rules are captured as immutable Artipoints, allowing clients to reconstruct history, evaluate access, and derive visibility semantics consistent with the append-only nature of the protocol.

For the complete governance substrate—including attribute definitions, the evaluation algorithm, virtual group resolution, RACI semantics, inheritance resolution, and security interactions—see the companion specification **ASCP Governance and Access Control**.

## **Trust Model**

The authenticity and integrity of all governance statements rely on ASCP’s cryptographic identity and trust architecture, defined in the companion specification **ASCP Identity & Trust**. That document describes:

- cryptographic identity provisioning
- trust root establishment
- validation of authorship via log-anchored signatures
- long-term identity evolution and revocation

All governance and access semantics assume the correctness of this underlying trust model.

## **Bootstrapping**

Initial provisioning of an ASCP replica—including retrieval of the trust root, discovery of organizational governance roots, and enrollment into Channels—is defined in the companion specification **ASCP: Bootstrap Process and Channel Discovery**. Governance evaluation depends on clients correctly acquiring and validating this bootstrap information.

# ASCP Architectural Layered Model

ASCP's technical architecture is organized into four distinct layers, each serving a specific role in the coordination infrastructure. This separation enables modular implementation while maintaining clear boundaries between protocol concerns.

## Layer 3 - View Layer (Application-Specific)

The View Layer materializes the coordination graph into user-facing interfaces—agendas, task lists, project dashboards, and contextual workspaces. This layer maintains the coordination DAG in memory, built from the immutable coordination log, and interprets it to present dynamic, interactive views tailored to specific workflows and user needs.

Applications at this layer render locally based on their access to decryptable Channels, creating personalized perspectives on shared coordination context. Since presentation requirements vary dramatically across use cases—from agent interfaces to human collaborative environments—this layer remains intentionally outside the ASCP protocol specification, allowing maximum flexibility for implementers.

## Layer 2 - Articulation Layer (The Coordination Grammar)

The Articulation Layer contains the serialized representation of the coordination graph as typed, addressable, immutable statements. This is where Artipoints and Articulation Statements live as structured data following the formal coordination grammar.

Each statement in this layer captures atomic coordination decisions—instantiations, connections, constructions, and annotations—that collectively build the persistent memory of collaborative work. The grammar ensures that both human applications and AI agents can deterministically parse, validate, and reason about coordination context.

**Technical specification:** The complete syntax, semantics, and parsing rules are defined in the *ASCP Artipoint Grammar* specification document.

## Layer 1 - ASCP Channels (Distribution and Security)

ASCP Channels provide the distribution and encryption infrastructure for Articulation Statements. Each Channel operates as a logical distribution group with its own membership, encryption keys, and access control policies.

All articulations are cryptographically signed (for authenticity and integrity) and optionally encrypted (for privacy) using JOSE standards (JWS for signatures, JWE for encryption). Channel membership directly controls who receives copies of articulation statements and can decrypt their payloads.

This layer handles key management, member enrollment/removal, and the cryptographic envelope that protects coordination context during distribution. It ensures that shared cognition remains secure and appropriately scoped to authorized participants.

**Technical specification:** Channel structure, encryption protocols, and membership management are detailed in the *ASCP Channels: Secure Distribution Layer Specification* document.

## Layer 0 - Log and Transport (ALSP)

The foundational layer provides local storage and peer-to-peer synchronization for encrypted articulation messages. Using conflict-free replicated data type (CRDT) like principles, Layer 0 ensures that all participants eventually converge on the same coordination state despite network partitions, offline work, and concurrent modifications.

The ASCP Log Sync Protocol (ALSP) handles the mechanics of distributing append-only logs across peers, managing backfill for new channel members, and maintaining consistency guarantees without requiring central coordination infrastructure.

**Technical specification:** Synchronization algorithms, logical log management, conflict resolution, and transport protocols are defined in the *ASCP Log Sync Protocol (ALSP)* specification document.

# ASCP Data Flow

The interaction between layers follows a clear data flow pattern:

1. **Articulation Creation:** Users or agents author coordination statements using the Layer 2 grammar
2. **Channel Encoding:** Layer 1 wraps statements in cryptographic envelopes and manages payload distribution for channel members
3. **Local Storage and Sync:** Layer 0 stores (optionally) encrypted articulation statements locally in channel log files and synchronizes with peers via ALSP
4. **Decryption and Indexing:** Recipients decrypt and index articulation statements they have access to
5. **View Materialization:** Layer 3 applications render personalized views from the coordination graph

This architecture ensures that coordination context remains persistent, secure, and eventually consistent across all participants while allowing flexible presentation and interaction patterns at the application layer.

# Why ASCP Matters

Just as HTTP standardized content delivery, ASCP standardizes context articulation. Its innovation lies not in inventing bookmarks or coordination—but in formalizing and distributing them as first-class, composable, secure primitives. This approach unlocks:

- Richer human-human collaboration
- Seamless human-agent teaming
- Persistent memory for multi-agent systems
- Auditable coordination history

By providing structured, persistent, and addressable coordination context, ASCP transforms the fundamental relationship between humans and AI agents—from repeated explanations to shared understanding, from isolated tools to collaborative partners.

Crucially, the coordination graph that makes this possible remains invisible to end users. They operate entirely through clients that render their work into natural, familiar forms—agendas, piles, threads—while the DAG ensures those views remain both **private where needed** and **coherent to the shared context** across participants. This removes the forced choice between “seeing everything” and “missing out” that plagues today’s tools, enabling natural human interaction with guaranteed continuity and relevance behind the scenes. ASCP is the missing substrate for the AI era.

# Conclusion

ASCP isn't just a feature or product—it's an architecture for shared cognition. Through Artipoints, Articulation Statements, and a formal coordination grammar, it replaces the brittle, isolated, stateless defaults of modern computing and AI with a secure, composable, and semantically rich protocol for shared context.

This forms the foundational layer for the next generation of collaboration: human, artificial, and hybrid.

# **References**

The following documents provide the broader context and layered exposition leading up to this ASCP whitepaper. They form a progressive narrative arc from high-level framing to technical specification:

### **Building the Missing Cortex Layer: Why Human and AI Collaboration Needs Common Persistent Context**

*By Jeffrey Szczepanski – June 2025*

This architectural explainer introduces the **Cortex Layer** as a foundational substrate missing from today’s agent ecosystems. It examines how current protocols (like MCP and A2A) fail to support structured, persistent collaboration, and positions the Agent Shared Cognition Protocol (ASCP) as the infrastructure required for shared understanding and coordination over time.

### **Introduction to the Agent Shared Cognition Protocol (ASCP)**

*By Jeffrey Szczepanski – July 2025*

This document introduces the key primitives of ASCP: **Artipoints**, **Articulation Statements**, **Streams**, and **Spaces**. It explains the rationale behind ASCP’s design, the layers of its architecture and how it supports persistent shared memory across agents and humans. Ideal for system architects and contributors looking to understand the conceptual model before diving into full specification.

# **Appendix: Normative Compliance Specification**

The authoritative definitions of these requirements reside in the companion normative specifications. This appendix consolidates them into a single compliance reference for convenience, providing implementers with a one-stop summary of **MUST / SHOULD / MAY** requirements for ASCP compliance.

## **1. Compliance Levels**

### **Core ASCP Compliance**

An implementation MUST:

- Support Layer 2 (Artipoint Grammar) and Layer 1 (Channels) fully.
- Implement at least one Layer 0 transport binding (WebSockets/TLS REQUIRED).
- Support baseline cryptographic algorithms.

### **Full ASCP Compliance**

An implementation MUST:

- Meet all Core ASCP requirements.
- Implement the full ALSP sync model (push and pull modes).
- Support channel bootstrap and discovery procedures.
- Implement all required governance and key rotation mechanisms.

## **2. Normative Requirements by Layer**

### **Layer 2 – Artipoint Grammar**

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

### **Layer 1 – Channels**

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

### **Layer 0 – ALSP**

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

## **3. Bootstrap and Discovery**

**MUST:**

- Retrieve and validate bootstrap documents.
- Discover organizational manifests including channel metadata and public keys.
- Verify bootstrap material signatures.

**SHOULD:**

- Support live bootstrap updates.

**MAY:**

- Cache bootstrap metadata locally with expiration policy.

## **4. Versioning and Extensibility**

**MUST:**

- Include ascp\_version in all protocol handshakes.
- Reject unsupported major versions.

**SHOULD:**

- Support minor version negotiation.

**MAY:**

- Ignore unknown attributes/operators while preserving them in logs.

## **5. Security Considerations**

**MUST:**

- Protect all payloads in transit with TLS 1.3 or higher.
- Enforce replay protection using nonces and timestamps.
- Validate all signatures before processing Artipoints.

**SHOULD:**

- Implement key revocation workflows.

**MAY:**

- Support post-quantum algorithms when available.

## **6. Conformance Testing**

An ASCP-compliant implementation SHOULD pass:

- **Grammar validation tests** (syntax parsing, serialization).
- **Channel crypto tests** (sign/verify, encrypt/decrypt, key rotation).
- **LogSync interoperability tests** (multi-peer sync scenarios).
- **Bootstrap trust chain validation**.

Official conformance test fixtures and validation tooling will be published separately.