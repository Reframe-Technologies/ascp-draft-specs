# Issue 01: Layer-2 `seq-id`

Author: Jeffrey Szczepanski
Date: 2026-05-30
Issue Status: Resolved
Resolution: Incorporated into `docs/ascp/ascp-artipoint-grammar-a-structure.md`, Section 5.3.1.

## Background

ASCP Layer-2 currently defines an Articulation Sequence header containing:

- `author_uuid`
- `seq_number`
- optional sequence timestamp

The current grammar defines `seq-number` as an author-scoped unsigned 64-bit value that MUST be strictly monotonically increasing across successive sequences for a given author.

At the same time:

- Layer-2 rendering requires `author_uuid` and `seq_number` to be resolved before canonical bytes can be emitted.
- Layer-0 log order, not `seq-number`, determines replay and evaluation order.
- Layer-3 surfaces such as `NodeContext` use `author_uuid:seq_number` as a convenient local correlation key.

This means the field is currently required and preserved, but its practical usage is closer to sequence correlation than to ordering semantics.

## Problem

Strict monotonic author-scoped sequence numbering is a poor fit for concurrent authoring across multiple Layer-3 surfaces.

In the current architecture, articulated sequences may be authored from multiple places, including:

- `NodeContext`
- `IdentityDirectory`
- `ChannelDirectory`
- helper or application-owned authoring paths

If multiple threads or modules author sequences for the same `author_uuid`, then strict monotonic numbering requires a shared allocator above all such surfaces. That allocator must issue the next number before Layer-2 rendering occurs, because the header is part of the canonical signed bytes.

This creates an awkward coordination point:

- sequence numbering must be resolved before canonical rendering
- rendering may happen on different threads
- outbound submission happens later and may not preserve authoring order between threads
- the burden of monotonicity therefore falls on a shared pre-render authoring coordinator

The result is architectural complexity around a field whose actual role appears weaker than its current specification.

## Why Current Semantics Are Fragile

The present `seq-number` rule appears stronger than the role the field actually plays.

In the current model, `seq-number` is not:

- a replay-order mechanism
- a graph-evaluation-order mechanism
- a trust-semantic input
- a durable graph-uniqueness mechanism on its own

Instead, it is mainly useful as:

- a sequence-level correlation identifier
- a local tracking key
- a debugging and operational reference

That makes strict monotonicity expensive relative to its value. The field carries ordering constraints that the rest of the architecture does not actually rely on.

## Proposed Direction

Change the Layer-2 sequence-header field from `seq-number` to `seq-id`.

The proposed `seq-id` model is:

- keep the field as an unsigned 64-bit integer
- rename it from `seq-number` to `seq-id`
- redefine it as an author-scoped sequence identifier rather than a monotonic sequence counter
- generate it from a cryptographically strong random source
- treat practical uniqueness as reliably determined by `(author_uuid, seq_id)`
- if absent in Draft form, allow the Layer-2 renderer to assign it automatically before canonical rendering

Under this model, the field remains useful for:

- correlation of authored sequences
- local tracking of pending or outbound requests
- replay acknowledgement and operational diagnostics

But it no longer requires a shared monotonic allocator across authoring surfaces.

## Rationale

This direction fits the architecture better for several reasons.

### 1. It supports concurrent authoring cleanly

Multiple threads or modules can author sequences for the same author identity without coordinating on a shared monotonic counter before rendering.

### 2. It preserves the useful part of the field

The field continues to provide a stable sequence-level correlation identifier that can be used by:

- `NodeContext`
- Layer-3 directories
- helpers
- application-level request tracking

### 3. It removes a coordination burden that is not justified by replay semantics

Because replay order already comes from Layer-0 log order, relaxing the field away from monotonic ordering does not weaken graph evaluation semantics.

### 4. A random 64-bit identifier is sufficient at realistic scales

For approximately 50 outstanding sequences per author, the collision probability for a cryptographically random 64-bit identifier is approximately:

- `6.6e-17`

That is low enough that duplicate checking does not need to be a normative requirement.

Optional local retry on collision may still be an implementation choice, but it is not central to the design.

## Affected Specifications

If this issue is accepted, the following specifications require inspection and likely update:

- `docs/ascp/ascp-artipoint-grammar-a-structure.md`
- `docs/rfc-ascp-layer-2-codec.md`
- `docs/rfc-ascp-layer-3-graph.md`
- `docs/rfc-ascp-layer-3-directories.md`

Additional specifications should be inspected for references to `seq-number`, author-local monotonicity, or correlation semantics built on the current field name.

## Affected Code Areas

If this issue is accepted, the following implementation areas require inspection and likely update:

- `src/ascp/layer2/types.py`
- `src/ascp/layer2/parser.py`
- `src/ascp/layer2/renderer.py`
- `src/ascp/layer2/codec.py`
- `src/ascp/layer3/context.py`
- `src/ascp/layer3/types.py`
- `src/ascp/layer3/graph.py`
- `src/ascp/layer3/runner_helper.py`
- runner submission paths under `src/ascp/runner/`
- tests and fixtures across Layer-2, Layer-3, runner, and RFC-alignment coverage

The main follow-up work would include:

- renaming `seq_number` references to `seq_id`
- updating parser and renderer behavior
- allowing renderer-side assignment when absent
- updating local correlation logic such as `author_uuid:seq_number`
- removing or revising any assumptions about monotonic ordering
- updating tests, fixtures, and documentation references

## Open Questions

- Should the grammar field be renamed immediately, or should there be an interim compatibility phase?
- Should renderer auto-assignment be the only automatic assignment point?
- Should any ingestion-side validation change once monotonicity is removed?
