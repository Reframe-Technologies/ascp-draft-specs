# Agents Shared Cognition Protocol (ASCP)

**The missing Cortex Layer for Human-AI collaboration.**

This repository contains the evolving specifications for the **Agents Shared Cognition Protocol (ASCP)**—a foundational protocol for shared cognition between humans and intelligent agents. ASCP introduces the **Cortex Layer**: a persistent, secure, and structured substrate for collaborative context.

> Just as HTTP standardized the web, ASCP standardizes shared context—making intent, memory, and coordination first-class digital artifacts.

---

## 🚧 Repository Status

This is an early-stage, **collaborator-controlled** repository. It is not yet an open standard but is being shared for review, issue tracking, and early implementation planning.

We welcome feedback via GitHub Issues. Public contributions are currently restricted but will be enabled in a future phase as the protocol stabilizes and reference implementations emerge.

---

## 📜 Specification Suite

The protocol is modular, layered, and formally defined through a suite of interoperable documents located in the [`/specs`](./specs/) directory:

| **Layer** | **Document** | **Purpose** |
| --------- | ------------ | ----------- |
| **Overview** | [`specs/the-agents-shared-cognition-protocol.md`](./specs/the-agents-shared-cognition-protocol.md) | Top-level architecture, core concepts (Artipoints, Streams, Spaces), and layered protocol model |
| **Layer 2 – Articulation** | [`specs/ascp-artipoint-grammar.md`](./specs/ascp-artipoint-grammar.md) | Formal grammar for immutable coordination statements (Artipoints) |
| **Whitepaper / Motivation** | [`specs/building-the-missing-cortex-layer.md`](./specs/building-the-missing-cortex-layer.md) | Strategic motivation and framing for the Cortex Layer and shared cognition infrastructure |
| **Remaining layers** | _(Coming soon)_ | Channels, ALSP (log sync), Identity & Trust, Bootstrap & Discovery |

A full spec index with descriptions is available in [`specs/README.md`](./specs/README.md).

---

## 🧠 What is ASCP?

ASCP is a protocol that turns coordination context into a durable, addressable data layer that both humans and AI agents can read from and write to. It enables:

- ✅ Structured, immutable context (via Artipoints)
- ✅ Shared memory across tools, time, and participants
- ✅ Human and agent co-authorship
- ✅ End-to-end encrypted distribution
- ✅ Decentralized, audit-friendly collaboration

ASCP is the infrastructure behind the **Cortex Layer**—the missing coordination substrate that allows AI agents to become trustworthy collaborators instead of stateless tools.

---

## 🔧 What This Repo Is For

This repository serves as:

- 📖 A **canonical reference** for ASCP specifications
- 🗂 A **version-controlled changelog** as the protocol evolves
- 🛠 A **place to file issues and track design decisions**
- 🧪 The **future home for reference implementations** in Python and Rust

---

## 🗺 Planned Structure

This repo will eventually include:
/
├── specs/                    # Core specifications and whitepapers
│   ├── the-agents-shared-cognition-protocol.md
│   ├── ascp-artipoint-grammar.md
│   └── …
├── reference/                # Language-specific implementations
│   ├── python/
│   └── rust/
├── tests/                    # Conformance tests and fixtures
├── docs/                     # Visual aids, presentations, explainers
├── LICENSE.md
├── CONTRIBUTING.md
└── CHANGELOG.md

---

## 🌐 Key Concepts

- **Artipoints** – Immutable, addressable coordination statements (the cognitive atoms of ASCP)
- **Articulation Statements** – Bundled, signed expressions of collaborative intent
- **Streams / Spaces / Piles** – Human-aligned coordination structures (threads, workspaces, buckets)
- **Channels** – Cryptographically scoped knowledge-sharing domains
- **Flags & Roles** – Attention tracking, authorship, access, and accountability

---

## 🧱 Roadmap

We’re building toward:

- ✅ Complete grammar and top-level spec
- 🔜 Secure distribution layer (Channels)
- 🔜 Log synchronization protocol (ALSP)
- 🔜 Identity, trust, and bootstrap model
- 🔜 Reference implementations in Python and Rust
- 🔜 Conformance test suite and example fixtures

---

## 🛡 License and Governance

All specifications are © Reframe Technologies, Inc. and will be released under a permissive open-source license (TBD). ASCP is intended to evolve into an open, community-governed protocol.

---

## 🤝 Contact

If you're interested in contributing, collaborating, or implementing ASCP, please open an issue or reach out directly.

---

_This is the shared foundation for human-aligned agent collaboration. Let's build the missing Cortex._
