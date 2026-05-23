# ASCP Specification Drafting Guidelines

## Scope and purpose

This file defines drafting standards for the formal ASCP protocol specifications in `specs/`.

For any file under `specs/`, this file is authoritative over the top-level [`AGENTS.md`](/Users/talljeff/repos/ascp-draft-specs/AGENTS.md) on drafting, editing, document structure, review standards, and other specification-specific guidance. The top-level file still applies for repo-wide process matters that do not conflict with this file.

If you are creating, editing, extending, or reviewing a specification in this directory:

1. Read [`specs/README.md`](/Users/talljeff/repos/ascp-draft-specs/specs/README.md) first for document roles and reading order.
2. Use this file for drafting rules, source-of-truth boundaries, and acceptance criteria.
3. Keep the specification suite internally consistent. When a change affects shared terminology, layered boundaries, or the master specification map, update the relevant companion documents in the same change.

## Document set and authority

The files in `specs/` are the formal ASCP specification suite. They are not implementation RFCs and should not be drafted as implementation-owned design notes.

Authoritative shared references within this directory:

- [`the-agents-shared-cognition-protocol.md`](/Users/talljeff/repos/ascp-draft-specs/specs/the-agents-shared-cognition-protocol.md) defines the architectural framework, system model, and specification map for the suite.
- [`ascp-terminology-primer.md`](/Users/talljeff/repos/ascp-draft-specs/specs/ascp-terminology-primer.md) is the authoritative terminology and layering reference.
- [`ascp-stack.md`](/Users/talljeff/repos/ascp-draft-specs/specs/ascp-stack.md) is the authoritative high-level map of the ASCP layer model.
- [`ascp-artipoint-grammar-a-structure.md`](/Users/talljeff/repos/ascp-draft-specs/specs/ascp-artipoint-grammar-a-structure.md) owns the Layer-2 grammar and syntax for Artipoint Expressions, Articulation Statements, and Articulation Sequences.

Other specifications in `specs/` should treat those shared documents as authoritative at their stated boundaries and should avoid silently redefining terms, layers, or syntax they already own.

## Source-of-truth rules

- Place normative requirements with the specification and section that actually owns the behavior.
- Do not duplicate detailed normative requirements across companion specifications.
- Do not redefine terminology already owned by the terminology primer unless the document is explicitly refining that term for its own scoped use.
- Do not redefine Layer-2 syntax or ABNF outside the grammar specification unless the grammar spec is being updated in the same change.
- Keep syntax, semantics, transport, trust, governance, and bootstrap responsibilities at their proper architectural boundaries.
- When a document depends on another ASCP specification, say so explicitly and describe the boundary clearly.
- When a broader architectural document summarizes requirements defined elsewhere, make the summary clearly non-owning.

## Drafting standards and conventions

These rules apply to all new or substantially revised specifications in `specs/`.

- Write from the current design, not from history. Do not specify behavior through phrases such as "used to", "previously", or other evolution language.
- Use direct, matter-of-fact specification prose. Avoid brainstorming style, roadmap prose, task lists, and implementation-note tone.
- Prefer short paragraphs and clear sectioning. Use bullet lists only when they improve clarity for distinct cases, ordered steps, or field inventories.
- Keep section structure semantically honest:
  - architecture and scope rules belong in introductory or boundary sections,
  - syntax rules belong with grammar-owned sections,
  - semantic rules belong with the construct or mechanism that owns them,
  - state-machine rules belong in explicit state or procedure sections,
  - security requirements belong in security sections unless another section clearly owns the behavior.
- Break long normative sections into concise subsections based on real ownership boundaries such as message types, constructs, procedures, or evaluation stages.
- Use normal prose capitalization for protocol concepts such as Artipoint, Articulation Statement, Channel, Stream, Space, and Effective Governance Set, consistent with the terminology primer.
- Use backticks only for literal identifiers, grammar tokens, field names, attribute keys, enum-like symbols, wire values, and similar literal forms.
- Do not combine emphasis markup with backticks for the same token.
- Distinguish clearly between normative and informative text. If a section mixes both, state which statements are normative.
- Prefer explicit boundary statements such as syntax vs. semantics, governance vs. enforcement, or transport vs. interpretation when those distinctions are important to correct implementation.
- Make requirements concrete enough that an implementer can build an interoperable implementation without further design negotiation.

## Normative language

- Use RFC 2119 and RFC 8174 keywords only for actual requirements.
- Use uppercase requirement words: `MUST`, `MUST NOT`, `SHOULD`, `SHOULD NOT`, and `MAY`.
- Do not scatter uppercase keywords through explanatory or motivational text.
- If a document is primarily informational, say that clearly in the status section and make clear when RFC 2119/8174 keywords are only summarizing requirements defined normatively elsewhere.
- If a section is informative except for explicit requirement sentences, say so directly.

## Required document shape

Unless the owning document has a clear reason to differ, new or substantially rewritten ASCP specifications should follow the suite's established form:

- Title and subtitle identifying the document's role.
- Version line and editors line near the top of the document.
- A numbered `Status of This Document` section beginning at Section 1.
- A numbered `Abstract` section.
- A numbered `Introduction` section.
- Additional numbered sections for scope, terminology, protocol model, procedures, security considerations, examples, references, and appendices as appropriate for the document.

The status section should state:

- what kind of document it is,
- whether it is normative or informational,
- whether it is an early draft or public comment draft,
- how RFC 2119/8174 keywords are being used,
- and, when relevant, that the document is not an IETF standards-track document.

## Section expectations

Use these section expectations as the default for protocol-facing specifications in this repo. They are intended to improve IETF-readiness without forcing every ASCP document into the same shape.

- `Security Considerations` is required for protocol specifications and normative subsystem specifications. It should discuss concrete threats, trust assumptions, failure consequences, and any limits of the specified defenses.
- `IANA Considerations` is required whenever a document defines registries, code points, message types, error codes, attribute names, media types, URI schemes, protocol identifiers, or any other extensibility namespace intended for interoperable reuse. If there are no such actions, the document should say so explicitly.
- `Privacy Considerations` is required when a specification exposes stable identifiers, authorship metadata, timestamps, discovery metadata, governance metadata, visibility metadata, or other data that could enable correlation across participants, Channels, replicas, or time.
- `Operational Considerations` is recommended for deployable protocol documents such as Channels, ALSP, Bootstrap, and Trust/Identity. Architecture overviews and terminology primers may omit it when operational behavior is not part of the document's purpose.
- `Normative References` and `Informative References` should be split when the distinction matters to correct implementation or review.
- Examples should be placed in clearly marked informative sections or appendices unless the document explicitly states that a worked procedure is normative.

## Specification-specific guidance

### Relationship to companion specifications

- State what companion ASCP specifications remain normative for related behavior.
- Use explicit boundary text when one document depends on another for syntax, semantics, cryptographic rules, transport behavior, governance evaluation, or bootstrap procedures.
- When subsystem boundaries matter, include a short relationship or scope-boundary subsection rather than leaving the dependency implicit.

### Shared terminology and layering

- Treat the terminology primer as the authoritative source for shared ASCP vocabulary.
- Treat the stack and master specification as the authoritative source for the layer model and suite structure.
- Treat the grammar specification as authoritative for syntactic form and canonical structural representation.
- Keep Layer-0, Layer-1, Layer-2, and Layer-3 responsibilities distinct unless the document is explicitly describing a cross-layer interaction.
- Do not use one term interchangeably for a semantic concept and its wire, grammatical, or persisted representation unless the distinction is explicitly defined in the document.
- Introduce new cross-spec terminology only when the terminology primer cannot carry the concept cleanly. If a new shared term is needed, update the terminology primer in the same change.

### Protocol precision

- Define message, envelope, record, and object fields with clear presence rules, allowed values, validation behavior, and receiver behavior for invalid or unsupported input.
- Specify state-dependent procedures with concrete sequencing rules, success conditions, failure behavior, retry expectations, replay handling, and idempotency expectations where those behaviors matter.
- Specify handling for unknown fields, unknown enum values, future extensions, and version skew whenever a format or procedure is intended to evolve.
- When examples show syntax, message layouts, or canonical forms, ensure they match the owning grammar and field rules exactly.

### Extensibility and registries

- Every extensible namespace should identify who assigns values, how collisions are prevented, and what the allocation policy is.
- New registries or extension spaces should define initial contents, registration procedure, change-control assumptions, and expert-review expectations if applicable.
- Do not leave extensibility behavior implicit. State what compliant implementations must do when they encounter unknown values, unsupported extensions, duplicate registrations, or conflicting definitions.
- Avoid creating extension points with no interoperability policy. If a namespace is private or experimental, say so explicitly.

### Examples and appendices

- Examples should be clearly marked informative unless the document explicitly says otherwise.
- Appendices should be used for worked examples, explanatory walkthroughs, migration notes, open issues, or supplemental guidance that would otherwise distract from the core normative flow.
- Do not move core normative requirements into appendices merely to keep the main body short.

### Conformance and references

- Where practical, identify the behaviors, message fields, algorithms, or evaluation rules that are necessary for interoperable implementation.
- Use separate normative and informative references when the distinction matters.
- Security-sensitive, cryptographic, and wire-format requirements should cite the governing external standards or companion ASCP specifications directly.
- Avoid making core normative behavior depend on unstable external drafts unless the dependency is explicit and justified.

### ASCP-specific security and privacy review

- Security analysis should cover at least eavesdropping, replay, insertion, deletion, modification, impersonation, downgrade paths, denial of service, and trust-boundary confusion where relevant.
- ASCP specifications should also evaluate ASCP-specific risks such as Channel confusion, identity-binding mistakes, bootstrap trust confusion, stale governance state, key-rotation failure, unauthorized semantic effect despite authenticated delivery, and mismatch between governance meaning and lower-layer enforcement.
- Privacy analysis should consider cross-Channel and cross-replica correlation, long-lived identifiers, authorship exposure, timing leakage, discovery metadata leakage, and governance or trust structures that reveal sensitive organizational relationships.
- Documents that expose durable logs, participant sets, or trust graphs should say what information remains observable even when payload confidentiality is otherwise protected.

### Draft-to-I-D readiness

- Use boilerplate-friendly BCP 14 wording when RFC 2119/8174 keywords are used normatively.
- State clearly whether the document is architecture-only, informative guidance, or normative protocol behavior.
- Keep section titles and reference structure close enough to common IETF practice that later Internet-Draft conversion is straightforward.
- If a document normatively depends on an Internet-Draft or similarly unstable external text, call that dependency out explicitly.

## Acceptance criteria

A new or substantially rewritten specification in `specs/` should meet all of the following before it is considered acceptable:

- It fits the ASCP specification suite rather than reading like an implementation RFC, product note, or planning document.
- It states its document type and normative status clearly in Section 1.
- It uses the suite's standard numbered structure beginning at Section 1.
- It defines one coherent protocol or architectural profile rather than a loose menu of alternatives, unless comparison is the explicit purpose of the document.
- It places requirements with the section that owns them and avoids duplicating detailed normative behavior from companion specifications.
- It uses RFC 2119/8174 keywords carefully and only for real requirements.
- It is concrete enough for interoperable implementation, especially around message structure, sequencing, validation, state transitions, authority boundaries, and error handling where applicable.
- It states its boundaries with related ASCP specifications clearly, especially for terminology, grammar, trust, governance, channels, synchronization, and bootstrap behavior.
- It includes `Security Considerations`, and includes `IANA Considerations`, `Privacy Considerations`, or `Operational Considerations` whenever the subject matter makes them relevant under this guide.
- It keeps informative examples, tutorials, and explanatory material clearly separated from normative requirements.
- It defines extensibility behavior explicitly when it introduces registries, namespaces, message types, error codes, attributes, or other reusable protocol values.
- It preserves the established ASCP distinction between syntax, semantics, cryptographic realization, and transport.
- It updates related shared documents in the same change when the draft changes shared terminology, the suite map, or master cross-document assumptions.

## File naming and maintenance

- New specification files should live in `specs/` and follow the existing naming pattern `ascp-<topic>.md` unless the document is the suite master specification.
- Keep headings descriptive and in sentence case.
- Preserve existing suite terminology and naming unless a deliberate terminology change is being made across the relevant documents.
- When adding a new specification or materially changing document roles, update [`specs/README.md`](/Users/talljeff/repos/ascp-draft-specs/specs/README.md).
