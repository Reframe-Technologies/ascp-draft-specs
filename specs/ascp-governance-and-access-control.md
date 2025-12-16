# ASCP Governance and Access Control Specification

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.32 — Informational (Pre-RFC Working Draft)  
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

A **Security Construct** is an Artipoint whose type establishes cryptographic material or security-relevant configuration. Security Constructs include Keyframes, certificate containers, and related Artipoint types that define keys, signing capabilities, verification parameters, or channel-level security state. They specify *what cryptographic material is available*, while trust relationships, attestations, or bindings are expressed as attributes of these Constructs.

## **4.6 Governance Attribute**

A **Governance Attribute** is a typed attribute attached to a Construct that expresses semantic participation, authority, or coordination roles within the ASCP coordination graph.

## **4.7 Inherited Attribute**

An **Inherited Attribute** is a governance attribute whose value is derived from an ancestor Construct in the structural DAG.

## **4.8 Effective Governance Set**

The **Effective Governance Set** is the computed result of evaluating all governance attributes for a given Construct after applying inheritance, overrides, denials, expirations, and virtual group expansion.

## **4.9 Virtual Group**

A **Virtual Group** is a symbolic participant reference (e.g., @members, @writers) that resolves dynamically to the current effective participant set of a specified role or attribute within a Construct.

## **4.10 Vector of Execution**

A **Vector of Execution** is a bounded, directed context within which articulated work progresses toward a specific outcome or purpose. Vectors of Execution are primarily expressed through Stream Artipoints, which define self-contained execution contexts with clear scope, participants, and accountability. The term emphasizes both directionality (oriented toward an outcome) and boundedness (containing only artifacts and context relevant to that outcome).

# **5. Governance Model**

Governance in ASCP defines **authoritative participation, authorship permission, and administrative stewardship** within the shared coordination graph. It establishes *who is permitted to act*, *who owns governance authority*, and *how responsibility and accountability are expressed*, while deliberately separating these semantics from cryptographic enforcement and transport mechanics.

Governance is expressed exclusively through **immutable Artipoints and governance attributes**, evaluated deterministically across the structural DAG. These articulations form a durable, auditable record of authority and coordination intent shared by humans and agents alike.

Governance does **not** directly enforce access or execute control logic. Instead, it defines **semantic authority** that ASCP-compliant applications, agents, and lower protocol layers MUST respect when making authorization, workflow, and distribution decisions.

## **5.1 Governance Semantics (Conceptual Overview)**

Governance in ASCP is the **articulation of authority** ***via*** **articulation**.

It captures statements such as:

- *Who may author new context into a Structure*
- *Who administratively stewards that Structure*
- *Who is responsible for execution and accountable for outcomes*

These statements are first-class coordination artifacts, not configuration state or mutable policy tables.

Governance semantics are:

- **Declarative** — expressed as immutable statements of intent
- **Contextual** — always scoped to a specific Construct
- **Authoritative** — defining permission to act
- **Semantic** — interpreted by agents and applications
- **Deterministic** — evaluated identically across replicas

Governance is therefore neither advisory nor merely descriptive. Attributes such as writer and owner define **permission and authority** that MUST be honored by compliant systems.

## **5.2 Governance Constructs and Scope**

Governance semantics are always evaluated **relative to a Construct**. There is no global or ambient governance state in ASCP.

Governance applies across the following Construct classes:

- **Contextual Constructs** — Spaces, Streams, and Piles
  - Define *where* governance meaning applies
  - Establish accountability and execution boundaries
- **Addressing Constructs** — Identities and Groups
  - Define *who* governance applies to
  - Resolve to participant sets
- **Distribution Constructs** — Channels
  - Consume governance output to provision cryptographic state
  - Do not interpret governance semantics directly
- **Security Constructs** — Certificates, Keyframes
  - Provide cryptographic material
  - Do not carry governance meaning themselves

Governance attributes MAY be attached to any Construct of a structural nature unless otherwise restricted by specification.

## **5.3 Governance Attributes as Authoritative Permission**

Certain governance attributes explicitly define **permission and authority**:

- **writer** — defines who is permitted to articulate new Artipoints into a Structure
- **owner** — defines who holds administrative stewardship over a Structure and its governance state

These attributes:

- Express **authoritative permission**, not suggestions
- Participate in inheritance, override, denial, and expiration semantics
- MUST be respected by ASCP-compliant applications and agents **when determining semantic effect**, including when emitting new articulations, accepting or acting upon received articulations, and rendering views of participation and responsibility

Governance permissions are **semantic**, not cryptographic. An articulation authored by a non-writer may exist immutably in a log, but applications and agents MAY treat it as invalid, ignore it, or surface it as a governance violation.

### **Non-authoritative articulations are expected and safe**

ASCP permits authenticated participants to articulate statements even when those statements fall outside their current governance authority. Such articulations are immutably recorded with verifiable authorship and evaluated at interpretation time. Governance determines whether a given articulation is operative within a particular context; it does not determine whether the articulation exists. This model preserves auditability and historical truth—who said what and when—while ensuring that only authorized articulations influence derived state, views, or actions.

## **5.4 Governance as Input, Not Enforcement**

Governance defines **what is permitted and authoritative**, but does not perform enforcement itself.

Responsibility is intentionally separated across layers:

- **Governance (Layer-2 / Layer-3)**
  - Defines permission, authority, participation, and roles
  - Produces the **Effective Governance Set**
- **Applications and Agents**
  - MUST consult governance semantics before acting
  - Enforce authoring and stewardship rules in workflow behavior
- **Channels (Layer-1)**
  - Enforce cryptographic access and distribution
  - Rely on governance output for participant provisioning
- **Transport (Layer-0)**
  - Performs append-only replication without semantic interpretation

This separation ensures that governance meaning remains human- and agent-interpretable while cryptographic execution remains deterministic and semantically agnostic.

## **5.5 Effective Governance Set**

The **Effective Governance Set** is the result of evaluating all applicable governance attributes for a Construct after applying:

- inheritance,
- explicit overrides,
- group resolution,
- virtual group expansion,
- denials, and
- expiration semantics.

The Effective Governance Set represents:

- Who is considered a participant
- Who may author (writer)
- Who administratively stewards (owner)
- Which coordination roles apply (e.g., role::responsible, role::accountable)

The Effective Governance Set is **semantic output**, not an execution plan. It serves as authoritative input to applications, agents, and Channel provisioning logic.

## **5.6 Relationship to Contextual Constructs and Inheritance**

Governance meaning flows along **Contextual Construct containment boundaries**, establishing default expectations that may be refined at more specific levels.

In typical usage:

- **Spaces** establish the *enclosing accountability context* for a body of work
- **Streams** establish *execution contexts* and MAY define additional or delegated accountability and responsibility specific to that execution
- **Piles** inherit governance and MAY carry responsibility or accountability for the stewardship of their contents

No RACI-style role is restricted to a specific Construct type. Roles such as role::accountable and role::responsible MAY be declared on any Contextual Construct to express local authority, stewardship, or execution ownership, subject to inheritance and override rules.

Governance inheritance and role evaluation are defined in detail in **Section 12**.

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

# **8. Contextual Construct Artipoints**

Contextual Constructs are Layer-2 Artipoints whose **type** declares a first-class semantic role within the ASCP coordination graph. They define the **contextual boundaries** within which articulated work is grouped, interpreted, governed, and related. Unlike Addressing or Distribution Constructs, Contextual Constructs specify **what the work is about**, not *who* participates or *how* it is distributed.

Each Construct contributes a distinct cognitive and structural function:

- **Bookmark** — atomic cognitive references to external content
- **Pile** — associative clustering of related items
- **Stream** — a context-switchable **Vector of Execution**
- **Space** — an **accountability container** governing Streams and subordinate Spaces

All Contextual Constructs:

1. **MUST** be instantiated using the ASCP Artipoint Grammar.
2. **MUST NOT** contain nested Artipoint expressions in their payloads.
3. **MAY** carry governance and addressing attributes.
4. **Participate in inheritance** as defined in Section 12 unless otherwise restricted.
5. Provide **semantic context** that higher-layer agents and applications use to materialize views of work.

Payloads serve as **semantic anchors**—they provide external references, descriptive definitions, or authoritative representations of the Construct, but **never** encode structure or governance.

## **8.1 Bookmark Artipoint**

A **Bookmark** is a Layer-2 Artipoint of type bookmark representing the most atomic Contextual Construct. It records an immutable cognitive decision that a particular external resource is relevant to a given context at a specific moment.

A Bookmark:

- Is a **terminal node** in containment: it **MUST NOT** contain other Contextual Constructs.
- Inherits all governance semantics from its parent Pile, Stream, or Space.
- Serves as a durable semantic pointer, enabling annotation, flagging, and relational articulation.

### **8.1.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["bookmark", "<bookmark title>", uri:"https://example.com/resource"]
]
```

### **8.1.2 Field Requirements (Normative)**

*(unchanged but clarified)*

- **type** — MUST be "bookmark".
- **label** — SHOULD provide a human-readable title; MAY be empty.
- **payload** — MUST be a URI or typed block with uri:; MUST identify the referenced resource; MUST NOT embed structured grammar.

### **8.1.3 Required Attributes**

None.

### **8.1.4 Optional Attributes**

Bookmarks MAY carry governance, flagging, or annotation-related attributes.

### **8.1.5 Semantic Role (Informative)**

A Bookmark captures the *Reference Principle*: ASCP separates **cognitive structure** (the Bookmark Artipoint) from **dynamic content** (the external resource). Bookmarks create durable anchors for collaboration even as referenced content evolves.

## **8.2 Pile Artipoint**

A **Pile** is a Layer-2 Artipoint of type pile representing a **flat, thematic grouping** of Artipoints. It provides associative clustering without hierarchy, sequence, or recursion.

A Pile:

- **MAY** contain document-level Artipoints (e.g., Bookmarks).
- **MAY** be articulated into Streams or Spaces.
- **MUST NOT** contain other Piles.
- Inherits governance from its parent Stream or Space.

Piles represent the “workbench trays” of articulated work—useful for gathering information, early-stage exploration, and loose categorization.

### **8.2.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["pile", "<pile title>", payload ]
]
```

### **8.2.2 Field Requirements (Normative)**

- **type** — MUST be "pile".
- **label** — SHOULD be descriptive; MAY be empty.
- **payload** — MAY be empty; if present, MAY be string or typed block. Payload does not determine structure.

### **8.2.3 Required Attributes**

None.

### **8.2.4 Optional Attributes**

Piles MAY declare governance or flagging attributes.

### **8.2.5 Semantic Role (Informative)**

A Pile represents a **flat associative grouping** within the ASCP coordination graph.

Piles exist to support **non-hierarchical organization of related artifacts**—allowing humans and agents to cluster information, references, and intermediate work products without imposing execution flow, ordering, or deep structural commitments.

Key characteristics of a Pile include:

- **Associative Organization:** Items grouped within a Pile are related by theme, relevance, or affinity rather than sequence or dependency. A Pile expresses *“these things belong together”* without specifying *why* or *in what order*.
- **Structural Flatness:** Piles MUST NOT contain other Piles. This constraint prevents the formation of deep, implicit hierarchies within active work contexts and encourages clarity between *organization* (Piles), *execution* (Streams), and *accountability* (Spaces).
- **Low Commitment Context:** Piles are well-suited for early-stage exploration, reference gathering, categorization, or maintenance of collections whose structure may evolve over time.
- **Composable and Addressable:** As first-class Artipoints, Piles can be referenced, annotated, evolved, or promoted through later articulations without modifying their contents.

Piles do not constrain governance semantics. Any governance attributes—including authorship permissions or RACI-style roles—MAY be applied to Piles to express stewardship or responsibility for their contents, subject to the inheritance and override rules defined elsewhere in this specification. The role of a Pile is to define *associative grouping*, not *organizational hierarchy or execution flow*.

## **8.3 Stream Artipoint**

A **Stream** is a Layer-2 Artipoint of type stream representing a **context-switchable thread of work**—the atomic unit of *execution* in the ASCP model.

A Stream:

- Defines a self-contained **Vector of Execution** directed toward a goal or task sequence.
- **MAY** contain Piles and arbitrary Artipoints.
- **MUST NOT** contain other Streams.
- **MUST** inherit governance from its parent Space unless explicitly overridden.
- Serves as the natural boundary for assigning roles such as role::responsible.

Streams ensure that execution occurs within bounded cognitive spaces, enabling both humans and agents to operate with clear, relevant context.

### **8.3.1 Canonical Form**

```asciidoc
[uuid, author, timestamp,
  ["stream", "<stream title>", payload ]
]
```

### **8.3.2 Field Requirements (Normative)**

- **type** — MUST be "stream".
- **label** — SHOULD identify the intended purpose of the execution thread.
- **payload** — SHOULD reference the Stream’s external representation (e.g., task brief, issue link) via uri:.

### **8.3.3 Required Attributes**

None.

### **8.3.4 Optional Attributes**

Streams MAY include governance attributes such as owner, writer, member, and RACI roles.

### **8.3.5 Semantic Role (Informative)**

A Stream represents a **bounded execution context** within the ASCP coordination graph.

Streams exist to provide a **context-switchable focus of work**—a scope within which humans and agents can reason, act, and coordinate without inheriting unnecessary cognitive or informational overhead from broader contexts.

Key characteristics of a Stream include:

- **Contextual Isolation:** A Stream defines a boundary for relevance. Artipoints articulated within a Stream are presumed to relate to a common objective, inquiry, or line of execution.
- **Execution Focus:** Streams are optimized for *doing work*: planning, decision-making, iteration, and coordination toward a specific outcome. They are the primary unit of active collaboration.
- **Non-Recursive Structure:** Streams MUST NOT contain other Streams. This constraint prevents fragmentation of execution contexts and enforces a clear distinction between *execution* (Streams) and *organizational containment* (Spaces).
- **Agent and Human Entry Point:** For both humans and agents, entering a Stream signals an intentional shift of attention. The Stream defines the working set of context, history, and artifacts relevant to the task at hand.

Streams do not constrain governance semantics. Any governance attributes—including authorship permissions and RACI-style roles—MAY be applied to Streams as defined elsewhere in this specification. The role of a Stream is to define *where work happens*, not *who holds authority* over that work.

## **8.4 Space Artipoint**

A **Space** is a Layer-2 Artipoint of type space representing an **accountability container**. Spaces define the highest-level contextual boundaries—corresponding to teams, programs, departments, or strategic initiatives.

A Space:

- **MAY** contain Streams and subordinate Spaces.
- Implicitly defines an administrative coordination Stream (“Stream Zero”).
- Is the **root of governance inheritance** for all contained Streams and Piles.
- Establishes the scope within which accountability, ownership, and policy are articulated.

### **8.4.1 Canonical Form**

```c
[uuid, author, timestamp,
  ["space", "<space title>", payload ]
]
```

### **8.4.2 Field Requirements (Normative)**

- **type** — MUST be "space".
- **label** — SHOULD meaningfully identify the accountability context.
- **payload** — SHOULD reference the Space’s authoritative external representation via uri:; MUST NOT embed grammar.

### **8.4.3 Required Attributes**

None.

### **8.4.4 Optional Attributes**

Spaces MAY declare any governance or addressing attributes, including inheritance modifiers.

### **8.4.5 Semantic Role (Informative)**

A Space represents a **contextual accountability boundary** within the ASCP coordination graph.

Spaces exist to organize work at scales where **ownership, continuity, and long-lived responsibility** matter. They provide the enclosing context within which Streams, Piles, and subordinate Spaces are interpreted, governed, and related.

Key characteristics of a Space include:

#### **1. Accountability Containment**

A Space defines an enclosing context for accountability and stewardship. It establishes the scope within which outcomes are owned, policies apply, and governance meaning is inherited by contained Constructs. This does not preclude additional or delegated accountability being defined on contained Streams or Piles.

#### **2. Recursive Structure**

Spaces MAY contain other Spaces. This allows multiple layers of accountability to be modeled explicitly, reflecting real-world organizational and project structures.

For example, an outermost Space may represent organization-wide accountability (e.g., executive or corporate ownership), containing nested Spaces representing departments, programs, or initiatives, each with progressively more specific accountability contexts.

#### **3. Implicit Coordination Stream (“Stream Zero”)**

Every Space implicitly defines an internal coordination context, informally referred to as *Stream Zero*.

Any Artipoints articulated directly into a Space—rather than into a named Stream—are semantically treated as belonging to this implicit Stream. Stream Zero serves as the coordination and management context for:

- organizing contained Streams and Spaces,
- maintaining reference materials, bookmarks, or Piles relevant to the Space as a whole,
- capturing administrative or cross-cutting context not specific to a single execution Stream.

**4. Structural vs. Execution Distinction**

Spaces provide organizational containment and contextual continuity, while Streams provide focused execution contexts. The recursive nature of Spaces enables scale, while the non-recursive nature of Streams preserves clarity of execution.

#### Summary

Spaces do not constrain governance semantics. Any governance attributes—including authorship permissions and RACI-style roles—MAY be applied to Spaces to express ownership, stewardship, or participation expectations at that level, subject to inheritance and override rules defined elsewhere in this specification. The role of a Space is to define *contextual enclosure and continuity*, not to prescribe how authority must be allocated within it.

## **8.5 Construct Evolution Semantics (Informative)**

Contextual Constructs may evolve through later articulations. The following examples illustrate common evolution patterns, though many other paths are possible:

- A **Pile** may be "upgraded" into a more formal execution context by creating a **Stream** and relating it using operators such as `promotes` or `replaces`.
- A **Stream** may similarly be promoted into a **Space** as work scope expands to encompass broader accountability.
- Such evolution preserves historical context (e.g., the original Pile or Stream) while establishing a new semantic center for execution.
- Payloads evolve accordingly: e.g., a Pile payload may be empty, while a Stream payload often points to a project brief, and a Space payload may reference organizational charters or program documentation.

These evolutionary paths allow lightweight clustering to mature into structured execution contexts without losing provenance. The specific evolution semantics depend on the operators used and the governance attributes applied during the transition.

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

# **12. Inheritance Model**

Governance inheritance in ASCP defines how **semantic authority, participation context, and coordination roles** propagate through the coordination graph (DAG). Inheritance provides continuity of meaning across nested contextual scopes while allowing precise, explicit override where required.

Inheritance is **declarative and semantic** in nature. It does not grant permissions, perform enforcement, or directly control cryptographic access. Instead, it establishes the contextual governance state from which applications and agents derive authorization, visibility, and workflow behavior.

## **12.1 Inheritance Semantics (Conceptual Overview)**

Inheritance in ASCP expresses the principle that **responsibility is exercised within accountability**, and that articulated work occurs within broader organizational and strategic context.

Accordingly:

- **Spaces** represent *accountability containers*
- **Streams** represent *vectors of execution*
- **Piles** represent *associative groupings of work artifacts*

Governance attributes defined at higher-level Contextual Constructs establish default expectations that flow downward to more specific execution contexts. This mirrors real-world collaborative structures, where teams own outcomes (Spaces), individuals execute tasks (Streams), and artifacts accumulate within both.

Inheritance is:

- **Contextual** — it applies only within containment relationships between Contextual Constructs
- **Directional** — flowing from enclosing accountability contexts to enclosed execution contexts
- **Declarative** — expressing meaning, not enforcement
- **Deterministic** — producing identical results across replicas

Inheritance does not apply across Addressing Constructs (e.g., Groups), Distribution Constructs (e.g., Channels), or non-containment relationships.

## **12.2 Default Inheritance**

Unless explicitly overridden, governance attributes follow a default inheritance model aligned with the containment hierarchy of Contextual Constructs:

- **Streams** inherit governance from their enclosing Space
- **Piles** inherit governance from their enclosing Stream or Space

This default reflects the assumption that execution contexts operate under the authority of their enclosing accountability contexts. It minimizes repetitive articulation while preserving explicit override as a first-class capability.

Inheritance applies uniformly to all governance attributes unless otherwise specified, including:

- `member`
- `writer`
- `owner`
- `role::*`

## **12.3 Encapsulation and Hierarchy Rules**

The following table defines the normative containment and inheritance relationships among Constructs:

| **Structure** | **May Contain**                         | **May Be Contained By** | **Inherits From**                          |
| ------------- | --------------------------------------- | ----------------------- | ------------------------------------------ |
| **Space**     | Bookmarks, Piles, Streams, other Spaces | Parent Spaces           | Parent Space (unless overridden)           |
| **Stream**    | Bookmarks, Piles, document Artipoints   | Spaces                  | Parent Space (unless overridden)           |
| **Pile**      | Bookmark Artipoints                     | Streams or Spaces       | Parent Stream or Space (unless overridden) |
| **Group**     | Participants, Groups                    | Any Structure           | *N/A*                                      |

Only **structural containment edges** participate in inheritance evaluation.

## **12.4 Rules**

The following rules are normative:

1. **Streams inherit from their parent Space**
2. **Piles inherit from their parent Stream or Space**
3. **Spaces inherit only from enclosing Spaces**
4. **Groups do NOT inherit governance attributes -** Groups are Addressing Constructs. They define *who* participates, not *where* work occurs or *under what accountability*. Allowing Groups to inherit would conflate participant definition with contextual authority and is therefore prohibited.
5. **Explicit attributes override inherited values**
6. **Denials override both local and inherited positives**
7. **Expired grants MUST be removed**
8. **Inheritance MUST NOT cross**:
   - Channel boundaries
   - Group boundaries
   - Non-containment relationships (e.g., references, supports)
9. **Payloads MUST NOT participate in inheritance evaluation**

Payloads serve as semantic anchors to external representations. They do not affect governance computation.

## **12.5 Explicit Inheritance Override**

The inherits := attribute allows a Construct to explicitly redefine its inheritance source.

```asciidoc
inherits := <structure-ref>
inherits := default
```

- When set to a structure reference, inheritance MUST be computed relative to that referenced Construct.
- When set to default, inheritance MUST revert to the standard containment-based model.
- If no valid inheritance source can be resolved, inheritance MUST terminate at the current Construct.

Explicit inheritance overrides MUST be applied **before** group and virtual group resolution.

## **12.6 Evaluation Algorithm (Normative)**

Clients **MUST** compute governance for a Construct by executing the following steps in order:

1. Gather local governance attributes
2. Apply inherited attributes from parent Constructs
3. Apply explicit inherits := overrides
4. Resolve referenced Groups (recursively, acyclic)
5. Resolve Virtual Groups
6. Apply denials (`deny::*`)
7. Remove expired grants (`expiration::*`)
8. Resolve RACI-style role assignments
9. Produce the **Effective Governance Set**

The Effective Governance Set represents **semantic participation and authority**. It does not itself grant cryptographic access or authorize runtime actions.

## **12.7 Accountability and Responsibility (Informative)**

In typical usage, a common pattern is where:

- **Spaces** declare accountability (e.g., `role::accountable`)
- **Streams** declare responsibility (e.g., `role::responsible`)
- **Piles** and document Artipoints inherit both

This structure ensures that every execution context is enclosed within a clear accountability boundary, while allowing responsibility to be scoped precisely to the work being performed.

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

