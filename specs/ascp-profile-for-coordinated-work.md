# ASCP Profile for Coordinated Work

**Public Comment Draft —** *Request for community review and collaboration*  
Version: 0.02 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# 1. Status of This Document

This document is part of the ASCP specification suite and defines a **Layer-3 semantic profile** for coordinated work. It specifies two complementary families of Coordination Constructs—**Rationale Constructs** and **Work Coordination Constructs**—and a cross-family articulation construct, **Commitment**, that binds decisions to coordinated work.

This document is **not** an Internet Standards Track specification. It has not undergone IETF review, has no formal standing within the IETF process, and is provided solely for early review and experimentation. Implementations based on this document should be considered **experimental**.

The key words “MUST”, “MUST NOT”, “SHOULD”, “SHOULD NOT”, and “MAY”, when used in this document, are to be interpreted as described in RFC 2119 and RFC 8174.

# 2. Abstract

Modern organizations rely on “systems of record” to maintain canonical state for operational objects such as accounts, opportunities, tickets, incidents, and projects. However, the **coordination substrate** that actually determines how these objects evolve—justifications, exceptions, approvals, precedents, and cross-system synthesis—often remains trapped in ephemeral conversations and informal judgment.

This document defines a Layer-3 ASCP profile for capturing that into substrate of durable **articulation work**: immutable, addressable, semantically typed Artipoints arranged into an interpretable coordination graph. The profile introduces:

- **Rationale Constructs** to capture issues, options, arguments, evidence, and decisions.
- **Work Coordination Constructs** to capture objectives, goals, tasks, lists, agendas, and discussion points.
- A **Commitment** bridging Construct that converts decisions into articulated plans tied to goals and tasks.

Together, these constructs support deterministic materializations such as decision traces, execution plans, goal progress, meeting packages, and precedent indexes—without requiring mutable overwrites or centralized workflow platforms.

# 3. Introduction

## 3.1 Problem Statement

Coordination today is fragmented. Critical work context is distributed across tickets, documents, chats, emails, calls, and informal memory. While “systems of record” capture the final state (e.g., a discount value, an escalation flag, a closed ticket), they often fail to capture **why that state was allowed to happen**, including:

- which policy or rule was applied or overridden,
- which exception path was invoked,
- who approved or objected,
- what precedent influenced the decision, and
- what work was committed to as a result.

For human teams this produces re-learning, inconsistent judgment, and brittle onboarding. For agentic systems it produces ambiguous execution, limited auditability, and an inability to improve via precedent.

## 3.2 Approach

ASCP addresses this gap by treating coordination as first-class data: **articulation work made durable**.

This profile intentionally separates the *justification of work* (why) from the *organization of work* (what). **Rationale Constructs** capture why a course of action was chosen, while **Work Coordination Constructs** capture what work is being undertaken as a result. Neither family is intended to function as an execution engine, policy system, or authority model.

This separation exists specifically to prevent coordination semantics from collapsing into workflow control or decision enforcement, while still enabling durable traceability between decisions and the work they authorize through an explicit articulation boundary.

Accordingly, this profile defines a minimal and interoperable Layer-3 semantics for coordinated work by standardizing:

- the **Rationale Construct (RC)** and **Work Coordination Construct (WCC)** families and their minimal completeness,
- required **link invariants** between construct instances, and
- deterministic **derived semantic structures** (“materializations”) that ASCP-compliant applications and agents can compute identically across replicas.

These materializations represent *interpreted coordination structures*, not executable control flows, automation logic, or enforcement mechanisms.

## 3.3 Relationship to Other ASCP Specifications

This profile depends on and assumes:

- the **ASCP Terminology Primer** for canonical vocabulary (Artipoint, Artipoint Expression, Articulation Statement, Articulation Sequence, Coordination Construct, Semantic Construct, Derived Semantic Structure, etc.),
- the **ASCP Artipoint Grammar** for syntactic validity and operator definitions (`references`, `supports`, `annotates`, `assembles`, `groups`, `replaces`, etc.),
- the **ASCP Governance and Access Control** specification for participation semantics and authoring authority, and
- the **ASCP Identity & Trust** and **ASCP Channels** specifications for verifiable authorship and distribution.

## 3.4 Syntax vs. Semantics Boundary

The ASCP Artipoint Grammar defines the syntax of Artipoint Expressions. This document assigns **Layer-3 semantics** to certain Artipoint types and establishes evaluation rules for coordinated-work interpretation.

As with other ASCP Layer-3 specifications, this profile does **not** reserve global type identifiers. It defines a recommended vocabulary and interoperability expectations. Alternative or extended type systems MAY be used so long as they preserve the required invariants and materialization behavior.

## 3.5 Out of Scope

This document does not define:

- UI/UX requirements or workflow automation engines,
- transport-layer enforcement or replication semantics,
- canonical schemas for external ticket/project systems, or
- “rules engines” for enterprise policy logic.

This profile focuses on durable articulation, traceability, and deterministic interpretation.

# 4. Terminology

All terminology is used according to the ASCP Terminology Primer. The following additional terms are introduced for this profile:

## 4.1 Rationale Constructs (RC)

A **Rationale Construct** is a Coordination Construct whose primary role is to capture *why* work is done, including justification, deliberation, and decision, but not execution or enforcement.

## 4.2 Work Coordination Constructs (WCC)

A **Work Coordination Construct** is a Coordination Construct whose primary role is to capture *what* work is being undertaken or tracked, but not how it is executed or enforced.

## 4.3 Commitment

A **Commitment** is a cross-family articulation construct that binds rationale to coordinated work. It represents the explicit declaration of intent by which a Decision is taken to authorize and frame a set of Goals and Tasks.

A Commitment is neither a Rationale Construct nor a Work Coordination Construct. It exists specifically to impose traceability and legitimacy across those families.

## 4.4 Decision Trace

A **Decision Trace** is a derived semantic structure materialized from a subgraph rooted at a Decision, including relevant evidence, arguments, commitments, tasks, and lifecycle state within a given scope.

Specific interpretive projections over a Decision Trace MAY be defined as named views (e.g., rationale-focused or work-focused views), but all such views derive from the same underlying trace.

## 4.5 Work Item Proxy

A **Work Item Proxy** is a Construct whose payload denotes an external mutable artifact (e.g., a ticket or document) by URI or other locator, while semantic coordination relationships are expressed through ASCP operators and scope placement.

# 5. Model Overview

This profile defines a coordinated-work system as two semantic families and a cross-family articulation boundary:

1. **Rationale Constructs (RC):** capture decisions and their justification.
2. **Work Coordination Constructs (WCC):** capture the organization and articulation of work, including goals, tasks, and status assertions, without defining execution or enforcement semantics.
3. **Commitment:** a cross-family articulation construct that binds Decisions to coordinated work by assembling Goals and Tasks under explicit intent.

A simplified conceptual structure is:

```
Issue → Options → (Arguments + Evidence) → Decision → Commitment → Tasks → Task State
                                      ↘              ↘
                                       ↘              Goal(s) → Objective(s)
                                        ↘
                                        Precedent (Decision ↔ Decision)
```

All semantics are introduced only through Articulation Statements and their resulting Artipoints.

Arrows in the conceptual structure denote semantic derivation and traceability relationships, not execution order, control flow, or temporal scheduling semantics.

# 6. Rationale Constructs

This profile defines a set of Rationale Construct (RC) Artipoint types that capture aspects of justification, deliberation, and decision-making.

## 6.1 Scope and Semantics

Rationale Constructs represent articulated reasoning and decision outcomes. They do NOT introduce execution, enforcement, or workflow control semantics.

Rationale Constructs describe deliberation and choice only as articulated and traced in context, not as executed or enforced.

## 6.2 Placement and Context

Rationale Constructs MAY appear as standalone Artipoints or as nodes within Contextual Constructs (Spaces, Streams, or Piles).

Contextual placement does not alter the semantic role of a Rationale Construct. Placement MAY affect visibility, aggregation, or materialization behavior.

## 6.3 Construct Types

### **6.3.1 issue**

An issue represents a question, problem statement, or matter requiring consideration or resolution.

An issue establishes the scope of deliberation and serves as the root context for options, arguments, evidence, and decisions. It defines *what is being decided*, not how or whether it will be executed.

### **6.3.2 option**

An option represents a proposed course of action or alternative resolution for a given issue.

Options enumerate the space of possible responses to an issue. They do not imply selection, commitment, or authorization, and may coexist without conflict until a decision is made.

### **6.3.3 argument**

An argument represents a reasoned position in favor of or against a specific option.

Arguments capture evaluative judgment, interpretation, or reasoning contributed by participants or agents. They express *why an option should or should not be selected*, but do not themselves determine outcomes.

### **6.3.4 evidence**

An evidence construct represents factual information, reference material, or external signals used to support or contextualize arguments, options, or decisions.

Evidence provides grounding for deliberation but does not encode interpretation or judgment. Its semantic role is to inform reasoning, not to resolve it.

### **6.3.5 decision**

A decision represents the articulated selection of an option in response to an issue.

A decision records *what was chosen and why*, based on the available options, arguments, and evidence at the time. A decision does not itself execute work or allocate responsibility; it establishes the authoritative rationale from which coordinated intent may later be articulated.

### **6.3.6 action**

An action represents an articulated intent, directive, or follow-on statement motivated by a decision.

An action may capture recommended next steps, requests, or declared intentions, but it does **not** represent the execution of work. Actions do not function as tasks and do not introduce workflow or enforcement semantics; they remain within the rationale domain as expressions of intent.

## 6.4 Denotation and Payload Guidance

Rationale Constructs MAY carry payloads that:

- summarize the construct for humans/agents (e.g., short rationale summary), and/or
- denote external resources (e.g., `evidence` payload as `uri:`).

Payloads are semantic anchors; structure and coordination meaning MUST NOT be encoded in payload.

## 6.5 Link Invariants (Normative)

Within a shared scope:

- A `decision` **SHOULD** `references` exactly one `issue`.
- A `decision` **SHOULD** identify a selected `option` (e.g., `supports {option}`).
- An `option` **MUST** be associated with an `issue`, either by:
  - being assembled/grouped under that issue, and/or
  - `references {issue}`.
- An `argument` **SHOULD** `references` exactly one `option`.
- `evidence` **SHOULD** `supports` an `argument`, `option`, or `decision`.
- An `action` **SHOULD** `references` the `decision` that motivated it.

## 6.6 Evolution and Supersession

A subsequent `decision` MAY `replaces` a prior decision for the same issue. When used, `replaces` MUST be interpreted according to ASCP scoped displacement behavior (SDB): masking applies only within scopes shared by both Artipoints.

# 7. Work Coordination Constructs

This profile defines a set of Work Coordination Construct (WCC) Artipoint types for articulating work organization and tracking. WCCs represent coordination artifacts—objectives, goals, tasks, and organizational groupings—as they are articulated and referenced within a coordination context. WCCs do NOT introduce execution semantics, enforcement logic, workflow control, or lifecycle management.

## 7.1 Scope and Semantics

Work Coordination Constructs describe work as it is articulated, tracked, and referenced by participants. They do not prescribe how work is executed, evaluated, or enforced. WCCs MAY appear as:

- standalone Artipoints within a shared scope, or
- nodes within Contextual Constructs (Spaces, Streams, or Piles) that provide organizational grouping or interpretive context.

## 7.2 Placement and Context

Contextual placement does not alter the semantic role of a Work Coordination Construct. Placement MAY affect visibility, aggregation, or materialization behavior as defined by the containing Contextual Construct.

## **7.3 Construct Types**

### **7.3.1 objective**

An objective represents a high-level outcome or strategic aim toward which coordinated work is directed.

Objectives provide framing and prioritization across multiple goals or initiatives. They do not imply specific execution paths, task ordering, or success criteria beyond articulated intent.

### **7.3.2 goal**

A goal represents a concrete, intended outcome that contributes to an objective.

Goals serve as coordination anchors for commitments and tasks. They express *what is intended to be achieved*, but do not prescribe how work is executed or evaluated beyond articulated context.

### **7.3.3 task**

A task represents a unit of articulated work to be undertaken or tracked.

Tasks function as **Work Item Proxies**, typically denoting external mutable artifacts (e.g., tickets, issues, documents) via payload locators. A task does not encode execution logic, lifecycle enforcement, or ownership semantics; it represents work *as coordinated and referenced*, not as executed.

### **7.3.4 list**

A list represents an explicit grouping of tasks or work items for organizational, planning, or tracking purposes.

Lists provide structure and ordering where useful, but do not impose execution order, dependency semantics, or completion requirements.

### **7.3.5 agenda (Optional)**

An agenda represents an articulated structure for a meeting or coordinated discussion.

Agendas organize discussion points, referenced artifacts, or intended topics. They do not prescribe meeting outcomes, decisions, or follow-up execution.

### **7.3.6 discussion\_point (Optional)**

A discussion\_point represents a specific topic, question, or focus area within an agenda.

Discussion points may reference rationale constructs, commitments, tasks, or evidence to provide context. They exist solely to frame conversation and coordination, not to assert decisions or execute work.

## 7.4 Work Item Proxy Binding (Normative)

A `task` SHOULD function as a Work Item Proxy:

- The `task` payload **SHOULD** be a locator that denotes the external mutable work item, typically `uri:"..."`.
- The locator in payload is the binding to the external system; `references` is reserved for semantic traceability to other Artipoints.

## 7.5 Decomposition and Organization (Normative)

Within a shared scope, producers SHOULD express planning structure using:

- `objective assembles {goal*}` (ordered) or `objective groups {goal*}` (unordered),
- `goal assembles {task*}` or `goal groups {task*}`, and/or
- `list assembles {task*}` or `list groups {task*}`.

## 7.6 Task Lifecycle and “Current State” (Normative)

Mutable task status MUST be representable without mutation of prior Artipoints.

This profile defines:

- `task_state` — an Artipoint type representing a state assertion for a task.

Rules:

- A `task_state` **MUST** `references` exactly one `task`.
- A later `task_state` **SHOULD** `replaces` the prior `task_state` for the same task within the shared scope, enabling deterministic “current status” computation through scoped interpretive masking, not protocol-enforced execution state.
- Task lifecycle history remains available as articulated context even when masked for “current” views.

Attributes MAY be used to carry additional metadata (owner, due, priority), but such attributes do not introduce coordination relationships.

## **8. Commitment (Cross-Family Articulation Construct)**

This section defines the commitment Artipoint type. A Commitment is a **singleton cross-family articulation construct** whose sole purpose is to bind Rationale Constructs to Work Coordination Constructs through explicit declaration of intent.

A Commitment is neither a Rationale Construct nor a Work Coordination Construct. It exists to preserve the semantic boundary between *why* work is justified and *what* work is coordinated, while enabling durable traceability between the two.

## **8.1 Construct Type (Normative)**

This profile defines the following Commitment construct type:

- `commitment`

No other Artipoint type serves the role of cross-family articulation in this profile.

## **8.2 Semantics (Normative)**

A commitment represents an explicit declaration of intent and an articulated coordination plan grounded in a decision and oriented toward one or more goals. It explicitly binds Rationale Constructs (decisions and their justification) to Work Coordination Constructs (goals, tasks, and related work structure).

A Commitment serves as the **normalization point** where rationale becomes articulated execution context. It does **not** represent:

- a binding obligation,
- an enforceable contract,
- an approval mechanism,
- or a protocol-level execution directive.

Commitments express *that* work is intended and *how it is framed for coordination*, not *how or whether* that work is executed.

## **8.3 Link Invariants (Normative)**

Within a shared scope, a commitment Artipoint MUST satisfy the following invariants:

- A commitment **SHOULD** references exactly one originating decision.
- A commitment **SHOULD** supports one or more goal Artipoints it advances.
- A commitment **MUST** assembles (ordered) or groups (unordered) the set of task Artipoints that implement the committed work.
- A commitment **MAY** assembles related action Artipoints, but such actions **MUST NOT** be treated as the executable work substrate.

These invariants ensure that coordinated work remains explicitly traceable to its justificatory decision without collapsing execution semantics into the protocol.

## **8.4 Architectural Role and Non-Bypass Guidance (Informative)**

By design, Commitments serve as the **canonical articulation boundary** between decisions and coordinated work.

Direct attachment of tasks to decisions without an intervening Commitment is permitted by the ASCP grammar but **discouraged by this profile** in order to preserve explicit articulation of intent, legitimacy, and traceability.

This guidance reflects architectural intent rather than syntactic restriction.

## **8.5 Plan Revision and Evolution (Normative)**

A subsequent commitment MAY replaces an earlier commitment to represent revision of articulated intent or coordination structure.

This mechanism enables:

- evolution of execution context,
- re-framing of planned work,
- or reassignment of tasks,

without rewriting decisions, mutating tasks, or erasing historical articulation. Scoped displacement behavior (SDB) applies as defined by ASCP.

## **8.6 Commitment as Articulation Work (Informative)**

In Computer-Supported Cooperative Work (CSCW) research, *articulation work* refers to the meta-work required to align tasks, roles, responsibilities, and interpretations so that cooperative activity can proceed. This includes not only deciding *what* should be done, but explicitly establishing *that* work is authorized, expected, and coordinated.

Within this profile, the commitment construct corresponds directly to this function. A Commitment is not a task, plan, or workflow step. Rather, it is the articulated act by which a decision is taken to legitimize and frame coordinated work.

By representing this articulation explicitly and immutably, ASCP enables durable traceability between justification and action without embedding execution, authority, or enforcement semantics in the protocol.

# 9. Derived Semantic Structures and Materialization

This section defines derived semantic structures (“views”) that materializers SHOULD be able to compute deterministically from Artipoints and operators within a shared scope.

These views define interpretive projections over coordination artifacts, not required user interfaces, runtime behaviors, automation semantics, or presentation formats.

## 9.1 Decision Trace View (Canonical)

Given a `decision` D, a materializer constructs the canonical Decision Trace View as the closure over:

- the referenced `issue`,
- the selected `option` (if present),
- supporting `argument` and `evidence` Artipoints reachable via `supports` and `references`,
- any `commitment` that `references {D}`,
- tasks assembled/grouped by such commitments,
- latest (unmasked) `task_state` for each task in that scope.

The Decision Trace View represents the complete, canonical coordination trace for a decision within a given scope. Other views defined in this section are deterministic projections over this trace, derived by selection or emphasis rather than by introducing distinct semantic structures.

## 9.2 Decision-to-Work View (Decision Trace Projection)

A materializer SHOULD be able to display the following projection of a Decision Trace:

`decision → commitment → tasks → current task_state`

This view answers “what work came from this decision, and where does it stand?”

## 9.3 Goal Progress View

Given a `goal` G, materialize all commitments that `supports {G}`, then aggregate:

- tasks assembled/grouped by those commitments, and
- each task’s latest `task_state` in scope.

Aggregation policy (e.g., percent complete) is implementation-defined, but input selection MUST be deterministic.

## 9.4 Precedent Index

Decisions MAY `references` other decisions as precedent. A materializer MAY expose:

- inbound precedent references (“what later decisions cite this?”),
- outbound precedent references (“what did this rely on?”),
- clustering by issue types or scope.

## 9.5 Meeting Package View

An `agenda` assembles/group discussion points. A discussion point SHOULD `references` the artifacts it intends to cover (decisions, commitments, tasks, evidence). A materializer SHOULD produce a “meeting package” that bundles the referenced traces and execution context.

# 10. Conformance

## 10.1 RC Producer

An implementation claiming RC Producer conformance:

- MUST emit syntactically valid Artipoint Expressions for RC types, and
- MUST satisfy RC link invariants in Section 6.3 for any decision trace it produces.

## 10.2 WCC Producer

An implementation claiming WCC Producer conformance:

- MUST emit syntactically valid Artipoint Expressions for WCC types, and
- MUST support proxy tasks (Section 7.2) and task lifecycle representation (Section 7.4).

## 10.3 Commitment Producer

An implementation claiming Commitment Producer conformance:

- MUST emit commitments satisfying Section 8.3 invariants, and
- SHOULD support plan revision via `replaces`.

## 10.4 Materializer

A Materializer:

- MUST apply ASCP operator semantics and scoped displacement behavior consistently, and
- SHOULD support the derived semantic structures defined in Section 9.

# **11. Related Work, Precedent, and Design Lineage**

This specification builds on several decades of research and practice in cooperative work, distributed systems, decision support, and protocol design. This section surveys relevant prior art to (a) situate the ASCP Coordinated Work Profile within established bodies of work, (b) highlight lessons learned from earlier systems and standards, and (c) clarify the specific ways in which ASCP advances the state of the art rather than duplicating or subsuming existing approaches.

This discussion is informative rather than normative.

## **11.1 Computer-Supported Cooperative Work (CSCW) and Articulation Work**

A foundational influence on this profile is the CSCW concept of **articulation work**: the meta-work required to align tasks, roles, dependencies, and interpretations so that cooperative activity can proceed. Classic CSCW research demonstrated that coordination is not incidental to work, but a primary activity that must be explicitly supported by systems rather than relegated to informal channels.

ASCP adopts this insight directly by treating coordination artifacts—goals, decisions, tasks, meetings, dependencies, and attention signals—as first-class, durable objects. Unlike many CSCW systems, however, ASCP does not embed these constructs within a single application or user interface. Instead, it defines a protocol-level substrate in which articulation work can be expressed, preserved, and re-interpreted over time by both humans and agents.

**Key distinction:** CSCW systems typically *host* articulation work; ASCP *protocolizes* it.

## **11.2 Design Rationale and Decision Representation Systems**

The Coordinated Work Profile is informed by prior efforts to formalize decision-making and design rationale, including Issue-Based Information Systems (IBIS), Questions-Options-Criteria (QOC), and related argumentation models. These approaches established durable patterns for representing:

- decision context and framing,
- alternative options or positions,
- evaluative criteria and trade-offs,
- and resulting commitments or conclusions.

While these models proved valuable for retrospection and analysis, they often suffered from high authoring overhead, rigid schemas, or limited integration with everyday work systems.

ASCP incorporates the *structural lessons* of these systems—namely, that decisions benefit from explicit representation—while avoiding prescriptive schemas. Artipoints allow rationale structures to emerge incrementally and organically through linked statements, without requiring participants to adopt a fixed ontology or formal notation.

**Non-goal:** ASCP does not standardize a universal decision model or argumentation grammar.

## **11.3 Append-Only Coordination History and Event-Derived State**

From distributed systems practice, ASCP draws on the principles of **append-only logs** and **event-derived state**, commonly associated with event sourcing architectures. In such systems, durable records of change form the authoritative history, while current views are materializations derived from that history.

ASCP applies this pattern to coordination rather than application state. Artipoints are immutable once authored; higher-order constructs such as Streams, Spaces, and Piles are interpreted as projections over an evolving graph of contributions. This approach enables replay, auditability, divergence analysis, and future reinterpretation as semantics evolve.

Where applicable, ASCP is compatible with replica convergence techniques such as CRDTs, but does not mandate a specific replication or merge algorithm at Layer-3.

## **11.4 Provenance, Attribution, and Accountability**

The profile aligns conceptually with established provenance models that emphasize authorship, derivation, and responsibility over time. Prior standardization efforts in this space demonstrate that durable attribution is critical for trust, accountability, and governance in distributed systems.

ASCP extends this lineage by binding provenance directly to coordination artifacts rather than treating it as auxiliary metadata. Every Artipoint carries verifiable authorship and temporal placement within a shared graph, enabling downstream systems to reason about how decisions, commitments, and interpretations emerged.

**Key distinction:** ASCP treats provenance as *structural*, not annotative.

## **11.5 Multi-Agent Coordination and Commitment Semantics**

Research in multi-agent systems has long explored coordination through social commitments: explicit, inspectable statements of obligation, expectation, and fulfillment between autonomous actors. These models are particularly relevant in environments where participants cannot be centrally controlled or fully trusted.

The Coordinated Work Profile is compatible with commitment-based reasoning but deliberately avoids embedding formal obligation logic into the protocol. Instead, it provides the substrate on which such semantics may be layered by higher-level profiles, organizational conventions, or agent frameworks.

This preserves flexibility while enabling rigorous interpretation where required.

## **11.6 Workflow, Process, and Case Management Standards**

Enterprise workflow and case management standards demonstrate the risks of over-formalizing knowledge work. While these systems excel in compliance-driven environments, they often impose rigid process models that fail to accommodate exploratory, collaborative, or cross-organizational coordination.

ASCP explicitly avoids executable process semantics at Layer-3. The Coordinated Work Profile is not a workflow engine, nor does it define task execution semantics. Instead, it focuses on *representing coordination artifacts themselves*, leaving execution, automation, and enforcement to tools built atop the protocol.

**Design principle:** Coordination structure should be durable and interpretable even when execution paths change.

## **11.7 Lightweight Decision Capture in Practice**

Informal practices such as Architecture Decision Records (ADRs) demonstrate that durable decision capture succeeds when it is lightweight, append-only in spirit, and colocated with everyday work artifacts. Their success reinforces a core ASCP design choice: favor minimal, composable units over comprehensive schemas.

Artipoints generalize this lesson across domains, participants, and tools.

## **11.8 Governance, Consensus, and Interpretive Stability**

Distributed technical communities have long recognized that decision legitimacy depends on transparent process, documented dissent, and the ability to revisit conclusions over time. Standards processes themselves embody these principles.

The Coordinated Work Profile supports governance not by encoding voting or authority models, but by preserving the interpretive record from which legitimacy can be assessed. Consensus, disagreement, and evolution are observable properties of the coordination graph rather than opaque outcomes.

## **11.9 Summary of Novel Contributions**

In contrast to prior systems and standards, the ASCP Profile for Coordinated Work introduces the following novel contributions:

1. A protocol-level substrate for articulation work, independent of applications or interfaces.
2. Immutable, addressable coordination primitives that support emergent structure rather than fixed schemas.
3. Explicit separation of participation, visibility, and attention as distinct coordination dimensions.
4. Durable, replayable coordination history suitable for both human and agent interpretation.
5. A clear boundary between coordination representation and execution or automation semantics.

These choices reflect a deliberate synthesis of lessons from CSCW, distributed systems, and standards practice, while addressing known adoption and scalability failures of earlier approaches.

# **12. Non-Goals and Explicitly Avoided Anti-Patterns**

This section enumerates design goals that are *explicitly out of scope* for the ASCP Profile for Coordinated Work, as well as common architectural anti-patterns deliberately avoided. These exclusions are as important to interoperability and long-term viability as the features defined elsewhere in this specification.

This section is informative.

## **12.1 ASCP Is Not a Workflow or Process Execution Engine**

The Coordinated Work Profile does not define executable workflows, task state machines, orchestration logic, or process control semantics.

While Artipoints may represent tasks, milestones, dependencies, or commitments, ASCP does not prescribe:

- task lifecycle states,
- execution ordering or branching logic,
- automation triggers,
- or enforcement mechanisms.

**Rationale:** Prior workflow standards demonstrate that executable semantics quickly become brittle in dynamic, human-centered work. ASCP focuses instead on representing coordination artifacts durably, leaving execution and automation to systems built atop the protocol.

## **12.2 ASCP Is Not a Universal Ontology or Knowledge Graph Schema**

This profile does not define a closed or comprehensive ontology for work, decisions, or organizations. It introduces no reserved semantic types for goals, tasks, meetings, or roles.

Semantics emerge through:

- immutable Artipoints,
- typed relations,
- and interpretive conventions shared by participants or tools.

**Anti-pattern avoided:** schema rigidity that prevents adaptation, extension, or cross-domain reuse.

## **12.3 ASCP Is Not a Replacement for Systems of Record**

The Coordinated Work Profile does not attempt to replace authoritative systems such as:

- project management tools,
- ticketing systems,
- document repositories,
- calendars,
- or financial systems.

Instead, ASCP complements such systems by providing a durable coordination substrate that records *why* actions were taken, *how* interpretations evolved, and *who* participated in decisions—context that is often absent from systems of record.

**Design intent:** ASCP preserves meaning, not transactional authority.

## **12.4 ASCP Does Not Impose Organizational Authority Models**

This profile does not encode:

- role hierarchies,
- managerial authority,
- voting rules,
- quorum definitions,
- or enforcement policies.

Although governance artifacts may be represented as Artipoints, their interpretation and enforcement are explicitly delegated to higher-level agreements, organizational norms, or application-layer logic.

**Rationale:** Authority and governance vary widely across contexts and must remain evolvable without protocol revision.

## **12.5 ASCP Is Not a Real-Time Collaboration or Messaging Protocol**

ASCP does not provide:

- real-time presence semantics,
- message delivery guarantees,
- typing indicators,
- or conversational ordering guarantees.

While Channels may reference communication contexts, ASCP does not replace messaging or conferencing protocols.

**Anti-pattern avoided:** conflating durable coordination memory with transient interaction transport.

## **12.6 ASCP Is Not an AI Reasoning or Planning Framework**

This profile does not specify:

- planning algorithms,
- goal decomposition logic,
- optimization strategies,
- or agent decision policies.

ASCP provides shared, inspectable context upon which agents *may* reason, but it does not attempt to standardize intelligence or behavior.

**Design principle:** shared cognition infrastructure should remain neutral with respect to reasoning strategies.

## **12.7 ASCP Does Not Require Global Agreement on Meaning**

The protocol does not require that all participants or tools interpret coordination artifacts identically at all times.

Divergent interpretations, contested meanings, and evolving consensus are treated as first-class realities rather than failures.

**Anti-pattern avoided:** premature semantic closure or enforced consensus.

## **12.8 ASCP Does Not Centralize Control or Coordination**

The Coordinated Work Profile does not assume:

- a central authority,
- a global coordinator,
- or a single source of interpretive truth.

All coordination emerges from shared, immutable contributions that can be replicated, replayed, and reinterpreted independently.

**Rationale:** centralization undermines resilience, trust, and long-term continuity.

## **12.9 ASCP Does Not Optimize for Maximum Automation**

This profile explicitly prioritizes human legibility, inspectability, and agency over maximal automation.

While automation and agents are expected consumers and producers of Artipoints, the protocol does not assume that automation is always desirable or that human involvement is a temporary phase.

**Design stance:** coordination infrastructure should amplify human agency, not obscure or replace it.

## **12.10 Summary of Scope Discipline**

In aggregate, these non-goals reflect a deliberate design philosophy:

- Represent coordination, not execution.
- Preserve meaning, not control.
- Enable interpretation, not enforcement.
- Support evolution, not closure.

By explicitly rejecting these anti-patterns, the ASCP Profile for Coordinated Work seeks to remain durable, adaptable, and aligned with the realities of human–agent collaboration over time.

# 13. Security Considerations

Coordinated work traces contain sensitive operational context. Implementations MUST:

- verify Artipoint authorship per ASCP identity and signature requirements,
- respect governance semantics when interpreting authority and responsibility (including who is permitted to author within a scope),
- consider that payload URIs may reveal sensitive system endpoints or identifiers, and
- treat external evidence links as potentially sensitive data requiring appropriate access control at lower layers.

This profile does not define cryptographic enforcement mechanisms; it assumes ASCP Channels and related security constructs provide distribution boundaries and confidentiality when required.

# 14. IANA Considerations

This document makes no requests of IANA.

# Appendix A. Examples (Informative)

## A.1 Decision → Commitment → Tasks → State (exception trace)

```clike
[ISSUE1, AUTH1, 2025-12-25T10:00:00Z,
  [issue, "Renewal discount over policy cap?", string:"Cap 10% unless exception."]];

[OPT1, AUTH1, 2025-12-25T10:02:00Z,
  [option, "Approve 20% with exception", string:"Route approvals + record precedent"]
  references {ISSUE1}];

[EVID1, AUTH1, 2025-12-25T10:03:00Z,
  [evidence, "SEV1 incident list", uri:"https://pagerduty.example/incidents?sev=1&since=30d"]];

[ARG1, AUTH1, 2025-12-25T10:04:00Z,
  [argument, "Service impact justifies exception", string:"pro"]
  references {OPT1}];

[EVID1] supports {ARG1};

[DEC1, AUTH1, 2025-12-25T10:06:00Z,
  [decision, "Approve 20% under policy v3.2 exception", uri:"https://policy.example/v3.2"]
  references {ISSUE1}
  supports   {OPT1}];

[GOAL1, AUTH1, 2025-12-25T10:07:00Z,
  [goal, "Retain account", string:"Renewal secured this quarter"]];

[TASK1, AUTH1, 2025-12-25T10:08:00Z,
  [task, "Route exception to Finance", uri:"https://linear.example/issue/ABC-123"]];

[TASK2, AUTH1, 2025-12-25T10:08:10Z,
  [task, "Update CRM discount terms", uri:"https://salesforce.example/opp/006…"]];

[COM1, AUTH1, 2025-12-25T10:09:00Z,
  [commitment, "Execute renewal exception path", string:"Finance approval + CRM update"]
  references {DEC1}
  supports   {GOAL1}
  assembles  {TASK1, TASK2}];

[TS1, AUTH1, 2025-12-25T10:10:00Z,
  [task_state, "Open", string:"open"] references {TASK1}];

[TS2, AUTH1, 2025-12-25T10:30:00Z,
  [task_state, "Done", string:"done"] references {TASK1} replaces {TS1}];
```

## A.2 Objective hierarchy with work lists

```clike
[OBJ1, AUTH1, 2025-12-25T12:00:00Z,
  [objective, "Reliability Q1", string:"Improve platform reliability"]];

[GOAL2, AUTH1, 2025-12-25T12:01:00Z,
  [goal, "Reduce P1 incidents", string:"< 2/month by end of quarter"]];

[OBJ1] assembles {GOAL2};

[LIST1, AUTH1, 2025-12-25T12:02:00Z,
  [list, "Sprint 12", string:"Current sprint backlog"]];

[LIST1] assembles {TASK1, TASK2};
```

## A.3 Agenda tying together rationale and work

```clike
[AGEN1, AUTH1, 2025-12-25T14:00:00Z,
  [agenda, "Renewal exception review", string:"Weekly exception and precedent review"]];

[DP1, AUTH1, 2025-12-25T14:01:00Z,
  [discussion_point, "Review DEC1 trace and outcomes", string:"Confirm precedent + audit approval chain"]
  references {DEC1, COM1}];

[AGEN1] assembles {DP1};
```

# Appendix B. Suggested Attribute Conventions (Informative)

Implementations MAY adopt consistent attribute keys for coordinated-work metadata, including:

- `role::responsible`, `role::accountable`, `role::consulted`, `role::informed` (as defined in governance profile)
- `priority := <scalar>`
- `due := <timestamp>`
- `owner := <participant>` (governance meaning)
- `status := <string>` (advisory; prefer `task_state` for semantic state)

Attributes do not create graph relationships; they are interpreted as metadata within the containing Artipoint Expression.