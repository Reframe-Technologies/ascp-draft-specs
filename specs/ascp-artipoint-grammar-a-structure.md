# **ASCP Artipoint Grammar: A Structure for Shared Cognition**

**Version:** Draft 0.2 - August 2025

**Scope:** Grammar for encoding immutable cognitive decisions as addressable statements within persistent, shared cognitive substrate—enabling auditable articulation work between humans and AI agents in the ASCP Cortex Layer

## **Background**

The Artipoint Grammar emerged in response to a fundamental gap in human-human and human-AI collaboration: the absence of durable, interpretable, and shared cognitive structure for coordination work. Most contemporary systems operate on ephemeral exchanges—messages, prompts, or transient memory updates—rather than persistent, structured meaning that makes collaborative work possible. What the web standardized for documents and APIs, ASCP seeks to standardize for **contextual coordination**.

This challenge connects directly to what the Computer-Supported Cooperative Work (CSCW) literature calls **"articulation work"**—the essential but often invisible work that cooperating individuals must perform to partition work into units, divide it amongst themselves, and reintegrate it after completion. Articulation work is all the work around cooperative work that makes it possible—a secondary work process that enables primary collaborative activities. In human-AI teams, this articulation work becomes even more critical as it must bridge fundamentally different cognitive architectures while maintaining shared understanding over time.

Traditional collaboration systems fail to capture the **auditable cognitive decisions** that constitute this articulation work. Instead of preserving the reasoning about *why* work is organized in particular ways, *how* pieces relate to each other, and *what* dependencies exist, they focus on content exchange. ASCP recognizes that effective collaboration requires a **persistent, shared cognitive substrate**—the immutable structure of how work gets organized, connected, and reasoned about.

We studied prior efforts in provenance modeling (W3C PROV), distributed content graphs (Merkle DAGs, IPFS), semantic triple stores (RDF), and collaborative data models (CRDTs). While each informed our thinking, none fully addressed the dual needs of:

1. **Expressing structured cognitive intent** in a form that is both machine-parseable and human-readable
2. **Supporting co-evolution of meaning** across humans and agents, while preserving an immutable, auditable record of articulation decisions

Where RDF emphasizes universal description and Merkle DAGs emphasize verifiability, the Artipoint Grammar emphasizes **semantic articulation**—the ability to express and preserve the shape of collaborative reasoning itself. This grammar sits between language and logic: minimal enough to be used conversationally, yet structured enough to support collaborative reasoning and interoperable memory at scale.

The result is a system that captures not just the final state of collaborative work, but the complete history of articulation decisions that built that shared understanding—creating a foundation for true shared cognition between humans and intelligent agents.

## **Overview**

The **Artipoint Grammar** defines a minimalist syntax for representing cognitive atoms—called **Artipoints**—as immutable, addressable statements. Multiple Artipoints form an **articulation sequence**, where each Artipoint is a semicolon-terminated line that captures a complete, atomic declaration of intent or structure.

The grammar is intentionally flat: **there is no nesting** within expressions. All composition happens via references to previously defined Artipoints using their UUID, creating a Directed Acyclic Graph (DAG) of shared cognition. This approach enables:

- Machine- and human-readable cognitive statements
- Individual referenceability and auditability
- Incremental composition into complex structures
- Stream-compatible, line-by-line processing
- Decentralized distribution via ASCP channels

### Structure vs. Content: The Reference Principle

A critical design insight: **Artipoints capture cognitive structure, not dynamic content itself**. The grammar tracks the immutable decisions, connections, and articulations that shape how work gets organized—the persistent cognitive substrate underlying collaboration. When an Artipoint needs to incorporate evolving content—documents, databases, real-time streams—that mutable state lives externally and is referenced through URIs in the payload of instantiated Artipoint.

This is the **"bookmark pattern"**: rather than embedding a 50-page research paper directly in an Artipoint, you create a cognitive statement like "this paper is highly relevant to our project" with a URI pointing to the external document. The paper may be updated, moved, or versioned, but the cognitive decision—the structural relationship between paper and project—remains immutable and auditable within the DAG.

This enables teams to work within a shared cognitive architecture where the structural relationships are universally visible and persistent, while the actual content remains dynamic and appropriately scoped.

The core unit follows this pattern:

```
[uuid, source, timestamp, expression];
```

Where the optional expression enables four fundamental articulation patterns: instantiation (creating new cognitive atoms), annotation (enriching existing ones), connection (linking atoms), and construction (creating and linking simultaneously).

## **Structural Model**

**The Foundation: Cognitive Structure, Not Content**

ASCP's core innovation lies in recognizing that effective collaboration requires a **persistent, shared cognitive substrate**—the immutable structure of how work gets organized, connected, and reasoned about. This is fundamentally different from the content itself.

Think of ASCP as tracking the **cognitive scaffolding** that teams use to coordinate their thinking: the decisions about what's relevant, how pieces relate, which tasks depend on others, what insights emerge from analysis. This scaffolding persists and remains auditable even as the underlying documents, data, and deliverables evolve rapidly.

**The Immutability Principle**

Each Artipoint represents a permanent, auditable **cognitive decision**—a moment when someone (human or AI) made a structured statement about relationships, relevance, or reasoning. These decisions accumulate into a DAG that captures not just the final state of collaborative work, but the complete history of how that shared understanding was built.

When an Artipoint needs to incorporate dynamic content—documents that get edited, databases that update, real-time streams—that mutable state lives externally and is referenced through URIs in the payload. This is the **"bookmark pattern"**: rather than embedding content directly, we create cognitive statements *about* that content.

**Example**: When an AI agent discovers a relevant research paper, it doesn't embed the paper's text:

```clike
[a1b2c3d4-e5f6-7890-abcd-ef1234567890,
     agent@research.system,
     2024-03-15T14:30:22.123Z,
 [ paper,
  "Attention Mechanisms in Transformer Models",  
  uri:"https://arxiv.org/abs/2023.12345" ]
];
```

This creates a permanent record: "this agent determined this paper was highly relevant to this project at this moment." The paper may be updated or moved, but the cognitive judgment—the structural relationship between paper and project—remains immutable and traceable.

**Enabling Shared Cognitive Architecture**

This design enables teams to work within a common cognitive structure while maintaining appropriate privacy and scope. Team members contribute to the same underlying DAG—the shared "tree" of how work is organized—while potentially having private branches and nodes that others cannot see. Everyone benefits from the structural coherence without sacrificing information security or cognitive autonomy.

The result is true **shared cognition**: not just exchanging messages or files, but building and evolving a persistent, jointly-accessible model of how the work itself is structured and interconnected.

### **Artipoint Format**

The core unit of an Artipoint in the grammar is as follows:

```bnf
artipoint = "[" uuid "," source "," timestamp ["," expression ] "]"

```

**Fields:**

- **uuid**: RFC-4122 compliant universally unique identifier for this Artipoint
- **source**: URI or addr-spec (email-style) indicating authorship or origination
- **timestamp**: UTC time in ISO 8601 format, with optional milliseconds
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

## **Articulation Patterns**

The true power of Artipoints emerges through **composition**—how individual cognitive atoms combine to form complex, interconnected structures within the DAG. Each of the four articulation patterns defined above serves a distinct role in building and evolving this shared cognitive graph:

- **Instantiation** creates new nodes—the foundational cognitive atoms that represent tasks, documents, insights, or any meaningful unit of thought
- **Connection** establishes directed edges between existing nodes, encoding relationships like dependencies, associations, or semantic links
- **Construction** combines instantiation and connection in a single statement, simultaneously creating a new node and linking it to existing context
- **Annotation** enriches existing nodes with metadata, state updates, or additional semantic information without changing the graph's topology

Together, these patterns enable incremental, collaborative construction of knowledge structures. An AI agent might instantiate a research task, connect it to relevant prior work, while a human collaborator annotates it with priority metadata. Each action adds to the permanent, auditable record of how the cognitive structure evolved—creating not just the final graph, but a complete history of the reasoning process that built it.

This compositional approach means that complex cognitive artifacts—project plans, research syntheses, collaborative decisions—emerge naturally from the accumulation of simple, atomic statements over time.

The following sub-sections detail each one of distinct kind of articulations one can make.

### **Instantiation**

```bnf
instantiation = "[" type "," label "," payload "]" [ "." attribute-list ]

```

Declares a typed and labeled unit of meaning with an embedded or referenced payload.

- **type**: A semantic label like "task", "doc", "stream", etc.
- **label**: A human-readable title or caption
- **payload**: The main content—can be a typed embedded inline structure (ie: typedBlock), or an ordinary UTF-8 string which would typically be a URL.
- **attribute-list**: Optional semantic metadata (see below)

### **Annotation**

```bnf
annotation = uuidReference "." attribute-list

```

Applies new attributes to an existing Artipoint. Used for modification, enrichment, or contextual update to prior context.

### **Connection**

```bnf
connection = uuidReference verb-operator uuidSet

```

Establishes a semantic relationship between a source Artipoint and a set of targets via an operator.

### **Construction**

```bnf
construction = instantiation verb-operator uuidSet

```

Combines a newly declared instantiation with one or more related Artipoints to define collections or aggregate forms like piles or streams.

## **Payloads and Typed Blocks**

The payload field, used both in instantiations and attribute values, accepts the following:

- A quoted-string
- A typed block (e.g., embedded JSON or structured string)

The quoted-string form is just shorthand for the string typed block pattern and often will be a URL referencing the dynamic content or state of the artipoint.

```bnf
payload = quoted-string / typedBlock

typedBlock = prefix ":" ( jsonObject / quoted-string )
prefix     = "json" / "string" / "uri" / "data" ; extensible
jsonObject = "{" *CHAR "}" ; A balanced JSON object per RFC 8259

```

## Articulation Operator Taxonomy and Semantics

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

## Verb Operator Semantics and Examples

### references

Indicates that the LHS item semantically refers to or is informed by the RHS item, without implying dependency or structural inclusion.

**Example:**

```
[uuidA, alice@notes, 2025-07-28T10:00Z,
  [decision,
     "Proceed with option B",
     uri:"https://workspace/docs/discussion-summary"]
  references {uuidB}
];
```

*A decision references a prior discussion point.*

### replaces

Indicates semantic supersession: the LHS is intended to supersede or override the RHS. The RHS remains in history but is no longer active.

**Example:**

```
[uuidA, bob@drafts, 2025-07-28T10:10Z,
  [document,
     "Final Draft",
     uri:"/docs/final.pdf"]
  replaces {uuidOldDraft}
];
```

*A new document replaces an older version.*

### extracts

The LHS is a derived or excerpted sub-part of the RHS—typically in a child role. Used for derivation without supersession.

**Example:**

```
[uuidA, agent@system, 2025-07-28T10:20Z,
  [snippet,
     "Key passage from study",
     uri:"https://papers.ai/conference2025/paper42"]
  extracts {uuidDoc}
];
```

*A snippet is extracted from a full document.*

### groups

LHS is a flat set or collection of RHS items, such as a pile or unordered list.

**Example:**

```
[uuidA, chris@team, 2025-07-28T10:30Z,
  [pile,
     "Articles for Review",
     uri:"https://mycompany.com/pileState.pile"]
  groups {uuid1, uuid2, uuid3}
];
```

*A pile groups together several items without implying order or structure.*

### assembles

LHS is constructed as a structured whole from multiple RHS components. Hierarchical containment is implied.

**Example:**

```
[uuidA, dora@ops, 2025-07-28T10:40Z,
  [agenda,
     "Monday Sync Agenda",
     uri:"https://calendar.company.com/agenda/monday-sync"]
  assembles {uuidTopic1, uuidTopic2}
];
```

*An agenda is assembled from individual discussion points.*

### promotes

LHS is a new, elevated form of RHS, which it displaces. Used when transforming one structure into a new one that absorbs its content.

**Example:**

```
[uuidA, elena@cortex, 2025-07-28T10:50Z,
  [stream, "Live Workstream", uuidOldPile] promotes {uuidOldPile} ];
```

*A stream promotes an earlier pile, replacing it as the active structure.*

### annotates

LHS provides a subordinate comment, note, or enrichment on the RHS item. It does not structurally alter the RHS.

**Example:**

```
[uuidA, finn@review, 2025-07-28T11:00Z,
  [comment,
     "Needs clarification",
     uri:"https://workspace/docs/proposal-draft"]
  annotates {uuidTarget}
];
```

*A comment annotates a document or statement.*

### supports

LHS is a functional subcomponent or enabler of the RHS. Use for required substructure or infrastructural dependency.

**Example:**

```
[uuidA, gina@infra, 2025-07-28T11:10Z,
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

**Example:**

```
uuidPile adds {uuidNewDoc};
```

*Adds a document to an existing pile.*

### removes

LHS excludes the RHS item from an existing structure, marking it as no longer active.

**Example:**

```
uuidPile removes {uuidOldDoc};
```

*Removes a document from a collection.*

## **Attributes**

An attribute list is a collection of key-value metadata pairs. Each key may optionally include a type declaration using ::, and values can include any valid payload type—giving attributes the same expressive power as instantiations.

```
attribute-list = "(" keyValuePair *(separator keyValuePair) ")"
keyValuePair   = [type "::"] key attr-operator payload

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

```
```

> **Note:** JSON payloads are marked with a json: prefix and must be balanced, well-formed objects per [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259). They are parsed externally from the grammar.

## **Timestamp**

```bnf
timestamp = date "T" time [fraction] "Z"

```

- UTC only (Z suffix mandatory)
- Supports millisecond precision via .sss optional fraction
- Used for ordering, traceability, and replay
- No timezone offsets permitted in this grammar (reserved for future)

## **Provenance and Authorship**

```bnf
source = URI / addr-spec

```

Represents the author, signer, or generator of the Artipoint.

- **addr-spec**: human or agent (e.g., <jeff@reframe.systems>)
- **URI**: agent endpoint, system address or even URL (e.g., did:example:agent1)

Cryptographic integrity, privacy, and audience scoping are all handled by **ASCP channel encoding**, not the grammar.

## **Strings and Escaping**

Strings must be double-quoted ("...") and support the following escapes:

```bnf
escaped = "\\" ( "\"" / "\\" / "n" / "r" / "t" )
safe-char = %x20-21 / %x23-5B / %x5D-7E

```

Example:

```
"This is a quoted string with a newline\n and a quote: \""

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

## Appendix: The Formal ASCP Grammar in ABNF

```bnf
; Artipoint Grammar Definition (ABNF)
; -----------------------------------
; This grammar captures the syntax of Artipoints for structured, immutable cognitive statements
; within the Cortex Layer of the ASCP (Agent Shared Cognition Protocol).

; Core definitions will cover:
; - Artipoint structure
; - Source attribution
; - Operators and articulation patterns
; - Referencing and typing
; - Nesting and composition

; Design Principles:
; - Everything is an Artipoint
; - Immutable, addressable units forming a DAG
; - Self-similar and recursive
; - No reserved keywords (types are extensible)
; - Minimalist but expressive operators (+, -, <, >, ::, :=)
; - Declarative, not imperative
; - Traceable provenance
; - No nesting; all composition is by reference only.
; - UUIDs are canonical ID for a statement in the grammar;
; - Cryptographic signing, hashing, handled by ASCP channel encoding.
; - Routing/visibility is scoped by the ASCP channel.

; The basis of statements in the grammar
articulation-sequence   = 1*(artipoint-statement)
articulation-statement  = artipoint ";" [ end-of-line ]

; The formally defined artipoint itself
artipoint      = "[" uuid "," source "," timestamp ["," expression ] "]"

; Each artipoint has an optional expression
expression     = instantiation / annotation / connection / construction

; There are 4 types of Artipoint expressions in the grammar
instantiation  = "[" type "," label "," payload "]" [ "." attribute-list ]
annotation     = uuidReference "." attribute-list
connection     = uuidReference verb-operator uuidSet
construction   = instantiation verb-operator uuidSet

; Verb-based operator for connection and construction patterns
verb-operator = "references" / "replaces" / "extracts" / 
                "groups" / "assembles" / "promotes" /
                "annotates" / "supports" / "adds" / "removes"

; The payload value of a new artipoint or attribute
payload        = quoted-string / typedBlock

; For linking uuid(s) in the connection or construction
uuidSet        = "{" (uuidReference) *(separator uuidReference) "}"

; Attributes themselves can have semantic types and complex payloads
attribute-list = "(" keyValuePair *(separator keyValuePair) ")"
keyValuePair   = [type "::"] key attr-operator ( payload / uuidReference )
attr-operator  = "+" / "-" / "=" / ":="

; Values or payloads that are typed blocks
typedBlock     = prefix ":" ( jsonObject / quoted-string )
prefix         = "json" / "string" / "uri" / "data"
jsonObject     = "{" *CHAR "}" ; A balanced JSON object per RFC 8259

; An ASCP uuidReference is a reference to an existing Artipont uuid
uuidReference  = uuid

; A source is either a URI/URL or an email-style address spec
source         = URI / addr-spec

; All the logistical types supporting the grammar
timestamp      = date "T" time [fraction] "Z"

date           = 4DIGIT "-" 2DIGIT "-" 2DIGIT
time           = 2DIGIT ":" 2DIGIT ":" 2DIGIT
fraction       = "." 1*3DIGIT
offset         = ("+" / "-") 2DIGIT ":" 2DIGIT ; reserved
end-of-line    = CRLF / LF / CR
separator      = ","

type           = ALPHA *(ALPHA / DIGIT / ".")
key            = ALPHA *(ALPHA / DIGIT / "-" / "_")
label          = quoted-string

quoted-string  = DQUOTE *(escaped / safe-char) DQUOTE
escaped        = "\\" ("\"" / "\\" / "n" / "r" / "t")
safe-char      = %x20-21 / %x23-5B / %x5D-7E

; address specification based on RFC-5322
addr-spec   = local-part "@" domain
local-part  = dot-atom / quoted-string
domain      = 1*atext *("." 1*atext)
atext       = ALPHA / DIGIT / SPECIAL_CHARS

; URI specification from RFC-3986 stubbed here
URI         = scheme ":" hier-part [ "?" query ] [ "#" fragment ]

; UUID specification flattened from RFC-4122 
uuid     = 8(hexOctet) "-" 4(hexOctet) "-" 4(hexOctet)"-"
           4(hexOctet) "-" 6(hexOctet)
hexOctet = hexDigit hexDigit
hexDigit =
   "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" /
   "8" / "9" / "a" / "b" / "c" / "d" / "e" / "f" /
   "A" / "B" / "C" / "D" / "E" / "F"

```

