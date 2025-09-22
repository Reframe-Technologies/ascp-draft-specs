# ASCP Specification Index

This directory contains the formal specifications for the **Agents Shared Cognition Protocol (ASCP)**, the coordination layer enabling structured, persistent, and composable context between humans and AI agents.

> ASCP is the protocol implementation of the Cortex Layer — the missing substrate for shared cognition.

---

## 🧭 Specification Documents

| **Document** | **Purpose** |
| ------------ | ----------- |
| [`the-agents-shared-cognition-protocol.md`](./the-agents-shared-cognition-protocol.md) | Master specification outlining the ASCP architecture, layered model, and coordination primitives |
| [`ascp-artipoint-grammar.md`](./ascp-artipoint-grammar.md) | Defines the grammar for immutable, addressable Artipoints — the atomic units of shared cognition |
| [`ascp-channels.md`](./ascp-channels.md) | Secure distribution of Artipoints using encrypted channels |
| [`ascp-logsync.md`](./ascp-logsync.md) | Transport and synchronization layer for distributed logs |
| [`ascp-identity-and-trust.md`](./ascp-identity-and-trust.md) | Cryptographic identity, key management, and trust anchoring |
| [`ascp-bootstrap-and-discovery.md`](./ascp-bootstrap-and-discovery.md) | Replica initialization and channel discovery |

---

## 🛠 Usage

Implementers should follow the layered reading order:

1. **Master Spec** – Start with `the-agents-shared-cognition-protocol.md`
2. **Grammar** – Understand `ascp-artipoint-grammar.md` to parse/create coordination statements
3. **Distribution** – Integrate secure channels and synchronization via `ascp-channels.md` and `ascp-logsync.md`
4. **Identity** – Defines how identity and trust comes together into your ASCP instance
5. **Bootstrapping** - How everything comes together into a coherent operating protocol stack including rooting trust, confirming identities, etc.

---

## 📐 Conformance

All normative requirements will be collected in a dedicated section of the master spec. Formal test cases and conformance tooling will be published separately.

For questions, feedback, or proposed edits, please use GitHub Issues in the parent repository.
