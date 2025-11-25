# ASCP Governance and Access Control Specification

Version 0.1 - Novermber 2025

Jeff Szczepanski, Founder and CEO Reframe Technologies

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

## Terminology

Key terms defined in this document:

- **Structure** — An Artpoint of type Space, Stream, Pile, or Group.
- **Governance Attribute** — A typed attribute of an Artipoint expressing membership, authorship, stewardship, or roles.
- **Inherited Attribute** — An Artipoint attribute whose effective value is derived from a parent structure.
- **Virtual Group** — A dynamic reference to a participant set (e.g., @members).
- **Effective Governance Set** — The evaluated set of participants or roles after inheritance, overrides, denies, expirations, and virtual group resolution.

Definitions of Artipoints, Articulation Statements, Channels, and the structural DAG appear in the ASCP Master Specification.

## Governance Model Overview

Governance in ASCP is defined by three principles:

1. Declarative Policy, External Enforcement
2. Inheritance with Explicit Override
3. Composable Participant Sets

Applications MAY use governance attributes for authorization, routing, notifications, workload assignment, role resolution, or context rendering.

## 4. Governance Attributes

Attributes may appear for any Structure such as Space, Stream, Pile, or Group unless otherwise restricted.

### 4.1 Membership

```
member + <participant>  
member - <participant>
```

Indicates semantic inclusion in a Structure.

### 4.2 Writers

```
writer + <participant>  
writer - <participant>
```

Indicates who may articulate into the Structure.

### 4.3 Ownership

```
owner := <participant>
```

Defines the administrative steward of a Structure.

### 4.4 Inheritance Override

```
inherits := <structure-ref | "default">
```

Defines explicit inheritance source.

### 4.5 Deny Semantics

```
deny::<attribute> := <participant>
```

Denials override inherited and local positives.

### 4.6 Expiration Semantics

```
expiration::<attribute> := (<participant>, <timestamp>)
```

Expired grants MUST NOT be effective after the timestamp.

# Groups

Groups are modular participant sets expressed as Structures of type *group*.

## Characteristics

- MAY contain membership and role attributes
- MAY reference other Groups
- Do NOT participate in structural inheritance
- Act as reusable governance components
- Provide identity stability

## Composition

Groups MAY be constructed from individuals, other Groups, or Virtual Groups.

# Virtual Groups

Virtual Groups dynamically resolve based on evaluated state.

Normative virtual groups:

- `@members`
- `@owners`
- `@writers`
- `@role::<role>`

Resolution MUST occur after inheritance, denies, and expiration pruning.

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

Clients MUST evaluate governance in this order:

1. Gather Local Attributes
2. Apply Inheritance
3. Apply Explicit `inherits :=`
4. Resolve Groups (recursive, acyclic)
5. Resolve Virtual Groups
6. Apply Denials
7. Remove Expired Grants
8. Resolve RACI Roles
9. Produce Effective Governance Set

# Security Considerations

Governance requires verified identities. All governance Artipoints MUST be signature-verified prior to evaluation. Identity verification, trust chain validation, and key material handling are defined in:

- **ASCP Identity & Trust**
- **ASCP Channels: Secure Distribution Layer Specification**

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

