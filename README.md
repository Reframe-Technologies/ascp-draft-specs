# Agents Shared Cognition Protocol (ASCP)

**A protocol for persistent, structured context in human-AI collaboration**

## Problem Statement

Current AI agent systems lack persistent, structured memory across interactions. Each conversation starts fresh, tools operate in isolation, and collaborative context disappears when sessions end. Existing agent protocols (MCP, A2A) focus on tool invocation and message passing but provide no substrate for durable, addressable shared context that both humans and agents can reference over time.

This creates fundamental limitations:

- Agents cannot build on prior work or maintain continuity across sessions
- Humans must repeatedly re-establish context and intent
- Multi-agent coordination lacks shared ground truth
- Audit trails and accountability are ephemeral
- Knowledge cannot be versioned, referenced, or composed

ASCP addresses this by treating coordination context as a first-class, immutable data layer—analogous to how Git treats code history, but designed for collaborative intent, decisions, and shared understanding.

## Approach

ASCP introduces **Artipoints**: immutable, cryptographically signed statements of collaborative intent that form an append-only log. These are organized into **Channels** (secure distribution contexts), **Streams** (workstream-based contexts), and **Spaces** (project workspaces). The protocol provides:

- Immutable, content-addressed coordination primitives
- Cryptographic identity and access control
- Local-first synchronization with conflict-free merging
- Human and agent co-authorship with attribution
- Structured references between context elements

The design draws on operational transformation theory, CRDTs for distributed systems, and CSCW research on shared workspaces, while borrowing architectural patterns from Git, IPFS, and Matrix.

## Repository Contents

This repository contains early-stage specifications and is actively seeking technical review and implementation feedback.

**Current Specifications (v0.1.0-draft):**

| Layer                  | Specification                                                                                  | Status |
| ---------------------- | ---------------------------------------------------------------------------------------------- | ------ |
| Overview               | [The Agents Shared Cognition Protocol](./specs/the-agents-shared-cognition-protocol.md)        | Draft  |
| Layer 2 – Articulation | [ASCP Artipoint Grammar](./specs/ascp-artipoint-grammar.md)                                    | Draft  |
| Layer 1 – Channels     | [ASCP Channels: Secure Distribution Layer](./specs/ascp-channels-secure-distribution-layer.md) | Draft  |
| Layer 0 – Transport    | [ASCP LogSync Protocol (ALSP)](./specs/ascp-logsync-protocol-alsp.md)                          | Draft  |

Additional specifications covering identity, governance, and bootstrapping are in active development. See [`specs/README.md`](./specs/README.md) for the complete index.

**Reference implementations** in Python and Rust are planned following specification stabilization.

## Architecture Overview

ASCP uses a four-layer model:

1. **Transport (ALSP)**: Local-first log synchronization ensuring eventual consistency across participants
2. **Distribution (Channels)**: Named, cryptographically scoped contexts with access control and optional encryption
3. **Articulation (Artipoints)**: Immutable coordination statements with formal grammar, signatures, and content addressing
4. **Coordination (Streams/Spaces)**: Higher-level structures for organizing collaborative work

Each Artipoint is a signed, timestamped statement (e.g., "task created," "decision recorded," "context linked") that references prior Artipoints, forming a directed acyclic graph of coordination history. Channels define who can read/write, Streams provide conversation-like ordering, and Spaces group related work.

The protocol is transport-agnostic and designed for both centralized and peer-to-peer deployment.

## Relationship to Existing Work

| Protocol/System                  | Focus                                  | Key Difference from ASCP                                                             |
| -------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------ |
| **MCP** (Model Context Protocol) | Tool invocation, resource access       | No persistent context layer; session-scoped                                          |
| **ActivityPub**                  | Social networking, federated timelines | Content-focused, not coordination-focused; lacks structured intent primitives        |
| **Matrix**                       | Real-time messaging, E2EE chat         | Message-oriented; no immutable coordination semantics or structured references       |
| **XMPP**                         | Instant messaging, presence            | Synchronous messaging; no durable context model                                      |
| **CRDTs** (e.g., Automerge)      | Conflict-free data replication         | General-purpose data structures; ASCP adds coordination semantics and access control |
| **Git**                          | Version control for code               | ASCP adapts Git's immutability model for collaborative context instead of files      |

ASCP is complementary to agent communication protocols (MCP, A2A) and could serve as their shared memory substrate. It differs from social protocols (ActivityPub) by focusing on structured coordination rather than content distribution, and from CRDTs by providing domain-specific semantics for human-AI collaboration.

## Theoretical Foundations

ASCP builds on research in:

- **Computer-Supported Cooperative Work (CSCW)**: Shared workspaces, common ground theory, coordination mechanisms
- **Distributed Systems**: Operational transformation, CRDTs, eventual consistency models
- **Discourse and Argumentation Theory**: Structured dialogue, speech acts, collaborative reasoning
- **Cryptographic Identity**: Self-sovereign identity, capability-based security

Key influences include Lamport's logical clocks, the IPFS content-addressing model, and Matrix's federated room model.

## Background Reading

The following blog posts provide additional context and motivation for the Agents Shared Cognition Protocol, exploring the foundational ideas and challenges that inspired its development:

1. [The Pathway to AGI isn't Intelligence, it's Context](https://blog.reframetech.com/the-pathway-to-agi-isnt-intelligence-its-shared-cognition) - An exploration of why persistent, structured context is critical for advancing human-AI collaboration beyond current narrow AI capabilities.
2. [Building the Missing Cortex Layer](https://blog.reframetech.com/building-the-missing-cortex-layer) - A deep dive into the architectural gaps in current AI systems and the need for a shared memory substrate that enables more nuanced, contextual interactions.
3. [Introduction to the Agents Shared Cognition Protocol](https://blog.reframetech.com/introduction-to-the-agents-shared-cognition-protocol-ascp) - A comprehensive overview of ASCP's design principles, technical approach, and vision for transforming human-AI collaborative systems.

## Standardization Intent

We intend to pursue standardization through the IETF once the protocol demonstrates viability through independent implementations and real-world deployment. This repository serves as the working space for specification development prior to formal Internet-Draft submission.

## Contributing and Feedback

We are actively seeking input from:

- Researchers in distributed systems, CSCW, and human-AI interaction
- Protocol implementers and standards developers
- Developers building agent frameworks or collaboration tools

Please open GitHub issues for technical questions, specification feedback, or implementation concerns. We particularly welcome discussion on:

- Conflict resolution strategies for concurrent edits
- Cryptographic identity and trust models
- Interoperability with existing agent protocols
- Performance characteristics and scalability limits

**Contact**: <ascp@reframetech.com>

## License and Governance

Specifications are provided under terms consistent with IETF Trust Legal Provisions (TLP). Code components are BSD-2 licensed; reference implementations will be Apache-2.0. Upon Internet-Draft submission, specification text will fall under IETF Trust governance. Reframe Technologies stewards this work and operates commercial services built on ASCP, but the protocol itself remains open and vendor-neutral.
