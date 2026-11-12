# Agents Shared Cognition Protocol (ASCP)

ASCP is a draft protocol for coordination across humans, AI agents, and applications. It defines a common model for recording decisions, commitments, responsibilities, and their history across independent implementations.

## Why Coordination Matters

People and AI agents can create documents, complete tasks, and share information. But working together also means setting goals, weighing options, making decisions, and accepting responsibilities. These acts of coordination help determine what the work means and what should happen next.

A suggestion is not the same as an agreed decision. A conversation about a task does not prove that someone accepted responsibility for it. Better memory may help participants recall what happened, but memory alone does not preserve these distinctions.

ASCP proposes a common way to record authored coordination so its meaning and history can persist across independent systems.

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

## ASCP and Organized Work Environments

An **Organized Work Environment (OWE)** is a proposed category of interactive computing environments designed around ongoing work, not just the applications and files used to carry it out. AI can help people notice changes, consider options, and clarify their intentions. People remain responsible for deciding what becomes an established decision or commitment. [Read the OWE Vision](https://reframetech.com/owe-vision).

ASCP is not tied to a particular OWE or to Reframe's own software. An agent framework, business application, or MCP gateway could use ASCP to take part in coordination without becoming an OWE. Different developers could also build their own OWE experiences. ASCP provides a proposed open foundation for sharing coordination across these systems.

## Relationship to Existing Work

ASCP is intended to complement existing communication and agent protocols. An MCP gateway, for example, could let agents read coordination history and contribute new coordination acts through their existing tools.

| Protocol/System                  | Focus                                  | Relationship to ASCP                                                                 |
| -------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------ |
| **MCP** (Model Context Protocol) | Tool invocation, resource access       | Can expose ASCP context and coordination operations through tools and resources |
| **ActivityPub**                  | Social networking, federated timelines | Addresses social activity exchange; ASCP focuses on shared coordination history and interpretation |
| **Matrix**                       | Real-time messaging, E2EE chat         | Addresses messaging and room state; ASCP focuses on authored coordination acts and their interpretation |
| **XMPP**                         | Instant messaging, presence            | Addresses messaging and presence; ASCP focuses on durable coordination across participants and applications |
| **CRDTs** (e.g., Automerge)      | Conflict-free data replication         | Address replicated data convergence; ASCP also defines coordination representation and semantics |
| **Git**                          | Version control for code               | Provides a useful analogy for preserved history; ASCP records coordination acts and relationships |


## Theoretical Foundations

ASCP builds on research in:

- **Computer-Supported Cooperative Work (CSCW)**: Shared workspaces, common ground theory, coordination mechanisms
- **Distributed Systems**: Operational transformation, CRDTs, eventual consistency models
- **Discourse and Argumentation Theory**: Structured dialogue, speech acts, collaborative reasoning
- **Cryptographic Identity**: Self-sovereign identity, capability-based security

## Background Reading

These articles explain the coordination problem, the OWE vision, and the ideas behind ASCP:

1. [Articulation Work and the Missing Coordination Layer](https://blog.reframetech.com/articulation-work-and-the-missing-coordination-layer) — Why ongoing work needs coordination that can be understood across systems.
2. [The Work Persists. The Agents Come and Go.](https://blog.reframetech.com/the-work-persists-the-agents-come-and-go) — Why goals, decisions, and commitments should outlast any one agent.
3. [The OWE Vision](https://reframetech.com/owe-vision) — How a computing environment could help people maintain the meaning and continuity of their work.
4. [AI Needs a Place to Work](https://blog.reframetech.com/ai-needs-a-place-to-work) — Three ways AI can take part in ongoing work.
5. [Introduction to the Agents Shared Cognition Protocol](https://blog.reframetech.com/introduction-to-the-agents-shared-cognition-protocol-ascp) — A longer introduction to ASCP's proposed design and purpose.

**Earlier thinking:**

- [The Pathway to AGI isn't Intelligence, it's Context](https://blog.reframetech.com/the-pathway-to-agi-isnt-intelligence-its-shared-cognition) — An early exploration of context and shared cognition in human–AI collaboration.
- [Building the Missing Cortex Layer](https://blog.reframetech.com/building-the-missing-cortex-layer) — An earlier framing of the infrastructure problem that led toward ASCP.

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

Specifications are provided under terms consistent with IETF Trust Legal Provisions (TLP). Code components are BSD-2 licensed; reference implementations will be Apache-2.0. Upon Internet-Draft submission, specification text will fall under IETF Trust governance. Reframe Technologies stewards this work, but the protocol itself remains open and vendor-neutral.
