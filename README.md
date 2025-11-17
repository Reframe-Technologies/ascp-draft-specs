# Agents Shared Cognition Protocol (ASCP)

**A Git like Protocol for Shared Cognition to Close the Gap Between Chatbots and Full Human-AI Collaboration**

This repository contains the evolving specifications for the **Agents Shared Cognition Protocol (ASCP)**—a foundational protocol for shared cognition between humans and intelligent agents. ASCP introduces the [**Cortex Layer**](https://blog.reframetech.com/building-the-missing-cortex-layer): a persistent, secure, and structured substrate for collaborative context.

> Just as HTTP standardized the web, ASCP standardizes shared context—making intent, memory, and coordination first-class digital artifacts.

---

## 📚 Background

The following blog posts should be very helpful additional context:

- [**The Pathway to AGI isn’t Intelligence, it’s Shared Cognition**](https://blog.reframetech.com/the-pathway-to-agi-isnt-intelligence-its-shared-cognition) - Discover why shared cognition, and the Cortex Layer, is the real pathway to AGI.
- [**Building the Missing Cortex Layer**](https://blog.reframetech.com/building-the-missing-cortex-layer) - Closing the Gap Between Chat and Full Human-AI Collaboration

---

## 🚧 Repository Status

This is an early-stage, **collaborator-controlled** repository. It is not yet an open standard but is being shared for review, issue tracking, and early implementation planning.

We’re building toward having drafts of:

- ✅ Complete grammar and top-level spec
- ✅  Secure distribution layer (Channels)
- ✅  Log synchronization protocol (ALSP)
- ✅ Identity, trust, bootstrap model
- 🔜 Bootstrap model - building a new org repository
- 🔜 Reference implementations in Python and Rust
- 🔜 Conformance test suite and example fixtures

We welcome feedback via GitHub Issues. Public contributions are currently fairly limited but will expanded soon as the protocol stabilizes and reference implementations emerge.

---

## 📜 Specification Suite

The protocol is modular, layered, and formally defined through a suite of interoperable documents located in the [`/specs`](./specs/) directory:

| **Layer**                  | **Document**                                                                                             | **Purpose**                                                                                                |
| -------------------------- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Overview**               | [`specs/the-agents-shared-cognition-protocol.md`](./specs/the-agents-shared-cognition-protocol.md)       | Top-level architecture, core concepts (Artipoints, Streams, Spaces), and layered protocol model            |
| **Layer 2 – Articulation** | [`specs/ascp-artipoint-grammar.md`](./specs/ascp-artipoint-grammar.md)                                   | Formal grammar for immutable coordination statements (Artipoints)                                          |
| **Layer 1 - Channels**     | [`specs/ascp-channels-secure-distribution-layer.md`](./specs/ascp-channels-secure-distribution-layer.md) | Named pathways for distribution and optional encryption of Artipoints with authorized participants.        |
| **Layer 0 - ALSP**         | [`specs/ascp-logsync-protocol-alsp.md`](./specs/ascp-logsync-protocol-alsp.md)                           | ASCP LogSync Protocol ensuring local-first, conflict-free syncing of channel logs across all participants. |

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

## 🔑 Key Concepts

- **Artipoints** – Immutable, addressable coordination statements (the cognitive atoms of ASCP)
- **Articulation Statements** – Bundled, signed expressions of collaborative intent
- **Streams / Spaces / Piles** – Human-aligned coordination structures (threads, workspaces, buckets)
- **Channels** – Cryptographically scoped knowledge-sharing domains
- **Flags & Roles** – Attention tracking, authorship, access, and accountability

---

## **🌐 Intended Standards Path**

ASCP is designed from inception to be an **open, vendor-neutral, internet-scale protocol**, following the same evolutionary path as:

- HTTP
- TLS
- WebRTC
- JOSE / JWT
- QUIC

The roadmap:

1. **Early Drafts (this repo)**
2. **Internet-Draft (I-D) submissions**
3. **Working Group Formation (BoF → WG)**
4. **Standard-Track RFCs**

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

```
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
```

---

## 🤝 **Community & Contact**

Interested in contributing, collaborating, or implementing ASCP? Open an issue or reach out directly via:

**Email:** <ascp@reframetech.com>

**Website:** <https://reframetech.com>

We especially welcome contributions from those working on: agent protocols, local-first sync, cryptographic identity, operational semantics, distributed systems, and human-centered collaboration tooling.

---

## 🛡 License and Governance

Copyright © 2025 Reframe Technologies, Inc.

This document adn the associated specification documents are provided under terms consistent with the IETF Trust’s Legal Provisions (TLP). You may copy, distribute, display, and make derivative works of this document. Derivative works may not be presented  
as the original specification.

The authors intend to submit this work to the IETF for standardization.

