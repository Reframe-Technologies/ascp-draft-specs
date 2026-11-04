# Agents Shared Cognition Protocol (ASCP)

ASCP is a draft protocol for coordination across humans, AI agents, and applications. It defines a common model for recording decisions, commitments, responsibilities, and their history across independent implementations.

## Problem Statement

AI agents can maintain persistent memory and project state, but coordination often remains tied to individual agents or applications. When work moves between them, participants must reconstruct context and determine which decisions, commitments, and responsibilities still apply.

This fragmentation creates several problems:

- Continuing work across agents and applications requires repeated context reconstruction
- Actual commitments and agreements are difficult to distinguish from suggestions, summaries, or inferences
- Participants lack a shared history of who decided or accepted what
- Authorship and authority are difficult to verify across implementations
- Relationships between tasks, decisions, and responsibilities remain tied to application-specific models

## Approach

ASCP represents coordination as **Artipoints**: immutable units of articulated meaning that participants can reference and relate over time. Authored contributions are preserved as signed records in append-only logs. The design combines:

- Durable coordination history with explicit authorship
- Cryptographic identity and access control
- Local-first replication with deterministic log convergence
- Structured relationships between coordination elements
- A common grammar and compatible semantic profiles for interpretation across implementations

## Repository Contents

This repository contains early-stage specifications and is actively seeking technical review and implementation feedback.

**Current Specifications:**

| Layer                  | Specification                                                                                  | Status |
| ---------------------- | ---------------------------------------------------------------------------------------------- | ------ |
| Overview               | [The Agents Shared Cognition Protocol](./specs/the-agents-shared-cognition-protocol.md)        | Draft  |
| Terminology & Stack    | [ASCP Terminology Primer](./specs/ascp-terminology-primer.md) and [ASCP Stack](./specs/ascp-stack.md) | Draft  |
| Layer 2 – Articulation | [ASCP Artipoint Grammar](./specs/ascp-artipoint-grammar-a-structure.md)                        | Draft  |
| Layer 1 – Channels     | [ASCP Channels: Secure Distribution Layer](./specs/ascp-channels-secure-distribution-layer.md) | Draft  |
| Layer 0 – Transport    | [ASCP LogSync Protocol (ALSP)](./specs/ascp-logsync-protocol-alsp.md)                          | Draft  |
| Governance             | [ASCP Governance and Access Control](./specs/ascp-governance-and-access-control.md)            | Draft  |
| Identity & Trust       | [ASCP Trust and Identity Architecture](./specs/ascp-trust-and-identity-architecture.md)        | Draft  |
| Bootstrap              | [ASCP Bootstrap Process and Channel Discovery](./specs/ascp-bootstrap-process-and-channel.md)  | Draft  |
| Profiles               | [ASCP Profile for Coordinated Work](./specs/ascp-profile-for-coordinated-work.md)              | Draft  |

See [`specs/README.md`](./specs/README.md) for the complete index and recommended reading order.

**Reference implementations** in Python and Rust are planned following specification stabilization.

## Architecture Overview

ASCP uses a four-layer model:

- **Layer 0 — Log and synchronization (ALSP):** Replicates and orders records in Channel logs
- **Layer 1 — Secure distribution (Channels):** Signs and optionally encrypts contributions using provisioned keys
- **Layer 2 — Articulation:** Defines the grammar for expressing Artipoints and their relationships
- **Layer 3 — Interpretation and governance:** Derives coordination views and evaluates meaning, authority, and trust

Channels scope the distribution of coordination history. Streams organize coherent threads of work, while Spaces provide broader organizational and accountability contexts.

The protocol is transport-agnostic and designed for both centralized and peer-to-peer deployment.

## Relationship to Existing Work

| Protocol/System                  | Focus                                  | Relationship to ASCP                                                                 |
| -------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------ |
| **MCP** (Model Context Protocol) | Tool invocation, resource access       | Can expose ASCP context and coordination operations through tools and resources |
| **ActivityPub**                  | Social networking, federated timelines | Addresses social activity exchange; ASCP focuses on shared coordination history and interpretation |
| **Matrix**                       | Real-time messaging, E2EE chat         | Addresses messaging and room state; ASCP focuses on authored coordination acts and their interpretation |
| **XMPP**                         | Instant messaging, presence            | Addresses messaging and presence; ASCP focuses on durable coordination across participants and applications |
| **CRDTs** (e.g., Automerge)      | Conflict-free data replication         | Address replicated data convergence; ASCP also defines coordination representation and semantics |
| **Git**                          | Version control for code               | Provides a useful analogy for preserved history; ASCP records coordination acts and relationships |

ASCP is intended to complement existing communication and agent protocols. An MCP gateway, for example, could let agents read coordination history and contribute new coordination acts through their existing tools.

## Theoretical Foundations

ASCP builds on research in:

- **Computer-Supported Cooperative Work (CSCW)**: Shared workspaces, common ground theory, coordination mechanisms
- **Distributed Systems**: Operational transformation, CRDTs, eventual consistency models
- **Discourse and Argumentation Theory**: Structured dialogue, speech acts, collaborative reasoning
- **Cryptographic Identity**: Self-sovereign identity, capability-based security

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

- Interpretation of concurrent or conflicting coordination acts
- Cryptographic identity and trust models
- Interoperability with existing agent protocols
- Performance characteristics and scalability limits

**Contact**: <ascp@reframetech.com>

## License and Governance

Specifications are provided under terms consistent with IETF Trust Legal Provisions (TLP). Code components are BSD-2 licensed; reference implementations will be Apache-2.0. Upon Internet-Draft submission, specification text will fall under IETF Trust governance. Reframe Technologies stewards this work and operates commercial services built on ASCP, but the protocol itself remains open and vendor-neutral.
