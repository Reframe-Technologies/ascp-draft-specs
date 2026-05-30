# ASCP Specification Index

This directory contains the formal specifications - current as early drafts for review and feedback - for the **Agents Shared Cognition Protocol (ASCP)**, the coordination layer enabling structured, persistent, and composable context between humans and AI agents.

> ASCP is the protocol implementation of the Cortex Layer — the missing substrate for shared cognition.

See the top-level [`README.md`](../README.md) file of this repo for more details.

## 🧭 Specification Documents

| **Document** | **Purpose** |
| ------------ | ----------- |
| [`the-agents-shared-cognition-protocol.md`](./the-agents-shared-cognition-protocol.md) | Master specification outlining the ASCP architecture, layered model, and coordination primitives |
| [`ascp-terminology-primer.md`](./ascp-terminology-primer.md) | Canonical terms, definitions, and shared language used across the specifications |
| [`ascp-stack.md`](./ascp-stack.md) | Layered stack overview and how the ASCP documents map to each layer |
| [`ascp-artipoint-grammar-a-structure.md`](./ascp-artipoint-grammar-a-structure.md) | Defines the grammar for Artipoint Expressions, Articulation Statements, and Articulation Sequences |
| [`ascp-channels-secure-distribution-layer.md`](./ascp-channels-secure-distribution-layer.md) | Secure distribution of Artipoints using encrypted channels and provisioned cryptographic state |
| [`ascp-logsync-protocol-alsp.md`](./ascp-logsync-protocol-alsp.md) | Transport and synchronization layer for distributed logs |
| [`ascp-governance-and-access-control.md`](./ascp-governance-and-access-control.md) | Governance attributes, groups, virtual groups, RACI-style roles, and the normative evaluation algorithm |
| [`ascp-trust-and-identity-architecture.md`](./ascp-trust-and-identity-architecture.md) | Cryptographic identity, key management, and trust anchoring |
| [`ascp-bootstrap-process-and-channel.md`](./ascp-bootstrap-process-and-channel.md) | Replica initialization and channel discovery |
| [`ascp-profile-for-coordinated-work.md`](./ascp-profile-for-coordinated-work.md) | Practical profile and guidance for coordinated work scenarios |

## 🛠 Usage

Implementers should follow the layered reading order:

1. **Master Spec** – Start with [`the-agents-shared-cognition-protocol.md`](./the-agents-shared-cognition-protocol.md)
2. **Terminology & Stack** – Read [`ascp-terminology-primer.md`](./ascp-terminology-primer.md) and [`ascp-stack.md`](./ascp-stack.md) to align on shared language and layering
3. **Grammar** – Understand [`ascp-artipoint-grammar-a-structure.md`](./ascp-artipoint-grammar-a-structure.md) to parse/create coordination statements
4. **Distribution** – Integrate secure channel envelopes and log synchronization via [`ascp-channels-secure-distribution-layer.md`](./ascp-channels-secure-distribution-layer.md) and [`ascp-logsync-protocol-alsp.md`](./ascp-logsync-protocol-alsp.md)
5. **Governance & Access Control** – Describes how governance and access control entities tie into Artipoint attributes in [`ascp-governance-and-access-control.md`](./ascp-governance-and-access-control.md)
6. **Identity & Trust** – Defines how identity and trust come together into your ASCP instance in [`ascp-trust-and-identity-architecture.md`](./ascp-trust-and-identity-architecture.md)
7. **Bootstrapping** - How everything comes together into coherent replicas including rooting trust, confirming identities, etc. in [`ascp-bootstrap-process-and-channel.md`](./ascp-bootstrap-process-and-channel.md)
8. **Profiles** – Apply [`ascp-profile-for-coordinated-work.md`](./ascp-profile-for-coordinated-work.md) for real-world coordination patterns

## 📐 Conformance

All normative requirements will be collected in a dedicated section of the master spec. Formal test cases and conformance tooling will be published separately.

For questions, feedback, or proposed edits, please use GitHub Issues in the parent repository.
