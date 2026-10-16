# The Work Persists. The Agents Come and Go.

*Why AI agents should join ongoing work instead of making us bring the work to each new agent*

**Reframe | Concept note | 14 October 2026**

## The way we use AI agents is backward

Today, we often start with an AI agent and bring our work to it. We write a prompt. We add links and attach files. We provide tools, summarize decisions, and explain what others have done. The agent builds its own working context and gets started.

When we need a different agent, we do much of this again. We send a handoff, copy a conversation, or ask the first agent to prepare a summary. If a person or another application needs to join, we repeat the process in yet another form.

Smarter agents can make these handoffs easier. They can read more material, prepare better summaries, and infer more of what happened. But this still assumes **the agent is the center of the work**. The work is treated as something we must package and deliver to each agent silo.

Human organizations work the other way around.

When an engineer joins a project, they get access to the team's existing work, learn what's been decided, and take on part of the job. The plans, decisions, and commitments stay with the project when that engineer moves on. Someone else can join and pick up where they left off.

People may take knowledge with them when they leave, **but the work persists even after people come and go.**

AI agents should be able to work this way too.

## Put the work at the center

Imagine that an ongoing initiative has its own durable, addressable context. That context is not owned by the first person, agent, chat session, or application that helped create it.

The context includes more than relevant documents. It can include the goals that guide the work; the decisions and reasons behind them; relationships among activities; proposals, concerns, and agreements; roles and accepted responsibilities; and the history of changes. Different participants may see different authorized parts of that context.

A person or agent can **join the work**, learn the relevant context, take on an agreed role, contribute, and eventually leave. The context keeps evolving. Others can continue without asking the departing participant to reconstruct everything first.

This reverses the usual model:

> **Instead of bringing the work to an agent, staff the agent into the work.**

The word *staff* is deliberate. In human organizations, staffing is not the same as assigning a task or handing someone a folder. It establishes a relationship between a participant and an ongoing effort. The participant has a role, a purpose, responsibilities to accept, limits on their authority, and other people to coordinate with.

To make this experience possible for AI agents—and for humans working with them—we need a new kind of infrastructure that preserves the work as its participants change.

## Joining work is not the same as receiving context

An agent can receive a perfect summary of a project and still not be a participant in its coordination.

It may know exactly who approved a release and who accepted responsibility for shipping it. Reading that record does not make the agent responsible for the release or give it authority to change the plan. Those relationships have to be established through participation.

More capable AI can understand these relationships more accurately. It cannot, through inference alone, turn a proposal into an approval or an expectation into an agreement.

That is the difference between **information about work** and **articulation of work**.

An *articulation* is an act through which a participant helps establish or change the coordination of work—for example, accepting a responsibility, raising an issue, approving a decision, or revising a commitment. An account of an act is not the act itself. Its author, authority, relationships, and place in the sequence of work matter.

*Shared cognition*, in the sense used in Computer-Supported Cooperative Work (CSCW), is the coordinated understanding and activity that participants achieve through collaboration. It does not require every participant to have the same beliefs or see the same information. In fact, a useful shared system must be able to represent disagreements and unresolved questions without silently turning them into consensus.

This is why a shared cognition protocol is not just an agent memory store. Memory helps a participant recall what it knows. **Articulation provides a way for independent participants to establish and evolve what they are doing together.**

## A small example: Staff, change, replace

Suppose a product team is preparing a release.

A human lead establishes a Workstream for the release. A *Workstream* is a persistent context for a particular thread of work, including the articulated relationships that connect its participants and activities.

The lead invites an AI engineering agent to join. The agent discovers the current decisions and dependencies relevant to engineering and accepts responsibility for preparing the release. It recommends shipping on Friday. That recommendation becomes an approved plan when the human lead approves it.

Next, a security-review agent from another provider joins. It has a different role and a different view of the same Workstream. It finds an access-control flaw and records the finding against the approved release plan.

The human lead withdraws approval until the flaw is fixed and the release is reviewed again. That change is recorded alongside the original approval and the security finding. Both agents can see why Friday's release is on hold and how that changes their work.

Later, the engineering agent leaves and a replacement from a different provider joins. The replacement joins the same work; it does not need the first agent's private conversation or internal memory. It can access the relevant, authorized history and take on a new delegation to fix the flaw. It does not inherit the first agent's identity or authority merely by reading the record. But it can continue the work without making the human rebuild the coordination context from scratch.

Even if the application that started the Workstream closes, other participants can continue through their own applications.

**The work did not live inside the original agent. It did not live inside its chat. It did not depend on one application to remain meaningful.**

That continuity includes the decisions, commitments, authority, and unresolved disagreements that give the work its meaning.

## Autonomy without surrendering control

A well-run human team does not ask a manager to approve every mouse click. Nor does every team member have unlimited authority. People work within roles, delegated authority, accepted responsibilities, and understood escalation paths. Trust, competence, and experience may inform those delegations, but they do not grant authority on their own.

An agent should be free to act within its established scope while recognizing the points at which it needs another participant's agreement or approval. If that scope changes, an explicit articulation can record the change. A different agent or application should be able to recognize it.

This does **not** replace technical security. Identity checks, policy enforcement, restricted credentials, sandboxes, and authorization systems remain essential. A shared articulation record helps explain *which authority has been established and why*; trusted systems must still enforce what an agent may actually do.

## Of course a database can store this

A database can track decisions, responsibilities, and the relationships among them. Existing applications already do much of this. But consider what we need that database to preserve when independent people, agents, and applications participate in the same work.

It needs to capture articulations as they happen: who proposed something, who agreed, what authority they had, what remained disputed, and what changed. The current state of the work must remain traceable to the acts that established it.

That calls for **immutable history**. When a decision changes, a new articulation supersedes the earlier one. When someone withdraws a commitment, that withdrawal becomes part of the record. The work evolves while the history of how it evolved remains intact.

It also calls for **auditability**. Participants need to be able to verify who authored an articulation and trace its relationships to earlier acts. A replacement agent should be able to distinguish an authorized decision from a recommendation, and a resolved objection from one that remains open.

And it calls for **end-to-end protection**. Authorship and integrity must remain verifiable as articulations move between implementations. Confidential context should be encrypted for its intended participants, so its privacy survives storage and distribution through other systems.

These requirements describe a special kind of graph database: one that preserves authored coordination acts, their relationships, and the history through which the work evolves. To let independent participants build on that work, its articulation model must be shared across implementations.

## A common protocol, independent implementations

For work to continue across independent implementations, those implementations need a common way to express, exchange, and interpret its articulations.

A shared grammar gives coordination acts a consistent representation. Compatible semantic profiles define how the relevant types of articulation and their relationships are interpreted—for example, how a decision supersedes an earlier decision or how a commitment connects a decision to planned work. Together, these let implementations preserve both the record and the meaning needed to participate in the work.

That is why an **open, common standard** matters. The work's continuity should not depend on one vendor's implementation. Different implementations can offer different interfaces, workflows, and capabilities while preserving the shared articulations and their relationships.

Participants remain independent. Humans and agents retain their own perspectives and responsibilities; applications retain their own internal architectures. What they share is a common way to articulate and recognize decisions, commitments, and disagreements.

## Why this matters beyond AI agents

The agent-staffing use case is a starting point, not the full ambition.

Computers have long been good at running applications and storing their output. Humans have carried much of the burden of connecting those outputs into coherent work: remembering why a document matters, who agreed to what, which plan is current, and what requires attention next.

AI can help reconstruct these connections. A shared cognition layer offers another possibility: capture the relevant articulations *as participants do the work*, so independent humans, applications, and agents can build on them later.

The World Wide Web made it possible to link and access information across independent systems. Shared cognition infrastructure aims at an analogous step for coordination: **make the work itself interoperable, not just the information and tools used to perform it**.

The parallel is in the infrastructure: a common protocol that independent implementations can support, allowing people to choose their tools while continuing to participate in the same shared medium.

**ASCP—the Agents Shared Cognition Protocol**—is Reframe's open protocol proposal for this kind of infrastructure. Its [draft specifications](https://github.com/Reframe-Technologies/ascp-draft-specs) bring together immutable articulation history, verifiable authorship, encrypted distribution, and a common grammar with semantic profiles for governance and coordinated work.

And it begins with a simple change in perspective:

> **Don't bring the work to each new agent. Let agents join the work.**
>
> **The work persists. The agents come and go.**

---

*This note describes a proposed computing model. ASCP is an open protocol proposal, not a claim that these capabilities are already universally implemented or standardized.*
