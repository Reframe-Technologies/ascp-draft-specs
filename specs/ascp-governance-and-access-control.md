# ASCP Governance and Access Control Specification

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.30 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the ASCP specification suite and defines the **governance and access control mechanisms** for ASCP collaborative structures, including membership attributes, inheritance semantics, RACI-style roles, and Channel governance. It is published at this time to gather community feedback on the governance model and evaluation semantics.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# 2. Abstract

This document defines the governance and access control model for the Agents Shared Cognition Protocol (ASCP). It specifies how collaborative structures—Spaces, Streams, Piles, and Groups—encode membership, authorship privileges, administrative stewardship, delegation, and role semantics as immutable Artipoints. The governance substrate defines the semantics of authority and participation but delegates enforcement to ASCP-compliant applications and agents.

The model enables decentralized, append-only, auditable governance for hybrid human–AI teams. It supports modular participant sets, hierarchical inheritance, dynamic virtual groups, RACI-style coordination roles, and explicit deny/expiration semantics. This specification describes the normative evaluation rules for deriving effective permissions and roles within the ASCP structural DAG.

This document depends on and integrates with the cryptographic identity model defined in **ASCP Identity & Trust** and the distribution semantics of **ASCP Channels**.

# 3. Introduction

Distributed collaborative systems require explicit mechanisms to express participation, authority, and accountability across autonomous replicas without centralized coordination. This specification defines the governance layer (Layer-2/3) of ASCP, which provides the semantic foundation for access control and role assignment. It operates above the cryptographic identity layer and provides policy input to the distribution layer, enabling applications to make consistent authorization decisions across the coordination graph.

## 3.1 Design Principles

The governance model is built on three core principles:

1. **Declarative Policy, External Enforcement** — Governance declares semantic authority; enforcement is performed by application logic and transport layers
2. **Inheritance with Explicit Override** — Attributes flow from parent to child structures unless explicitly overridden
3. **Composable Participant Sets** — Groups and virtual references enable flexible, reusable participant definitions

## 3.2 Scope of this Specification

This specification defines the governance attributes that express participation and authority within ASCP structures, including Group composition for reusable participant sets, RACI-style coordination roles for responsibility assignment, and inheritance rules that propagate governance from parent to child structures. It further specifies deny and expiration semantics for overriding and time-limiting grants, virtual group resolution for dynamic participant references, and the normative evaluation algorithm for computing the effective governance set from the structural DAG.

#### Syntax vs. Semantics Boundary

The ASCP Artipoint Grammar specification defines syntax and ABNF for expressing Artipoints, including the structural form of types, attributes, operators, and payloads. It does not declare or assign semantics to specific Artipoint types or attribute keys.

This specification provides the normative semantic definitions for governance-related Constructs (eg: `Space`, `Stream`, `Group`) and governance attributes (eg: `member`, `writer`, `owner`). These constructs and attributes are formally defined only within this document or other normative ASCP subspecifications.

Implementations MUST use the ASCP Artipoint Grammar for syntactic validity and MUST use this specification for semantic interpretation, evaluation rules, inheritance behavior, and effective governance computation.

## 3.3 Out of Scope

This specification does not define application-level UI behavior or runtime enforcement mechanisms, as these are implementation-specific concerns left to individual applications. Transport-layer access enforcement, including cryptographic access control and key distribution, is addressed in either Layer-0 **ASCP LogSync Protocol** or Layer-1 **ASCP Channels** specification. Security Constructs such as Certificates and Tokens are defined in the **ASCP Identity & Trust** and **ASCP Channels** specifications, which together establish the cryptographic foundation upon which this governance model operates.

# 4. Terminology

Key terms defined or referenced in this document beyond the definitions of Artipoints, Articulation Statements and other key terms which appear in the ASCP top-level specification:

## **4.1 Coordination Construct**

A **Coordination Construct** is an Artipoint whose type declares a first-class role within the ASCP coordination graph. Constructs define high-level coordination functions such as contextual scoping, participant addressing, cryptographic distribution, or security provisioning. Constructs MAY carry attributes that express governance, trust, attestation, or configuration semantics, but such semantics ARE NOT themselves Constructs.

## **4.2 Contextual Construct**

A **Contextual Construct** organizes Artipoints into meaningful scopes of work. Contextual Constructs—such as Spaces, Streams, and Piles—define the contextual boundaries within which articulated work is grouped, interpreted, and related. They specify *what the work is about*, independent of participant identity, distribution scope, or security properties.

## **4.3 Addressing Construct**

An **Addressing Construct** defines one or more addressable participants. Addressing Constructs—including Identities and Groups—resolve to sets of Identity Artipoints for participation, membership, or audience evaluation. Addressing Constructs specify *who is involved*, while related governance semantics (roles, delegations, constraints) are expressed via attributes, not as separate constructs.

## **4.4 Distribution Construct**

A **Distribution Construct** defines the cryptographic delivery boundaries for articulated context. Distribution Constructs, realized as Channels, determine how Artipoints are signed, encrypted, transported, and synchronized across replicas. They specify *how* articulated context flows, distinct from contextual meaning, participant identity, or security metadata stored as attributes.

## **4.5 Security Construct**

A **Security Construct** is an Artipoint whose type establishes cryptographic material or security-relevant configuration. Security Constructs include KeyFrames, certificate containers, and related Artipoint types that define keys, signing capabilities, verification parameters, or channel-level security state. They specify *what cryptographic material is available*, while trust relationships, attestations, or bindings are expressed as attributes of these Constructs.

## **4.6 Governance Attribute**

A **Governance Attribute** is a typed attribute attached to a Construct that expresses semantic participation, authority, or coordination roles within the ASCP coordination graph.

## **4.7 Inherited Attribute**

An **Inherited Attribute** is a governance attribute whose value is derived from an ancestor Construct in the structural DAG.

## **4.8 Effective Governance Set**

The **Effective Governance Set** is the computed result of evaluating all governance attributes for a given Construct after applying inheritance, overrides, denials, expirations, and virtual group expansion.

## **4.9 Virtual Group**

A **Virtual Group** is a symbolic participant reference (e.g., @members, @writers) that resolves dynamically to the current effective participant set of a specified role or attribute within a Construct.

# **5. Governance Model**

Governance in ASCP defines who participates in collaborative structures, what authority they hold, and how that authority flows through the coordination graph (DAG). It provides the semantic foundation for authorization, distribution, role resolution, and context rendering.

Governance is orchestrated through Artipoint-based Constructs and their governance-related attributes. These Constructs include:

- **Contextual Constructs** (Spaces, Streams, Piles)
- **Addressing Constructs** (Identities, Groups)
- **Distribution Constructs** (Channels)
- **Security Constructs** (Certificates, Tokens)

Governance attributes such as `member`, `writer`, `owner`, `role::*`, `deny::*`, and `expiration::*` are **Construct-agnostic** and may be attached to any Artipoint of a structural nature unless otherwise restricted. This enables a consistent governance model across all ASCP coordination structures.

Governance defines **semantic authority** and **coordination meaning**. It does not enforce cryptographic access. Governance attributes are interpreted through DAG evaluation at Layer-2 and Layer-3, producing a semantic model of participation and rights. Cryptographic enforcement—including access to encrypted payloads and distribution scopes—is the responsibility of Layer-1 Channels, which receive their effective participant sets and key provisioning instructions from the governance state evaluated at Layer-3.

The result of governance evaluation at Layer-3 is called the **effective governance set**: the resolved set of participants, roles, and permissions after applying inheritance, overrides, denies, expirations, and group resolution. 

# 6. Governance Attributes

All Governance attributes defined in this specification are **Coordination Construct** agnostic. They **MAY** apply to Spaces, Streams, Piles, Channels, Groups and other future Artipoint types.

## 6.2 Writer

```
writer + <participant>  
writer - <participant>
```

Indicates who may articulate into the Structure.

## 6.3 Owner

```
owner := <participant>
```

Defines the administrative steward of a Structure.

## 6.4 Inheritance Override

```
inherits := <structure-ref | "default">
```

Defines explicit inheritance source.

## 6.5 Deny Semantics

```
deny::<attribute> := <participant>
```

Denials override inherited and local positives.

## 6.6 Expiration Semantics

```
expiration::<attribute> := (<participant>, <timestamp>)
```

Expired grants MUST NOT be effective after the timestamp.

# 7. Addressing Attributes

## 7.1 Member

```
member + <participant>  
member - <participant>
```

The \`member\` attribute expresses semantic inclusion of a participant within a Structure. It defines participation and coordination visibility at the semantic layer and is evaluated according to the inheritance and override rules described in this specification.

Membership does not itself grant cryptographic access to Channel content or any other transport-layer authorization. Cryptographic access is determined exclusively by the Layer-1 Channel key provisioning mechanism, which is configured based on governance evaluation performed by Layer-3 clients.

Governance defines \*who is considered part of a Structure\*. Channels define \*who receives decryptable payloads\*. These domains are intentionally orthogonal but connected through Keyframes and Layer-3 interpretation.

**Note**: Keyframes are NOT governance Artipoints but may be created as a result of governance evaluations provided to Layer-1.

## 7.2 Flag

```
flag + <participant>  
flag - <participant>
```

Operations:

- flag + user\@domain — user is actively tracking this Artipoint
- flag - user\@domain — user is no longer tracking it

Scope:

- **Applies to any Artipoint**, including Streams, Piles, Spaces, and content-level statements
- Flags are visible to all Channel members and act as a social contract—indicating who is maintaining awareness
- Used to drive attention, notifications, inboxes, and working memory interfaces

# 8. **Contextual Contruct Artipoints**

## 8.1 Bookmark Artipoint

A **Bookmark** is a Layer-2 Artipoint of type `bookmark` that represents the most fundamental Contextual Construct in ASCP. It provides a stable, addressable reference to an external resource—such as a document, file, or web page—identified by a URI carried in the payload. A Bookmark conveys no structural semantics by itself; it serves as an immutable pointer that MAY be grouped, related, or organized via later Articulation Statements.

### **8.1.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["bookmark", "<bookmark title>",
    uri:"https://github.com/Reframe-Technologies/ascp-draft-specs"
  ]
]
```

### **8.1.2 Field Requirements (Normative)**

The Bookmark Artipoint **MUST** conform to the Artipoint Grammar defined in the ASCP Artipoint Grammar specification. The following requirements apply specifically to Bookmark instantiation expressions:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be the literal string "bookmark".
  - This value unambiguously identifies the Artipoint as a Bookmark Construct.
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be an empty string.
  - The label **SHOULD** provide a human-readable title for the referenced resource.
- **payload**
  - The instantiation expression **MUST** include a payload field.
  - The payload **MUST** be a quoted string or a typed block.
  - If a typed block is used, the payload type **MUST** be uri:.
  - The payload value **MUST** contain a URI identifying the referenced external resource.
  - The payload **MUST NOT** embed Artipoint grammar elements or nested articulation expressions.

### **8.1.3 Required Attributes**

A Bookmark Artipoint defines no required attributes beyond its instantiation fields. No additional governance, addressing, or semantic attributes are mandated.

### **8.1.4 Optional Attributes**

A Bookmark Artipoint **MAY** include any attribute permitted by the Artipoint Grammar, including but not limited to:

- Governance attributes (e.g., roles, membership, ownership)
- Addressing attributes
- Flagging attributes

Such attributes, when present, MUST conform to the normative semantics defined for attributes elsewhere in this specification suite.

## 8.2 Pile Artipoint

A **Pile** is a Layer-2 Artipoint of type pile that represents a **flat, thematic grouping** of Artipoints within a collaborative context. Piles are associative “collections,” used to gather related items without imposing ordering or hierarchy. Piles **MAY** be articulated into Streams or Spaces but **MUST NOT** contain other Piles (Piles of Piles are disallowed). A Pile is itself an Artipoint and therefore addressable, immutable, and composable.

### **8.2.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["pile", "<pile title>", payload ]
]
```

### **8.2.2 Field Requirements (Normative)**

A Pile Artipoint **MUST** conform to the Artipoint Grammar defined in the ASCP Artipoint Grammar specification. The following additional requirements apply to Pile instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be the literal string "pile".
  - This value identifies the Artipoint as a Pile Construct.
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be an empty string.
  - The label **SHOULD** provide a human-readable descriptive title.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MAY** be an empty quoted string or typed block.
  - If typed, the payload usage is... (TBD)

### **8.2.3 Required Attributes**

A Pile defines **no required attributes** beyond those mandated by the core Artipoint format.

### **8.2.4 Optional Attributes**

A Pile Artipoint **MAY** include any optional attribute permitted by the grammar, including:

- Governance attributes (e.g., member, owner, role::\*)
- Addressing attributes
- Flagging attributes (for working-memory semantics)

All optional attributes **MUST** follow the normative semantics defined elsewhere in this specification suite.

## **8.3 Stream Artipoint**

A **Stream Artipoint** is a Layer-2 Artipoint of type stream that represents a **context-switchable thread of work**. A Stream defines the scope of a self-contained coordination effort—such as a project initiative, task sequence, or thematic workstream. A Stream **MAY** contain Piles and arbitrary Artipoints but **MUST NOT** contain other Streams. Streams inherit default governance from their enclosing Space unless explicitly overridden through governance attributes.

### **8.3.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["stream", "<stream title>", payload ]
]
```

### **8.3.2 Field Requirements (Normative)**

A Stream Artipoint **MUST** conform to the Artipoint Grammar specification. The following requirements apply specifically to Stream instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be "stream".
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be empty.
  - The label **SHOULD** identify the Stream’s intended purpose or thematic focus.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MAY** be a quoted string or typed block.
  - If typed, the payload **SHOULD** use the uri: prefix referencing the Stream’s external representation.

### **8.3.3 Required Attributes**

None.

### **8.3.4 Optional Attributes**

A Stream Artipoint **MAY** declare any optional attribute, including:

- Governance attributes (e.g., owner, writer, role::\*)
- Addressing attributes
- Membership attributes
- Flagging attributes, indicating working-memory relevance

Optional attributes **MUST** follow the semantics defined in the Governance and Artipoint Grammar specifications.

## **8.4 Space Artipoint**

A **Space Artipoint** is a Layer-2 Artipoint of type `space` representing an **accountability container** that is used to organize and govern multiple Streams and optional nested Spaces. Spaces serve as top-level coordination scopes corresponding to organizational structures, programs, teams, or strategic work areas. Every Space implicitly defines a coordination Stream (commonly called **Stream Zero**) used for the administration and organization of its subordinate Streams.

### **8.4.1 Canonical Form**

```c
[uuid, author, timestamp,
  ["space", "<space title>",
    payload
  ]
]
```

### **8.4.2 Field Requirements (Normative)**

A Space Artipoint **MUST** conform to the Artipoint Grammar specification. The following requirements apply to Space instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be "space".
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be empty.
  - The label **SHOULD** meaningfully identify the Space to participants.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MAY** be a quoted string or a typed block.
  - If typed, the payload **SHOULD** use the uri: prefix pointing to the Space’s authoritative external representation.
  - The payload **MUST NOT** embed grammar-level expressions or other Artipoints.

### **8.4.3 Required Attributes**

None.

### **8.4.4 Optional Attributes**

A Space Artipoint **MAY** declare optional attributes including:

- Governance attributes (e.g., owner, member, role::\*, delegation attributes)
- Addressing attributes
- Membership attributes
- Inheritance modifiers (e.g., inherits := \<rule>)
- Flagging attributes

All attributes **MUST** adhere to the normative semantics defined in the Governance and Artipoint Grammar specifications.

# 9. **Addressing** Contruct Artipoints

Addressing Constructs define reusable participant sets for use with the other types of Collaboration Contructs. These constructs are expressed as Artipoints following the Layer-2 Artipoint Grammar.

## **9.1 Identity Artipoint (Informational)**

Declares a human, agent, or system identity. The normative reference for this Artipoint type can be found in **ASCP Trust and Indentity Architecture**.

## **9.1 Group Artipoint**

A **Group Artipoint** is a Layer-2 Artipoint of type group representing a **reusable, addressable participant set** used for governance and access-control semantics. A Group functions as a modular component that can be referenced by Spaces, Streams, Piles, Channels, or other Groups. Groups **MAY** contain explicit participants, referenced Groups, or Virtual Groups (e.g., @owners, @members), and **MAY** carry governance attributes such as role assignments. Groups **DO NOT** participate in structural inheritance and have no ordering or hierarchical semantics beyond explicit composition expressed via Articulation Statements.

A Group is immutable once instantiated; all evolution of its definition occurs through additional articulations referencing it.

### **9.1.1 Canonical Form**

```c
[uuid, author, timestamp,
  ["group", "<group name>",
   uri:"https://app.example.com/group/<group-id>"
  ]
]
```

### **9.1.2 Field Requirements (Normative)**

A Group Artipoint **MUST** conform to the Artipoint Grammar defined in the ASCP Artipoint Grammar specification. The following requirements apply specifically to Group instantiation:

- **type**
  - The instantiation expression **MUST** include a type field.
  - The value **MUST** be the literal string "group".
  - This value unambiguously identifies the Artipoint as a Group Construct.
- **label**
  - The instantiation expression **MUST** include a label field.
  - The label **MAY** be empty.
  - The label **SHOULD** contain a human-readable name for the Group.
- **payload**
  - A payload field **MUST** be present.
  - The payload **MAY** be a quoted string or typed block.
  - If typed, the payload **SHOULD** use the uri: prefix, referencing the Group’s authoritative external representation (e.g., a directory or application metadata).
  - The payload **MUST NOT** contain nested Artipoint grammar constructs or embedded articulation expressions.

### **9.1.3 Required Attributes**

A Group Artipoint defines **no required attributes** beyond those mandated by the core Artipoint structure.

### **9.1.4 Optional Attributes**

All optional attributes **MUST** adhere to the normative governance semantics defined elsewhere in the ASCP specification suite. A Group Artipoint **MAY** include optional attributes to express participant membership and governance semantics, including but not limited to:

#### **9.2.4.1 Membership attributes**

- e.g.,
- member + \<participant-uuid>
- member + \<group-uuid>

#### **9.2.4.2 Role attributes**

Roles in the management of the group itself.

- e.g., role::responsible + \<uuid>, role::owner + \<uuid>

**9.2.4.3 Governance extensions**

Delegation, responsibility, or other governance semantics permitted by the Governance specification. (TBD)

## **9.3 Virtual Groups (Normative)**

A **Virtual Group** is **not** an Artipoint and **MUST NOT** be represented as a Construct. Instead, a Virtual Group is a **symbolic identifier** that resolves dynamically to a participant set during governance evaluation. Virtual Groups provide a concise mechanism to reference derived participant sets—such as all members, all writers, or all participants holding a specific role—without requiring those sets to be explicitly modeled as Artipoints.

Virtual Groups behave *as if* they were Groups only for the purpose of resolution in the governance algorithm. They do not have:

- a UUID,
- authorship,
- persistence,
- immutability, or
- a place in the structural DAG.

### **9.3.1 Syntax**

Virtual Groups are identified using the @ prefix:

```c
@members
@owners
@writers
@role::<role-name>
```

### **9.3.2 Resolution Semantics**

Virtual Group resolution **MUST** occur during governance evaluation **after**:

1. Local attributes are gathered
2. Inheritance is applied
3. Explicit inherits := overrides are applied
4. Explicit Group references are fully expanded

Resolution of a Virtual Group **MUST**:

- Produce a set of Identity Artipoints (UUIDs),
- Reflect the *current* effective governance state,
- Incorporate denials and expiration semantics,
- Resolve recursively until no symbolic references remain.

Virtual Groups **MAY NOT** reference other Virtual Groups directly; resolution simply follows derived semantics.

### **9.3.3 Defined Virtual Groups**

The following Virtual Groups are normatively defined:

| **Virtual Group** | **Resolves To**                                                                                     |
| ----------------- | --------------------------------------------------------------------------------------------------- |
| @members          | The effective set of participants with member privileges on the Structure                           |
| @owners           | The effective set of participants holding the owner attribute                                       |
| @writers          | The effective set of participants permitted to articulate into the Structure (writer attribute)     |
| @role::\<role>    | The effective set of participants assigned the specified RACI-style role (e.g., @role::responsible) |

### **9.3.4 Behavior Relative to Real Groups**

Virtual Groups MUST follow these rules:

- **MUST NOT** be instantiated as Artipoints
- **MUST NOT** appear as payloads
- **MUST NOT** be treated as Governance Constructs
- **MUST** behave as a participant set when resolving member, writer, owner, or role::\* attributes
- **MUST** expand before applying denials and expiration semantics
- **MUST** not appear in evaluation output (i.e., the Effective Governance Set contains only Identity UUIDs)

### **9.3.5 Example (Informative)**

```c
role::consulted := @members
writer + @role::responsible
member - @writers
```

During evaluation, each Virtual Group expands into a concrete set of identity UUIDs according to current governance state.

# **10. Channels as Governance-Controlled Distribution Structures**

Channels are the ASCP **Distribution Construct** that define the cryptographic delivery boundaries for articulated context. Channels determine how Artipoints are signed, encrypted, transported, and synchronized across replicas. Governance interacts with Channels by determining *who* participates in a Channel and *when* a Channel’s cryptographic configuration must change. These governance decisions are reflected in **Keyframes**, which act as declarative, immutable markers of cryptographic epochs.

This section defines how governance influences Channel state, how Keyframes express that state, and how Layer-3 provisions the resulting cryptographic material to lower layers.

## **10.1 Governance Role in Channels**

Channels may carry any governance attributes defined by this specification, including `member`, `writer`, `owner`, `role::*`, and `deny` or `expiration` semantics. These attributes determine:

- who may articulate into a Channel,
- who is considered a participant of the Channel,
- who is responsible or accountable for the Channel’s stewardship,
- whether Channel membership changes require cryptographic rotation.

Governance defines *semantic authority* and *participation meaning*.

It does **not** define the mechanics of encryption, signing, or replication.

Changes in governance state—such as adding or removing participants, modifying responsibilities, or responding to a compromise event—MAY require a new cryptographic epoch for the Channel. These epochs are expressed through **Keyframes**.

## **10.2 Keyframes as Channel Cryptographic Epochs**

A **Keyframe** is a Layer-2 Artipoint of type keyframe that declares the intended cryptographic state of a Channel at a particular point in time. Keyframes:

- identify a new cryptographic epoch,
- reference the Channel they configure,
- carry recipient-specific wrapped keys (as Layer-2 attributes),
- provide immutable, auditable records of cryptographic state transitions.

Keyframes are **semantic declarations**, not executable instructions.

Their contents are interpreted only by Layer-3.

Layer-1 and Layer-0 treat Keyframes as opaque objects.

## **10.3 Layer-3 Responsibilities for Channel State (Normative)**

Layer-3 (Governance & Identity/Trust) is responsible for all semantic interpretation of governance and Keyframes. Layer-3 MUST:

1. **Evaluate governance attributes**  
   Determine Channel participants, writers, owners, and role assignments.
2. **Determine when a new cryptographic epoch is required**  
   Based on governance changes, rotation policy, or compromise conditions.
3. **Interpret Keyframes**  
   Resolve the active epoch, historical epochs, and the cryptographic state they represent.
4. **Resolve participant sets**  
   Expand Groups, apply Virtual Groups, apply inheritance, apply denies/expirations.
5. **Construct wrapped key envelopes**  
   For each eligible participant, generate or obtain the necessary cryptographic keys and wrap them using recipient public keys.
6. **Produce and attach Keyframe payload attributes**  
   Including JOSE kid identifiers, envelope metadata, and configuration values.
7. **Provision Layers-1 and -0 with the resulting cryptographic material**, including:
   - active and historical AES Channel keys,
   - Channel Access Key (CAK) for ALSP replication,
   - identity certificates and trust anchors,
   - JOSE kid → key mappings.
8. **Enforce key-usage invariants**, including:
   - new messages encrypted with the latest epoch key,
   - old keys retained for decryptability,
   - consistent interpretation across replicas.

Layer-3 MUST NOT rely on Layer-1 or Layer-0 to interpret Keyframes or apply governance semantics.

## **10.4 Channel Processing in Lower Layers (Informative)**

Because Layer-3 provisions fully evaluated cryptographic state:

- **Layer-1** performs message signing, encryption, decryption, and verification, selecting keys solely by JOSE kid.
- **Layer-0 (ALSP)** performs append-only replication and CAK-based authorization.
- Both layers treat Keyframes and all Layer-2 artifacts as opaque; they do not parse governance attributes or Keyframe semantics.
- Lower layers have no concept of membership, RACI roles, or governance inheritance.

This design ensures deterministic cryptographic behavior across all replicas without requiring lower layers to understand governance semantics.

## **10.5 Separation of Semantic Authority and Cryptographic Execution**

ASCP enforces a layered separation between governance, provisioning, and cryptographic execution:

| **Layer**   | **Role**                                                                                                                                            |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Layer-3** | Determines *who* participates, *why* the epoch changes, computes participant sets, constructs wrapped envelopes, provisions cryptographic material. |
| **Layer-2** | Provides declarative, immutable records of cryptographic configuration and state transitions.                                                       |
| **Layer-1** | Executes signing, encryption, and decryption using provisioned keys; has no semantic interpretation capabilities.                                   |
| **Layer-0** | Ensures ordered, authenticated, append-only replication; uses CAK for authorization but does not access Layer-2 content.                            |

This separation ensures:

- governance remains human- and policy-driven,
- cryptography remains deterministic and semantically agnostic,
- replicas remain interoperable across organizational or agent boundaries,
- governance evolution never requires modification of lower layers.

## **10.6 Examples of Governance-Driven Channel Changes (Informative)**

The following illustrate governance intent and the resulting Channel behavior:

- **Participant Added:** Governance adds a new member → Layer-3 creates a new Keyframe or updates envelopes accordingly.
- **Participant Removed:** Governance revokes access → Layer-3 produces a new epoch excluding that participant from key wrapping.
- **Scheduled Rotation:** A policy requires periodic rotation → Layer-3 generates new keys and provisions them through a new Keyframe.
- **Compromise Recovery:** A key compromise is detected → Layer-3 immediately issues a new epoch with fresh keys.

Only Layer-3 interprets these conditions; lower layers act solely on the provisioned cryptographic material.

# 11. RACI-Style Roles

RACI-style responsibility assignment matrices are widely used in collaborative and organizational workflows. ASCP adopts a minimal, protocol-friendly subset of this model to express semantic responsibility and participation expectations within a Structure. These roles do **not** affect cryptographic access, authorship rights, or distribution logic; rather, they serve as governance attributes that applications and agents MAY interpret to guide workflow behavior.

For background on the RACI model, see:

[*Responsible–Accountable–Consulted–Informed*](https://en.wikipedia.org/wiki/Responsibility_assignment_matrix) (Informative reference)

## **11.1 Semantics**

ASCP defines the following RACI-style role attributes:

```
role::responsible := <participant(s)>  
role::accountable := <participant(s)>  
role::consulted := <participant(s)>  
role::informed := <participant(s)>  
role::approver := <participant(s)>  
role::auditor := <participant(s)>  
role::observer := <participant(s)>
```

- RACI roles are **governance attributes** attached to a Structure.
- They participate in inheritance, denials, and expiration rules in the same manner as other governance attributes.
- Layer-3 evaluation MUST incorporate role assignments into the Effective Governance Set.
- Roles have **no direct effect** on authorship, membership, encryption, or Channel access.
- Roles MAY inform application-level behaviors such as notifications, workflows, approvals, or UI presentation; such behaviors are outside the scope of this specification.

## **11.2 Constraints**

- Multiple responsible participants MAY be assigned.
- Only one accountable participant is **RECOMMENDED**, though this specification does not enforce uniqueness.
- All roles MAY be inherited unless overridden or explicitly denied.
- Role attributes MUST contain only Identity or Group references (including Virtual Groups).

## **11.3 Notes (Informative)**

- Traditional RACI frameworks treat “Accountable” as the single entity ultimately answerable for an outcome. ASCP does not enforce uniqueness, but implementers SHOULD consider the implication of multiple accountables in application logic.
- consulted, informed, and observer roles are intended as semantic visibility indicators for higher-level systems and agents.
- These roles do not imply access-control privileges; access control is determined exclusively by member, writer, owner, and Channel Keyframe provisioning.

# 12. Inheritance Model

## 12.1 Default Inheritance

Unless otherwise specified, governance attributes (writer, member, owner) follow a default inheritance model that traverses up the structural DAG—Streams inherit from their parent Space, Piles inherit from their parent Stream or Space. This inheritance may be explicitly overridden using `inherits :=`. While explicit articulation is always preferred, defaults provide robust fallback semantics in under-specified contexts.

## **12.2 Encapsulation and Hierarchy Rules Table**

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

## 12.3 Rules

- Streams inherit from parent Space
- Piles inherit from parent Stream or Space
- Groups do NOT inherit
- Explicits override inherited values
- Denials override both local and inherited positives
- Expired grants MUST be removed

## 12.4 Evaluation Algorithm (Normative)

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

# 13. Security Considerations

Governance requires verified identities. All governance Artipoints MUST be signature-verified prior to evaluation. Identity verification, trust chain validation, and key material handling are defined in:

- **ASCP Identity & Trust**
- **ASCP Channels: Secure Distribution Layer Specification**

**Important**: Governance evaluation **MUST** occur before any cryptographic state changes in Channels. Incorrect or malicious governance interpretation can result in unauthorized inclusion or exclusion at the semantic layer, but Layer-1’s cryptographic rules remain strict and independent.

# 14. Interaction with Bootstrap (TBD)

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

