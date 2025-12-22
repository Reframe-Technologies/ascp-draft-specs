# **ASCP Artipoint Grammar: A Structure for Shared Cognition**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.69 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document is part of the ASCP specification suite and defines the Layer-2 grammar used to express immutable, addressable **Artipoint Expressions** within ASCP. It is published at this time to gather community feedback on the structure, clarity, and interoperability of the grammar, articulation patterns, and structural coordination semantics it defines.

This is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174. Their use here is intended to convey the authors’ expectations for future interoperability profiles; the normative requirements are provisional and subject to change.

Feedback from implementers, protocol designers, distributed systems researchers, and security reviewers is explicitly requested to guide further development toward a future Internet-Draft.

# **2. Abstract**

This draft defines the Artipoint Grammar, the Layer-2 coordination syntax of the Agents Shared Cognition Protocol (ASCP). The grammar specifies the structure of **Artipoint Expressions**, expressed as immutable, timestamped, and author-attributed **Articulation Statements** that introduce and relate **Artipoints** within the ASCP coordination graph. This specification defines articulation patterns, operator semantics, payload formats, and the ABNF required for deterministic parsing and interoperability.

# **3. Introduction & Background**

This Artipoint Grammar emerged in response to a fundamental gap in human-human and human-AI collaboration: the absence of durable, interpretable, and shared cognitive structure for coordination work. Most contemporary systems operate on ephemeral exchanges—messages, prompts, or transient memory updates—rather than persistent, structured meaning that makes collaborative work possible. What the web standardized for documents, ASCP seeks to standardize for **contextual coordination**.

This challenge connects directly to what the Computer-Supported Cooperative Work (CSCW) literature calls **"articulation work"**—the essential but often invisible work that cooperating individuals must perform to partition work into units, divide it amongst themselves, and reintegrate it after completion. Articulation work is all the work around cooperative work that makes it possible—a secondary work process that enables primary collaborative activities. In human-AI teams, this articulation work becomes even more critical as it must bridge fundamentally different cognitive architectures while maintaining shared understanding over time.

Traditional collaboration systems fail to capture the **auditable cognitive decisions** that constitute this articulation work. Instead of preserving the reasoning about *why* work is organized in particular ways, *how* pieces relate to each other, and *what* dependencies exist, they focus on content exchange. ASCP recognizes that effective collaboration requires a **persistent, shared cognitive substrate**—the immutable structure of how work gets organized, connected, and reasoned about.

We studied prior efforts in provenance modeling (W3C PROV), distributed content graphs (Merkle DAGs, IPFS), semantic triple stores (RDF), and collaborative data models (CRDTs). While each informed our thinking, no prior art addressed the dual needs of:

1. **Expressing structured cognitive intent** in a form that is both deterministically machine-parseable and also fully human-relatable
2. **Supporting co-evolution of meaning** across humans and agents, while preserving an immutable, auditable record of articulation decisions

Where RDF emphasizes universal description and Merkle DAGs emphasize verifiability, the Artipoint Grammar emphasizes **semantic articulation**—the ability to express and preserve the shape of collaborative reasoning itself. This grammar sits between language and logic: minimal enough to be used conversationally, yet structured enough to support collaborative reasoning and interoperable contextual memory at scale.

The result is a system that captures not just the final state of collaborative work, but the complete history of articulation decisions that built that shared understanding—creating a foundation for true shared cognition between humans and intelligent agents.

## 3.1 Coordination Invariant (Informative)

ASCP represents coordination exclusively through **immutable, append-only articulation**.

Context is never modified, deleted, or overwritten. All evolution of meaning, structure, or relevance is expressed solely by the creation of additional Artipoints that extend the coordination graph. While interpretations of context may change over time, the historical record of articulation remains complete, addressable, and auditable.

This invariant applies uniformly across all articulation patterns and operator semantics defined in this specification.

## 3.2 Terminology Reference

Terminology in this document follows the definitions established in the **ASCP Terminology Primer**, particularly the distinction between **Artipoints** (semantic units) and **Articulation Statements** (acts of coordination).

# **4. Structural Model of the Grammar**

The Artipoint Grammar defines a minimalist syntax for representing **acts of articulation** that introduce cognitive atoms—called Artipoints—as immutable, addressable statements. Multiple Articulation Statements form an **Articulation Sequence**, where each Artipoint is a semicolon-terminated line that captures a complete, atomic declaration of intent or structure.

#### The Coordination DAG

The **coordination DAG** is a directed acyclic graph with two types of elements:

- **Nodes:** semantic Artipoints introduced by Articulation Statements
- **Edges:** verb-operator articulations that connect those nodes

The coordination DAG is global, append-only, and fully historical: once an Artipoint or edge is introduced, it remains addressable for the lifetime of the system. No articulation ever removes or mutates existing nodes or edges; all change occurs through monotonic extension of the graph.

The grammar is intentionally flat: **there is no nesting** within expressions. All composition happens via references to previously defined Artipoints using their UUID, creating a Directed Acyclic Graph (DAG) of shared cognition. This approach enables:

- Machine-readable and human-relatable cognitive statements
- Individual referenceability and auditability
- Incremental composition into complex structures
- Stream-compatible, line-by-line processing
- Decentralized distribution via ASCP Channels

ASCP’s core innovation is recognizing that collaboration requires a **persistent, shared cognitive substrate**—the articulated structure of how work is organized, connected, and reasoned about—distinct from the content itself. ASCP records the articulated structure of coordination: decisions about relevance, relationship, dependency, and organization, independent of the underlying content. This scaffolding persists and remains auditable even as underlying documents and data evolve.

All Articulation Statements are applied to the coordination DAG in a strictly ordered, append-only manner; the evaluation model governing this application is defined in Section 9.

#### Structural Effects vs Semantic Effects

For the purposes of this specification, it is essential to distinguish between:

- **Structural effects**, which describe how an articulation extends the coordination DAG (i.e., the introduction of new semantic Artipoints via Articulation Statements and/or new labeled edges); and
- **Semantic effects**, which describe how articulated relationships are *interpreted* when materializing views of the graph within a given context.

The Artipoint Grammar defines **structural effects only**, as recorded in the coordination DAG. Semantic effects — including activation, masking, prioritization, supersession, or displacement — are deterministic evaluation rules applied during interpretation and **MUST NOT** modify the underlying DAG.

This distinction is relied upon throughout the operator semantics defined in Section 8.

## 4.1 The Reference Principle

Key design insight: **Artipoints capture cognitive structure, not dynamic content itself**.

When an Artipoint needs to incorporate evolving content—documents, databases, real-time streams—that mutable state lives externally and is referenced through URIs in the payload. This is the **"bookmark pattern"**: rather than embedding a 50-page research paper directly, you create a cognitive statement about it with a URI to the external document. The paper may be updated, moved, or versioned, but the cognitive decision—the structural relationship between paper and project—remains immutable and auditable within the DAG.

The core unit follows this non-normative pattern:

```
[uuid, author, timestamp, expression];
```

Where the optional expression enables four fundamental articulation patterns: instantiation (creating new cognitive atoms), annotation (enriching existing ones), connection (linking atoms), and construction (creating and linking simultaneously).

## 4.2 **Expression of Coordination Relationships**

Verb-operators are the **only** grammar mechanism that introduce coordination relationships into the DAG.

Attributes and payloads MAY reference other Artipoints by UUID, but such references do **not** create structural edges and do **not** participate in DAG topology. Only connection and construction expressions using verb-operators produce coordination relationships.

## **4.3 Externalized Context Principle (Informative)**

The Artipoint Grammar enforces a strict separation between **articulated structure** and **embedded content**.

All contextual meaning—such as grouping, scope, precedence, supersession, evolution, or relevance—MUST be expressed through **relationships between Artipoints in the coordination graph**, never by embedding structure or semantics inside payloads.

Payloads are intentionally opaque to the grammar. They serve as **semantic anchors** to external or domain-specific content, while all composition, interpretation, and coordination semantics arise exclusively from:

- the immutable sequence of Artipoints,
- the directed relationships introduced by operators, and
- the positions of those Artipoints within the resulting DAG.

This design ensures that contextual meaning remains durable, auditable, and interoperable even as referenced content evolves independently. The grammar therefore captures the **persistent cognitive scaffolding** of collaboration, rather than transient or mutable state.

## **4.4 Immutability in Practice**

Because Artipoints capture the persistent cognitive substrate rather than dynamic content, each statement becomes a permanent, auditable decision point in the DAG. This design enables rich collaborative histories: an AI agent's relevance judgment, a human's organizational decision, or a team's dependency mapping all accumulate as an immutable record—even as the documents, data, and deliverables they reference continue to evolve externally.

**Example**: When an AI agent discovers a relevant research paper, it doesn't embed the paper's text:

```clike
[<uuid-of-this-articulation>,
     <uuid-of-author-identity>,
     2024-03-15T14:30:22.123Z,
 [ "bookmark",
  "Attention Mechanisms in Transformer Models",  
  uri:"https://arxiv.org/abs/2023.12345" ]
] supports <relevant-stream-uuid>;
```

This creates a permanent record: "this agent determined this paper was highly relevant to this project at this moment." The paper may be updated or moved, but the cognitive judgment—the structural relationship between paper and project stream—remains immutable and traceable.

## 4.5 Coordination Scopes

A **scope** is any articulated contextual structure (such as a Space, Stream, Pile, or other aggregate Artipoint) in which one or more Artipoints are explicitly composed via coordination relationships.

Scope is always derived from explicit articulation within the coordination DAG. Scope is never implicit, global, or inferred from payload content; it exists only where established by explicit articulation.

## 4.6 Structural Benefits

This design enables teams to work within a common cognitive structure while maintaining appropriate privacy and scope. Team members contribute to the same underlying DAG—the shared "tree" of how work is organized—while potentially having private branches and nodes that others cannot see. Everyone benefits from the structural coherence without sacrificing information security or cognitive autonomy.

The result is true **shared cognition**: not just exchanging messages or files, but building and evolving a persistent, jointly-accessible model of how the work itself is structured and interconnected.

# **5. Core Articulation Elements**

## **5.1 Artipoint Expression**

The core normative grammatical representation of an Artipoint is as follows:

```bnf
artipoint = "[" uuid "," author "," timestamp "," expression "]"
```

**Fields:**

- **uuid**: RFC-4122 compliant universally unique identifier for this Artipoint
- **author**: the UUID reference to the *identity* who is authoring this Artipoint. An author is always referencing a person or agentic identity. This must be the UUID of an Indentity Artipoint. See next section for details.
- **timestamp**: This contains the RFC 3339 UTC timestamp for the time of articulation.
- **expression**: One of four supported articulation patterns (see below)

## 5.2 Articulation Statement

Each Artipoint is a single, semicolon-terminated line (called an `artipoint-statement` in the formal grammar):

```bnf
articulation-statement = artipoint ";" [ end-of-line ]

```

The Articulation Statement forms a "Cognitive Atom"—an atomic **act of articulation** that captures a specific coordination assertion made by an author at a specific time, whose semantic result is the introduction or modification of one or more Artipoints.

## 5.3 Articulation Sequences

One or more Articulation Statements are then logically form into an `articulation-sequence`:

```abnf
articulation-sequence = 1*(artipoint-statement)

```

An Articulation Sequence is passed from Layer-2 to Layer-1 Channels for distribution. All **Articulation Statements** in a sequence MUST share the same Author, whose credentials secure the sequence at Layer-1.

## **5.4 Statement Author**

An Articulation Sequence is passed from Layer-2 to Layer-1 Channels for distribution. All **Articulation Statements** in a sequence MUST share the same Author, whose credentials secure the sequence at Layer-1. The author field contains a **uuidReference** that points to an **Identity Artipoint**—an immutable record containing the author's attributes including handles, decentralized identifiers (DIDs), email addresses, and cryptographic key material.

This design ensures that authorship becomes an integral part of the immutable DAG of cognition itself: statements are always authored, and authors are themselves first-class Artipoints with persistent, verifiable identities.

The author field **MUST** contain a UUID reference to a valid Identity Artipoint. The grammar requires that author contains a UUID referencing an Identity Artipoint. Validation of signatures, key relationships, and authorship correctness is defined in the ASCP Channels and ASCP Identity specifications. External identifiers such as email addresses, DIDs, or URLs **SHOULD** be stored as attributes within the Identity Artipoint and **MUST NOT** appear directly in the author field, maintaining clean separation between identity and identification methods.

More generally, the Artipoint Grammar expresses **what is being articulated**, not **who is permitted to articulate**, **who must receive it**, or **how it is enforced**.

The grammar captures **authorial intent and structural meaning** only. Questions of permission, authority, participation, and accountability are articulated separately as governance metadata and evaluated by higher protocol layers. Questions of delivery, visibility, and cryptographic access are handled exclusively by ASCP Channels.

This separation allows the grammar to remain minimal, deterministic, and universally interpretable, while supporting rich coordination semantics through composition rather than control logic.

## **5.5 Statement Recipients (Informational)**

The Artipoint Grammar defines the structure and semantics of Artipoints and Articulation Sequences, but it **does not encode explicit recipients** or perform any distribution or access-control function.

However, **governance itself is articulated through Artipoints**. Membership attributes, group definitions, and inheritance rules (normatively defined in the *ASCP Governance & Access Control* specification) determine **who is contextually part of a Space, Stream, or other collaborative structure**. These governance statements express the *semantic audience*—that is, who *ought* to participate in the coordination context.

Actual delivery, visibility, and privacy are enforced solely by **ASCP Channels** (normatively defined in the *ASCP Channels: Secure Distribution Layer* specification). Channels determine **who receives, stores, and can decrypt** an Articulation Sequence.

In short:

- **Governance Artipoints** and **Attributes** express *who is included* in a collaborative scope;
- **Channels** where Articulation Sequences are distributed enforce *who actually receives* articulated statements.

This maintains strict layering: governance lives at Layer-3, articulation semantics lives at Layer-2, while distribution and access-control are implemented via Layer-1.

# **6. Articulation Patterns**

The true power of Artipoints emerges through **composition**—how individual cognitive atoms combine to form complex, interconnected structures within the DAG. Each of the four articulation patterns defined below serves a distinct role in building and evolving this shared cognitive graph:

- **Instantiation** creates new nodes—the foundational cognitive atoms that represent tasks, documents, insights, or any meaningful unit of thought
- **Connection** establishes directed edges between existing nodes, encoding relationships like dependencies, associations, or semantic links
- **Construction** combines instantiation and connection in a single statement, simultaneously creating a new node and linking it to existing context
- **Annotation** enriches existing nodes with metadata, state updates, or additional semantic information without changing the graph's topology

Together, these patterns enable incremental, collaborative construction of knowledge structures. An AI agent might instantiate a research task, connect it to relevant prior work, while a human collaborator annotates it with priority metadata. Each action adds to the permanent, auditable record of how the cognitive structure evolved—creating not just the final graph, but a complete history of the reasoning process that built it.

This compositional approach means that complex cognitive artifacts—project plans, research syntheses, collaborative decisions—emerge naturally from the accumulation of simple, atomic statements over time.

The following normative sub-sections detail each distinct kind of articulation one can make along with the associated grammar.

## **6.1 Instantiation**

```bnf
instantiation = "[" artipoint-type "," label "," payload "]" [ "." attribute-list ]

```

Declares a typed and labeled unit of meaning with an embedded or referenced payload.

- **type**: A semantic label like "task", "doc", "stream", etc. Types are not reserved keywords; they are open-ended domain vocabulary. New types MAY be introduced by an implementation to suit application-specific needs. Common types are defined in the Default Symbol Dictionary for efficient encoding, but implementations are not constrained to this set.
- **label**: A human-readable title or descriptive caption. Implementations **MAY** display this in a manner similar to how the title of browser bookmark is typically rendered.
- **payload**: The main content—can be a typed embedded inline structure (ie: typedBlock), literal numeric value using various encodings, or an ordinary quoted UTF-8 string which would typically be a URL.
- **attribute-list**: Optional semantic metadata. See the section on Artipoint Attributes.

## **6.2 Annotation**

```bnf
annotation = uuidReference "." attribute-list

```

Applies new Attributes or updates existing Attributes onto an existing Artipoint. Used for modification, enrichment, or contextual update to prior context. Annotations and operators on them are fully explained in the **Artipoint Annotation Attributes** section.

## **6.3 Connection**

```bnf
connection = uuidReference verb-operator uuidSet

```

Establishes a semantic relationship between an existing source Artipoint (LHS) and one or more existing target Artipoints (RHS) using a verb-operator.

A connection:

- **MUST** be evaluated atomically as a single articulation event with no intermediate states.
- **MUST NOT** create new Artipoint&#x73;**.** Only construction or instantiation expressions create new Artipoints.
- **MUST** apply all structural effects as determined by the operator semantics, including hierarchical placement, masking behavior, and any Scoped Displacement Behavior (SDB) defined for that operator.

See Section 8 for detailed definitions of structural, hierarchical, and Scoped Displacement Behavior effects.

## **6.4 Construction**

```bnf
construction = instantiation verb-operator uuidSet

```

A construction expression creates a new Artipoint and establishes a relationship with one or more existing Artipoints (RHS) using a verb-operator.

A construction:

- **MUST** be evaluated atomically as a single articulation event with no intermediate states. The new Artipoint instantiation and operator application occur in one inseparable operation.
- **MUST NOT** be treated as two separate operations (instantiation followed by connection).
- **MUST** establish all structural effects through operator semantics, including the new Artipoint's initial hierarchical placement, masking behavior, and any Scoped Displacement Behavior (SDB) evaluated using the birth context.

See **Operator Semantics** for the definitions of topological, hierarchical, and SDB effects.

# **7. Payloads and Typed Blocks**

## 7.1 Payload Types

The normative pattern for the payload of an Artipoint MUST be formed according to the following:

```bnf
payload = quoted-string / typedBlock / scalar-value

typedBlock = payload-type ":" ( jsonObject / quoted-string / scalar-value )

payload-type = "json" / "string" / "uri" / "data" / "uuid" / future-type

jsonObject = "{" *CHAR "}" ; A balanced JSON object per RFC 8259

future-type = ALPHA *(ALPHA / DIGIT)

```

The payload field, used in both instantiations and attribute values, **MUST** conform to one of three forms as defined in the grammar above:

- A `quoted-string`
- A `typedBlock` (e.g., `json:`, `uuid:`, `uri:`, `string:`, or `data:` followed by the value)
- A `scalar-value` (numeric literals: integers, hexadecimal, or binary constants)

### 7.1.1 Type Safety and Semantic Correctness

The simple `quoted-string` form is syntactic shorthand for the `string:` typed block pattern. Implementations **MUST** interpret bare quoted strings as string-typed content.

When representing semantically-typed values, the appropriate typed block form **MUST** be used:

- **UUIDs** **MUST** use the `uuid:` prefix (e.g., `uuid:"550e8400-e29b-41d4-a716-446655440000"`)
- **URIs and URLs** **MUST** use the `uri:` prefix (e.g., `uri:"https://example.org/resource"`)
- **Structured data** **MUST** use the `json:` prefix when representing JSON objects
- **Arbitrary binary or opaque data** **SHOULD** use the `data:` prefix

Bare quoted strings **MUST NOT** be used to represent UUIDs, URIs, or other semantically-typed values. This ensures type safety, enables proper validation, and allows implementations to apply type-specific processing (e.g., URI resolution, UUID canonicalization).

Implementations **MAY** reject payloads that use bare quoted strings where typed blocks are semantically required, or **MAY** issue warnings while treating them as opaque string content.

## 7.2 JSON Payloads

The `jsonObject` pattern is intentionally opaque to the ASCP grammar. The Layer-2 parser identifies only the outer `{ ... }` boundaries and consumes everything between as a JSON payload. ASCP requires that these braces form a syntactically balanced region so the payload can be correctly delimited, but it does not enforce JSON's internal structure or semantics. JSON payloads are marked with a `json:` prefix and, when interpreted, must be well-formed JSON objects per [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259). All whitespace, formatting, and JSON-specific validation is handled externally. This separation keeps the ABNF grammar focused on articulation structure while allowing full JSON flexibility.

## **7.3 No Expressions in Payloads**

Expressions **MUST NOT** appear inside payloads. Payloads are opaque content-bearing structures and cannot contain instantiation, annotation, connection, or construction expressions. Composition occurs **only** via referencing previously articulated Artipoints through UUIDs. Implementations **MUST** reject or treat as invalid any payload that attempts to embed grammar-level expressions.

## **7.4 Payload Type Extensibility**

The standard payload types (`json`, `string`, `uri`, `data`, `uuid`) are fixed and defined by this specification. Custom payload types MAY be used for experimental or domain-specific purposes and, if proven valuable, MAY be incorporated into future versions of this specification. Implementations encountering unrecognized payload types SHOULD treat the typed block as opaque content and preserve it without interpretation.

# 8. Articulation Operator Taxonomy and Semantics

## 8.1 General Principles

Verb-operators define **coordination relationships** between Artipoints. Their semantics are governed by the following principles:

- **Declarative Only:** Operators state relationships; they do not execute behavior, enforce policy, or encode workflow logic.
- **Append-Only Structure:** Operators **MUST NOT** modify, delete, or overwrite any existing Artipoint or relationship. Each operator invocation extends the coordination DAG solely by the addition of new edges (and, in the case of constructions, a new node plus edges).
- **Structural vs Semantic Effects:** Operators define **structural effects** on the coordination DAG (as defined in Section 4). Any masking, supersession, prioritization, or displacement behavior is a **semantic evaluation effect** applied during interpretation and **MUST NOT** alter the underlying DAG.
- **Determinism:** Given an identical articulation history replayed in the same Layer-0 log order, and evaluated per Section 4, all conforming implementations MUST derive identical structural DAG effects and identical semantic evaluation outcomes.
- **Forward Compatibility:** Operators MAY be extended with future verbs. Unrecognized operators **MUST** be admitted to the log and treated as **no-op** with respect to structural and semantic effects, while remaining fully addressable.

## 8.2 Verb Operator Structure and Hierarchy

This table defines a set of verb-based operators used in the Artipoint grammar and classifies them by their semantic intent and structural behavior. Each verb enables fine-grained, semantically rich articulation of cognitive structure within the Cortex Layer, allowing both humans and agents to reason explicitly over relationships—from provenance chains to agenda construction to scoped replacements and incremental composition.

The following table classifies verb-operators by their **coordination intent** and **semantic profile**. It does not define application behavior, enforcement, or presentation. The table exists to ensure consistent interpretation of operator effects on the coordination DAG and on scope-bounded semantic evaluation.

| Verb       | Type                 | Orientation | **Hierarchical**? | SDB? |
| ---------- | -------------------- | ----------- | ----------------- | ---- |
| references | semantic link        | LHS → RHS   | No                | No   |
| replaces   | semantic override    | LHS → RHS   | No                | Yes  |
| extracts   | derivation           | LHS ← RHS   | LHS is Child      | No   |
| groups     | flat collection      | LHS → {RHS} | No                | No   |
| assembles  | hierarchy builder    | LHS → {RHS} | LHS is Parent     | No   |
| promotes   | elevation            | LHS ← RHS   | RHS raised to LHS | Yes  |
| annotates  | weak subordinate     | LHS → RHS   | LHS is Child      | No   |
| supports   | strong subordinate   | LHS → RHS   | LHS is Child      | No   |
| adds       | structural inclusion | LHS += RHS  | RHS joins LHS     | No   |
| removes    | structural exclusion | LHS -= RHS  | RHS leaves LHS    | Yes  |

Table Column Definitions:

- **Verb**: The name of the relationship operator, used in declarative Artipoint grammar statements.
- **Type**: A high-level classification of the relationship based on its cognitive or structural function.
- **Orientation**: Indicates the directional relationship between LHS and RHS, such as "LHS += RHS" for inclusion or "LHS ← RHS" for derivation. In instantiation and construction, the LHS is created with the specified relationship to existing RHS nodes. In connections, both nodes exist and the operator defines their new or updated edge relationship.
- **Hierarchical**: Specifies if/how the relationship implies a structural or hierarchical containment or dependency.
- **SDB**: Scoped Displacement Behavior (yes or no) indicates whether the operator produces scoped masking or displacement semantics within the contextual structures that contain both LHS and RHS. That is,  whether the operator suggests a semantic replacement, displacement, or archival of the RHS item.

## 8.3 Scope and Masking

Certain operators exhibit **Scoped Displacement Behavior (SDB)**. SDB is a **semantic evaluation rule**, not a structural mutation. When present, SDB affects how Artipoints are interpreted within specific contextual structures, without modifying the coordination DAG.

1. **Scope:** This is defined as the set of contextual structures (e.g., Spaces, Streams, Piles, or other aggregate Artipoints) in which **both** the LHS and RHS appear through explicit composition relations.
2. **Displacement**: Displacement effects **MUST** be applied **only** within scopes shared by both the LHS and RHS.
3. **Masking Rule:** Within any shared scope, the RHS **MUST** be treated as inactive in default materializations of that scope. The underlying DAG, including the RHS Artipoint and all prior relationships, remains unchanged.
4. **No Scope → No Displacement:** If the LHS and RHS share **no** contextual structure, displacement **MUST NOT** occur. Implementations **MAY** emit a diagnostic.
5. **Non-Propagation:** Displacement effects **MUST NOT** apply outside shared scopes and MUST NOT propagate to unrelated structures.

## **8.4 Operator Implications on the DAG**

This section defines the **Layer-2 structural effects** of verb-operators on the coordination DAG, independent of any semantic evaluation or view materialization.

The verb-operator determines the complete structural effect of both **connection** and **construction** articulations. Operators define how the LHS relates to the RHS within the coordination DAG, including:

- hierarchical containment
- grouping and ordering
- successor/predecessor relationships
- semantic linkage
- Scoped Displacement Behavior (SDB)
- parent/child relationships
- structural participation in contextual boundaries established through explicit composition

All operators MUST be applied atomically within a single articulation event, whether invoked through a **connection** or **construction** expression. Construction additionally performs instantiation, but evaluation of operator semantics MUST remain identical across both forms once the LHS exists.

The semantics of each operator—its orientation, hierarchical rules, displacement profile, and DAG update effects—are normative and MUST be implemented exactly as defined in this section.

## 8.5 Operator Semantics Summary

This section defines the **normative semantics** of each operator in the Artipoint Grammar. The purpose is to ensure that all compliant implementations interpret operator intent consistently, even as application views and UI materializations may differ.

| Operator   | Output / Effect                                                                                                                                                                                           |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| references | Declares that the LHS Artipoint semantically refers to each RHS Artipoint. No containment, no order implied.                                                                                              |
| replaces   | Declares that the LHS Artipoint supersedes the RHS **within shared scope**. The RHS remains in the coordination DAG but **MUST** be masked in default materializations of any scope in which both appear. |
| extracts   | Declares each LHS as a sub-part derived from RHS. Directional dependency, but does not mask RHS.                                                                                                          |
| groups     | Declares LHS as a flat collection of RHS items. Order of RHS is not semantically meaningful.                                                                                                              |
| assembles  | Declares LHS as a structured whole composed of RHS items. Order of RHS MUST be preserved.                                                                                                                 |
| promotes   | Declares the LHS as a new elevated form of the RHS. Within shared scope, the RHS **MUST** be masked in default materializations. All prior structure and history remain preserved.                        |
| annotates  | Declares LHS as metadata or commentary on RHS. RHS remains active; LHS enriches but does not supersede.                                                                                                   |
| supports   | Declares LHS as a required subcomponent enabling RHS. Dependency is directional.                                                                                                                          |
| adds       | Declares RHS is now included in the structure. Semantically equivalent to post-facto grouping.                                                                                                            |
| removes    | Declares that the RHS is excluded from the addressed structure **within shared scope**. The RHS remains in the coordination DAG and may remain active in other scopes.                                    |

## 8.6 Detailed Description of each Operator

Each operator described below follows the same invariant pattern: articulation extends the coordination DAG with new relationships, while any supersession or exclusion semantics are applied only during interpretation and only within explicitly shared scope.

### 8.6.1 references

Indicates that the LHS item semantically refers to or is informed by the RHS item, without implying dependency or structural inclusion.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:00Z,
  [decision,
     "Proceed with option B",
     uri:"https://workspace/docs/discussion-summary"]
  references {uuidB}
];
```

*A decision references a prior discussion point.*

### 8.6.2 replaces

Indicates semantic supersession: the LHS is intended to supersede or override the RHS. The RHS remains in history but is no longer active.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:10Z,
  [document,
     "Final Draft",
     uri:"/docs/final.pdf"]
  replaces {uuidOldDraft}
];
```

*A new document replaces an older version.*

### 8.6.3 extracts

The LHS is a derived or excerpted sub-part of the RHS—typically in a child role. Used for derivation without supersession.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:20Z,
  [snippet,
     "Key passage from study",
     uri:"https://papers.ai/conference2025/paper42#passage26"]
  extracts {uuidDoc}
];
```

*A snippet is extracted from a full document.*

### 8.6.4 groups

LHS is a flat set or collection of RHS items, such as a pile or unordered list.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:30Z,
  [pile,
     "Articles for Review",
     uri:"https://mycompany.com/pileState.pile"]
  groups {uuid1, uuid2, uuid3}
];
```

*A pile groups together several items without implying order or structure.*

### 8.6.5 assembles

LHS is constructed as a structured whole from multiple RHS components. Hierarchical containment is implied.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:40Z,
  [agenda,
     "Monday Sync Agenda",
     uri:"https://calendar.company.com/agenda/monday-sync"]
  assembles {uuidTopic1, uuidTopic2}
];
```

*An agenda is assembled from individual discussion points.*

### 8.6.6 promotes

LHS is a new, elevated form of RHS, which it displaces. Used when transforming one structure into a new one that absorbs its content.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:50Z,
  [stream, "Live Workstream", uuidOldPile] promotes {uuidOldPile} ];
```

*A stream promotes an earlier pile, replacing it as the active structure.*

### 8.6.7 annotates

LHS provides a subordinate comment, note, or enrichment on the RHS item. It does not structurally alter the RHS.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T11:00Z,
  [comment,
     "Needs clarification",
     uri:"https://workspace/docs/proposal-draft"]
  annotates {uuidTarget}
];
```

*A comment annotates a document or statement.*

### 8.6.8 supports

LHS is a functional subcomponent or enabler of the RHS. Use for required substructure or infrastructural dependency.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T11:10Z,
  [scene, "Initial Scene",
    json:{
      "timestamp": "00:00:00",
      "scene": "intro",
      "camera": "static",
      "caption": "Welcome to the session"
    }
  ] supports {uuidStream}
];
```

*A scene supports a stream.*

### 8.6.9 adds

LHS includes new RHS items into an existing structure or set. This is a post-facto inclusion.

Example:

```
uuidPile adds {uuidNewDoc};
```

*Adds a document to an existing pile.*

### 8.6.10 removes

LHS excludes the RHS item from an existing structure, marking it as no longer active.

Example:

```
uuidPile removes {uuidOldDoc};
```

*Removes a document from a collection.*

# **9. Articulation Evaluation Model (Normative)**

This section defines the **evaluation model** governing how Articulation Statements are applied to the coordination DAG over time. It specifies the authoritative ordering, replay guarantees, and lifecycle rules under which the structural and semantic definitions in this specification MUST be interpreted.

This section does **not** define operator semantics, scope rules, or Scoped Displacement Behavior (SDB). Those behaviors are defined exclusively in Section 8 and MUST be applied in accordance with the evaluation model defined here.

## **9.1 Log-Order Invariant**

All Articulation Statements MUST be evaluated **strictly in the order they are received from Layer-0 log replay**.

- Layer-0 log order is the **sole authoritative ordering source** for Layer-2 evaluation.
- Implementations MUST NOT reorder, buffer, batch, delay, speculate on, or retroactively reinterpret Articulation Statements.
- Within a single Articulation Sequence, Articulation Statements MUST be applied **in sequence order**.

Metadata — including timestamps, UUID values (including UUIDv7), payload contents, or operator types — MUST NOT be used to alter evaluation order.

## **9.2 Append-Only Replay Semantics**

Evaluation of Articulation Statements is **monotonic and append-only**.

- Each Articulation Statement is applied exactly once, at the time it is encountered during replay.
- No previously applied structural or semantic result may be undone, reordered, or re-evaluated due to later statements.
- All evolution of meaning, relevance, or activation is expressed solely by the addition of new Articulation Statements.

This model ensures that evaluation is **prefix-deterministic**: given the same ordered prefix of a channel log, all conforming implementations MUST derive identical structural and semantic results for that prefix.

## **9.3 Structural Replay vs Semantic Interpretation**

Evaluation proceeds in two strictly ordered phases for each Articulation Statement:

1. **Structural application**, in which the coordination DAG is extended according to the grammar and operator structure outlined in Section 4.
2. **Semantic interpretation**, in which operator-specific meaning — including scope, masking, prioritization, supersession, or displacement — is evaluated according to Section 8.

Semantic interpretation MUST NOT modify the underlying coordination DAG. Structural effects are permanent; semantic effects are interpretive.

## **9.4 Unresolved References and Deferred Interpretation**

Articulation Statements MAY reference UUIDs that have not yet been observed during replay.

- Such references MUST be admitted and recorded without buffering or rejection.
- Any semantic interpretation that depends on an unresolved Artipoint MUST be deferred until that Artipoint becomes resolvable through later replay.
- Deferred interpretation MUST NOT retroactively reorder or invalidate prior evaluation results.

This rule enables deterministic local-first operation under out-of-order delivery and partial replication.

## **9.5 Idempotency and Historical Fidelity**

Duplicate or repeated Articulation Statements MAY appear in the log.

- Repetition MUST NOT alter the effective structure or interpretation beyond the first applicable articulation.
- Historical duplication MUST be preserved as part of the immutable record, even when semantically redundant.

Idempotency is defined relative to **prior replayed history**, not by global deduplication or speculative analysis.

## **9.6 Deterministic Evaluation Requirement**

> **Given an identical Layer-0 log replayed in the same order, all conforming ASCP implementations MUST construct identical coordination DAGs and MUST derive identical semantic interpretation results.**

No implementation-defined discretion is permitted at Layer-2 with respect to ordering, replay, or evaluation.

This requirement is foundational to interoperability, shared cognition, and auditable collaboration within ASCP.

# **10. Artipoint Annotation Attributes**

Attributes provide optional metadata for Artipoints and are commonly used in annotations and bookmarks. Attributes do not introduce coordination relationships and do not modify the coordination DAG.

## 10.1 Attribute Syntax

The normative pattern for annotating Artipoints with attributes is as follows:

```
attribute-list = "(" keyValuePair *(separator keyValuePair) ")"
keyValuePair   = [class "::"] key attr-operator ( payload / uuidReference )

```

An attribute list is a collection of key-value metadata pairs. Each key MAY optionally be qualified with a class prefix using the `::` separator. Values MUST be either a payload (quoted-string, typedBlock, or scalar-value) or a uuidReference to another Artipoint.

## **10.2 Class-qualified keys**

When using the `class::key` pattern, implementations MUST NOT insert whitespace around the `::` separator. The class prefix, separator, and key form a single lexical token (e.g., `role::owner`, `signature::protocol`). Parsers encountering whitespace around `::` SHOULD emit a diagnostic warning but MAY accept the attribute as valid.

## 10.3 Attribute Operators

Operators modify the value using the following available semantic patterns:

| **Operator** | **Meaning**                                                                                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| +            | Appends to a value set. If no value exists, this becomes the first member of the set.                                                                                                         |
| -            | Removes a value from an existing set. If the value is inherited or implied, it is explicitly excluded.                                                                                        |
| :=           | Assigns an explicit value displacing any previous value or set. You can think of this as rebasing the value. An empty value string can be used to, in effect, clear and remove the attribute. |
| =            | Equivalence. The attribute is considered an alias or equivalent to whatever value is specified. In this way, it makes that attribute an alias to be referenced elsewhere.                     |

## **10.4 Attribute Key Extensibility**

Attribute keys are fully extensible and represent open-ended domain vocabulary. Implementations MAY freely introduce custom attribute keys to support application-specific metadata needs. Common attributes (such as `member`, `owner`, `role::`, `signature::`) are defined in the Default Symbol Dictionary for efficient encoding, but the attribute namespace is not constrained to this set. Vendor-specific or application-specific attributes SHOULD use namespaced class prefixes (e.g., `myapp::custom_field`) to avoid conflicts. Implementations encountering unknown attributes MUST preserve them without error.

## 10.5 Attribute Examples (non-normative)

```
(document_metadata :=
  json:{
    "word_count": 2847,
    "last_modified": "2024-03-15T14:30:22Z",
    "version": "draft-0.4",
    "collaborators": ["jeff@reframe.systems", "ai-agent-7"]
  }
);

(signature::protocol :=
  json:{
    "algorithm": "ed25519",
    "timestamp": "2024-03-15T14:30:22.123Z",
    "signer_pubkey": "a1b2c3d4...",
    "signature_hex": "f9e8d7c6..."
  }
);

( members + "alice@research.org",
  members + "bob@cogtech.dev",
  members + "charlie@ai-systems.net"
);
```

# **11. Universally Unique Identifiers (UUIDs)**

Artipoints and Artipoint references use UUIDs as their canonical identifiers.

Because these identifiers participate directly in DAG construction, causal evaluation, immutability semantics, and reference resolution, the UUID format is a compulsory part of the Artipoint Grammar.

### **11.1 UUID Version Requirement**

UUIDs is ASCP are defined based on RFC-4122, but they **MUST** be generated as **UUID version 7** (time-ordered, Unix epoch milliseconds + randomness) as defined in \[IETF draft-peabody-dispatch-new-uuid-format].

```
550e8400-e29b-41d4-a716-446655440000
```

UUIDv7 is required at the grammar layer because Artipoint UUIDs are not opaque tokens. Instead, they are structural identifiers with temporal-sortable properties that ASCP uses to support deterministic DAG construction and evaluation.

Implementations **MUST** reject Artipoints whose UUID field is not a valid UUIDv7.

### **11.2 Encoding Patterns**

- In the ASCP Grammar when encoded in Hexadecimal form, they may be written with or without hyphens (both accepted)
- Hyphens are never present in binary forms such as in Binary Value Island (BVI) or in wire encodings.
- For debugging, logs, and CLI tools, implementations **SHOULD** support parsing and emitting the canonical **RFC-4122 hex form with hyphens**:

**Examples**:

- Raw bytes → 550e8400e29b41d4a716446655440000
- Hex with hyphens → 550e8400-e29b-41d4-a716-446655440000
- Hex without hyphens → 550e8400e29b41d4a716446655440000

### **11.3 Validation Rules**

- The grammar accepts uppercase or lowercase hexadecimal.
- Version nibble (bits 48–51) **MUST** equal 0111 (v7).
- Variant bits **MUST** conform to RFC-4122 (10x).
- Parsers **MUST** accept hyphenated and non-hyphenated hex, Base64URL encoding may be accepted as well, but implementations MUST normalize internally to raw 16-byte representation.

### **11.4 UUID Role in DAG Construction**

UUIDv7 identifiers are **structural elements** of the Artipoint Grammar, not opaque tokens. UUIDs provide:

- **immutable identity** of every Artipoint
- **reference stability** across all articulation statements
- **collision resistance** suitable for distributed authorship

Because the semantics of instantiation, reference, masking, and supersession require stable identifiers, UUIDv7 is a **normative structural requirement** of the grammar. The DAG **MUST** be evaluated using the UUID as the authoritative identity of each Artipoint.

# **12. Timestamps**

```bnf
timestamp = date "T" time [fraction] "Z"
date      = 4DIGIT "-" 2DIGIT "-" 2DIGIT
time      = 2DIGIT ":" 2DIGIT ":" 2DIGIT
fraction  = "." 1*DIGIT
```

Timestamps in ASCP Artipoints represent the **articulation time**—the moment when the author (human or AI agent) made the cognitive decision to create, modify, or relate cognitive structure within the shared DAG. This is fundamentally different from content creation time, system processing time, or wall-clock synchronization, although the changes to the DAG *should* propagate into the ASCP channel log with minimal latency and typically within one second or less.

- MUST be **RFC 3339 compliant** UTC-only timestamps (Z suffix mandatory)
- Fractional seconds SHOULD be limited to millisecond precision (3 digits maximum)
- Timestamps SHOULD be precise to at least the nearest second
- Timestamps SHOULD be accurate to within 100 milliseconds of actual wall-clock time when the system is operating online
- Systems MAY tolerate wall-clock drift of up to a few seconds when operating offline or with limited network connectivity
- Hours restricted to 00-23 (following RFC 3339's clarification over ISO 8601)
- No timezone offsets permitted in this grammar (reserved for future extensions)
- Case-insensitive: both "T"/"Z" and "t"/"z" are valid per RFC 3339

Timestamps represent authorial articulation time and provenance metadata. They MUST NOT be used to reorder, delay, or reschedule Articulation Statements during Layer-2 evaluation. Evaluation order is defined exclusively by Layer-0 log replay as specified in Section 9.

# **13. Provenance and Authorship**

```bnf
author = uuidReference

```

Represents the author, signer, and generator of the Artipoint.

- **uuidReference**: The RFC-4122 standard UUID of an IdentityArtipoint of the human or agent generating the Artipoint.

Cryptographic integrity, privacy, and audience scoping are all handled by **ASCP channel encoding**, not the grammar.

# **14. Strings and Escaping**

Strings **MUST** be double-quoted ("...") and support the full JSON standard string escaping mechanisms per RFC 8259:

```bnf
escaped    = "\\" ( escape-char / unicode-escape )
escape-char = "\"" / "\\" / "/" / "b" / "f" / "n" / "r" / "t"
unicode-escape = "u" 4HEXDIG
safe-char  = %x20-21 / %x23-5B / %x5D-7E

```

**Supported escape sequences:**

- `\"` - Double quote (U+0022)
- `\\` - Backslash (U+005C)
- `\/` - Forward slash (U+002F)
- `\b` - Backspace (U+0008)
- `\f` - Form feed (U+000C)
- `\n` - Line feed (U+000A)
- `\r` - Carriage return (U+000D)
- `\t` - Horizontal tab (U+0009)
- `\uXXXX` - Unicode code point (where XXXX is 4 hexadecimal digits)

**Examples:**

```
"This is a quoted string with a newline\n and a quote: \""
"Unicode example: \u03B1\u03B2\u03B3 (Greek letters)"
"Special chars: \t\r\n\b\f\/"
"File path: C:\\Users\\Documents\\file.txt"
```

# **15. Collections and Structures**

There is no nesting of expressions within expressions. Instead, **collections** (e.g. streams, piles, lists) are formed using the construction pattern with a bookmark that refers to an external document or resource.

The **Artipoint's own UUID** serves as the persistent ID of that structure. Metadata lives in the referenced document; relations live in the DAG.

# **16. Examples (Future Section)**

Coming soon: a set of validated Artipoint examples including bookmarks, annotations, equivalence chains, collections, and agent-generated updates.

# **17. Summary**

This grammar defines a minimal but powerful **cognitive substrate**—a way to persistently structure thought, reasoning, and coordination into a universally interpretable form. It is:

- Declarative, immutable, and traceable
- Friendly to both humans and agents
- Built to support persistent shared cognition across systems
- Designed for cryptographically scoped, decentralized distribution via ASCP
- Organized as articulated coordination statements, each explicitly terminated with a semi-colon (;) for clarity and structure

It provides the foundational syntax for durable, auditable collaboration between humans and intelligent systems.

# 18. Future Considerations

In the future we could/should add sections covering:

- **Extension Points**: Formal rules for extending the operator vocabulary
- **Unknown Element Handling**: How parsers should behave with unrecognized operators
- **Lifecycle Management**: A deprecation model with clear phases and timelines

# Appendix A: The Formal ASCP Grammar in ABNF

```ebnf
; Artipoint Grammar Definition (ABNF)
; -----------------------------------
; This grammar captures the syntax of Artipoints Expressions for 
; structured, immutable cognitive statements within the Cortex Layer
; of the ASCP (Agent Shared Cognition Protocol).

; Core definitions cover:
; - Artipoint Expression structure
; - Author attribution
; - Operators and articulation patterns
; - Referencing, typing and composition

; Design Principles:
; - All articulated meaning is represented as Artipoints
; - Artipoints are introduced and related via Articulation Statements
; - Immutable, addressable units forming a DAG
; - Self-similar and recursive
; - No reserved keywords for types (artipoint-type, payload-type, etc.)
; - Expressive verb operators
; - Declarative, not imperative
; - Traceable provenance
; - No nesting; all composition is by reference only.
; - UUIDs are canonical ID for a statement in the grammar;
; - Cryptographic signing, hashing, handled by ASCP channel encoding.
; - Routing/visibility is scoped by the ASCP channel.

; ----- Core sequence -----
articulation-sequence   = 1*(articulation-statement)
articulation-statement  = OWS artipoint OWS ";" OWS [ end-of-line ]

; ----- Artipoint Expression -----
artipoint      = "[" OWS uuid separator author separator timestamp
                     separator expression OWS "]"
expression     = instantiation / annotation / connection / construction

; ----- Expression Forms -----
instantiation  = "[" OWS artipoint-type separator
                     label separator payload OWS "]"
                     [ dot attribute-list ]
annotation     = uuidReference dot attribute-list
connection     = uuidReference verb-operator uuidSet
construction   = instantiation verb-operator uuidSet

; ----- Operators -----
verb-operator  = OWS ( "references" / "replaces" / "extracts" /
                 "groups" / "assembles" / "promotes" /
                 "annotates" / "supports" / "adds" / "removes" /
                 future-verbs ) OWS
future-verbs   = ALPHA *(ALPHA / DIGIT)

; ----- Payloads -----
payload        = OWS ( quoted-string / scalar-value / typedBlock ) OWS
typedBlock     = payload-type ":" OWS
                 ( jsonObject / quoted-string / scalar-value )
payload-type   = "json" / "string" / "uri" / "data" / "uuid" / future-type
jsonObject     = "{" *CHAR "}"  ; Externally Parsed as RFC 8259 JSON
future-type    = ALPHA *(ALPHA / DIGIT)

; ----- UUID sets -----
uuidSet        = "{" uuidReference *(separator uuidReference) "}"

; ----- Attributes -----
classbind      = %x3A %x3A             ; "::"
class-key      = class classbind key   ; no whitespace around '::'

attribute-list = "(" OWS keyValuePair *(separator keyValuePair) OWS ")"

; Use either 'class::key' or bare 'key'
keyValuePair   = ( class-key / key ) attr-operator ( payload / uuidReference )

; Operator spacing tolerant (your choice)
attr-operator  = OWS ( "+" / "-" / "=" / ":=" ) OWS

; ----- Identity & refs -----
uuidReference  = OWS uuid OWS
author         = uuidReference

; ----- Timestamps -----
timestamp      = date "T" time [fraction] "Z"
date           = 4DIGIT "-" 2DIGIT "-" 2DIGIT
time           = 2DIGIT ":" 2DIGIT ":" 2DIGIT
fraction       = "." 1*DIGIT

; ----- Separators / EOL -----
separator      = OWS "," OWS
dot            = OWS "." OWS
end-of-line    = CRLF / LF / CR

; ----- Identifiers -----
artipoint-type = ALPHA *(ALPHA / DIGIT / ".")   ; Fully extensible
class          = ALPHA *(ALPHA / DIGIT / ".")
key            = ALPHA *(ALPHA / DIGIT / "-" / "_")
label          = quoted-string

; ----- Values can be integers or byte strings -----
scalar-value   = integer / hex-number / bin-number

; ----- Hex Specificed Byte String ------
hex-number     = %x30 %x78 1*(HEXOCTET)  ; "0x" prefix - whole bytes
HEXOCTET       = 2HEXDIG

; ----- Binary Specified Byte Strings -----
bin-number     = %x30 %x62 1*(BINOCTET)  ; "0b" prefix - whole bytes
BINOCTET       = 8BIT
BIT            = %x30 / %x31

; ----- Signed Integer value -----
integer        = [ minus ] int
minus          = %x2D
int            = %x30 / (DIGIT19 *DIGIT)

; ----- Strings (RFC 8259 compatible escapes) -----
quoted-string  = DQUOTE *string-char DQUOTE
DQUOTE         = %x22
string-char    = safe-char / escaped
safe-char      = %x20-21 / %x23-5B / %x5D-7F / UTF8-NonASCII
escaped        = %x5C ( escape-char / unicode-escape )
escape-char    = %x22 / %x5C / %x2F / %x62 / %x66 / %x6E / %x72 / %x74
unicode-escape = %x75 4HEXDIG

; ----- UUID (RFC 4122 textual but using v7 encoding) -----
uuid           = 32HEXDIG / (8HEXDIG "-" 4HEXDIG "-" 4HEXDIG "-"
                 4HEXDIG "-" 12HEXDIG)

; ----- Core RFC 5234 -----
ALPHA    = %x41-5A / %x61-7A   ; A-Z / a-z
HEXALPHA = %x41-46 / %x61-66   ; A-F / a-f
DIGIT    = %x30-39             ; 0-9
DIGIT19  = %x31-39             ; 1-9 (skips zero)
HEXDIG   = DIGIT / HEXALPHA    ; 0-9 / A-F / a-f
CRLF     = CR LF               ; Internet standard newline

; ----- Whitespace -----
WSP     = SP / HTAB / CR / LF / VT / FF
SP      = %x20        ; Space
HTAB    = %x09        ; Horizontal Tab
CR      = %x0D        ; Carriage Return
LF      = %x0A        ; Line Feed
VT      = %x0B        ; Vertical Tab
FF      = %x0C        ; Form Feed

OWS     = *WSP        ; Optional whitespace
RWS     = 1*WSP       ; Required whitespace

; ----- UTF-8 Characters -----
CHAR = %x01-1D / %x20-7F / UTF8-NonASCII
; 0x00 (NUL) prohibited by ASCP
; 0x1E (RS) Reserved for Symbol Dictionary
; 0x1F (US) Reserved for Binary Value Islands (BVI)

; ----- UTF8-NonASCII defined by RFC 3629 -------
UTF8-NonASCII = %xC2-DF UTF8-tail /
                %xE0 %xA0-BF UTF8-tail /
                %xE1-EC 2UTF8-tail /
                %xED %x80-9F UTF8-tail /
                %xEE-EF 2UTF8-tail /
                %xF0 %x90-BF 2UTF8-tail /
                %xF1-F3 3UTF8-tail /
                %xF4 %x80-8F 2UTF8-tail
UTF8-tail = %x80-BF
```

# Appendix B: ASCP Grammar Encodings

As formally specified in Appendix A, the Layer-2 grammar is defined as **UTF-8 text** for human-friendly authoring and inspection. To reduce verbosity in frequently occurring constructs (UUIDs, timestamps, keywords, operators), ASCP defines compact **encoding conventions**. These allow binary or symbolic forms to appear directly in the grammar without changing semantics.

When presenting the grammar to users, it should always be rendered per Appendix A specifications, however when encoding for Layer 1 channel processing, the full suite of compaction options SHOULD be used.

The two key ways of achieving more compact encoding are through **Binary Value Islands (BVI)** and the **Default Symbol Dictionary (DSD)**, which are described in the following sections.

## B.1 Binary Value Islands (BVI)

For efficiency, ASCP reserves **0x1F** as a **binary island introducer**. A BVI carries a compact header so a parser knows exactly how many bytes to consume.

```
0x1F <valueType> [<ULEB128-length>] <payloadBytes>
```

Where:

- **0x1F** = Binary Value Island introducer (reserved byte)
- **valueType** = Single byte identifying the data type (0x01-0xFF)
- **ULEB128-length** = Variable-length encoding of payload byte count, but not used by all types as some types have fixed or self-encoded lengths.
- **payloadBytes** = Raw binary data of the specified length

### **B.1.1 Standard ValueTypes**

| **Value Type** | **Description**    | ULEB128 Needed? | **Notes**                                                                                |
| -------------- | ------------------ | --------------- | ---------------------------------------------------------------------------------------- |
| 0x00           | Bytes (opaque)     | YES             | Arbitrary binary data from hex or binary string representations                          |
| 0x01           | UTF-8 string (raw) | YES             | UTF-8 quoted string without JSON-style escaping required                                 |
| 0x02           | UUID Set           | YES             | Set of UUIDs enclosed by braces {} in textual form. Length will be multiple of 16 bytes. |
| 0x03 - 0x1f    | Reserved           | YES             | Variable Length reserved                                                                 |
| 0x20           | UUID               | NO              | Single UUID on an artipoint or author field with a fixed 16-byte payload                 |
| 0x21           | ISO Timestamp      | NO              | 4-byte timestamp encoding                                                                |
| 0x22           | ISO Timestamp      | NO              | 8-byte timestamp encoding                                                                |
| 0x23 - 0x2F    | Reserved           | NO              | Fixed Length items reserved                                                              |
| 0x30–0x7F      | Reserved           |                 | Reserved for future standard types                                                       |
| 0x80–0xFF      | Vendor/extension   |                 | Available for vendor-specific encodings                                                  |

### **B.1.2 ULEB128 Length Encoding**

The segment length uses ULEB128 (Unsigned Little Endian Base 128), a variable-length integer encoding widely adopted in binary protocols including DWARF debugging information, WebAssembly, and Protocol Buffers. ULEB128 represents integers by breaking them into 7-bit chunks, with each byte's most significant bit serving as a continuation flag: 1 indicates more bytes follow, while 0 marks the final byte. The encoding is little-endian at the byte level, meaning the least significant 7-bit group is transmitted first.

For example, the value 1 encodes as a single byte 0x01, while 128 requires two bytes: 0x80 0x01 (the first byte's high bit indicates continuation, its low 7 bits represent zero, and the second byte contains the remaining value 1). The value 624485 encodes as 0xE5 0x8E 0x26, demonstrating the format's efficiency for larger integers.

In the BVI, the ULEB128 length indicates the number of bytes to follow as the payload.

### **B.1.3 UUID Encoding (Type = 0x20)**

Encodes a single UUID from textual representation into compact binary form.

**Textual Form**

```
a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**Binary Format**

```
0x1F | 0x20 | <16-byte-UUID>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x20` = UUID type identifier
- `<16-byte-UUID>` = Raw UUID bytes in RFC 4122 canonical order

**Encoding Process**

1. Parse textual UUID format (with or without hyphens)
2. Convert to 16 raw bytes following RFC 4122 byte order
3. Encode as BVI with type 0x20 and 16-bytes of payload

**Examples**

- **Standard UUID:** `550e8400-e29b-41d4-a716-446655440000`
- **Binary form:** `0x1F 0x20` followed by 16 bytes: `550E8400E29B41D4A716446655440000`

**Format Requirements**

- **Input validation:** Must be valid RFC 4122 UUID format
- **Case handling:** Accept both upper and lowercase hex digits
- **Hyphen handling:** Accept with or without hyphens in textual form
- **Output format:** Always store as 16 raw bytes, no hyphens

**Usage Context**

- Used for single UUID values in author fields, references, and scalar contexts
- Distinguished from UUID sets (Type 0x02) which contain multiple UUIDs enclosed by braces in the textual form
- Required format for all Artipoint UUIDs and author identity references

### **B.1.4 UUID Set Encoding (Type = 0x02)**

Encodes a collection of UUIDs from the textual { uuid1, uuid2, ..., uuidN } syntax into a compact binary representation.

#### **Textual Form**

```plaintext
{ uuid1, uuid2, ..., uuidN }

```

#### **Binary Format**

```plaintext
0x1F | 0x02 | <ULEB128-length> | <UUID-bytes>

```

Where:

- 0x1F = Binary Value Island (BVI) introducer
- 0x02 = UUID Set type identifier
- \<ULEB128-length> = Number of bytes in the UUID data (always multiple of 16)
- \<UUID-bytes> = Concatenated 16-byte UUIDs in sequence

#### **Encoding Process**

1. Count the UUIDs in the set: N
2. Calculate byte length: N × 16
3. Encode length as ULEB128
4. Concatenate all UUID bytes in order

#### **Examples**

- **3 UUIDs:** `0x1F 0x02 0x30` followed by 48 UUID bytes
- **1 UUID:** `0x1F 0x02 0x10` followed by 16 UUID bytes
- **Empty set:** encoded as `0x1F 0x02 0x00`

#### **Semantic Properties**

- **Order preserved:** UUIDs maintain their textual sequence
- **Duplicates allowed:** Consumer implementations MAY deduplicate
- **Validation:** Byte count MUST be divisible by 16

#### **Single UUID vs UUID Set Distinction**

- A `0x20` single UUID is always a bare scalar value with no braces.
- A `0x02` UUID Set is always brace-delimited in textual form and encoded as a compact block in binary form.
- A `0x20` UUID is **not** equivalent to a singleton `0x02` UUID Set—braces are implied only in the set case.
- Implementations MUST preserve this distinction.

#### **Alternate Representation of Sets (with braces)**

- Sets may alternatively remain in the **textual grammar form** with braces { … } and commas separating elements.
- Inside this alternate form, each element MAY be encoded individually as a `0x20` UUID BVI or left as a textual UUID.
- This form is valid but **less efficient**, since braces and commas remain part of the serialized form rather than collapsing into the compact 0x02 binary block.
- Importantly, such brace-preserving encodings are **not the same as a** **0x02** **UUID Set**—they are textual-set representations with per-element encodings.

#### **Encoding Examples:**

**Original textual form:**

```
{ a1b2c3d4-e5f6-7890-abcd-ef1234567890, b2c3d4e5-f6a7-8901-bcde-f23456789012,
  f9e8d7c6-b5a4-9382-7160-594837261504, c4d5e6f7-a8b9-0123-cdef-456789abcdef }
```

**Mixed encoding (textual + binary):**

```
{ a1b2c3d4-e5f6-7890-abcd-ef1234567890, 0x1F 0x20 <16-bytes>,  
  f9e8d7c6-b5a4-9382-7160-594837261504, 0x1F 0x20 <16-bytes> }
```

**Compact 0x02 UUID Set:**

```
0x1F 0x02 0x40 <64-bytes-of-concatenated-UUIDs>
```

The mixed form preserves brace syntax while allowing per-element encoding. The compact form eliminates all textual overhead by encoding the entire set as a single binary block.

#### **Parser Requirements**

- MUST accept both canonical 0x02 binary form and brace-preserving textual form.
- MUST NOT collapse a 0x20 single UUID without braces into a singleton 0x02 UUID Set or vice versa.
- MUST consume the surrounding braces and separating commas and whitespace within the braces.
- SHOULD consume leading and trailing whitespace outside the braces as the 0x02 binary form is a full token unto itself.

This distinction ensures consistent interpretation of singleton UUIDs, compact binary sets, and alternate brace-preserving encodings.

### B.1.5 Binary Values (Type = 0x00)

Encodes arbitrary binary data from hex or binary string representations into direct binary form.

#### **Textual Forms**

```
0x48656c6c6f        ; hex representation
0b0100100001100101  ; binary representation
```

#### **Binary Format**

```
0x1F | 0x00 | <ULEB128-length> | <raw-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x00` = Binary values type identifier
- `<ULEB128-length>` = Number of bytes in the payload
- `<raw-bytes>` = Direct binary data

#### **Encoding Process**

1. Parse hex (`0x...`) or binary (`0b...`) string representation
2. Convert to raw bytes (hex: 2 chars → 1 byte, binary: 8 bits → 1 byte)
3. Calculate byte length and encode as ULEB128
4. Append raw binary data

#### **Examples**

- **Hex input:** `0x48656c6c6f` → `0x1F 0x00 0x05 48656c6c6f`
- **Binary input:** `0b0100100001100101` → `0x1F 0x00 0x02 4865`
- **Empty data:** `0x` → `0x1F 0x00 0x00`

#### **Validation Requirements**

- **Hex format:** Must contain even number of hex digits (full bytes)
- **Binary format:** Must contain multiple of 8 bits (full bytes)
- **Case handling:** Accept both upper and lowercase hex digits
- **Output format:** Always store as raw binary data

### B.1.6 UTF-8 Strings (Type = 0x03)

Encodes UTF-8 text strings without requiring JSON-style escaping for common characters.

#### **Textual Form**

```plaintext
"Hello, 世界! 🌍"
```

#### **Binary Format**

```plaintext
0x1F | 0x03 | <ULEB128-length> | <utf8-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x03` = UTF-8 string type identifier
- `<ULEB128-length>` = Number of bytes in UTF-8 encoding
- `<utf8-bytes>` = Raw UTF-8 encoded string data

#### **Encoding Process**

*Encoders MUST first unescape JSON-style sequences into fully UTF-8 literal stream before encoding as a BVI UTF-8 escaped sequence*

1. Remove surrounding quotes from textual representation
2. Fill the UTF-8 bytes payload with the expanded UTF-8 stream.
3. Calculate byte length and encode into ULEB128
4. Append UTF-8 byte sequence

#### **Examples**

- **ASCII:** `"Hello"` → `0x1F 0x03 0x05` + `48656c6c6f`
- **Unicode:** `"世界"` → `0x1F 0x03 0x06` + `e4b896e7958c`
- **Empty string:** `""` → `0x1F 0x03 0x00`

#### **Character Handling**

- **No escaping required:** Direct UTF-8 encoding of all valid Unicode characters
- **Newlines and tabs:** Encoded directly as UTF-8 bytes
- **Null bytes:** Not permitted in string content
- **Validation:** Must be valid UTF-8 sequence

### **B.1.7 Timestamps (Types = 0x21, 0x22)**

Encodes RFC 3339 compliant UTC timestamps using variable-width binary formats optimized for typical precision requirements.

#### **Textual Form**

```asciidoc
2025-08-18T12:34:56Z           ; no fractional seconds
2025-08-18T12:34:56.789Z       ; millisecond precision  
2025-08-18T12:34:56.123456Z    ; microsecond precision
```

#### **Binary Format**

```asciidoc
0x1F | 0x21 | <time32-bytes>
0x1F | 0x22 | <time64-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x21` or `0x22` = Timestamp type identifier (time32) or (time64) follows
- `<time-bytes>` = Encoded timestamp data in big-endian format

#### **Format Selection**

- **time32 (4 bytes):** No fractional seconds AND fits in 32-bit Unix timestamp range (1970-2106) for type code 0x21
- **time64 (8 bytes):** Has fractional seconds OR exceeds 32-bit range (extends to year 2554) for type code 0x22

#### **Encoding Process**

1. Parse RFC 3339 timestamp and extract Unix epoch seconds + fractional component
2. Validate fractional seconds precision (RFC 3339 supports arbitrary precision via `time-secfrac = "." 1*DIGIT`)
3. Select format based on presence of fractional seconds and timestamp range
4. Encode using appropriate binary representation

#### **Examples**

- **Basic timestamp:** `2025-08-18T12:34:56Z` → `0x1F 0x21 0x66C7B310`  
  *(1,723,988,096 seconds since Unix epoch, encoded as 4-byte time32)*
- **With milliseconds:** `2025-08-18T12:34:56.789Z` → `0x1F 0x22 0x2F0A83DE66C7B310`  
  *(789,000,000 nanoseconds in upper 30 bits, same epoch seconds in lower 34 bits)*
- **With microseconds:** `2025-08-18T12:34:56.123456Z` → `0x1F 0x22 0x1E74DF8066C7B310`  
  *(123,456,000 nanoseconds in upper 30 bits, same epoch seconds in lower 34 bits)*

#### **Format Specifications**

- **time32:** Direct `uint32` seconds since Unix epoch (1970-01-01T00:00:00Z)
- **time64:** `uint64` = (fractional\_ns << 34) | seconds, where both formats share the same Unix epoch base and upper 30 bits encode nanoseconds (0-999,999,999)

#### **RFC 3339 Compliance**

- UTC timezone only (Z suffix mandatory, no timezone offsets)
- Hours restricted to 00-23 (RFC 3339 clarification over ISO 8601)
- Case-insensitive 'T' and 'Z' characters accepted per RFC 3339
- Leap seconds handled per RFC 3339 specification (time-second may be "60")

## B.2 Default Symbol Dictionary (DSD v1)

Frequent grammar tokens can be replaced by a **two-byte escape** to reduce verbosity in commonly occurring constructs like verbs, operators, and type prefixes. This compression mechanism significantly reduces the size of encoded Artipoints while maintaining full semantic equivalence with the textual grammar.

### **B.2.1 Escape Mechanism**

```
0x1E <code>
```

Where `0x1E` is the reserved **symbol dictionary escape** introducer and `<code>` is a single byte that selects a predefined token from the Default Symbol Dictionary.

### **B.2.2 How It Works**

1. **Encoding:** When serializing Artipoints, producers can replace any occurrence of a dictionary token with its two-byte escape sequence. The encoder SHOULD consume all leading and trailing whitespace to the adjacent language tokens to minimize encoding size.
2. **Decoding:** Parsers encountering `0x1E` read the following byte as a dictionary lookup and substitute the corresponding token. If the parsing process is outputting text to the ABNF grammar, the decoder MUST add any required or readability desired leading or trailing whitespace to maintain the tokens of the language.
3. Decoders **MUST** accept any combination of encodings with or without dictionary encodings.
4. **Semantic equivalence:** The escaped form is functionally identical to the full textual token with leading and trailing whitespace.
5. **Optional optimization:** Encoder implementations MAY choose when to apply DSD substitutions based on size/speed/readability tradeoffs.

### **B.2.3 Example Usage**

```
uuidA 0x1E 0x00 {uuidB}    ; "references" compressed
uuidC 0x1E 0x40 {data}     ; "json:" prefix compressed
```

### **B.2.4 Dictionary Structure**

The DSD v1 organizes tokens into functional ranges to enable efficient lookup and future extension:

#### **Verbs (0x00–0x1F)**

| Code      | Token          |
| --------- | -------------- |
| 0x00      | references     |
| 0x01      | replaces       |
| 0x02      | extracts       |
| 0x03      | groups         |
| 0x04      | assembles      |
| 0x05      | promotes       |
| 0x06      | annotates      |
| 0x07      | supports       |
| 0x08      | adds           |
| 0x09      | removes        |
| 0x0A–0x1F | Reserved verbs |

#### **Artipoint Types (0x20–0x3F)**

| Code       | Type            |
| ---------- | --------------- |
| 0x20       | **channel**     |
| 0x21       | **keyframe**    |
| 0x22       | **identity**    |
| 0x23       | **group**       |
| 0x24       | **space**       |
| 0x25       | **stream**      |
| 0x26       | **pile**        |
| 0x27       | **document**    |
| 0x28       | **comment**     |
| 0x29       | **decision**    |
| 0x2A-0x03F | Reserved Future |

#### **Typed-Block Prefixes (0x40–0x5F)**

| Code      | Token                         |
| --------- | ----------------------------- |
| 0x40      | json:                         |
| 0x41      | string:                       |
| 0x42      | uri:                          |
| 0x43      | data:                         |
| 0x44      | uuid:                         |
| 0x45–0x4F | Reserved typed-block prefixes |

#### **Common Attributes (0x50–0x7F)**

| Code      | Token                 |
| --------- | --------------------- |
| 0x50      | member                |
| 0x51      | owner                 |
| 0x52      | writer                |
| 0x53      | inherits              |
| 0x54      | flag                  |
| 0x55      | deny::                |
| 0x56      | expiration::          |
| 0x57      | keyframe::            |
| 0x58      | keyframe::kid         |
| 0x59      | role::                |
| 0x5A      | role::accountable     |
| 0x5B      | role::approver        |
| 0x5C      | role::auditor         |
| 0x5D      | role::consulted       |
| 0x5E      | role::informed        |
| 0x5F      | role::observer        |
| 0x60      | role::responsible     |
| 0x61      | endorsement::         |
| 0x62      | purpose::             |
| 0x63      | purpose::auth         |
| 0x64      | purpose::assert       |
| 0x65      | purpose::keyAgreement |
| 0x66–0x7F | Reserved attributes   |

#### **Vendor/Extension Range (Future)**

| Range     | Purpose                       |
| --------- | ----------------------------- |
| 0x80–0xFF | Vendor/extension dictionaries |

# Appendix C: Compliance

## C.1 Core Requirements

1. **Signing:** At Layer-1 Channel encoding, producers sign the payload as emitted. No re-serialization or normalization.
2. **Reserved Bytes:** 0x1E and 0x1F are reserved as escape introducers for DSD and BVI respectively.
3. **Reserved Byte Restrictions:** These reserved bytes (0x1E and 0x1F) MUST NOT appear in textual grammar regions or in JSON objects.
   - Producers MUST NOT emit these bytes outside of BVIs/DSD contexts.
   - Decoders MUST error if encountered in textual regions, dropping the current articulation statement.
4. **NUL Characters:** U+0000 is prohibited; encoders MUST error if present; decoders MUST reject.
5. **Extensibility:** Unknown codes in the reserved/vendor ranges MUST cause parse error unless explicitly declared in a profile.
6. **Round-tripping:** Tools MAY expand escapes back to full text for readability, but any rewrite produces a distinct artifact with a new Layer-1 signature.

## C.2 Textual Format Recommendations

Articulation Statements SHOULD follow these formatting conventions:

- **Character Encoding:** MUST be UTF-8 encoding with no BOM (Byte Order Mark)
- **Line Endings:** SHOULD use Unix-style line endings (LF without CR)
- **Whitespace:** SHOULD NOT have trailing whitespace
- **Spacing:** SHOULD have a single space after commas and semicolons, except when at end of line
- **Attribute Ordering:** Attribute lists SHOULD be ordered alphabetically by key

## C.3 JSON Payload Formatting

JSON payload formatting within the grammar (e.g., `json:` blocks or attribute values) SHOULD follow these conventions:

- **Canonicalization:** SHOULD follow best practices from [RFC 8785 – JSON Canonicalization Scheme (JCS)](https://datatracker.ietf.org/doc/html/rfc8785), unless the structure being encoded requires a different format
- **Character Encoding:** MUST be strictly UTF-8 encoded just like the rest of the Articulation statement
- **Key Ordering:** Keys SHOULD be sorted lexicographically (by UTF-16 codepoint)
- **Whitespace:** SHOULD have no insignificant whitespace
- **String Format:** Strings MUST use double quotes (") with proper JSON escaping
- **Number Format:** Numbers SHOULD be in exact format (no trailing .0, scientific notation, or inconsistent formatting)

**NOTE:** The formatting conventions in "Textual Format Recommendations" and "JSON Payload Formatting" are **not required for signature correctness**, but are highly encouraged to promote consistent encoding, diffability, and tooling interoperability across ASCP implementations.

# Appendix D: Validation and Error Handling (Normative) 

This section defines **accept‑and‑record** behavior for an immutable log. Once an articulation enters a channel log, it is **never altered or removed** by protocol action. All implementations MUST converge on identical handling by following the rules below.

## D.1 Core Principles

- **Immutability:** Clients MUST store every received, well‑formed Layer‑1 envelope exactly as emitted. No rewriting, normalization, or redaction.
- **Report‑and‑proceed:** Validation errors are **reported** via diagnostics; evaluation/materialization proceeds with deterministic **no‑op defaults** where applicable.
- **Determinism:** Given the same sequence, all conforming implementations MUST produce identical DAG effects (including when effects are intentionally **no‑op**).

## D.2 Validation Phases

1. **Envelope Validation (Layer‑1):** Signature, integrity, audience/recipient checks.
   - **FAIL** ⇒ The envelope MUST NOT be passed to Layer-2 and should be marked invalid in the local log.
   - **PASS** ⇒ Proceed to grammar validation.
2. **Grammar Validation (Layer‑2):** Syntactic correctness per ABNF (tokenization, UUID/timestamp lexemes, statement structure).
   - **FAIL** ⇒ Report as invalid syntax reporting any parseable uuid, author, timestamp. DAG effect = **no‑op**.
   - **PASS** ⇒ Proceed to semantic validation.
3. **Semantic Validation (Layer‑3):** Operator, reference, and attribute semantics.
   - **FAIL (non‑critical)** ⇒ Admit and index fully; apply DAG effect per decision table (often **no‑op**); attach diagnostic code(s).

## D.3 Deterministic Handling Table

Use the following decision table to produce identical outcomes across implementations.

| Case    | Condition                                                              | DAG Effect                                                                                                    | Diagnostic             |
| ------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------- |
| **E1**  | Unknown **operator** in connection/construction                        | **no‑op** for edges implied by the operator; instantiation (if any) remains valid                             | op\_unknown            |
| **E2**  | Unknown **typedBlock** prefix (e.g., foo:{...})                        | Keep payload opaque; treat instantiation valid                                                                | payload\_type\_unknown |
| **E3**  | **Malformed attributes**(cannot parse key op value)                    | Ignore offending attribute(s) only                                                                            | attr\_malformed        |
| **E4**  | **Invalid UUID syntax** in any reference                               | Do not create edges involving invalid UUID; instantiation (if present) remains                                | uuid\_invalid          |
| **E5**  | **Reference to non‑existent UUID** (not observed yet or never will be) | Create **dangling reference** entry; edge materializes if/when target arrives                                 | uuid\_unresolved       |
| **E6**  | **Timestamp missing/invalid**                                          | Report diagnostic; process normally.                                                                          | ts\_invalid            |
| **E7**  | replaces/promotes target **invalid or unresolved**                     | No masking applied; source remains ordinary node                                                              | mask\_target\_invalid  |
| **E8**  | adds/removes on **non‑collection** LHS                                 | Treat as **no‑op**                                                                                            | op\_context\_invalid   |
| **E9**  | **Oversize payload** (exceeds implementation/profile cap)              | Store envelope; treat payload as opaque; apply operator sans payload semantics                                | payload\_oversize      |
| **E10** | Unknown **DSD/BVI** codes                                              | Preserve bytes; attempt textual fallback if present; otherwise treat affected token as unknown ⇒ follow E1/E2 | encoding\_unknown      |

> **no‑op** means: do not add edges, masks, inclusions, or exclusions implied by the failing part. The remainder of the statement (e.g., an instantiation) remains effective if valid.

## D.4 Masking & Supersession Determinism

- replaces and promotes **mask** their RHS **only within the same applicable scope** (e.g., same parent collection/structure as established by accompanying relations or application context). If scope cannot be resolved (E7), **no mask occurs**.
- removes excludes RHS from the addressed structure only; history is unaffected.

## D.5 Diagnostics (SHOULD)

Implementations SHOULD expose a diagnostics feed with fields like { uuid, envelope\_id, phase, code, details, first\_seen\_at }. Clients MAY surface these to users/agents.

## D.6 Forward Compatibility

- Unknown verbs/types/codes MUST NOT block ingestion. They are preserved for future interpreters and evaluated as **no‑op** today per E1/E2/E10.
- Profiles MAY further constrain acceptance (e.g., disallow certain typed blocks) but MUST still follow these deterministic outcomes.

## D.7 Conformance

Implementations MUST demonstrate, via test vectors, that given identical articulation sequences they yield identical DAG effects and identical diagnostics for all cases E1–E10.