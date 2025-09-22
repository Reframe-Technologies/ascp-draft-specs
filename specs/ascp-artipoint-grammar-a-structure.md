# **ASCP Artipoint Grammar: A Structure for Shared Cognition**

**Version:** Draft 0.4 - September 2025

**Scope:** Grammar for encoding immutable cognitive decisions as addressable statements within persistent, shared cognitive substrate—enabling auditable articulation work between humans and AI agents in the ASCP Cortex Layer

# **Background**

The Artipoint Grammar emerged in response to a fundamental gap in human-human and human-AI collaboration: the absence of durable, interpretable, and shared cognitive structure for coordination work. Most contemporary systems operate on ephemeral exchanges—messages, prompts, or transient memory updates—rather than persistent, structured meaning that makes collaborative work possible. What the web standardized for documents and APIs, ASCP seeks to standardize for **contextual coordination**.

This challenge connects directly to what the Computer-Supported Cooperative Work (CSCW) literature calls **"articulation work"**—the essential but often invisible work that cooperating individuals must perform to partition work into units, divide it amongst themselves, and reintegrate it after completion. Articulation work is all the work around cooperative work that makes it possible—a secondary work process that enables primary collaborative activities. In human-AI teams, this articulation work becomes even more critical as it must bridge fundamentally different cognitive architectures while maintaining shared understanding over time.

Traditional collaboration systems fail to capture the **auditable cognitive decisions** that constitute this articulation work. Instead of preserving the reasoning about *why* work is organized in particular ways, *how* pieces relate to each other, and *what* dependencies exist, they focus on content exchange. ASCP recognizes that effective collaboration requires a **persistent, shared cognitive substrate**—the immutable structure of how work gets organized, connected, and reasoned about.

We studied prior efforts in provenance modeling (W3C PROV), distributed content graphs (Merkle DAGs, IPFS), semantic triple stores (RDF), and collaborative data models (CRDTs). While each informed our thinking, none fully addressed the dual needs of:

1. **Expressing structured cognitive intent** in a form that is both deterministically machine-parseable and also fully human-relatable
2. **Supporting co-evolution of meaning** across humans and agents, while preserving an immutable, auditable record of articulation decisions

Where RDF emphasizes universal description and Merkle DAGs emphasize verifiability, the Artipoint Grammar emphasizes **semantic articulation**—the ability to express and preserve the shape of collaborative reasoning itself. This grammar sits between language and logic: minimal enough to be used conversationally, yet structured enough to support collaborative reasoning and interoperable contextual memory at scale.

The result is a system that captures not just the final state of collaborative work, but the complete history of articulation decisions that built that shared understanding—creating a foundation for true shared cognition between humans and intelligent agents.

# **Overview**

The **Artipoint Grammar** defines a minimalist syntax for representing cognitive atoms—called **Artipoints**—as immutable, addressable statements. Multiple Artipoints form an **articulation sequence**, where each Artipoint is a semicolon-terminated line that captures a complete, atomic declaration of intent or structure.

The grammar is intentionally flat: **there is no nesting** within expressions. All composition happens via references to previously defined Artipoints using their UUID, creating a Directed Acyclic Graph (DAG) of shared cognition. This approach enables:

- Machine-readable and human-relatable cognitive statements
- Individual referenceability and auditability
- Incremental composition into complex structures
- Stream-compatible, line-by-line processing
- Decentralized distribution via ASCP Channels

### Structure vs. Content: The Reference Principle

A critical design insight: **Artipoints capture cognitive structure, not dynamic content itself**. The grammar tracks the immutable decisions, connections, and articulations that shape how work gets organized—the persistent cognitive substrate underlying collaboration. When an Artipoint needs to incorporate evolving content—documents, databases, real-time streams—that mutable state lives externally and is referenced through URIs in the payload of instantiated Artipoint.

This is the **"bookmark pattern"**: rather than embedding a 50-page research paper directly in an Artipoint, you create a cognitive statement like "this paper is highly relevant to our project" with a URI pointing to the external document. The paper may be updated, moved, or versioned, but the cognitive decision—the structural relationship between paper and project—remains immutable and auditable within the DAG.

This enables teams to work within a shared cognitive architecture where the structural relationships are universally visible and persistent, while the actual content remains dynamic and appropriately scoped.

The core unit follows this pattern:

```
[uuid, author, timestamp, expression];
```

Where the optional expression enables four fundamental articulation patterns: instantiation (creating new cognitive atoms), annotation (enriching existing ones), connection (linking atoms), and construction (creating and linking simultaneously).

# **Structural Model**

## **The Foundation: Cognitive Structure, Not Content**

ASCP's core innovation lies in recognizing that effective collaboration requires a **persistent, shared cognitive substrate**—the immutable structure of how work gets organized, connected, and reasoned about. This is fundamentally different from the content itself.

Think of ASCP as tracking the **cognitive scaffolding** that teams use to coordinate their thinking: the decisions about what's relevant, how pieces relate, which tasks depend on others, what insights emerge from analysis. This scaffolding persists and remains auditable even as the underlying documents, data, and deliverables evolve rapidly.

## **The Immutability Principle**

Each Artipoint represents a permanent, auditable **cognitive decision**—a moment when someone (human or AI) made a structured statement about relationships, relevance, or reasoning. These decisions accumulate into a DAG that captures not just the final state of collaborative work, but the complete history of how that shared understanding was built.

When an Artipoint needs to incorporate dynamic content—documents that get edited, databases that update, real-time streams—that mutable state lives externally and is referenced through URIs in the payload. This is the **"bookmark pattern"**: rather than embedding content directly, we create cognitive statements *about* that content.

**Example**: When an AI agent discovers a relevant research paper, it doesn't embed the paper's text:

```clike
[a1b2c3d4-e5f6-7890-abcd-ef1234567890,
     uuid-of-author-identity,
     2024-03-15T14:30:22.123Z,
 [ paper,
  "Attention Mechanisms in Transformer Models",  
  uri:"https://arxiv.org/abs/2023.12345" ]
];
```

This creates a permanent record: "this agent determined this paper was highly relevant to this project at this moment." The paper may be updated or moved, but the cognitive judgment—the structural relationship between paper and project—remains immutable and traceable.

## **Enabling Shared Cognitive Architecture**

This design enables teams to work within a common cognitive structure while maintaining appropriate privacy and scope. Team members contribute to the same underlying DAG—the shared "tree" of how work is organized—while potentially having private branches and nodes that others cannot see. Everyone benefits from the structural coherence without sacrificing information security or cognitive autonomy.

The result is true **shared cognition**: not just exchanging messages or files, but building and evolving a persistent, jointly-accessible model of how the work itself is structured and interconnected.

## **Artipoint Format**

The core unit of an Artipoint in the grammar is as follows:

```bnf
artipoint = "[" uuid "," author "," timestamp ["," expression ] "]"

```

**Fields:**

- **uuid**: RFC-4122 compliant universally unique identifier for this Artipoint
- **author**: the UUID reference to the *identity* who is authoring this Artipoint. An author is always referencing a person or agentic identity. This must be the UUID of an Indentity Artipoint. See next section for details.
- **timestamp**: This contains the RFC 3339 UTC timestamp for the time of articulation.
- **expression**: Optional. One of four supported articulation patterns (see below)

> Omitting the expression results in a "no-op" placeholder—a valid, referenceable cognitive atom that can be enriched or connected later.

Each Artipoint is a single, semicolon-terminated line (called an `artipoint-statement` in the formal grammar):

```bnf
articulation-statement = artipoint ";" [ end-of-line ]

```

Multiple such statements form an `articulation-sequence`:

```abnf
articulation-sequence = 1*(artipoint-line)

```

## **Articulation Statement Author**

Every Artipoint represents a statement made by an **author**, creating a direct link between cognitive assertions and their originators. The author field contains a **uuidReference** that points to an **Identity Artipoint**—an immutable record containing the author's attributes including handles, decentralized identifiers (DIDs), email addresses, and cryptographic key material.

This design ensures that authorship becomes an integral part of the immutable DAG of cognition itself: statements are always authored, and authors are themselves first-class Artipoints with persistent, verifiable identities.

The author field **MUST** contain a UUID reference to a valid Identity Artipoint. Clients **MUST** validate that any signing key used in the enclosing channel envelope either matches or is endorsed by key material stored on the referenced Identity Artipoint. External identifiers such as email addresses, DIDs, or URLs **SHOULD** be stored as attributes within the Identity Artipoint and **MUST NOT** appear directly in the author field, maintaining clean separation between identity and identification methods.

# **Articulation Patterns**

The true power of Artipoints emerges through **composition**—how individual cognitive atoms combine to form complex, interconnected structures within the DAG. Each of the four articulation patterns defined above serves a distinct role in building and evolving this shared cognitive graph:

- **Instantiation** creates new nodes—the foundational cognitive atoms that represent tasks, documents, insights, or any meaningful unit of thought
- **Connection** establishes directed edges between existing nodes, encoding relationships like dependencies, associations, or semantic links
- **Construction** combines instantiation and connection in a single statement, simultaneously creating a new node and linking it to existing context
- **Annotation** enriches existing nodes with metadata, state updates, or additional semantic information without changing the graph's topology

Together, these patterns enable incremental, collaborative construction of knowledge structures. An AI agent might instantiate a research task, connect it to relevant prior work, while a human collaborator annotates it with priority metadata. Each action adds to the permanent, auditable record of how the cognitive structure evolved—creating not just the final graph, but a complete history of the reasoning process that built it.

This compositional approach means that complex cognitive artifacts—project plans, research syntheses, collaborative decisions—emerge naturally from the accumulation of simple, atomic statements over time.

The following sub-sections detail each one of distinct kind of articulations one can make.

## **Instantiation**

```bnf
instantiation = "[" type "," label "," payload "]" [ "." attribute-list ]

```

Declares a typed and labeled unit of meaning with an embedded or referenced payload.

- **type**: A semantic label like "task", "doc", "stream", etc.
- **label**: A human-readable title or caption
- **payload**: The main content—can be a typed embedded inline structure (ie: typedBlock), literal numeric value using various encodings, or an ordinary UTF-8 string which would typically be a URL.
- **attribute-list**: Optional semantic metadata (see below)

## **Annotation**

```bnf
annotation = uuidReference "." attribute-list

```

Applies new attributes to an existing Artipoint. Used for modification, enrichment, or contextual update to prior context.

## **Connection**

```bnf
connection = uuidReference verb-operator uuidSet

```

Establishes a semantic relationship between a source Artipoint and a set of targets via an operator.

## **Construction**

```bnf
construction = instantiation verb-operator uuidSet

```

Combines a newly declared instantiation with one or more related Artipoints to define collections or aggregate forms like piles or streams.

# **Payloads and Typed Blocks**

The payload field, used both in instantiations and attribute values, accepts the following:

- A quoted-string
- A typed block (e.g., embedded JSON or structured string)
- Numeric values as integers, hex or binary constants in a C-language like way

The quoted-string form is just shorthand for the string typed block pattern and often will be a URL referencing the dynamic content or state of the artipoint.

```bnf
payload = quoted-string / typedBlock / value

typedBlock = type ":" ( jsonObject / quoted-string / value )
type       = "json" / "string" / "uri" / "data" ; extensible
jsonObject = "{" *CHAR "}" ; A balanced JSON object per RFC 8259

```

# Articulation Operator Taxonomy and Semantics

## General Principles

- Operators **do not** alter prior Artipoints; they define new relationships only.
- Operators are **deterministic**: given the same inputs, every conforming client MUST produce the same resulting coordination graph.
- Operators MAY be extended with future verbs, but unrecognized operators MUST be ignored without interpretation.

## Verb Operator Structure and Hierarchy

This table defines a set of verb-based operators used in the Artipoint grammar and classifies them by their semantic intent and structural behavior. Each verb enables fine-grained, semantically rich articulation of cognitive structure within the Cortex Layer, allowing both humans and agents to reason explicitly over relationships—from provenance chains to agenda construction to scoped replacements and incremental composition.

| Verb       | Type               | Orientation | **Hierarchical**? | Mutation? |
| ---------- | ------------------ | ----------- | ----------------- | --------- |
| references | semantic link      | LHS → RHS   | No                | No        |
| replaces   | semantic override  | LHS → RHS   | No                | Yes       |
| extracts   | derivation         | LHS ← RHS   | LHS is Child      | No        |
| groups     | flat collection    | LHS → {RHS} | No                | No        |
| assembles  | hierarchy builder  | LHS → {RHS} | LHS is Parent     | No        |
| promotes   | elevation          | LHS ← RHS   | RHS raised to LHS | Yes       |
| annotates  | weak subordinate   | LHS → RHS   | LHS is Child      | No        |
| supports   | strong subordinate | LHS → RHS   | LHS is Child      | No        |
| adds       | mutative inclusion | LHS += RHS  | RHS joins LHS     | No        |
| removes    | mutative exclusion | LHS -= RHS  | RHS leaves LHS    | Yes       |

Table Column Definitions:

- **Verb**: The name of the relationship operator, used in declarative Artipoint grammar statements.
- **Type**: A high-level classification of the relationship based on its cognitive or structural function.
- **Orientation**: Indicates the directional relationship between LHS and RHS, such as "LHS += RHS" for inclusion or "LHS ← RHS" for derivation. In instantiation and construction, the LHS is created with the specified relationship to existing RHS nodes. In connections, both nodes exist and the operator defines their new or updated edge relationship.
- **Hierarchical**: Specifies if/how the relationship implies a structural or hierarchical containment or dependency.
- **Mutation**: Indicates whether the use of this operator suggests a semantic replacement, displacement, or archival of the RHS item.

## Operator Semantics

This section defines the **normative semantics** of each operator in the Artipoint Grammar. The purpose is to ensure that all compliant implementations interpret operator intent consistently, even as application views and UI materializations may differ.

| Operator   | Output / Effect                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| references | Declares that the LHS Artipoint semantically refers to each RHS Artipoint. No containment, no order implied.                           |
| replaces   | Declares that the LHS Artipoint supersedes each RHS. RHS remains in history but MUST be masked in default views within the same scope. |
| extracts   | Declares each LHS as a sub-part derived from RHS. Directional dependency, but does not mask RHS.                                       |
| groups     | Declares LHS as a flat collection of RHS items. Order of RHS is not semantically meaningful.                                           |
| assembles  | Declares LHS as a structured whole composed of RHS items. Order of RHS MUST be preserved.                                              |
| promotes   | Declares LHS as a new elevated form of RHS. RHS MUST be masked in default views.                                                       |
| annotates  | Declares LHS as metadata or commentary on RHS. RHS remains active; LHS enriches but does not supersede.                                |
| supports   | Declares LHS as a required subcomponent enabling RHS. Dependency is directional.                                                       |
| adds       | Declares RHS is now included in the structure. Semantically equivalent to post-facto grouping.                                         |
| removes    | Declares RHS is excluded from the structure. RHS remains in history but MUST be excluded from current views.                           |

## Detailed Description of each Operator

### references

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

### replaces

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

### extracts

The LHS is a derived or excerpted sub-part of the RHS—typically in a child role. Used for derivation without supersession.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:20Z,
  [snippet,
     "Key passage from study",
     uri:"https://papers.ai/conference2025/paper42"]
  extracts {uuidDoc}
];
```

*A snippet is extracted from a full document.*

### groups

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

### assembles

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

### promotes

LHS is a new, elevated form of RHS, which it displaces. Used when transforming one structure into a new one that absorbs its content.

Example:

```
[uuidA, uuidAuthorIdentity, 2025-07-28T10:50Z,
  [stream, "Live Workstream", uuidOldPile] promotes {uuidOldPile} ];
```

*A stream promotes an earlier pile, replacing it as the active structure.*

### annotates

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

### supports

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
  ] supports {uuidStream} ];
```

*A scene supports a stream.*

### adds

LHS includes new RHS items into an existing structure or set. This is a post-facto inclusion.

Example:

```
uuidPile adds {uuidNewDoc};
```

*Adds a document to an existing pile.*

### removes

LHS excludes the RHS item from an existing structure, marking it as no longer active.

Example:

```
uuidPile removes {uuidOldDoc};
```

*Removes a document from a collection.*

# **Artipoint Annotation Attributes**

An attribute list is a collection of key-value metadata pairs. Each key may optionally include a class declaration using ::, and values can include any valid payload type—giving attributes the same expressive power as instantiations.

```
attribute-list = "(" keyValuePair *(separator keyValuePair) ")"
keyValuePair   = [class "::"] key attr-operator payload

```

Attributes are optional metadata attached to an Artipoint, particularly in annotations or bookmarks. Values can be strings or references to other Artipoints.

Operators modify the value using the following available semantic patterns:

| **Operator** | **Meaning**                                                                                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| +            | Appends to a value set. If no value exists, this becomes the first member of the set.                                                                                                         |
| -            | Removes a value from an existing set or by implication, the value is excluded from inclusion as a subtractive property from some sort of inherented value.                                    |
| :=           | Assigns an explicit value displacing any previous value or set. You can think of this as rebasing the value. An empty value string can be used to, in effect, clear and remove the attribute. |
| =            | Equivalence. The attribute is considered an alias or equivalent to whatever value is specified. In this way, it makes that attribute a macro of sorts to be referenced elsewhere.             |

Examples:

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

> **Note:** JSON payloads are marked with a json: prefix and must be balanced, well-formed objects per [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259). They are parsed externally from the formal ABNF grammar itself. As such, the grammar defines any arbitrary sequence of well formed UTF-8 encoded JSON.

# **Universally Unique Identiers (UUID's)**

All identifiers in ASCP (Artipoints, Identities, Channels, Streams, Spaces, etc.) **MUST** be represented as UUIDs following the rules below.

### **1. UUID Version Requirement**

UUID is ASCP are defined based on RFC-4122, but they **MUST** be generated as **UUID version 7** (time-ordered, Unix epoch milliseconds + randomness) as defined in \[IETF draft-peabody-dispatch-new-uuid-format].

```
550e8400-e29b-41d4-a716-446655440000
```

### **2. Encoding Patterns**

- In the ASCP Grammer when encoded in Hexadecimal form, they may be written with or without hyphens (both accepted)
- Hyphens are never present in binary forms such as in Binary Value Island (BVI) or in wire encodings.
- For debugging, logs, and CLI tools, implementations **SHOULD** support parsing and emitting the canonical **RFC-4122 hex form with hyphens**:

**Examples**:

- Raw bytes → 550e8400e29b41d4a716446655440000
- Hex with hyphens → 550e8400-e29b-41d4-a716-446655440000
- Hex without hyphens → 550e8400e29b41d4a716446655440000

### **5. Validation Rules**

- Version nibble (bits 48–51) **MUST** equal 0111 (v7).
- Variant bits **MUST** conform to RFC-4122 (10x).
- Parsers **MUST** accept hyphenated and non-hyphenated hex, Base64URL encoding may be accepted as well, but implementations MUST normalize internally to raw 16-byte representation.

# **Timestamps**

Timestamps in ASCP Artipoints represent the **articulation time**—the moment when the author (human or AI agent) made the cognitive decision to create, modify, or relate cognitive structure within the shared DAG. This is fundamentally different from content creation time, system processing time, or wall-clock synchronization, although the changes to the DAG *should* propagate into the ASCP channel log with minimal latency and typically within one second or less.

```bnf
timestamp = date "T" time [fraction] "Z"
date      = 4DIGIT "-" 2DIGIT "-" 2DIGIT
time      = 2DIGIT ":" 2DIGIT ":" 2DIGIT
fraction  = "." 1*DIGIT
```

- **RFC 3339 compliant** UTC-only timestamps (Z suffix mandatory)
- Fractional seconds SHOULD be limited to millisecond precision (3 digits maximum)
- Timestamps SHOULD be precise to at least the nearest second
- Timestamps SHOULD be accurate to within 100 milliseconds of actual wall-clock time when the system is operating online
- Systems MAY tolerate wall-clock drift of up to a few seconds when operating offline or with limited network connectivity
- Hours restricted to 00-23 (following RFC 3339's clarification over ISO 8601)
- No timezone offsets permitted in this grammar (reserved for future extensions)
- Case-insensitive: both "T"/"Z" and "t"/"z" are valid per RFC 3339

# **Provenance and Authorship**

```bnf
author = uuidReference

```

Represents the author, signer, and generator of the Artipoint.

- **uuidReference**: The RFC-4122 standard UUID of an IdentityArtipoint of the human or agent generating the Artipoint.

Cryptographic integrity, privacy, and audience scoping are all handled by **ASCP channel encoding**, not the grammar.

# **Strings and Escaping**

Strings must be double-quoted ("...") and support the full JSON standard string escaping mechanisms per RFC 8259:

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

## **Collections and Structures**

There is no nesting of expressions within expressions. Instead, **collections** (e.g. streams, piles, lists) are formed using the construction pattern with a bookmark that refers to an external document or resource.

The **Artipoint's own UUID** serves as the persistent ID of that structure. Metadata lives in the referenced document; relations live in the DAG.

## **Identity, Canonicalization, and Channels**

- The UUID is the **immutable, canonical identity** of each Artipoint. It must be universally unique per RFC-4122 and deterministic (typically generated by a higher-layer tool or content hash).
- Artipoints are **flat, line-based records** that can be serialized, signed, and version-controlled.
- Canonicalization rules (e.g., key ordering in attributes, whitespace trimming) are external to the grammar and defined by ASCP transport layers.
- **Recipients** and visibility are not embedded in the Artipoint itself—they are determined by the **ASCP channel** in which the Artipoint is distributed.

## **Examples (Future Section)**

Coming soon: a set of validated Artipoint examples including bookmarks, annotations, equivalence chains, collections, and agent-generated updates.

## **Summary**

This grammar defines a minimal but powerful **cognitive substrate**—a way to persistently structure thought, reasoning, and coordination into a universally interpretable form. It is:

- Declarative, immutable, and traceable
- Friendly to both humans and agents
- Built to support persistent shared cognition across systems
- Designed for cryptographically scoped, decentralized distribution via ASCP
- Organized as **semantic statements**, each explicitly terminated with a ; for clarity and structure

It is the **syntax layer of the Cortex**—the foundation of durable, auditable collaboration between humans and intelligent systems.

## Future Considerations:

In the future we could/should add sections covering:

- **Extension Points**: Formal rules for extending the operator vocabulary
- **Unknown Element Handling**: How parsers should behave with unrecognized operators
- **Lifecycle Management**: A deprecation model with clear phases and timelines

# Appendix 1: The Formal ASCP Grammar in ABNF

```ebnf
; Artipoint Grammar Definition (ABNF)
; -----------------------------------
; This grammar captures the syntax of Artipoints for structured,
; immutable cognitive statements within the Cortex Layer
; of the ASCP (Agent Shared Cognition Protocol).

; Core definitions cover:
; - Artipoint structure
; - Author attribution
; - Operators and articulation patterns
; - Referencing, typing and composition

; Design Principles:
; - Everything is an Artipoint
; - Immutable, addressable units forming a DAG
; - Self-similar and recursive
; - No reserved keywords (types are extensible)
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

; ----- Artipoint -----
artipoint      = "[" OWS uuid separator author separator timestamp
                     [ separator expression ] OWS "]"
expression     = instantiation / annotation / connection / construction

; ----- Expressions -----
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
payload        = OWS ( quoted-string / value / typedBlock ) OWS
typedBlock     = payload-type ":" OWS ( jsonValue / quoted-string / value )
payload-type   = "json" / "string" / "uri" / "data" / "uuid" / future-type
jsonValue      = "{" *string-char "}"  ; Externally Parsed as RFC 8259 JSON
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
artipoint-type = ALPHA *(ALPHA / DIGIT / ".")
class          = ALPHA *(ALPHA / DIGIT / ".")
key            = ALPHA *(ALPHA / DIGIT / "-" / "_")
label          = quoted-string

; ----- Values can be integers or byte strings -----
value          = integer / hex-number / bin-number

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
safe-char      = %x20-21 / %x23-5B / %x5D-10FFFF ; Full Unicode support
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

```

# Appendix 2: ASCP Grammar Encodings

As formally specified in Appendix 1, the Layer-2 grammar is defined as **UTF-8 text** for human-friendly authoring and inspection. To reduce verbosity in frequently occurring constructs (UUIDs, timestamps, keywords, operators), ASCP defines compact **encoding conventions**. These allow binary or symbolic forms to appear directly in the grammar without changing semantics.

When presenting the grammar to users, it should always be rendered per Appendix 1 specifications, however when encoding for Layer 1 channel processing, the full suite of compaction options SHOULD be used.

The two key ways of achieving more compact encoding are through **Binary Value Islands (BVI)** and the **Default Symbol Dictionary (DSD)**, which are described in the following sections.

## Binary Value Islands (BVI)

For efficiency, ASCP reserves **0x1F** as a **binary island introducer**. A BVI carries a compact header so a parser knows exactly how many bytes to consume.

```
0x1F <valueType> [<ULEB128-length>] <payloadBytes>
```

Where:

- **0x1F** = Binary Value Island introducer (reserved byte)
- **valueType** = Single byte identifying the data type (0x01-0xFF)
- **ULEB128-length** = Variable-length encoding of payload byte count, but not used by all types as some types have fixed or self-encoded lengths.
- **payloadBytes** = Raw binary data of the specified length

### **Standard ValueTypes**

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

### **ULEB128 Length Encoding**

The segment length uses ULEB128 (Unsigned Little Endian Base 128), a variable-length integer encoding widely adopted in binary protocols including DWARF debugging information, WebAssembly, and Protocol Buffers. ULEB128 represents integers by breaking them into 7-bit chunks, with each byte's most significant bit serving as a continuation flag: 1 indicates more bytes follow, while 0 marks the final byte. The encoding is little-endian at the byte level, meaning the least significant 7-bit group is transmitted first.

For example, the value 1 encodes as a single byte 0x01, while 128 requires two bytes: 0x80 0x01 (the first byte's high bit indicates continuation, its low 7 bits represent zero, and the second byte contains the remaining value 1). The value 624485 encodes as 0xE5 0x8E 0x26, demonstrating the format's efficiency for larger integers.

In the BVI, the ULEB128 length indicates the number of bytes to follow as the payload.

### **UUID Encoding (Type = 0x20)**

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

### **UUID Set Encoding (Type = 0x02)**

Encodes a collection of UUIDs from the textual { uuid1, uuid2, ..., uuidN } syntax into a compact binary representation.

**Textual Form**

```plaintext
{ uuid1, uuid2, ..., uuidN }

```

**Binary Format**

```plaintext
0x1F | 0x02 | <ULEB128-length> | <UUID-bytes>

```

Where:

- 0x1F = Binary Value Island (BVI) introducer
- 0x02 = UUID Set type identifier
- \<ULEB128-length> = Number of bytes in the UUID data (always multiple of 16)
- \<UUID-bytes> = Concatenated 16-byte UUIDs in sequence

**Encoding Process**

1. Count the UUIDs in the set: N
2. Calculate byte length: N × 16
3. Encode length as ULEB128
4. Concatenate all UUID bytes in order

**Examples**

- **3 UUIDs:** `0x1F 0x02 0x30` followed by 48 UUID bytes
- **1 UUID:** `0x1F 0x02 0x10` followed by 16 UUID bytes
- **Empty set:** encoded as `0x1F 0x02 0x00`

**Semantic Properties**

- **Order preserved:** UUIDs maintain their textual sequence
- **Duplicates allowed:** Consumer implementations MAY deduplicate
- **Validation:** Byte count MUST be divisible by 16

**Single UUID vs UUID Set Distinction**

- A `0x20` single UUID is always a bare scalar value with no braces.
- A `0x02` UUID Set is always brace-delimited in textual form and encoded as a compact block in binary form.
- A `0x20` UUID is **not** equivalent to a singleton `0x02` UUID Set—braces are implied only in the set case.
- Implementations MUST preserve this distinction.

**Alternate Representation of Sets (with braces)**

- Sets may alternatively remain in the **textual grammar form** with braces { … } and commas separating elements.
- Inside this alternate form, each element MAY be encoded individually as a `0x20` UUID BVI or left as a textual UUID.
- This form is valid but **less efficient**, since braces and commas remain part of the serialized form rather than collapsing into the compact 0x02 binary block.
- Importantly, such brace-preserving encodings are **not the same as a** **0x02** **UUID Set**—they are textual-set representations with per-element encodings.

**Encoding Examples:**

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

**Parser Requirements**

- MUST accept both canonical 0x02 binary form and brace-preserving textual form.
- MUST NOT collapse a 0x20 single UUID without braces into a singleton 0x02 UUID Set or vice versa.
- MUST consume the surrounding braces and seperating commas and whitespace within the braces.
- SHOULD consume leading and trailing whitespace outside the braces as the 0x02 binary form is a full token unto itself.

This distinction ensures consistent interpretation of singleton UUIDs, compact binary sets, and alternate brace-preserving encodings.

### Binary Values (Type = 0x00)

Encodes arbitrary binary data from hex or binary string representations into direct binary form.

**Textual Forms**

```
0x48656c6c6f        ; hex representation
0b0100100001100101  ; binary representation
```

**Binary Format**

```
0x1F | 0x00 | <ULEB128-length> | <raw-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x00` = Binary values type identifier
- `<ULEB128-length>` = Number of bytes in the payload
- `<raw-bytes>` = Direct binary data

**Encoding Process**

1. Parse hex (`0x...`) or binary (`0b...`) string representation
2. Convert to raw bytes (hex: 2 chars → 1 byte, binary: 8 bits → 1 byte)
3. Calculate byte length and encode as ULEB128
4. Append raw binary data

**Examples**

- **Hex input:** `0x48656c6c6f` → `0x1F 0x00 0x05 48656c6c6f`
- **Binary input:** `0b0100100001100101` → `0x1F 0x00 0x02 4865`
- **Empty data:** `0x` → `0x1F 0x00 0x00`

**Validation Requirements**

- **Hex format:** Must contain even number of hex digits (full bytes)
- **Binary format:** Must contain multiple of 8 bits (full bytes)
- **Case handling:** Accept both upper and lowercase hex digits
- **Output format:** Always store as raw binary data

### UTF-8 Strings (Type = 0x03)

Encodes UTF-8 text strings without requiring JSON-style escaping for common characters.

**Textual Form**

```plaintext
"Hello, 世界! 🌍"
```

**Binary Format**

```plaintext
0x1F | 0x03 | <ULEB128-length> | <utf8-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x03` = UTF-8 string type identifier
- `<ULEB128-length>` = Number of bytes in UTF-8 encoding
- `<utf8-bytes>` = Raw UTF-8 encoded string data

**Encoding Process**

*Encoders MUST first unescape JSON-style sequences into fully UTF-8 literal stream before encoding as a BVI UTF-8 escaped sequence*

1. Remove surrounding quotes from textual representation
2. Fill the UTF-8 bytes payload with the expanded UTF-8 stream.
3. Calculate byte length and encode into ULEB128
4. Append UTF-8 byte sequence

**Examples**

- **ASCII:** `"Hello"` → `0x1F 0x03 0x05` + `48656c6c6f`
- **Unicode:** `"世界"` → `0x1F 0x03 0x06` + `e4b896e7958c`
- **Empty string:** `""` → `0x1F 0x03 0x00`

**Character Handling**

- **No escaping required:** Direct UTF-8 encoding of all valid Unicode characters
- **Newlines and tabs:** Encoded directly as UTF-8 bytes
- **Null bytes:** Not permitted in string content
- **Validation:** Must be valid UTF-8 sequence

### **Timestamps (Types = 0x21, 0x22)**

Encodes RFC 3339 compliant UTC timestamps using variable-width binary formats optimized for typical precision requirements.

**Textual Form**

```asciidoc
2025-08-18T12:34:56Z           ; no fractional seconds
2025-08-18T12:34:56.789Z       ; millisecond precision  
2025-08-18T12:34:56.123456Z    ; microsecond precision
```

**Binary Format**

```asciidoc
0x1F | 0x21 | <time32-bytes>
0x1F | 0x22 | <time64-bytes>
```

Where:

- `0x1F` = Binary Value Island (BVI) introducer
- `0x21` or `0x22` = Timestamp type identifier (time32) or (time64) follows
- `<time-bytes>` = Encoded timestamp data in big-endian format

**Format Selection**

- **time32 (4 bytes):** No fractional seconds AND fits in 32-bit Unix timestamp range (1970-2106) for type code 0x21
- **time64 (8 bytes):** Has fractional seconds OR exceeds 32-bit range (extends to year 2554) for type code 0x22

**Encoding Process**

1. Parse RFC 3339 timestamp and extract Unix epoch seconds + fractional component
2. Validate fractional seconds precision (RFC 3339 supports arbitrary precision via `time-secfrac = "." 1*DIGIT`)
3. Select format based on presence of fractional seconds and timestamp range
4. Encode using appropriate binary representation

**Examples**

- **Basic timestamp:** `2025-08-18T12:34:56Z` → `0x1F 0x21 0x66C7B310`  
  *(1,723,988,096 seconds since Unix epoch, encoded as 4-byte time32)*
- **With milliseconds:** `2025-08-18T12:34:56.789Z` → `0x1F 0x22 0x2F0A83DE66C7B310`  
  *(789,000,000 nanoseconds in upper 30 bits, same epoch seconds in lower 34 bits)*
- **With microseconds:** `2025-08-18T12:34:56.123456Z` → `0x1F 0x22 0x1E74DF8066C7B310`  
  *(123,456,000 nanoseconds in upper 30 bits, same epoch seconds in lower 34 bits)*

**Format Specifications**

- **time32:** Direct `uint32` seconds since Unix epoch (1970-01-01T00:00:00Z)
- **time64:** `uint64` = (fractional\_ns << 34) | seconds, where both formats share the same Unix epoch base and upper 30 bits encode nanoseconds (0-999,999,999)

**RFC 3339 Compliance**

- UTC timezone only (Z suffix mandatory, no timezone offsets)
- Hours restricted to 00-23 (RFC 3339 clarification over ISO 8601)
- Case-insensitive 'T' and 'Z' characters accepted per RFC 3339
- Leap seconds handled per RFC 3339 specification (time-second may be "60")

## Default Symbol Dictionary (DSD v1)

Frequent grammar tokens can be replaced by a **two-byte escape** to reduce verbosity in commonly occurring constructs like verbs, operators, and type prefixes. This compression mechanism significantly reduces the size of encoded Artipoints while maintaining full semantic equivalence with the textual grammar.

### **Escape Mechanism**

```
0x1E <code>
```

Where `0x1E` is the reserved **symbol dictionary escape** introducer and `<code>` is a single byte that selects a predefined token from the Default Symbol Dictionary.

### **How It Works**

1. **Encoding:** When serializing Artipoints, producers can replace any occurrence of a dictionary token with its two-byte escape sequence. The encoder SHOULD consume all leading and trailing whitespace to the adjacent language tokens to minimize encoding size.
2. **Decoding:** Parsers encountering `0x1E` read the following byte as a dictionary lookup and substitute the corresponding token. If the parsing process is outputting text to the ABNF grammar, the decoder MUST add any required or readability desired leading or trailing whitespace to maintain the tokens of the language.
3. Decoders **MUST** accept any combination of encodings with or without dictionary encodings.
4. **Semantic equivalence:** The escaped form is functionally identical to the full textual token with leading and trailing whitespace.
5. **Optional optimization:** Encoder implementations MAY choose when to apply DSD substitutions based on size/speed/readability tradeoffs.

### **Example Usage**

```
uuidA 0x1E 0x00 {uuidB}    ; "references" compressed
uuidC 0x1E 0x40 {data}     ; "json:" prefix compressed
```

### **Dictionary Structure**

The DSD v1 organizes tokens into functional ranges to enable efficient lookup and future extension:

**Verbs (0x00–0x1F)**

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

**Artipoint Types (0x20–0x3F)**

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

**Typed-Block Prefixes (0x40–0x5F)**

| Code      | Token                         |
| --------- | ----------------------------- |
| 0x40      | json:                         |
| 0x41      | string:                       |
| 0x42      | uri:                          |
| 0x43      | data:                         |
| 0x44      | uuid:                         |
| 0x45–0x4F | Reserved typed-block prefixes |

**Common Attributes (0x50–0x7F)**

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

**Vendor/Extension Range (Future)**

| Range     | Purpose                       |
| --------- | ----------------------------- |
| 0x80–0xFF | Vendor/extension dictionaries |

## Compliance

1. **Signing:** At Layer-1 Channel encoding, producers sign the payload as emitted. No re-serialization or normalization.
2. **Reserved bytes:** 0x1E and 0x1F are reserved as escape introducers and MUST NOT appear in textual grammar regions. Producers MUST NOT emit these bytes outside of BVIs/DSD; decoders MUST error if encountered in textual regions thereby dropping the current articulation statement.
3. **NUL Characters**: U+0000 is prohibited; encoders MUST error if present; decoders MUST reject.”
4. **Extensibility:** Unknown codes in the reserved/vendor ranges MUST cause parse error unless explicitly declared in a profile.
5. **Round-tripping:** Tools MAY expand escapes back to full text for readability, but any rewrite produces a distinct artifact with a new signature.

## Appendix 3: Validation and Error Handling (Normative) 

This section defines **accept‑and‑record** behavior for an immutable log. Once an articulation enters a channel log, it is **never altered or removed** by protocol action. All implementations MUST converge on identical handling by following the rules below.

## Core Principles

- **Immutability:** Clients MUST store every received, well‑formed Layer‑1 envelope exactly as emitted. No rewriting, normalization, or redaction.
- **Report‑and‑proceed:** Validation errors are **reported** via diagnostics; evaluation/materialization proceeds with deterministic **no‑op defaults** where applicable.
- **Determinism:** Given the same sequence, all conforming implementations MUST produce identical DAG effects (including when effects are intentionally **no‑op**).

## Validation Phases

1. **Envelope Validation (Layer‑1):** Signature, integrity, audience/recipient checks.
   - **FAIL** ⇒ The envelope MUST NOT be passed to Layer-2 and should be marked invalid in the local log.
   - **PASS** ⇒ Proceed to grammar validation.
2. **Grammar Validation (Layer‑2):** Syntactic correctness per ABNF (tokenization, UUID/timestamp lexemes, statement structure).
   - **FAIL** ⇒ Report as invalid syntax reporting any parseable uuid, author, timestamp. DAG effect = **no‑op**.
   - **PASS** ⇒ Proceed to semantic validation.
3. **Semantic Validation (Layer‑2):** Operator, reference, and attribute semantics.
   - **FAIL (non‑critical)** ⇒ Admit and index fully; apply DAG effect per decision table (often **no‑op**); attach diagnostic code(s).

## Deterministic Handling Table

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

### Masking & Supersession Determinism

- replaces and promotes **mask** their RHS **only within the same applicable scope** (e.g., same parent collection/structure as established by accompanying relations or application context). If scope cannot be resolved (E7), **no mask occurs**.
- removes excludes RHS from the addressed structure only; history is unaffected.

### Diagnostics (SHOULD)

Implementations SHOULD expose a diagnostics feed with fields like { uuid, envelope\_id, phase, code, details, first\_seen\_at }. Clients MAY surface these to users/agents.

### Forward Compatibility

- Unknown verbs/types/codes MUST NOT block ingestion. They are preserved for future interpreters and evaluated as **no‑op** today per E1/E2/E10.
- Profiles MAY further constrain acceptance (e.g., disallow certain typed blocks) but MUST still follow these deterministic outcomes.

### Conformance

Implementations MUST demonstrate, via test vectors, that given identical articulation sequences they yield identical DAG effects and identical diagnostics for all cases E1–E10.