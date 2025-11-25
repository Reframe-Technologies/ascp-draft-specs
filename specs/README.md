# ASCP Specification Index

This directory contains the formal specifications - current as early drafts for review and feedback - for the **Agents Shared Cognition Protocol (ASCP)**, the coordination layer enabling structured, persistent, and composable context between humans and AI agents.

> ASCP is the protocol implementation of the Cortex Layer — the missing substrate for shared cognition.

See the top-level [`README.md`](../README.md) file of this repo for more details.

## 🧭 Specification Documents

| **Document** | **Purpose** |
| ------------ | ----------- |
| [`the-agents-shared-cognition-protocol.md`](./the-agents-shared-cognition-protocol.md) | Master specification outlining the ASCP architecture, layered model, and coordination primitives |
| [`ascp-artipoint-grammar.md`](./ascp-artipoint-grammar-a-structure.md) | Defines the grammar for immutable, addressable Artipoints — the atomic units of shared cognition |
| [`ascp-channels.md`](./ascp-channels-secure-distribution-layer.md) | Secure distribution of Artipoints using encrypted channels |
| [`ascp-logsync.md`](./ascp-logsync-protocol-alsp.md) | Transport and synchronization layer for distributed logs |
| [`ascp-trust-and-identity.md`](./ascp-trust-and-identity-architecture.md) | Cryptographic identity, key management, and trust anchoring |
| [`ascp-governance-and-access-control.md`](./ascp-governance-and-access-control.md) | Governance attributes, groups, virtual groups, RACI-style roles, and the normative evaluation algorithm. |
| [`ascp-bootstrap-and-discovery.md`](./ascp-bootstrap-process-and-channel.md) | Replica initialization and channel discovery |

## 🛠 Usage

Implementers should follow the layered reading order:

1. **Master Spec** – Start with `the-agents-shared-cognition-protocol.md`
2. **Grammar** – Understand `ascp-artipoint-grammar.md` to parse/create coordination statements
3. **Distribution** – Integrate secure channels and synchronization via `ascp-channels.md` and `ascp-logsync.md`
4. **Governance & Access Control** – Describes how governance and access control entities ties right into Artipoint attributes.
5. **Identity & Trust** – Defines how identity and trust comes together into your ASCP instance
6. **Bootstrapping** - How everything comes together into a coherent replicas including rooting trust, confirming identities, etc.

## 📐 Conformance

All normative requirements will be collected in a dedicated section of the master spec. Formal test cases and conformance tooling will be published separately.

For questions, feedback, or proposed edits, please use GitHub Issues in the parent repository.
