# The Agents Shared Cognition Protocol (ASCP)

**The Implementation of a Cortex Layer for Humans and AI**

Version 0.2 - August 2025

Jeffrey Szczepanski, Founder and CEO, Reframe Technologies, Inc.

# Abstract

Picture this: A designer opens Slack to explain the same project constraints for the third time this week. Meanwhile, their AI assistant—which helped draft those very constraints yesterday—sits waiting for the same background information to be provided again, as if meeting for the first time.

This scene plays out millions of times daily because we lack a foundational Cortex Layer—a persistent substrate for shared cognition. While the modern internet excels at content delivery, it provides no native infrastructure for managing evolving context across collaborators, tools, and agents. As a result, our digital work environments scatter shared understanding across emails, chat threads, and siloed apps—each a black box to the others. Humans are left to coordinate through ad hoc, informal exchanges that fail to scale. Meanwhile, AI agents operate with access to everything and understanding of nothing—stateless participants in workflows they cannot remember or reason about over time.

What's missing is a protocol for context itself.

The **Agents Shared Cognition Protocol (ASCP)** introduces this missing layer through a universal coordination grammar rooted in Artipoints: structured, immutable, addressable statements that form a distributed graph of articulated context. ASCP enables secure, decentralized, and composable collaboration between humans and agents—finally giving both shared memory and coordinated intent.

# Introduction

This document serves as the comprehensive technical specification for the Agents Shared Cognition Protocol (ASCP)—the foundational protocol for enabling persistent, structured coordination between humans and AI agents. While ASCP’s core substrate is a cryptographically secure coordination graph (a DAG of immutable Artipoints), end users never see or manipulate this directly. They work through familiar, human-centric interfaces—timelines, tasks, discussions—while the underlying structure quietly ensures private context stays coherent with the shared whole.

## Document Purpose and Scope

Unlike the companion white paper that introduces ASCP's high-level vision and strategic value, this specification provides the complete technical foundation needed to understand and implement ASCP systems. It establishes:

- **Theoretical foundations**: The discourse theory and coordination principles underlying ASCP's design
- **Core architectural concepts**: Artipoints, Articulation Statements, Streams, Spaces, and Channels
- **Implementation patterns**: How these primitives compose into practical coordination infrastructure
- **Security and governance models**: The cryptographic and social frameworks that make ASCP trustworthy
- **Layered architecture**: How ASCP's four layers work together to enable shared cognition

## What This Document Is Not

This specification focuses on **what** ASCP does and **how** its components work together, but does not include:

- **Formal grammar definition**: See the *ASCP Artipoint Grammar* specification
- **Layer implementation details**: See individual layer specifications (ALSP, Channels, etc.)
- **API specifications**: See implementation-specific documentation

## Intended Audience

This document is designed for:

- **System architects** designing ASCP-compatible systems
- **Protocol implementers** building ASCP clients and servers
- **Researchers** studying coordination protocols and shared cognition
- **Technical decision-makers** evaluating ASCP for organizational adoption

## Reading Guide

The document follows a deliberate progression from theory to implementation. It begins by establishing the coordination infrastructure gap and ASCP's fundamental approach, then defines the core primitives and their relationships. From there, it details how these components compose into working systems before situating ASCP within the broader collaboration ecosystem.

Readers already familiar with the strategic context from the companion white paper may wish to jump directly to the solution architecture. Those focused on implementing specific components will find the most relevant technical details in the later sections on architecture and implementation, while referencing the detailed companion specifications for formal grammar definitions and protocol specifications.

### Specification Map

This section provides a complete reference to the ASCP specification suite, showing how each document fits into the layered architecture and where to look for specific implementation details.

| **Layer / Scope**                       | **Document**                                                 | **Purpose**                                                                                  | **Key Contents**                                                                                                                                                          |
| --------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Top-Level Overview**                  | **The Agents Shared Cognition Protocol**(this document)      | Architectural overview and unifying narrative for ASCP.                                      | Vision, problem statement, core concepts (Artipoints, Streams, Spaces, Channels), four-layer architecture, governance/security principles, normative compliance appendix. |
| **Layer 2 – Articulation Layer**        | **ASCP Artipoint Grammar: A Structure for Shared Cognition** | Defines the formal grammar for immutable, addressable coordination statements.               | Four articulation patterns (Instantiation, Annotation, Connection, Construction), operator taxonomy, payload formats, attributes, ABNF grammar.                           |
| **Layer 1 – Secure Distribution Layer** | **ASCP Channels: Secure Distribution Layer Specification**   | Cryptographic envelope and membership model for distributing Artipoints.                     | Signing/encryption with JOSE (JWS/JWE), channel membership governance, keyframe rotation, channel key envelope format, trust model.                                       |
| **Layer 0 – Log and Transport Layer**   | **ASCP LogSync Protocol (ALSP)**                             | Transport-agnostic append-only log synchronization across replicas.                          | Lamport ordering, push/pull sync models, message formats, channel access proofs (CAKs), deduplication rules, error handling.                                              |
| **Identity & Trust**                    | **ASCP Identity & Trust**                                    | Establishes cryptographic identity, Root anchoring, and trust verification for participants. | Key provisioning, PKI anchoring, identity binding via JWTs/JWKs, recovery strategies, certificate-related Artipoints.                                                     |
| **Bootstrap & Discovery**               | **ASCP: Bootstrap Process and Channel Discovery**            | First-time replica provisioning and channel discovery process.                               | Bootstrapping trust root, retrieving Bootstrap channels, validating trust graph, discovering channel manifests.                                                           |

### Reading Order for Implementers

1. **The Agents Shared Cognition Protocol** – Understand the architecture, core concepts, and layered model.
2. **ASCP Artipoint Grammar** – Implement the syntax/semantics for immutable coordination statements.
3. **ASCP Channels** – Build secure distribution and membership governance for Artipoints.
4. **ASCP LogSync Protocol (ALSP)** – Implement log replication and ordering at the transport layer.
5. **ASCP Identity & Trust** – Integrate identity provisioning and trust anchoring into your security model.
6. **ASCP: Bootstrap Process and Channel Discovery** – Implement first-time setup and channel discovery.

# The Problem: Context Has No Infrastructure

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

## **The Invisible Coordination Burden**

The result is that all the surrounding context that melds these independent digital artifacts into a coherent stream of work remains locked in human memory and informal communication. We have the content, but we lack the coordination context as a first-class digital entity.

ASCP solves this by making coordination context itself a new digital content type—one that both humans and AI agents can directly use and reference. This eliminates the manual burden of conveying and maintaining context, replacing ad-hoc human coordination with structured, persistent, and addressable coordination infrastructure.

# The Solution: A Protocol for Shared Cognition

ASCP addresses the context fragmentation problem through a fundamental shift: instead of building better tools for managing scattered information, we create a **shared cognitive substrate** that both humans and AI agents can read from and write to.

## The Missing Cortex Layer

Today's AI agents operate in isolation, lacking coherent memory and context continuity. Current agent protocols like MCP provide plugin access and A2A enables messaging between agents, but what's missing is the persistent substrate for shared cognition itself available for both humans or agent actions alike.

ASCP implements this missing Cortex Layer through a grammar that agents can read and write, a shared log of articulated coordination, and addressable context scoped by task, person, or goal. This foundation enables agents to contribute meaningfully to workflows, evaluate and adjust plans dynamically, and collaborate transparently with both humans and other agents—finally bridging the gap between stateless tools and true collaborative partners.

## Three Key Innovations

**1. Context as First-Class Content**  
Rather than treating coordination as an informal, human-only activity, ASCP makes context itself a structured, addressable digital entity. Just as HTTP made content universally accessible, ASCP makes coordination context universally readable and writable by both humans and agents.

**2. Immutable Coordination History**  
Instead of forcing consensus on a single "current state," ASCP captures what people actually articulated—when, by whom, and in what context. This creates an auditable trail of collaborative thinking that supports coordination without eliminating the nuanced human reasoning that emerges from seeing how decisions evolved.

**3. Distributed Shared Memory**  
The protocol provides persistent memory that spans across tools, sessions, and participants. When you work with an AI agent on Monday and return on Wednesday, both you and the agent have access to the same coordination context—no re-explanation required.

## How ASCP Works

The protocol operates through **Artipoints**—immutable, addressable statements that capture coordination decisions—organized into collaborative structures called **Streams** and **Spaces**. These elements combine to form a **Cortex Layer**: the missing infrastructure for shared cognition between humans and AI agents.

Unlike current approaches that bolt coordination features onto existing platforms, ASCP provides the foundational protocol that any application can implement, creating true interoperability for collaborative context across the entire digital ecosystem.

## Articulation: The Foundation of Coordination

In Computer-Supported Cooperative Work (CSCW), **articulation work** refers to the process by which cooperating individuals partition work into units, divide it amongst themselves and, after the work is performed, reintegrate it. This foundational concept, introduced by Schmidt and Bannon in 1992, captures the essential coordination challenge: articulation work is the work that makes other work exist and possible—the often invisible effort required to ensure that distributed activities mesh together effectively toward a common goal.

In ASCP, an **Articulation** is an action that results in context being written into working memory—the act of making something explicit and shareable in collaborative work.

To see how this plays out in practice, consider the difference between today's fragmented workflow and what ASCP will enable through Artipoints:

### Before ASCP

You bookmark meeting notes as `https://notion.so/project-kickoff-notes`. Three weeks later, when you ask your AI assistant "Should we add user authentication?", you must:

- Re-upload the notes
- Re-explain that authentication was deprioritized
- Provide context the agent has no memory of—even if it helped you draft those original meeting notes

### After ASCP

When you bookmark those same meeting notes as an Artipoint with a specific working context, your AI assistant gains direct access to the project's foundational context through the shared articulation log.

Three weeks later, when you ask "Should we add user authentication?", the agent can reference those initial decisions without any re-explanation—it has persistent memory of your collaborative work together.

## From Bookmarks to Cognitive Atoms

**Artipoints** are the fundamental building blocks of ASCP—but they start as something deceptively familiar. In their most basic form, think of them as enhanced bookmarks that solve the core limitations of traditional browser bookmarks.

Here's where they diverge from anything we call a "bookmark" today: traditional bookmarks capture *what* you want to remember, while Artipoints capture *why* you're remembering it and *how* it fits into your broader thinking. When you bookmark a research paper, you're saying "this exists and I might want it later." When you create an Artipoint for that same paper, you're making a statement like "this paper provides the theoretical foundation for our approach to user authentication" or "this contradicts our earlier assumption about market size."

This shift from *referencing* to *reasoning* is why we call them cognitive atoms rather than enhanced bookmarks.

## Making Statements About Context

As we dive deeper into Artipoints, we need to shift our mental model from "fancy bookmarks" to **statements about things**. An Artipoint is fundamentally someone saying something specific about something—a point of articulation where context becomes explicit and shareable.

When Jeff creates an Artipoint linking a document to a project stream, he's not just bookmarking—he's making the statement "this document is relevant to this project context." When Sarah flags that same Artipoint for review, she's stating "I need to maintain awareness of this." These aren't just organizational actions; they're declarative statements that become part of the coordination record.

## The Structure of an Artipoint

An Artipoint is essentially a **cognitive atom**—an immutable, addressable statement that captures a single, complete decision or piece of structured thinking. Think of it as a permanent record of a cognitive judgment that can be referenced, built upon, and connected to other thoughts over time.

The key insight is that Artipoints capture **cognitive structure, not content itself**. They record the permanent decisions about how work is organized, what's relevant, how pieces relate to each other, and what dependencies exist—the "scaffolding" of collaborative thinking.

Every Artipoint has four fundamental components:

- **UUID**: A unique identifier that makes each Artipoint permanently addressable and referenceable
- **Source**: Who or what created this cognitive statement (human, AI agent, system)
- **Timestamp**: When this cognitive decision was made (for ordering and auditability)
- **Payload**: The static content associated with the Artipoint or, more commonly, a reference to dynamic content.
- **Expression** (optional): The actual cognitive statement being made

## Expression Types and Immutable Patterns

When an Artipoint includes an expression, it can take one of four forms:

**Instantiation**: Creates a new cognitive atom (like declaring "this is a relevant research paper")

**Connection**: Links existing Artipoints together (like "this task depends on that analysis")

**Construction**: Creates something new while simultaneously connecting it to existing context

**Annotation**: Adds metadata or enriches an existing Artipoint without changing the overall structure

### The "Bookmark Pattern" and Immutability

Rather than embedding dynamic content directly, Artipoints reference external resources through URIs. This means the cognitive judgment ("this paper is highly relevant") remains permanent and auditable, while the actual paper can be updated, moved, or versioned independently.

Why are Artipoints immutable? Consider this scenario: Jeff creates a project stream with documents for Scott to review. Scott syncs before a flight, then articulates discussion points into an agenda. Meanwhile, Jeff adds his own points. When Scott lands and syncs, Jeff reorganizes the agenda. Scott adds another topic, inspiring Jeff to invite Chance to the discussion.

What should Chance see? The "current" agenda? A merged version?

The answer reveals why ASCP prioritizes immutable statements over mutable state. There likely isn't a single "correct" agenda—the participants need to coordinate based on the full context of what everyone actually said and when they said it. Like a court stenographer recording sworn testimony, ASCP captures statements as they were made, creating an auditable trail that supports coordination rather than enforcing a single version of truth.

This creates a persistent **shared cognitive substrate**—a permanent record of not just what was decided, but how the collaborative thinking evolved over time.

## Articulation Statements: Formal Encoding

An **Articulation Sequence** is the formal encoding of one or more **Articulation Statements** formed from **Artipoints** bundled into an immutable, atomic, timestamped, and cryptographically signed sequence. When we say someone has "articulated something," we mean an Articulation Sequence of one or more Articulations Statements has been generated and persisted.

This system prioritizes the human and agent capacity to make statements about work—not to overwrite it, but to evolve it through an append-only coordination log.

### Addressability and Governance

The addressability of Articulation Statements enables effective collaboration by solving a core challenge in Computer-Supported Cooperative Work: notifying everyone about everything increases awareness but reduces relevance to the point of information overload, while notifying only about critical items guarantees awareness but risks missing things people care about.

ASCP solves this through a three-layer governance model that separates orthogonal concerns:

| **Mechanism** | **Governs**                | **Purpose**                                                               |
| ------------- | -------------------------- | ------------------------------------------------------------------------- |
| **Channels**  | Visibility / Access        | Who can receive, decrypt, and store Artipoints (cryptographic scoping)    |
| **Members**   | Structural Inclusion       | Who is part of a Space, Stream, Channel or any other contextual structure |
| **Flags**     | Attention / Working Memory | Who is actively tracking an Artipoint (transparent social contract)       |

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

# Contextual Architecture

Once we have Artipoints and Articulation Statements with the notions of addressability, we have the mechanisms for coordination and collaboration within a specific scope of work. But now the question becomes: what structures do we have to organize each scope of work? To solve for this, ASCP defines three types of scoping.

### Piles: Thematic Groups of Artipoints

**Piles** are thematic groupings of Artipoints, often used to collect related items within a Stream (e.g., "Open Questions" or "Design Ideas"). Piles may be articulated into Streams or directly into Spaces. **Piles of Piles are not allowed**, reinforcing their flat, associative nature. By default, a Pile inherits from the Stream or Space it is articulated into.

### Streams: Context-Switchable Threads of Work

**Streams** represent atomic units of collaboration—shareable containers for each context-switchable thread of work. They can contain Piles and Artipoints but **cannot contain other Streams**. Unlike arbitrary buckets of bookmarks, Streams represent the full scope of Artipoints that relate to a specific goal or outcome.

For example, one might have a Stream for hiring a new IT Manager, implementing a product feature, or preparing for a tradeshow. Each Stream becomes the entire "stadium" for that work. When shared between multiple people, Streams sync Artipoints while each user maintains their own view, creating awareness and enabling collaboration. Each Stream inherits its governance context from its parent Space unless overridden.

### Spaces: Accountability Bubbles

**Spaces** serve as top-level accountability containers that may encapsulate other Spaces and Streams. They represent "Accountability Bubbles"—organizational structures that create distinctions between who is accountable for outcomes and who is responsible for tasks.

Each Space has a "Stream Zero"—a special coordination Stream for managing all other Streams within that Space. Spaces can be hierarchically organized into trees representing both organizational structure or strategic priorities, with full administrative flexibility for managing complex collaborative work.

### Integration: A Unified Grammar

All structures including Piles, Streams and Spaces are implemented using the same Articulation Statement grammar and are implemented as addressable Artipoints. This means:

- You can reference Piles, Streams and Spaces in coordination statements
- They can be flagged, shared, and organized like any other Artipoint
- The same operators and composition rules apply
- All coordination history is preserved in the same immutable log

This unified approach, which we'll formalize in the grammar itself, eliminates the artificial boundaries between "content" and "structure"—everything becomes part of the same addressable, coordination grammar that both humans, through the collaborative environment, and AI agents can read, write, and reason about.

# **Governance and Security**

Beyond structural organization, ASCP provides a declarative governance system for encoding access, delegation, and accountability alongside robust security measures. This comprehensive approach integrates with organizational operations while maintaining the semantic substrate principle: it encodes policies and roles in a durable, auditable form, but does not itself enforce them. Enforcement and behavior are handled by applications and agents that read from and write to this substrate.

The **normative trust model** for the entire ASCP stack is defined in the companion document **ASCP Identity & Trust**. That specification describes how cryptographic identities are established, anchored to a root of trust, and validated over time using ASCP’s immutable log-anchored model. All security mechanisms described here—including channel authentication, membership control, and signature validation—rely on that trust model for correctness, interoperability, and long-term auditability.

The **initial provisioning and discovery process** for ASCP repositories is defined in the companion document **ASCP: Bootstrap Process and Channel Discovery**. This specification describes how new replicas securely obtain the organizational root of trust, retrieve and validate bootstrap content, and discover the channels they are authorized to join. It operationalizes the trust model by defining the secure starting point for all ASCP interactions.

## Cryptographic Security Foundation

All ASCP coordination occurs within cryptographically secured channels using JOSE standards (JWS for signatures, JWE for encryption). This ensures that shared cognition remains secure and appropriately scoped to authorized participants, providing both authenticity verification and privacy protection for sensitive coordination context.

## Access Control and Delegation

Beyond structural organization, ASCP provides a declarative governance system for encoding access, delegation, and accountability. This system operates as a semantic substrate: it encodes policies and roles in a durable, auditable form, but does not itself enforce them. Enforcement and behavior are handled by applications and agents that read from and write to this substrate.

## Groups as Reusable Access Blocks

**Groups** are modular participant sets that may be referenced within any governance attribute. Groups are not part of the structural containment DAG, but they can be composed from other groups using the construction pattern, serving as reusable access or role blocks across structures.

This governance model provides the foundational semantics for access control, structural delegation, and accountability—creating a durable substrate for shared cognition that both humans and AI agents can read, write, and reason about.

## **Access & Delegation**

These attributes may be applied to any Artipoint representing a structural unit (space, stream, pile, or group):

| **Attribute**               | **Meaning**                                                                                                                                                                  |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| member + / -                | Semantic inclusion in a structure (not access control). Can reference users or groups.                                                                                       |
| writer + / -                | Who may articulate into this structure. Can reference users or groups.                                                                                                       |
| owner :=                    | Canonical administrative steward for this structure. Can be a user or group.                                                                                                 |
| inherits :=                 | Explicit specification of inheritance source. If default inheritance is intended, one should specify “default”                                                               |
| deny::\<attribute> :=       | Explicit denial of specific access rights (member, writer, owner, etc.) for a given user or group                                                                            |
| expiration::\<attribute> := | Optional expiration of a specific permission (e.g., writer, member, owner) for a user or group. Value should include both the identity and an ISO 8601 expiration timestamp. |

> **Note:** member implies inclusion or awareness (e.g., being consulted or informed), whereas writer implies authorship privileges.

## **Group Membership and Virtual References**

Groups can be declared as Artipoints of type group, and may contain their own governance attributes and RACI roles. Additionally, implicit virtual groups such as @members, @owners, @role::responsible, and similar may be referenced within other Artipoint governance attributes to dynamically resolve participant sets scoped to a specific structure.

Groups may also be composed from other groups using the construction pattern, forming modular, reusable sets of participants.

### **Example Group Artipoint:**

```bnf
[group-irt-guid, source, timestamp,
  ["group", "Incident Response Team", "urn:group:irt"] .
  ( member + alice@org.com,
     member + bob@org.com,
     owner := carol@org.com,
     role::responsible := alice@org.com,
     role::accountable := carol@org.com
  )
] < {group-guid-ops-team}
```

This defines a group named “Incident Response Team” that inherits members from another group (group-guid-ops-team) and augments it with specific RACI role assignments.

Groups can be declared as Artipoints of type group, and may contain their own governance attributes and RACI roles. Additionally, implicit virtual groups such as @members, @owners, @role::responsible, and similar may be referenced within other Artipoint governance attributes to dynamically resolve participant sets scoped to a specific structure.

## **RACI-Style Roles**

In collaborative settings, clarity around responsibility and accountability is just as important as access and authorship. ASCP adopts a RACI-style model—borrowed from organizational workflows—to explicitly encode who is responsible, accountable, consulted, and informed for a given structure. These roles enable both human and agent participants to understand not just who can act, but who is expected to care, coordinate, or decide.

| **Attribute**        | **Meaning**                               |
| -------------------- | ----------------------------------------- |
| role::responsible := | Person(s) executing the work              |
| role::accountable := | Person ultimately answerable for outcomes |
| role::consulted :=   | People to consult before action           |
| role::informed :=    | People to notify of developments          |
| role::approver :=    | People whose approval is required         |
| role::auditor :=     | People responsible for monitoring changes |
| role::observer :=    | Passive observers included in visibility  |

## Default Inheritance

Unless otherwise specified, governance attributes (writer, member, owner) follow a default inheritance model that traverses up the structural DAG—Streams inherit from their parent Space, Piles inherit from their parent Stream or Space. This inheritance may be explicitly overridden using `inherits :=`. While explicit articulation is always preferred, defaults provide robust fallback semantics in under-specified contexts.

## **Encapsulation and Hierarchy Rules Table**

| **Structure** | **May Contain**                                  | **May Be Contained By** | **Inherits from**                           |
| ------------- | ------------------------------------------------ | ----------------------- | ------------------------------------------- |
| **Space**     | Streams, other Spaces                            | Parent Spaces           | Parent Space (unless overridden)            |
| **Stream**    | Piles, document Artipoints                       | Spaces                  | Parent Space (unless overridden)            |
| **Pile**      | Document Artipoints                              | Streams or Spaces       | Parent Stream or Space (unless overridden)  |
| **Group**     | Users, other Groups by reference or construction | Any structure           | N/A (flat, but composable via construction) |

## **Evaluation Rules**

- **Explicit attributes take precedence** over inferred inheritance
- When inherits := is absent, clients MUST traverse upward in the hierarchal scopes indicated by the DAG to locate inherited values
- writer, owner, and role attributes propagate down the hierarchy unless overridden
- Virtual group references (e.g., @members) MUST be resolved at runtime by inspecting the current attribute state of the parent structure
- Permissions that include expiration timestamps SHOULD be interpreted as invalid after the specified time

# The Grammar of Coordination

## From Bookmarks to Statements About Things

As we delve deeper into ASCP, now we need to shift our mental model of Artipoints from being "fancy bookmarks" to being **statements about things**. An Artipoint is fundamentally someone saying something specific about something—a point of articulation where context becomes explicit and shareable.

Drawing from programming language design, ASCP implements a context-free grammar where **Articulation Statements** contain **Artipoint expressions** using **verb operators**, and can be combined into **compound statements**. However, we deliberately avoid Turing completeness—we're not trying to compute or iterate, just make declarative statements about coordination context.

### Basic Statement Forms

The most fundamental Articulation Statement in ASCP corresponds to an **Instantiation** expression and can be described in English as:

```plaintext
Bookmark this <thing> as <type>
```

This represents creating a new Artipoint with a typed payload that gets added to the current working context. The statement includes the essential components: UUID (generated), source (who's making the statement), timestamp (when), and the instantiation expression itself.

### Extended Statement Forms

Beyond the basic instantiation form, the grammar's three additional expression types enable increasingly sophisticated coordination statements:

**Annotation** expressions enrich existing Artipoints with metadata:

```plaintext
<existing artipoint> is relevant to <user>
<existing artipoint> no longer relevant to <user>
```

**Construction** expressions combine instantiation with relationships:

```plaintext
Bookmark this <thing> as <type> constructed from <existing artipoint(s)>
Bookmark this <thing> as <type> which references <existing artipoint>
```

**Connection** expressions establish pure relationships between existing Artipoints:

```plaintext
<existing artipoint A> references <existing artipoint B>
```

These examples illustrate how **Annotation** expressions enrich existing Artipoints, **Construction** expressions combine instantiation with relationships, and **Connection** expressions establish pure relationships between existing Artipoints. Note that these English descriptions are conceptual representations—the actual formal syntax uses UUIDs, structured payloads, and verb operators as defined in the ABNF specification.

### Operators and Composition

Operators enable composition, transformation, and relationship expression without mutating past statements. The grammar includes semantic verbs like `references`, `replaces`, `extracts`, and `groups` for expressing relationships, alongside mutative operators like `adds` and `removes` for scope management. Hierarchical composition is supported through operators like `assembles` (building structured wholes from components) and `promotes` (elevating one structure into a new form that displaces the original).

These operators work together to enable:

- Addition and combination of context through grouping and assembly operations
- Removal from active scope through archiving operations that preserve history
- Hierarchical relationships that reflect both dependency and containment structures
- Equivalence and transformation statements that capture how cognitive structures evolve over time

This operator taxonomy allows complex coordination patterns to emerge from simple, atomic statements while maintaining the immutable, auditable record that makes collaboration trustworthy.

### Compound Statements and Blocks

Multiple related statements can be bundled into atomic operations, enabling complex coordination actions while maintaining the immutable, append-only principle.

## Formal Grammar Definition

This coordination language is formally specified in ABNF (Augmented Backus-Naur Form), enabling deterministic parsing by both human user interfaces and AI agents. The grammar ensures that every articulation can be unambiguously interpreted and reasoned about, creating a true "context-free grammar for context."

The formal specification enables:

- **Parser development** for different applications and platforms
- **Agent reasoning** about coordination state and implications
- **Audit and verification** of coordination history
- **Interoperability** across tools and systems

This grammatical foundation transforms ad-hoc coordination into structured, machine-readable, and human-interpretable statements—finally giving both humans and AI agents a shared language for collaborative context.

The complete ABNF specification of this coordination grammar—including detailed syntax rules, operator semantics, and parsing requirements—is provided in the accompanying Artipoint Grammar specification document. This formal definition serves as the authoritative reference for implementers building ASCP-compatible systems.

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

## Data Flow

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

# **Normative Compliance Specification**

This section is intended to be appended to the **top-level Master ASCP Specification**. It consolidates all normative requirements from the layered specifications into a single compliance reference. Rather than creating a separate document, this section is designed to live at the end of the Master Spec, providing implementers with a one-stop summary of **MUST / SHOULD / MAY** requirements for ASCP compliance.

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
- Use UUIDv4 or equivalent RFC-4122 compliant UUIDs.
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