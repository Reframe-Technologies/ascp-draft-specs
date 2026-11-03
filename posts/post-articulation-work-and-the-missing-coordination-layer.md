# Articulation Work and the Missing Coordination Layer

*CSCW, Human-Agent Collaboration, and the Case for an Open Protocol*

**Jeff Szczepanski | 3 November 2026**

## **The Coordination Gap**

The internet has repeatedly advanced by making important structures explicit and interoperable. Open protocols first gave us durable communication across systems, then shared representations for addressable content. Yet one essential structure of collaborative work remains outside that progression: coordination itself.

Modern work is distributed across messaging systems, document systems, issue trackers, workflow tools, and domain-specific systems of record. Each captures part of the collaborative process, but there is no general, interoperable representation of coordination across them. Coordination lacks a native data model in current systems. The result is familiar: reasoning is repeatedly reconstructed, relationships remain implicit, and shared understanding decays as soon as it crosses tool boundaries.

The rise of AI systems makes the absence of such a substrate easier to see.

Human interaction with language models routinely involves substantial articulation: task decomposition, clarification of constraints, iterative refinement of goals, explanation of tradeoffs, and alignment on next steps. Yet this articulated structure is often tool-specific and transient. It may be bound to a session, hidden in prompts, or dispersed across application state and memory rather than preserved as structured, durable shared context.

## **Terminology**

To help our discussion here, some clarification of terminology will be useful:

- **Collaboration** refers to the broader process through which humans and agents work together across time.
- **Coordination** refers to the structured organization of that collaboration: the relationships, rationale, roles, dependencies, and evolving context that make joint work coherent.
- **Articulation Work** refers to the ongoing work required to establish and maintain that coordination. This term comes from the disicpline of Computer-Supported Cooperative Work ([CSCW](https://en.wikipedia.org/wiki/Computer-supported_cooperative_work)) and is a key concept we cover in detail later.
- **Shared Cognition** refers to the higher-order condition that can only emerge when participants operate from a common, durable, and coherent shared context. When humans and agents collaborate and coordinate effectively, they are undergoing a state of shared cognition.

## **Thesis**

Without a shared structure, coordination cannot accumulate coherently over time; it must instead be repeatedly reconstructed from tools, documents, and conversation. As collaborative work becomes more distributed, more software-mediated, and more entangled with human-agent interaction, the pressure to represent coordination more directly continues to grow. The absence of a network-native coordination substrate is becoming increasingly hard to ignore.

The **Agents Shared Cognition Protocol ([ASCP](https://github.com/Reframe-Technologies/ascp-draft-specs))** is presented here as one open, layered approach to that problem: a way to represent articulated context as durable, private, and interoperable coordination history.

## **From Conversation to Content to Coordination**

The evolution of networked systems can be understood as a progression in explicit structure.

Early distributed systems such as Network News Transfer Protocol standardized large-scale conversation. Messages could be appended, replicated, and exchanged across peers. This was a major step for communication, but the structure of meaning remained mostly implicit. Threads provided only a limited scaffold. Relationships such as justification, dependency, supersession, or relevance still had to be reconstructed by human readers.

The web introduced a different kind of structure. Documents became addressable, linkable, and interpretable across systems. Hyperlinks made relationships between resources explicit enough to support global navigation, composition, and discovery. Open standards transformed content from an application-local artifact into a network-native one.

Coordination has not yet undergone the same transition. Modern systems can transmit messages, synchronize files, and track state, but they do not share a native representation of coordination. There is still no general, interoperable data model for how work is organized, why decisions were made, which items depend on which others, how roles and responsibilities are situated, and how context evolves across time.

All of this falls into the domain of Articulation Work.

## **Articulation Work**

The central CSCW insight behind this paper is that collaboration does not hold together on its own. It depends on the ongoing work of aligning interpretations, negotiating dependencies, assigning responsibility, and reconciling contributions into coherent outcomes. CSCW gave this work a name: **articulation work**.

Articulation work is often described as overhead, but this description is misleading. It suggests that coordination is secondary to the work itself, when in practice it is often the structure that makes the work intelligible, accountable, and durable. A project plan, a technical decision, an escalation path, a hiring loop, or a research agenda all depend on articulated relationships that give isolated actions and artifacts their meaning.

Current systems tend to preserve either **content** or **state**. They capture documents, messages, records, tickets, and status fields. What they do not preserve cleanly is the articulated structure that explains how these items relate, why they matter, what changed, and how shared understanding developed around them.

The absence of this layer becomes especially visible in human-agent collaboration. A significant portion of productive interaction with AI systems is not simple content generation but collaborative articulation: establishing scope, selecting evidence, connecting reasoning to decisions, and iteratively refining interpretations. Without a formal structure to these interactions, collaboration becomes repetitive and brittle. The problem is not merely limited or imperfect memory. It is the absence of a durable representation for articulation itself.

For this reason, articulation should be treated not as an accidental residue of collaboration, but as a **system primitive**. If coordination is the structured layer that makes collaboration coherent, then articulation work is the means by which that structure is produced and maintained. A substrate capable of supporting collaboration well must therefore preserve articulation directly rather than forcing participants to reconstruct it from messages, documents, and state after the fact.

## **Limits of Existing Systems**

Existing collaboration systems are useful precisely because they solve real problems well. Messaging systems support rapid interaction. Document systems support persistent content. Workflow systems support explicit tracking and execution. Systems of record maintain canonical operational state.

The difficulty is that each of these systems captures coordination only indirectly.

Messaging systems preserve conversation, but usually in chronological form. They are good at expression and weak at durable structure. Document systems preserve content, but typically embed coordination inside mutable prose. Workflow systems preserve status and transition, but often abstract away the reasoning and negotiation that produced those states. Systems of record preserve outcomes, but rarely the evolving structure of interpretation and decision around them.

In practice, coordination spans all of them at once. A decision may begin in chat, be explained in a document, produce tickets in a tracker, and eventually alter the state of a domain system. What remains missing is a shared substrate that can represent the coordination itself as first-class structure rather than leaving it fragmented across adjacent tools and trapped in people's heads.

This is not an argument for replacing current systems. It is an argument that they are solving neighboring problems. Communication, content, and state remain essential. But none of them, alone or in combination, provides a general-purpose coordination layer that remains grounded in human coordination practice while also being explicit enough to be structured, composed, and interpreted across machines and systems.

## **Requirements for a Coordination Substrate**

If coordination is to become a first-class network concern, the substrate beneath it must preserve articulation work in a form that remains grounded in human coordination practice while also being explicit enough for machine interpretation, composition, and exchange.

| Requirement | Description |
| --- | --- |
| **Append-only representation** | Coordination unfolds over time through addition, revision, exception, and reinterpretation. Preserving that evolution matters. A coordination substrate should extend prior context rather than overwrite it, allowing past states, disagreements, and refinements to remain inspectable. |
| **Addressable units of articulation** | Coordination cannot remain trapped inside monolithic documents or opaque message streams. The substrate must support discrete, referenceable units that can be connected, reused, and interpreted across contexts. |
| **Composability through explicit relationships** | Coordination is inherently relational: decisions support goals, tasks depend on prior work, artifacts are grouped into scopes, and interpretations are refined over time. A useful substrate must represent these relationships directly rather than leaving them implicit in adjacency or prose. |
| **Separation of representation and interpretation** | The substrate should preserve what was articulated without collapsing that record into a single application-specific meaning. Different systems, participants, and domains must be able to interpret shared structure without rewriting the underlying representation. |
| **Human legibility and machine interpretability** | The substrate must represent structures that are native to human coordination practice without leaving them trapped in unstructured prose or conversation. Participants need not interact directly with the formal grammar, but the underlying representation must still be explicit enough to support validation, composition, replay, and shared machine interpretation. |
| **Decentralization and interoperability** | Collaborative work crosses application boundaries, organizational boundaries, and trust boundaries. A coordination substrate should not require a single product surface or a single administrative center in order to be useful. |
| **Integrity, authorship, and verifiable history** | Coordination only becomes shareable if participants can attribute articulated contributions, validate that they have not been altered, and replay the relevant history that gave them meaning. |
| **End-to-end privacy and scoped visibility** | Coordination is often more sensitive than the content it references. Strategy, rationale, delegation, dissent, and responsibility cannot be assumed to belong in a public or globally replicated substrate. A viable coordination layer must allow durable shared structure without requiring universal disclosure. |

Taken together, these requirements describe a substrate that does more than transport messages or preserve application state. It must preserve articulation work as durable, structured coordination history: grounded in human collaborative practice, yet explicit enough to be validated, composed, replicated, and interpreted across systems.

The question, then, is what kind of protocol architecture can satisfy these requirements without collapsing coordination into public ledgers, proprietary application models, or overly rigid schemas. The following sections introduce the **Agents Shared Cognition Protocol ([ASCP](https://github.com/Reframe-Technologies/ascp-draft-specs))** as one such approach and clarify both what it is designed to provide and what it is not.

## **What ASCP Is and Is Not**

ASCP is a protocol suite for representing **articulated context** and the articulation work that produces it as durable, addressable, and structured coordination history. Its purpose is not to replace messages, documents, or systems of record, but to provide a shared coordination substrate through which they can be connected by explicit articulated structures.

More specifically, ASCP formalizes structures that arise from human coordination practice into protocol artifacts that can be validated, composed, replicated, and interpreted across systems, without requiring participants to interact directly with the underlying grammar. Several boundaries are especially important.

ASCP is **append-only**, but it is not a blockchain. It does not require global consensus, public visibility, or a single universal ledger. It is **distributed**, but that distribution is built around scoped channels, channel replication, and cryptographic visibility boundaries rather than world-readable state.

ASCP is **structured**, but it is not a universal ontology. It does not claim to define a single canonical process or schema for all work. Instead, it defines a small set of protocol-level primitives for representing articulation work and leaves higher-order semantics, domain models, and application views to higher layers and profiles.

ASCP is not a workflow engine, a project management system, semantic web schema, or or a collaboration product. It is better understood as an **open coordination substrate**: a protocol architecture on which many kinds of tools, interfaces, and domain-specific systems can interoperate without surrendering their own local models.

The next section turns from these boundaries to the core architectural elements through which ASCP does this: immutable units of articulation, explicit relationships, append-only history, and a layered protocol design.

## **The Core Design of ASCP**

At the center of ASCP is the **Artipoint**: an immutable, addressable unit of articulated meaning. The name is a contraction of *point of articulation*, and it is meant quite literally. An Artipoint is not a mutable record or a document fragment. It is a durable semantic contribution that can be referenced, connected, superseded, grouped, or interpreted in relation to other contributions.

One useful way to think about an Artipoint is as a kind of structured bookmark. A traditional bookmark says where something is. An Artipoint says why it matters in a line of work. It may point to a document, ticket, repository, decision, or external system of record, but it is not trying to replace those mutable systems. They continue to do the operational heavy lifting. The Artipoint provides the durable contextual glue: the articulated judgment that something is relevant, related, grouped, dependent, superseded, or worth carrying forward.

This is why immutability is a feature rather than a barrier. The underlying document, ticket, or database record may change over time, but the articulated coordination move remains worth preserving. Instead of rewriting prior context, ASCP records new Artipoints and new relationships that clarify, extend, or supersede what came before. The result is a history of how context evolved, not just a snapshot of where things happen to stand now.

The main elements can be summarized as follows:

| Element | What It Is | Why It Matters |
| --- | --- | --- |
| **Artipoint** | An immutable, addressable unit of articulated meaning that serves as a kind of structured bookmark. | Preserves a durable coordination move without replacing the mutable systems it references. |
| **Articulation Statement** | A single authored coordination move that introduces or relates Artipoints through a small operator vocabulary. | Makes relationships explicit rather than leaving them embedded in prose or tool state. |
| **Stream** | A coherent line of work aimed at a specific outcome. | Provides a stable contextual frame for a specific effort or thread of activity. |
| **Pile** | A thematic grouping within a line of work. | Organizes related Artipoints without relying on hidden application structure. |
| **Space** | A broader accountability context that can contain streams and other structures. | Provides durable scope for coordination across a team, project, or domain. |
| **Channel** | The structure that scopes who receives articulated context. | Separates visibility from meaning, governance, and authorship. |

These elements are then realized through a layered protocol design. Articulated history is represented in a deterministic grammar, secured for distribution, replicated through append-only logs, and finally interpreted by applications and agents into views such as streams, spaces, governance state, and domain-specific structures. This separation helps shared structure remain stable even while higher-level interpretations evolve.

ASCP also treats humans and agents as first-class participants in the same substrate. Both can articulate, relate, and interpret context. This is not a cosmetic symmetry. It allows collaborative reasoning to accumulate in a shared form rather than being repeatedly translated between human workflow and machine session state.

## **Privacy, Authorship, and Trust**

A coordination substrate must be shareable without being universally visible. It becomes useful only if it can preserve durable structure without forcing collaboration into a public or centralized visibility model. For that reason, privacy is not an optional refinement in ASCP. It is part of the architecture.

ASCP separates three concerns that many systems blur together: **authorship**, **authorization**, and **visibility**. Authorship concerns who articulated something. Authorization concerns whether that articulation should have effect in a given context. Visibility concerns who can receive and interpret the relevant records. These are related, but they are not the same, and treating them as identical produces brittle systems.

In ASCP, articulated contributions are attributable and verifiable. They are carried through signed sequences and retained in append-only history. But visibility is scoped through channels and cryptographic distribution rather than assumed to be public. Participants may share a coordination structure without sharing it with everyone, and they may be included in a context structurally without having visibility into every part of it.

Trust is similarly grounded in replayable history rather than in constant dependence on live centralized services. ASCP's trust model is log-anchored: identities, bindings, key state, and related trust evidence can be evaluated from signed history and local policy. This makes it possible to preserve provenance and continuity over time while supporting decentralized and local-first operation.

Governance sits above these lower mechanisms. The protocol does not collapse policy into transport or encryption. Instead, it allows authority, participation, and administrative roles to be articulated and interpreted semantically, while lower layers enforce only the cryptographic and replication consequences assigned to them.

This combination matters. Durable coordination does not require a public ledger. Private collaboration does not have to give up interoperability. Authorship can remain auditable even when authority changes or visibility is tightly scoped.

## **What Open Coordination Makes Possible**

When coordination becomes durable, addressable, and shared, collaboration changes from reconstruction to accumulation. A different class of collaboration becomes possible.

Human-agent collaboration can become continuous rather than session-bound. Instead of repeatedly restating context, participants can extend shared articulated history. Decisions can remain connected to their rationale. Tasks can remain connected to the commitments, exceptions, and interpretations that shaped them. Systems of record can remain what they are while gaining explicit links to the coordination structures that explain their state.

A coordination substrate also enables interoperability without flattening everything into one application model. Messaging systems can remain messaging systems. Documents can remain documents. Operational systems can remain systems of record. Yet all of them can participate in a shared coordination layer that records how their artifacts relate, why they matter, and how understanding changes around them.

For multi-agent systems, the implications are equally important. Agents no longer need to rely solely on stateless prompting or hidden application memory. They can participate in a durable substrate of articulated context shared with humans and other agents, making coordination less dependent on repeated reconstruction and more dependent on explicit, replayable structure.

Under these conditions, what begins to emerge is not merely better metadata or more persistent chat history, but a more durable form of **shared understanding**. It remains bounded by privacy, governed by explicit relationships, and open to reinterpretation over time, yet it is no longer lost whenever work crosses a session, a tool, or a participant boundary.

## **Toward an Open Coordination Standard**

The broader importance of a protocol like ASCP is not that it proposes a new product category, but that it points toward an open standards direction for coordination itself.

The web succeeded in part because open standards made documents and links interoperable across many systems without prescribing a single application model. A coordination substrate should aspire to the same openness, allowing applications, domain profiles, interfaces, and deployment models to participate without being confined to one vendor or institutional boundary.

This is particularly important because coordination already spans heterogeneous environments. Teams work across tools, organizations use different trust models, and human-agent workflows do not belong to a single platform. If durable coordination remains proprietary, shared context will continue to fragment along application lines. An open protocol creates the possibility of a broader ecosystem of interoperable systems.

## **Shared Coordination as Infrastructure**

ASCP is one proposed response to the need for a network-native coordination substrate. It does not claim to answer every representational question or to be guaranteed a place as the coordination layer of the future. It shows how such a substrate can be approached without collapsing into public ledgers, proprietary silos, or application-specific state models.

Communication became infrastructure. Content became infrastructure. Coordination may now be next.
