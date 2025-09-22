# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Overview

This repository contains the evolving specifications for the **Agents Shared Cognition Protocol (ASCP)**—a foundational protocol for shared cognition between humans and intelligent agents. ASCP introduces the **Cortex Layer**: a persistent, secure, and structured substrate for collaborative context.

The repository is currently in **early-stage, collaborator-controlled** status. It serves as:
- 📖 A canonical reference for ASCP specifications  
- 🗂 A version-controlled changelog as the protocol evolves
- 🛠 A place to file issues and track design decisions
- 🧪 The future home for reference implementations in Python and Rust

## Architecture Overview

ASCP is built on a layered architecture with four distinct layers:

### Core Concepts
- **Artipoints**: Immutable, addressable coordination statements (the cognitive atoms of ASCP)
- **Articulation Statements**: Bundled, signed expressions of collaborative intent  
- **Streams/Spaces/Piles**: Human-aligned coordination structures (threads, workspaces, buckets)
- **Channels**: Cryptographically scoped knowledge-sharing domains
- **Flags & Roles**: Attention tracking, authorship, access, and accountability

### Layer Architecture
1. **Layer 3 - Overview**: Top-level architecture and core concepts
2. **Layer 2 - Articulation**: Formal grammar for immutable coordination statements (Artipoints)
3. **Layer 1 - Secure Distribution**: Cryptographic envelope and membership model for distributing Artipoints
4. **Layer 0 - Log and Transport**: Transport-agnostic append-only log synchronization across replicas

## Specification Documents Structure

The specifications are organized in the `specs/` directory with a deliberate reading order:

### Primary Specifications
- `the-agents-shared-cognition-protocol.md` - Master specification with architecture overview
- `ascp-artipoint-grammar-a-structure.md` - Formal grammar for Artipoints (cognitive atoms)
- `ascp-channels-secure-distribution-layer.md` - Secure distribution and membership
- `ascp-logsync-protocol-alsp.md` - Log synchronization across replicas
- `ascp-trust-and-identity-architecture.md` - Cryptographic identity and trust model
- `ascp-bootstrap-process-and-channel.md` - Replica initialization and discovery

### Reading Order for Implementers
1. Start with the master specification to understand the architecture
2. Study the Artipoint Grammar to understand the syntax/semantics  
3. Implement secure distribution via Channels
4. Build log replication with ALSP
5. Integrate identity and trust anchoring
6. Implement bootstrap and channel discovery

## Common Development Tasks

Since this is a specification repository (not a code implementation), development tasks focus on documentation and specification work:

### Working with Specifications
```bash
# View all specification files
ls specs/

# Check recent changes to specifications  
git --no-pager log --oneline -10

# View current repository status
git status

# Search for specific terms across specifications
grep -r "Artipoint" specs/

# Find references to specific concepts
grep -r "Cortex Layer" specs/
```

### Specification Validation
The specifications use formal grammar notation (ABNF/BNF) for the Artipoint Grammar. When editing:

- Maintain consistency with the established layered architecture
- Ensure cross-references between documents remain accurate
- Follow the established terminology (Artipoints, Articulation Statements, etc.)
- Preserve the immutability principles fundamental to ASCP design

## Key Design Principles

When working with these specifications, understand these fundamental principles:

### Immutability by Design
Artipoints are immutable cognitive atoms that capture **cognitive structure, not dynamic content**. They record permanent decisions about how work is organized via the "bookmark pattern" - referencing external resources through URIs rather than embedding dynamic content directly.

### Distributed Shared Memory  
ASCP provides persistent memory spanning tools, sessions, and participants. The protocol creates a shared cognitive substrate that both humans and AI agents can read from and write to.

### Articulation Work Focus
Based on CSCW (Computer-Supported Cooperative Work) theory, ASCP addresses "articulation work" - the often invisible coordination effort required to make distributed activities mesh effectively toward common goals.

### Four Articulation Patterns
1. **Instantiation**: Creates new cognitive atoms
2. **Connection**: Links existing Artipoints together  
3. **Construction**: Creates new atoms while connecting to existing context
4. **Annotation**: Adds metadata without changing structure

## Future Development

The repository is planned to evolve into:

```
/
├── specs/                    # Core specifications (current focus)
├── reference/                # Language-specific implementations  
│   ├── python/
│   └── rust/
├── tests/                    # Conformance tests and fixtures
└── docs/                     # Visual aids, presentations, explainers
```

Reference implementations in Python and Rust are planned but not yet present in the repository.

## Working with Issues

When filing issues or contributing feedback:

- Reference specific specification documents and sections
- Consider impacts across the layered architecture
- Distinguish between architectural concerns vs. implementation details
- Understand this is currently a collaborator-controlled repository (public contributions restricted until protocol stabilizes)

## Terminology Consistency

Key terms used throughout specifications:

- **ASCP**: Agents Shared Cognition Protocol
- **Artipoint**: Immutable, addressable coordination statement
- **Cortex Layer**: The persistent substrate for shared cognition
- **Articulation**: The act of making context explicit and shareable
- **DAG**: Directed Acyclic Graph of cognitive structure
- **ALSP**: ASCP LogSync Protocol
- **CSCW**: Computer-Supported Cooperative Work

Maintain consistency with this established terminology when working with the specifications.