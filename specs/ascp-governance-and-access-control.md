# ASCP Governance and Access Control Specification

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.22 — Informational (Pre-RFC Working Draft)  
November 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **Status of This Document**

This document is part of the ASCP specification suite and defines the **governance and access control mechanisms** for ASCP collaborative structures, including membership attributes, inheritance semantics, RACI-style roles, and Channel governance. It is published at this time to gather community feedback on the governance model and evaluation semantics.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# Abstract

This document defines the governance and access control model for the Agents Shared Cognition Protocol (ASCP). It specifies how collaborative structures—Spaces, Streams, Piles, and Groups—encode membership, authorship privileges, administrative stewardship, delegation, and role semantics as immutable Artipoints. The governance substrate defines the semantics of authority and participation but delegates enforcement to ASCP-compliant applications and agents.

The model enables decentralized, append-only, auditable governance for hybrid human–AI teams. It supports modular participant sets, hierarchical inheritance, dynamic virtual groups, RACI-style coordination roles, and explicit deny/expiration semantics. This specification describes the normative evaluation rules for deriving effective permissions and roles within the ASCP structural DAG.

This document depends on and integrates with the cryptographic identity model defined in **ASCP Identity & Trust** and the distribution semantics of **ASCP Channels**.

# Introduction

ASCP governance expresses who participates, how authority flows, and how accountability is represented within the coordination graph. Governance is encoded as immutable, signed Artipoints and forms part of the shared substrate of articulated context.

This specification defines:

- Governance attributes
- Group semantics
- RACI-style coordination roles
- Inheritance rules
- Deny/expiration rules
- Virtual group resolution
- Evaluation semantics

This document does NOT define:

- Application UI behavior
- Runtime enforcement
- Transport-layer access enforcement
- Security primitives (delegated to ASCP Channels and Identity & Trust)

Governance in ASCP is declarative, append-only, and semantically interpretable by both humans and AI agents.

# **Governance Scope Across All ASCP Structures**

Governance in ASCP applies uniformly across all **Structures**, including Spaces, Streams, Piles, Groups, and Channels. Governance attributes such as member, writer, owner, role::\*, deny::\*, and expiration::\* are **Structure-agnostic** and MAY be attached to any Artipoint of a structural nature unless otherwise restricted.

These governance attributes define **semantic participation**, **authorship rights**, **stewardship**, **role assignments**, and **inheritance behavior** within the coordination graph. Their interpretation is handled entirely within Layer-2 and Layer-3 through DAG evaluation. Governance semantics do **not** enforce cryptographic access; they define **semantic authority** and **coordination meaning**.

Cryptographic enforcement, including access to encrypted payloads and distribution scopes, is the responsibility of Layer-1 Channels, which receive their effective participant sets and key provisioning instructions from the governance state evaluated in Layer-3.

Governance therefore provides the **semantic model**, while Channels provide the **cryptographic transport**.

# Terminology

Key terms defined or referenced in this document:

- **Structure** — An Artpoint of type Space, Stream, Pile, Channel or Group.
- **Governance Attribute** — A typed attribute of an Artipoint expressing membership, authorship, stewardship, or roles.
- **Channel** - The Layer-1 secure distribution mechanism of ASCP: a cryptographically scoped pathway through which Articulation Statements are signed, optionally encrypted, transmitted, and stored in an append-only log. A Channel establishes the visibility and privacy boundaries of shared context.
- **Inherited Attribute** — An Artipoint attribute whose effective value is derived from a parent structure.
- **Virtual Group** — A dynamic reference to a participant set (e.g., @members).
- **Effective Governance Set** — The evaluated set of participants or roles after inheritance, overrides, denies, expirations, and virtual group resolution.

Definitions of Artipoints, Articulation Statements and the structural DAG appear in the ASCP Master Specification.

# Governance Model

Governance in ASCP is defined by three principles:

1. Declarative Policy, External Enforcement
2. Inheritance with Explicit Override
3. Composable Participant Sets

Applications MAY use governance structures as the basis for authorization, routing, notifications, workload assignment, role resolution, or context rendering.

## Governance Attributes

All Governance attributes defined in this specification are **Coordination Contruct** agnostic. They **MAY** apply to Spaces, Streams, Piles, Channels, Groups and other future Artipoint types.

This enables a consistent governance model across ASCP while preserving the independence between:

- **Contextual Constructs** (Spaces, Streams, Piles)
- **Addressing Contructs** (Identities, Groups)
- **Distribution Contructs** (Channels)
- **Security Contructs** (Certificates, Tokens)

### Membership

```
member + <participant>  
member - <participant>
```

The \`member\` attribute expresses semantic inclusion of a participant within a Structure. It defines participation and coordination visibility at the semantic layer and is evaluated according to the inheritance and override rules described in this specification.

Membership does not itself grant cryptographic access to Channel content or any other transport-layer authorization. Cryptographic access is determined exclusively by the Layer-1 Channel key provisioning mechanism, which is configured based on governance evaluation performed by Layer-3 clients.

Governance defines \*who is considered part of a Structure\*. Channels define \*who receives decryptable payloads\*. These domains are intentionally orthogonal but connected through Keyframes and Layer-3 interpretation.

**Note**: Keyframes are NOT governance Artipoints but may be created as a result of governance evaluations provided to Layer-1.

### Writers

```
writer + <participant>  
writer - <participant>
```

Indicates who may articulate into the Structure.

### Ownership

```
owner := <participant>
```

Defines the administrative steward of a Structure.

### Inheritance Override

```
inherits := <structure-ref | "default">
```

Defines explicit inheritance source.

### Deny Semantics

```
deny::<attribute> := <participant>
```

Denials override inherited and local positives.

### Expiration Semantics

```
expiration::<attribute> := (<participant>, <timestamp>)
```

Expired grants MUST NOT be effective after the timestamp.

## Groups

Groups are modular participant sets expressed as a Structures where the Artipoint type is `Group`. Groups:

- MAY contain membership and role attributes
- MAY reference other Groups
- Do NOT participate in structural inheritance
- Act as reusable governance components
- Provide identity stability

### Composition

Groups MAY be constructed from individuals, other Groups, or Virtual Groups.

### Virtual Groups

Virtual Groups dynamically resolve based on evaluated state.

Normative virtual groups:

- `@members`
- `@owners`
- `@writers`
- `@role::<role>`

Resolution MUST occur after inheritance, denies, and expiration pruning.

## **Governance and Keyframe Semantics**

Keyframes are Layer-2 Artipoints that encode the **intended cryptographic state** of a Channel or other secure distribution context. Their payloads typically include:

- encryption algorithm declarations
- signing algorithm declarations
- kid version identifiers
- references to participants for whom key envelopes must be generated

Governance defines the **semantic reasons** for creating or modifying a Keyframe, including:

- membership additions or removals
- changes in responsibility or stewardship
- explicit rotation events driven by policy

However, Governance itself does **not** define the cryptographic mechanics of Keyframes. Layer-1 Channels carry out the actual:

- generation of symmetric keys
- creation of per-recipient JWE envelopes
- enforcement of cryptographic access control

Layer-3 implementations interpret Governance attributes, determine the effective participant set for a Channel, and supply this resolved set to Layer-1 for Keyframe execution.

This ensures strict separation between **semantic authority** (Governance) and **cryptographic enforcement** (Channels).

# RACI-Style Roles

In collaborative settings, clarity around responsibility and accountability is just as important as access and authorship. ASCP adopts a RACI-style model—borrowed from organizational workflows—to explicitly encode who is responsible, accountable, consulted, and informed for a given structure. These roles enable both human and agent participants to understand not just who can act, but who is expected to care, coordinate, or decide.

```
role::responsible := <participant(s)>  
role::accountable := <participant(s)>  
role::consulted := <participant(s)>  
role::informed := <participant(s)>  
role::approver := <participant(s)>  
role::auditor := <participant(s)>  
role::observer := <participant(s)>
```

Roles define participation and responsibility expectations.

Constraints:

- Multiple responsibles allowed
- Only one accountable recommended
- Roles MAY be inherited

# Inheritance Model

## Default Inheritance

Unless otherwise specified, governance attributes (writer, member, owner) follow a default inheritance model that traverses up the structural DAG—Streams inherit from their parent Space, Piles inherit from their parent Stream or Space. This inheritance may be explicitly overridden using `inherits :=`. While explicit articulation is always preferred, defaults provide robust fallback semantics in under-specified contexts.

## **Encapsulation and Hierarchy Rules Table**

| **Structure** | **May Contain**                                  | **May Be Contained By** | **Inherits from**                           |
| ------------- | ------------------------------------------------ | ----------------------- | ------------------------------------------- |
| **Space**     | Streams, other Spaces                            | Parent Spaces           | Parent Space (unless overridden)            |
| **Stream**    | Piles, document Artipoints                       | Spaces                  | Parent Space (unless overridden)            |
| **Pile**      | Document Artipoints                              | Streams or Spaces       | Parent Stream or Space (unless overridden)  |
| **Group**     | Users, other Groups by reference or construction | Any structure           | N/A (flat, but composable via construction) |



Structures follow this containment model:

- Spaces contain Spaces and Streams
- Streams contain Piles and document Artipoints
- Piles contain document Artipoints
- Groups are not in the structural hierarchy

## Rules

- Streams inherit from parent Space
- Piles inherit from parent Stream or Space
- Groups do NOT inherit
- Explicits override inherited values
- Denials override both local and inherited positives
- Expired grants MUST be removed

# Evaluation Algorithm (Normative)

Clients **MUST** evaluate governance in this order:

1. Gather Local Attributes
2. Apply Inheritance
3. Apply Explicit `inherits :=`
4. Resolve Groups (recursive, acyclic)
5. Resolve Virtual Groups
6. Apply Denials
7. Remove Expired Grants
8. Resolve RACI Roles
9. Produce Effective Governance Set

# **Channels as Governance-Controlled Distribution Structures**

Channels are Structures that specialize in **cryptographic distribution**. Like all Structures, they MAY carry all the normal governance attributes defined by this specification. These attributes define the **semantic** participants and stewards of a Channel.

However, Channels differ from other Structures in two ways:

## **Cryptographic Enforcement**

1. Channels enforce **payload confidentiality** and **distribution scope** using symmetric encryption and per-recipient key envelopes.
2. This enforcement is not derived from governance directly; instead, Layer-3 interprets governance attributes and provides the effective participant list to Layer-1 for key provisioning.

## **Keyframe Binding**

1. Channels use Keyframes to express desired cryptographic configuration as Layer-2 Artipoints.
2. Governance attributes influence Keyframe creation but do not dictate cryptographic behavior directly.

This design allows Channels to be governed like any other Structure while preserving their role as transport-layer security boundaries.

# Security Considerations

Governance requires verified identities. All governance Artipoints MUST be signature-verified prior to evaluation. Identity verification, trust chain validation, and key material handling are defined in:

- **ASCP Identity & Trust**
- **ASCP Channels: Secure Distribution Layer Specification**

**Important**: Governance evaluation **MUST** occur before any cryptographic state changes in Channels. Incorrect or malicious governance interpretation can result in unauthorized inclusion or exclusion at the semantic layer, but Layer-1’s cryptographic rules remain strict and independent.

# Interaction with Bootstrap

Bootstrap establishes the initial trust root and authoritative governance metadata. Clients MUST follow:

- **ASCP: Bootstrap Process and Channel Discovery**

Incorrect bootstrap results in incorrect governance state.

# Appendix A — Examples (Informative)

## A.1 Group Composition Example

```
[group-irt-guid, source, timestamp,
  ["group", "Incident Response Team", "urn:group:irt"] .
  (
    member + alice@org.com,
    member + bob@org.com,
    owner := carol@org.com,
    role::responsible := alice@org.com,
    role::accountable := carol@org.com
  )
] < {group-guid-ops-team}
```

## A.2 Inheritance Override Example

```
inherits := default  
writer + eng-team@org.com  
deny::writer := temp-contractor@org.com
```

## A.3 Virtual Group Use Case

```
role::consulted := @members
```

