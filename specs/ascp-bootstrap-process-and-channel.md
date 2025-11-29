# ASCP: **Bootstrap Process and Channel Discovery**

**Building a Local Replica and forming the Root of Trust.**

Version 0.2 - Novermber 2025

Jeff Szczepanski, Founder and CEO Reframe Technologies

## **Introduction**

This document specifies the ASCP Bootstrapping and Channel Discovery process—the foundational mechanism that establishes a root of trust for secure, distributed coordination. Each ASCP repository maps 1:1 to an organization, serving as both the logical and cryptographic trust boundary for all coordination artifacts.

Beyond establishing organizational trust, the bootstrap process also governs how users and agents are securely added to the repository's trust graph, enabling mutual authentication between any participants—whether in fully distributed peer-to-peer scenarios or traditional client-server deployments.

This specification defines the structures, conventions, and operational procedures for establishing and maintaining the root of trust that enables robust ASCP operation across distributed replicas.

## **Overview**

Bootstrapping brings together all layers of ASCP (Layers 0-3) into a coherent system, enabling new replicas to discover, authenticate, and maintain their view of an ASCP organization. The bootstrap process logically operates at **Layer 3** by leveraging the facilities of all lower ASCP layers to solve a fundamental chicken-and-egg problem: we cannot validate articulation statements until we have the trust graph, but we need validated statements to build that graph.

Bootstrapping coordinates across all layers:

- **Layer 0** (ALSP): Retrieves channel messages as append-only logs of signed, optionally encrypted articulation statements.
- **Layer 1**: Extracts and decrypts articulation information from the opaque payloads.
- **Layer 2**: Parses articulation statements into ASCP grammar structures.
- **Layer 3**: Constructs a pre-seeded DAG from bootstrap content for post-hoc validation.

The process *assumes* initial bootstrap content is valid to construct the foundational trust graph, then uses that graph to cryptographically validate all bootstrap statements. User and agent identities are articulated into the same immutable logs, creating a unified trust model where individual signing certificates chain back to the organizational root.

Once validation completes successfully, normal ASCP operation can proceed with a fully established root of trust that supports global coherence, decentralization, and long-term auditability.

## **Document Guide**

This document follows the same sequence as a real bootstrap event:

1. **Foundations** – Context, glossary, and key principles establish the bootstrap’s role in ASCP’s trust model.
2. **Requirements** – Normative MUST/SHOULD rules set the baseline for compliant implementations.
3. **Process** – Step‑by‑step lifecycle from first connection to full channel replication.
4. **Acquisition & Security** – How bootstrap content is securely obtained and validated.
5. **Structure & Content** – Definitions of bootstrap documents, manifests, and @References.
6. **Updates & Recovery** – Handling changes, conflicts, and divergence.
7. **Implementation Details** – Naming conventions, trust material, and channel reference examples.

This order ensures readers first grasp **why** and **what’s required**, then **how** to execute, followed by guidance for edge cases and concrete implementation specifics.

## **Glossary of Terms**

- **Articulation Statement**: The atomic unit of context and articulation. Immutable, signed, and optionally encrypted statements that can represent anything from bookmarks to permissions to policy declarations.
- **Artipoint**: A core component of ASCP grammar representing a point of articulation (like an enhanced bookmark). While some Articulation Statements contain only an Artipoint, they are distinct concepts—similar to how URLs are a type of URI but not equivalent.
- **Bootstrapping Channel**: The initial Channel used to create and configure the root of trust for an ASCP organization. This Channel operates unencrypted to enable initial key distribution.
- **Channel**: A named group communication pathway used for distribution and optional encryption of Artipoints. Each Channel has a universally unique identifier (UUID) for clear identification and a friendly name for user selection (e.g., @Org, @HiringTeam).
- **Channel Log**: The actual persisted history of articulation statements for a given channel. Normally maintained file-by-file in local file systems and replicated between peers via ALSP.
- **localState**: Implementation-specific mutable state not replicated across peers.
- **Repository**: The logical aggregation of all ASCP channel logs associated with a specific organization.
- **Replica**: A local node's slice of the overall organizational Repository, which may contain the entire ASCP repository if the node acts as a server serving multiple ASCP clients.

## **Key Elements of ASCP Design**

The bootstrap process implements several core ASCP design principles that enable secure, distributed coordination:

### **Log-Anchored Trust Model**

ASCP uses **log-time validation** where trust decisions are based on immutable, signed entries in channel logs rather than real-time certificate checks. This ensures signatures remain verifiable indefinitely as long as their trust chain exists in the log, preserving meaning "as it was understood at the time."

The bootstrap process establishes this foundational trust anchor, enabling all subsequent operations to operate through the same transparent, auditable system.

### **Layered Architecture**

ASCP maintains clear separation across four layers, with bootstrapping coordinating all layers:

- **Layer 0** (ALSP): Transport and storage of opaque articulations
- **Layer 1**: Security, privacy, and distribution scope
- **Layer 2**: Grammar parsing and semantic resolution
- **Layer 3**: User's current DAG view and coordination logic

### **Universal Participation Model**

Both humans and autonomous agents use identical cryptographic mechanisms for identity, signing, and coordination. The bootstrap process establishes trust relationships that work equivalently for all participant types.

### **Implementation Flexibility**

Channel-to-document mapping is implementation-defined, supporting various deployment strategies while maintaining a central bootstrap registry for auditable channel discovery.

> **For complete details** on ASCP's trust architecture, identity binding mechanisms, and cryptographic foundations, see the companion document **ASCP Trust and Identity Architecture**.

## **Bootstrap Trust Foundation**

The bootstrap process establishes ASCP's log-anchored trust model through a special **bootstrap channel log** that serves as the root of trust for the entire repository.

### **Internal Trust Root**

- The **first Artipoint** in the bootstrap log contains the organizational **rootCA public key**
- This Artipoint is **self-signed** by the rootCA private key
- All subsequent trust relationships trace back to this immutable foundation

### **External PKI Anchoring**

To enable portable, verifiable trust beyond the ASCP instance:

- The rootCA can be **endorsed** by external PKI certificates (enterprise CA, Let's Encrypt, etc.)
- Uses **detached signatures** over the rootCA public key with captured certificate chains
- Creates **two-way binding**: internal ASCP trust + external PKI verification

### **Identity Integration**

Once the organizational root is established, individual users and agents join the trust graph through:

- **Self-sovereign key generation** with third-party identity attestation
- **Cryptographic binding** via identity and certificate Artipoints
- **Chain of trust** ultimately anchored by the bootstrap rootCA

> **For comprehensive coverage** of identity verification flows, key recovery mechanisms, cryptographic binding procedures, and detailed Artipoint structures, refer to **ASCP Trust and Identity Architecture**.

This trust foundation enables the bootstrap process to securely distribute channel references, validate all subsequent articulations, and establish the coordination infrastructure for robust ASCP operation.

## **Normative Requirements Summary**

This section defines mandatory, recommended, and optional requirements for bootstrap handling, using RFC 2119 keywords:

| Requirement                                                                            | Level  | Notes                                                            |
| -------------------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------- |
| Bootstrap content MUST be obtained and validated before channel sync begins            | MUST   | Applies to all new replicas                                      |
| Bootstrap channel MUST be unencrypted to allow initial key distribution                | MUST   | Protects availability of initial trust graph                     |
| Bootstrap retrieval SHOULD occur via ALSP hello exchange unless explicitly out-of-band | SHOULD | Alternate mechanisms allowed for air-gapped or offline scenarios |
| All bootstrap content MUST be validated via organizational root certificate            | MUST   | Validates authenticity of trust anchor                           |
| Implementations SHOULD persist validated bootstrap content for offline re-joins        | SHOULD | Improves reconnect performance                                   |
| Bootstrap updates SHOULD propagate via normal ALSP sync                                | SHOULD | Simplifies update handling                                       |

## **Bootstrapping Process**

The following steps describe the full lifecycle for initializing an ASCP-compatible client or agent, beginning from first contact with an ASCP server / peer and culminating in full replication of all necessary coordination context.

### **1. Connect to the ASCP Repository**

- The client connects to the ASCP server or peer hosting the ASCP repository.
- The repository is identified by a globally unique name derived from the organization (e.g., via DNS name or certificate hash).

### **2. Replicate the Bootstrap Channel**

- The client fetches the bootstrap channel, the canonical root document for the organization and identifies the @References channel log which contains manifests type information and other critical stuff
- The Bootstrap Channel can't be encrypted as it is really just the root of trust as the foundational log. 

### **3. Validate Organizational Trust Anchor**

- The client loads and cryptographically validates the organizational signing certificate located in the @Bootstrap log.
- This forms the root of trust for all articulated structures in the system.
- Local the necessary references to the @References channel which is always encrypted.

*See* ***ALSP § ALSP Hello Message*** *and* ***§ JWS-Based Message Authentication*** *for the handshake, nonce binding, and replay prevention mechanisms that secure the unencrypted @Bootstrap channel during first contact. For organizational trust anchoring and optional PKI endorsements, see* ***Trust & Identity § Provenance of the ASCP RootCA**.*

### **4. Load Bootstrap Articulations**

- The client parses and verifies all signed opaque articulations in the @Bootstrap log.
- These include key material, delegation statements, and other top-level coordination declarations.
- The result is a trusted DAG (directed acyclic graph) of foundational semantic structure.

### **5. Replicate the References Channel**

- This document contains the authoritative @References channel log of articulation statements related identifying information. The most critical element of this is the articulated map for all channels by their UUID's.
- References are not put in the @Bootstrap for an extra level of security and safety as bootstrap can't be encruypted, but @References call and should be.

### **6. Load and Index Channel References**

- The client processes the @References log to discover all known channels.
- Each reference includes the log name, document location, and UUID of the channel declaration.
- The client (re)generates a fast local index from channel ID → {document, log name, embedded status}.

### **7. Resolve Needed Channels**

- Based on the user’s identity, roles, and preferences, the client determines which channels it is authorized to access or subscribe to.
- This may include:
  - Org-wide channels like @Org
  - Team-specific channels like @HiringTeam
  - Stream-level or space-scoped discussions

### **8. Replicate Needed Channels**

- For each referenced channel, the client fetches the appropriate Automerge document and synchronizes the specified channel log within it.
- These may reside in the same or separate documents depending on the placement strategy articulated in @References.

## **Bootstrap Acquisition Mechanisms**

*Normative connection setup, peer authentication, and channel access proofs are defined in* ***ALSP § ALSP Hello Message*** *and* ***§ Channel Access Proof**. Participant onboarding and first-time identity binding are specified in* ***Trust & Identity § Identity Bootstrap and Verification**. This bootstrap document assumes those procedures are correctly implemented when retrieving initial bootstrap content.*

To be completed:

- Define retrieval via ALSP hello flow.
- Define out-of-band retrieval methods (HTTPS, signed package import, QR-based provisioning, etc.).
- Specify trust-on-first-use vs. pre-provisioned certificate trust models.
- Define initial certificate pinning or DNSSEC validation options.

## **Security Considerations for Initial Acquisition**

To be completed:

- Define MITM resistance for unencrypted @Bootstrap channel.
- Outline certificate-based validation process.
- Specify replay prevention for bootstrap artifacts.
- Detail signature verification of bootstrap manifest.

> *See **ALSP § ALSP Hello Message** and **§ JWS-Based Message Authentication** for the handshake, nonce binding, and replay prevention mechanisms that secure the unencrypted @Bootstrap channel during first contact. For organizational trust anchoring and optional PKI endorsements, see **Trust & Identity § Provenance of the ASCP RootCA**.*

## **Bootstrap Content and Structure**

- Detailed format of the bootstrap document and channel manifests
- How the @References channel serves as the authoritative channel index
- Certificate directories for user authentication material
- Channel manifests containing CAK public keys, access permissions, and metadata
- Relationship between bootstrap content and the log-anchored trust model

> *Formal definitions for certificate, rootCA, and identity Artipoints are provided in **Trust & Identity § Security and Identity Artipoints**, which should be used when authoring or parsing bootstrap manifests containing trust material.*

### Need Context through Bootstrapping

Information to be built up from the bootstrapping process

- The **root of trust** for the organization
- The **routing ledger** for all channels in the system
- The **certificate authority root**, containing the org’s signing certificate and other identity anchors

```json
{
  // Root Bootstrapping Materials
  "BootstrapInfo": {
    "orgId": "string",
    "createdBy": "Identity",
    "createdAt": "Timestamp",
    "cert": "PublicKey | Cert"
  },
  
  // Root of Trust Articulation Logs
  "rootChannelLogs": {
    "@Bootstrap": "<uuid>",
    "@References": "<uuid>"
  },
  
  // User Defined Logs
  "channelLogs": Map<uuid, List<local-filenames>>",
  
  // Implementation-specific optimizations (not replicated)
  "localstate": {
    // Possible Implementation Optimization examples
    "identityIndex": "Map<EntityID, PublicKey | Cert>",  // users, agents, org cert
    "derivedIndex": "Map<ChannelID, ChannelRefInfo>",    // locally extracted from @References
    // Other mutable items used locally to manage access
  }
}
```

## **Bootstrap Evolution and Updates**

- How replicas learn about changes to available channels, rotated keys, or membership updates
- Whether bootstrap changes propagate through normal ALSP sync or require special handling
- How bootstrap updates interact with active sessions and cached authorization

Stub Additions:

- Link key rotation handling explicitly to **Channels Keyframe mechanism**.
- Clarify that updates to @Bootstrap and @References propagate via normal channel sync.
- Specify how replicas should treat stale bootstrap data.

> *Key rotation, re-authentication triggers, and revocation handling are covered in **ALSP § Active session handling during CAK rotation** and **Trust & Identity § Identity: Certificate Binding & Rotation**. Bootstrap update policies should reference those mechanisms to ensure continuity of trust.*

## **Conflict Resolution and Split-Brain Scenarios**

- What happens if two replicas have different views of what channels exist?
- How are conflicts resolved if replicas have divergent bootstrap information?
- What's the recovery process if a replica becomes isolated and generates messages with stale Lamport counters?
- Reconciliation procedures when replicas reconnect after extended isolation
- Authority resolution when bootstrap sources provide conflicting information

## **Operational Workflow**

- Standard sequence: bootstrap acquisition → channel discovery → authentication → sync
- How replicas determine which discovered channels they should actually sync
- Dynamic membership changes and access revocation notification
- Recovery procedures when bootstrap information becomes stale or corrupted
- Handling updates to bootstrap information.
- Clarify security measures during bootstrap.

Given that bootstrap enables channel discovery, authentication, and authorization, this specification is critical for implementation completeness and interoperability.

## Repository Naming

When we connect to the ALSP server, we have to indicate which repository we are replicating. Repository naming follows globally unique conventions derived from verifiable organizational identity, typically:

- The DNS domain used to access the ALSP server (e.g., org.example.com)
- A hash or fingerprint of the organization's bootstrap signing certificate

## **Bootstrap Trust Material**

The \`BootstrapInfo\` in the @Bootstrap log MUST include:

- A signed organizational certificate (X.509-style or compatible)
- A declaration of the organization’s root identity
- Any key material required to validate other signing identities (or delegation thereof)

This allows the bootstrap log to serve as both a cryptographic root of trust and a semantic root of structure for all user defined structures (ie: Channels)

## **@References Channel Log**

This is a dedicated channel log inside the bootstrap.doc that publishes **signed Artipoints** for each known channel.

Each entry defines:

```
[
  uuid,
  source,
  timestamp, [
    "channel-ref",
    "Hiring Team",
    "@HiringTeam"
  ] . (
        doc := "shard-8f9a.doc",
        log := "@HiringTeam",
        embedded := false,
        declaredIn := "UUID-xyz"
    )
]
```

This structure makes:

- Channel discovery deterministic
- Routing auditably transparent
- Reference records cryptographically verifiable

## **Channel Creation Flow**

1. A new Channel (e.g., @HiringTeam) is articulated into a parent Channel log (e.g., @Org). This act of articulation is a semantic declaration of the new Channel’s existence and purpose. 
2. A **channel-ref Artipoint** is signed and appended to the @References
3. All clients can now deterministically resolve and replicate the log if authorized.

## **Security Section Stub (New)**

To be completed:

- Define handling of compromised bootstrap material.
- Outline remediation and re-keying procedures.
- Define logging and audit expectations during bootstrap.

## Summary

Once complete, the client is fully initialized and able to:

- Author and sign new articulations
- Interpret and act on coordination structure
- Maintain a consistent and auditable local replica of all relevant ASCP data