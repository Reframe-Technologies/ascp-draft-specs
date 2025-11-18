# Agents Shared Cognition Protocol (ASCP)

**A Git-like Protocol for Context instead of Code**

This repository contains the evolving specifications for the **Agents Shared Cognition Protocol (ASCP)**—a foundational protocol for shared cognition between humans and intelligent agents. ASCP introduces the [**Cortex Layer**](https://blog.reframetech.com/building-the-missing-cortex-layer): a persistent, secure, and structured substrate for collaborative context.

> Just as HTTP standardized the web, ASCP standardizes shared context—making intent, memory, and coordination first-class digital artifacts.

---

## 📚 Background

**You can think of ASCP as a Git-like Protocol for context instead for code and content.** The following blog posts should be very helpful additional context on the basis behind, and inspiration for, ASCP:

1. [**The Pathway to AGI isn’t Intelligence, it’s Shared Cognition**](https://blog.reframetech.com/the-pathway-to-agi-isnt-intelligence-its-shared-cognition) - *Discover why shared cognition, and the Cortex Layer, is the real pathway to AGI.* This post and manifesto of sorts by Reframe Founder & CEO argues that the pathway to AGI is not greater intelligence but "Shared Cognition"—persistent, structured context that enables continuous collaboration between humans and AI agents. He proposed the Cortex Layer as open infrastructure to solve AI's "amnesia problem," where current systems lack memory across interactions, preventing them from becoming true collaborative partners rather than isolated task executors. This post, in effect, defines the technical box that ASCP lives in.
2. [**Building the Missing Cortex Layer**](https://blog.reframetech.com/building-the-missing-cortex-layer?hs_preview=AlrhJCSr-302574469307) - *Closing the Gap Between Chat and Full Human-AI Collaboration.* This post examined existing agent infrastructure protocols (MCP, A2A, ACP, ANP) and identified their shared limitation: none provide persistent, human-centered context for collaborative reasoning across sessions and tools. It explains the Cortex Layer concept and previews the Agent Shared Cognition Protocol (ASCP) as complementary infrastructure designed to enable durable shared memory between humans and AI agents.
3. [**Introduction to the Agents Shared Cognition Protocol (ASCP)**](https://blog.reframetech.com/introduction-to-the-agents-shared-cognition-protocol-ascp?hs_preview=WjrikjTz-303278941376)**** *- A Git-like Protocol for Context instead of Code*. This post Introduced the Agents Shared Cognition Protocol (ASCP) architecture, including its core primitives (Artipoints, Channels, Spaces), immutable coordination model, and four-layer technical stack for enabling durable, addressable context in human-AI collaboration.

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

> Reframe will not trademark the protocol name; anyone may implement or deploy ASCP without restriction. Reframe’s trademarks and commercial offerings apply exclusively to higher-level experiences (e.g., the Cortex Layer™, Reframe Cloud™, Reframe OWE™), not to the protocol itself.

---

## 🔧 What This Repo Is For

This repository serves as:

- 📖 A **canonical reference** for ASCP specifications
- 🗂 A **version-controlled changelog** as the protocol evolves
- 🛠 A **place to file issues and track design decisions**
- 🧪 The **future home for reference implementations** in Python and Rust
- Clarifying the boundary between the **open, public-good protocol** (ASCP) and Reframe’s **proprietary experience layer** (Cortex Layer™, Reframe Cloud™, Reframe OWE™).

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

## 🛡 License and Governance

Copyright © 2025 Reframe Technologies, Inc.

The ASCP specifications in this repository are provided under terms consistent with the IETF Trust Legal Provisions (TLP). Code components are BSD-2 licensed, and reference implementations are Apache-2.0 licensed. This ensures ASCP remains a public-good protocol that anyone may implement or host.

Upon eventual submission of Internet-Drafts, the related specification text will fall under the governance of the IETF Trust. Reframe will continue to steward the broader **Cortex Layer™** experience, Reframe-branded products, and certification programs which remain separate from the protocol itself.

The authors intend to submit this work to the IETF for standardization.

---

## **🌍 ASCP as a Public Good & Reframe’s Stewardship Philosophy**

ASCP is intentionally designed as an **open, vendor-neutral, public-good protocol**, similar in spirit to foundational internet standards such as **TCP/IP**, **HTTP/HTML**, **DNS**, and **SMTP**. The goal is broad adoption across the industry—tools, vendors, cloud platforms, and AI ecosystems—without central ownership or proprietary control over the protocol itself.

### **What This Means in Practice**

- **ASCP is not trademarked**, and anyone may implement, host, extend, or integrate the protocol.
- All **specification text** in this repository is licensed under terms consistent with the *IETF Trust Legal Provisions (TLP)*, allowing open redistribution and derivative works (with attribution and non-falsification requirements).
- Upon submission of ASCP Internet-Drafts, the related specification text will fall under the *IETF Trust*, ensuring long-term neutrality.
- **Code components** (grammar, examples, schemas) are BSD-2 licensed, consistent with IETF norms and enabling free incorporation into independent implementations.
- **Reference implementations** are Apache-2.0 licensed, encouraging broad, permissive use with explicit patent grants.

### **What Does Not Transfer to the IETF**

While the ASCP **protocol** is a public good, a few things remain the long-term responsibility and intellectual property of Reframe:

- **Cortex Layer™ -** Reframe’s branded conceptual model and human-aligned experience built on top of ASCP.
- **Reframe’s product ecosystem -** including the Reframe OWE, AI Desktop, Reframe Cloud, developer tooling, UX patterns, and other first-party applications built using the protocol.
- **Reframe tradenames and trademarks -** such as “Reframe,” “Cortex Layer,” “Reframe Cloud,” and any future marks representing our products or certification programs.
- **Reframe-hosted services -** including secure ASCP cloud hosting, governance dashboards, auditing tools, enterprise compliance layers, and premium operational infrastructure.

These elements constitute Reframe’s commercial stack **above** the public protocol.

### **Reframe Monetization Philosophy (for reference)**

Reframe does **not** extract value by restricting the protocol. Instead:

- We contribute ASCP freely to the open ecosystem.
- We encourage independent hosting and cloud deployments by any vendor.
- We focus on delivering world-class **products, experiences, and infrastructure** that sit *above* the protocol.
- A healthy, diverse ASCP ecosystem benefits everyone—including Reframe.
- Reframe may offer **certification programs** (e.g., “Cortex Layer Compatible™”) that validate interoperability and trustworthiness without limiting participation.

This approach mirrors successful open-protocol strategies seen in the evolution of the Internet, Linux, Git, Kubernetes, and Postgres:

**Open standards at the bottom, shared value at the top.**

---

## 🤝 **Community & Contact**

Interested in contributing, collaborating, or implementing ASCP? Open an issue or reach out directly via:

**Email:** <ascp@reframetech.com>

**Website:** <https://reframetech.com>

We especially welcome contributions from those working on: agent protocols, local-first sync, cryptographic identity, operational semantics, distributed systems, and human-centered collaboration tooling.

