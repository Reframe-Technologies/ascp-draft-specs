# Issue 02: Keyframe encoding and payload shape.

Author: Jeffrey Szczepanski
Date: 2026-05-29
Issue Status: Open

## Background

The current ASCP specifications and implementation expose several related concerns around Keyframe Artipoints.

First, the Trust and Identity specification currently places recipient identity in the keyframe attribute name using forms like:

- `envelope::<recipient-identity-uuid> := "<JWE>"`

This is inconsistent with the Layer-2 grammar, which defines class-qualified attribute names using the shape:

- `class::key`

where:

- `class` must begin with `ALPHA`
- `key` must begin with `ALPHA`

The same value-shape principle applies to the ASCP `member` attribute. Referenced Artipoint UUIDs belong in attribute values encoded as strings, and the stable attribute name `member` carries the semantic key.

Second, the current implementation places CAK public-key material on the Keyframe as an attribute such as `cak_public_jwk`, even though ASCP already uses the Artipoint payload as the natural home for public JWK material in Certificate and RootCA Artipoints.

Third, the current Keyframe and recipient-envelope rules do not cleanly express the now-required optionality around CAK presence and recipient encryption material. A Keyframe may legitimately carry no CAK at all, and recipient JWE material may still be present even in that no-CAK case because encryption remains optionally supported.

Fourth, channel-side authoring now increasingly operates through a profile-driven model. Rather than articulating several low-level cryptographic policy attributes directly on each Keyframe, the long-term ASCP direction is to articulate a profile name and let that profile define the lower-level Layer-1 and Layer-0 consequences.

At the same time, the current bootstrap and discovery rules for Channel Reference Artipoints remain correct structurally:

- `cak_kid`
- `cak_public_keys`

Those discovery-facing `channel-ref` attributes are required for bootstrap and ALSP behavior and remain part of the Channel Reference contract. What this issue clarifies is when their effective values may legitimately be empty.

## Problem

The current Keyframe shape has become internally inconsistent.

The issues are:

- `envelope::<recipient-identity-uuid>` is not legal under the Layer-2 grammar because it places a UUID into the `key` position
- `member` attribute examples using unquoted forms such as `member + <uuid>` obscure the Layer-2 value contract; the concrete articulated value should be a quoted UUID string such as `member + "<uuid>"`
- Keyframe-side CAK public-key material is currently being treated like an attribute even though comparable ASCP public-key constructs use payload-carried JWKs
- the current Keyframe shape does not clearly distinguish CAK-bearing Keyframes from no-CAK Keyframes through payload presence alone
- the current recipient-envelope wording does not cleanly express that JWE is mandatory in CAK-bearing cases but still permitted in no-CAK cases
- a separate Keyframe-side `cak_kid` attribute is redundant because `ascp:cak:<keyframe-uuid>` is deterministically derivable from the Keyframe Artipoint UUID
- low-level Keyframe crypto-policy attributes such as `payload_cipher`, `message_signing`, and `channel_access_alg` are poorer fits once a named crypto/profile contract becomes the governing articulation mechanism

This is not one isolated mismatch. It is a broader Keyframe encoding problem spanning:

- recipient-envelope articulation
- `member` attribute UUID value shape
- CAK public-key placement
- CAK optionality and recipient-JWE optionality
- redundant Keyframe-side identifiers
- profile expression at the wrong level of abstraction

## Why Current Semantics Are Fragile

The current Keyframe design is fragile for several reasons.

### 1. It puts recipient identity into grammar-significant attribute-name structure

The Layer-2 grammar defines stable attribute naming through `class::key`. Dynamic `envelope::<uuid>` naming steps outside that model.

### 2. It separates Keyframe public-key identity from the place ASCP already uses for public JWK material

Certificate Artipoints use payload-carried JWKs:

- `["certificate", <label>, json:{ <JWK> }]`
- payload MUST contain a valid JWK

Keyframes should follow the same basic public-key publication pattern when they carry CAK public-key material.

### 3. It redundantly articulates values that are derivable from the Keyframe UUID

If the CAK `kid` is always `ascp:cak:<keyframe-uuid>`, then a Keyframe-side `cak_kid` attribute is not carrying new information.

### 4. It articulates low-level crypto settings where a named profile should be the governing semantic handle

Once Keyframes are authored through a profile-driven channel model, repeating low-level policy fields on every Keyframe makes the articulation noisier and less coherent.

### 5. It diverges from the correct separation between discovery state and semantic keyframe state

`channel-ref` discovery artifacts should continue to expose the CAK material needed for bootstrap and ALSP admission. That does not imply the Keyframe must mirror that same discovery encoding internally.

## Proposed Direction

This issue proposes a coherent Keyframe encoding model with four parts:

- `channel-ref` remains unchanged
- Keyframe recipient-envelope articulation uses `envelope::recipients`
- Keyframe payload carries CAK public-key material when a CAK is required
- Keyframe attributes are reduced to `crypto_profile` and recipient-envelope state

### 1. Channel Reference remains unchanged structurally

The `channel-ref` contract is already correct and remains required.

This issue does not propose changes to:

- `cak_kid`
- `cak_public_keys`

on Channel Reference Artipoints.

Those fields remain the bootstrap and discovery contract for Layer-0 admission and ALSP CAK verification.

This issue does, however, clarify their effective value semantics:

- `cak_kid` is empty when there is no active CAK
- `cak_public_keys` is empty when the channel has never had any CAKs
- `cak_public_keys` may still carry historical CAK public keys when there is no active CAK now

For example, if a channel previously used CAKs `K1` and `K2`, and then rotates to a no-CAK Keyframe, the Channel Reference may legitimately expose:

- `cak_kid = ""`
- `cak_public_keys = [ <K1 public JWK>, <K2 public JWK> ]`

That is coherent: there is no currently active CAK, but historical CAK discovery material still exists.

### 2. Keyframe recipient-envelope articulation uses `envelope::recipients`

The Trust and Identity specification should no longer define recipient placement using:

- `envelope::<recipient-identity-uuid> := "<JWE>"`

Instead, it should define one stable keyframe attribute name:

- `envelope::recipients`

Each articulated value represents exactly one recipient envelope entry encoded as one deterministic UTF-8 string using one of two legal forms:

- `"<recipient-identity-uuid>"`
- `"<recipient-identity-uuid>|<JWE compact serialization>"`

The `|` character is the explicit delimiter when a JWE is present. Accordingly:

- the complete string with no `|` represents an identity-only recipient entry
- the prefix before `|` MUST be the canonical UUID text of the intended recipient Identity Artipoint
- the suffix after `|` MUST be the compact JWE serialization carrying that recipient's CKE

Multiple recipients are specified by articulating multiple values under the same stable key, for example:

```text
envelope::recipients + "550e8400-e29b-41d4-a716-446655440010|<jwe-for-recipient-1>"
envelope::recipients + "550e8400-e29b-41d4-a716-446655440011|<jwe-for-recipient-2>"
```

and in a no-CAK but still recipient-declared case:

```text
envelope::recipients + "550e8400-e29b-41d4-a716-446655440010"
envelope::recipients + "550e8400-e29b-41d4-a716-446655440011|<optional-jwe-for-recipient-2>"
```

This preserves the `envelope::key` namespace style while keeping the encoding legal under the Layer-2 grammar and directly compatible with the current shared Layer-3 attribute reduction model.

### 3. Keyframe payload carries the CAK public JWK when required

When a Keyframe carries a CAK, the Keyframe payload should carry the CAK public JWK directly.

The intended pattern is the same as the current Certificate Artipoint pattern:

- Certificate canonical form uses `["certificate", <label>, json:{ <JWK> }]`
- Certificate payload MUST contain a valid JWK

The Keyframe should follow the same payload model for CAK public-key publication:

- the payload SHOULD be `json:{ <CAK public JWK> }`
- when present, the payload MUST contain a valid public JWK suitable for the CAK profile in effect

This aligns Keyframe public-key publication with existing ASCP public-key constructs and matches the implementation posture already used by `IdentityDirectory`, which materializes certificate public keys from node payload rather than from auxiliary attributes.

### 4. Empty Keyframe payload means no CAK is present

An empty Keyframe payload implies that the Keyframe carries no CAK public-key material.

Under this model:

- payload present with JWK => this Keyframe carries CAK public-key material
- payload empty => this Keyframe carries no CAK

CAK presence is therefore determined by payload presence, not by `crypto_profile` alone.

### 5. Keyframe-side `cak_kid` is unnecessary

The Keyframe-side `cak_kid` attribute should be removed from the Keyframe contract.

Its value is deterministically derived from the Keyframe Artipoint UUID:

- `ascp:cak:<keyframe-uuid>`

This is consistent with the existing ASCP `kid` model and keeps the Keyframe articulation from restating information already implied by its identity.

### 6. Keyframe attributes are simplified to profile plus recipients

Under the profile-driven model, the Keyframe should not need to articulate low-level crypto policy directly through several separate attributes.

The intended Keyframe attribute set becomes:

- `crypto_profile`
- `envelope::recipients`

Under this proposal:

- `crypto_profile` names the ASCP profile whose rules define the lower-layer cryptographic behavior
- `envelope::recipients` carries recipient-specific recipient state, including optional wrapped key material when present

Accordingly, Keyframe-side articulation no longer needs to carry separate low-level policy attributes such as:

- `payload_cipher`
- `message_signing`
- `channel_access_alg`
- `cak_kid`
- `cak_public_jwk`

## Proposed Operator Model

The proposed recipient-envelope encoding should participate in the normal Layer-2 attribute operator model using the single stable key `envelope::recipients`.

The intended model is:

- `+` is the canonical articulation form for recipient-envelope state
- each `+` adds one encoded recipient entry to the effective articulated set
- `:=` is legal and resets the effective set to the supplied encoded value, but it is not the ordinary authoring form
- `-` is legal under generic Layer-2 attribute mechanics but SHOULD NOT be used as the ordinary mechanism for removing recipients from a channel keyframe

Ordinary recipient removal should occur by introducing a new Keyframe rather than subtractively mutating the retained recipient set of an existing Keyframe. This reflects the actual security model: once a recipient should no longer be able to decrypt future channel traffic, the channel should rotate to a new Keyframe carrying the new effective recipient set.

Under the recipient-entry encoding rules:

- if the Keyframe payload carries CAK public JWK, each effective recipient entry MUST include JWE
- if the Keyframe payload is empty, recipient entries MAY be identity-only or identity-plus-JWE
- no-CAK Keyframes MAY still carry recipient JWE because encryption remains optionally supported even without a CAK
- when JWE is present, it remains meaningful regardless of whether the Keyframe carries a CAK

## Rationale

This direction fits the ASCP architecture better for several reasons.

### 1. It resolves the grammar inconsistency directly

`envelope::recipients` is a valid `class::key` attribute name under the existing Layer-2 grammar.

### 2. It aligns Keyframes with the existing Certificate public-key pattern

Certificate and RootCA Artipoints already use payload-carried JWKs. Keyframes should use the same public-key publication shape whenever they carry CAK public-key material.

### 3. It removes redundant Keyframe-side identifiers

The CAK `kid` is derivable from the Keyframe UUID, so articulating it again on the Keyframe adds no independent semantic value.

### 4. It preserves the existing semantic namespace

The proposal retains the `envelope::...` class-qualified naming style already established by the Trust and Identity design.

The same principle applies to `member` attribute values: the stable attribute key remains `member`, and the referenced UUID is encoded as a string value.

### 5. It keeps discovery concerns and semantic Keyframe concerns cleanly separated

`channel-ref` remains the discovery and bootstrap carrier for `cak_kid` and `cak_public_keys`, while the Keyframe becomes the semantic carrier of the current epoch's CAK state, profile, and recipient state.

### 6. It aligns with the implementation direction while simplifying it

The current implementation already treats recipient envelope state as aggregate keyframe-side state and already derives canonical `kid` strings from UUID-based identifiers. This proposal regularizes that direction and removes the redundant Keyframe-side CAK attribute layer.

### 7. It matches the profile-driven authoring model

A named `crypto_profile` is a better articulation boundary than repeating low-level cryptographic policy fields on each Keyframe.

## Proposed Specification Change

If this issue is accepted, the Trust and Identity and related Keyframe specifications should evolve as follows.

### Keyframe payload

- A Keyframe payload MAY be empty.
- An empty Keyframe payload means the Keyframe carries no CAK.
- When a Keyframe carries a CAK, the Keyframe payload MUST contain a valid public JWK for that CAK, following the same payload pattern used by Certificate Artipoints.

### Keyframe attributes

- Keyframes SHOULD articulate `crypto_profile` as the profile selector for the cryptographic behavior of the Keyframe epoch.
- Keyframes SHOULD articulate recipient-specific wrapped material using `envelope::recipients`.
- Keyframes SHOULD NOT articulate `cak_kid` as a Keyframe attribute.
- Keyframes SHOULD NOT articulate `cak_public_jwk` as a Keyframe attribute.
- Keyframes SHOULD eventually stop articulating low-level policy attributes such as `payload_cipher`, `message_signing`, and `channel_access_alg` once those concerns are fully owned by the profile contract.

### Recipient-envelope entries

- The stable keyframe recipient-envelope attribute name is `envelope::recipients`.
- Each articulated value is one encoded UTF-8 string of the form `"<identity_uuid>"` or `"<identity_uuid>|<jwe>"`.
- `identity_uuid` identifies the intended recipient Identity Artipoint.
- when present, `jwe` carries that recipient's wrapped CKE.
- If the Keyframe payload carries CAK public JWK, each effective recipient entry MUST include `jwe`.
- If the Keyframe payload is empty, recipient entries MAY omit `jwe` or include it.
- Implementations MUST interpret the effective recipient-envelope set from the value history of `envelope::recipients`.
- `+` is the canonical articulation operator for recipient-envelope entries.
- `:=` is legal and resets the effective recipient set to the supplied encoded value, but is not the ordinary authoring form.
- `-` SHOULD NOT be used as the ordinary mechanism for removing recipients from future channel traffic.
- Recipient removal for ordinary channel evolution SHOULD occur through Keyframe rotation.

### `member` attribute values

- Membership articulation SHOULD use the stable attribute key `member`.
- Each `member` value SHOULD be a quoted UUID string, for example `member + "<uuid>"` or `member - "<uuid>"`.
- The quoted UUID string identifies the referenced Artipoint according to the owning Layer-3 semantic profile.
- Layer-3 consumers should canonicalize valid UUID strings according to their owning RFC.
- Layer-3 consumers should reject or skip malformed `member` values according to their owning RFC.

### Channel Reference

- No changes are proposed to the required `channel-ref` discovery attributes.
- `cak_kid` and `cak_public_keys` remain correct and required on Channel Reference Artipoints.
- `cak_kid` is empty when there is no active CAK.
- `cak_public_keys` is empty when the channel has never had any CAKs.
- `cak_public_keys` may remain non-empty when there is no active CAK but the channel has historical CAKs.

## Affected Specifications

If this issue is accepted, the following specifications require inspection and likely update:

- `docs/ascp/ascp-trust-and-identity-architecture.md`
- `docs/ascp/ascp-governance-and-access-control.md`
- `docs/rfc-ascp-layer-3-directories.md`

Additional specifications should be inspected for references to:

- `envelope::<recipient>`
- `envelope::<recipient-identity-uuid>`
- Keyframe-side `cak_public_jwk`
- Keyframe-side `cak_kid`
- low-level Keyframe crypto-policy attributes
- Keyframe payload shape and emptiness semantics
- `member` attribute examples using unquoted UUID placeholders such as `member + <uuid>`

The Layer-2 grammar and Layer-2 codec specifications should also be inspected for references, examples, or explanatory text that may implicitly assume the older Trust and Identity encoding. However, this proposal does not currently imply a normative Layer-2 grammar change and does not appear to require a functional Layer-2 codec change.

## Affected Code Areas

If this issue is accepted, the following implementation areas require inspection and likely update:

- `src/ascp/layer3/channel_directory.py`
- `src/ascp/layer3/identity_directory.py`
- tests and fixtures across Layer-2, Layer-3, crypto-context, and RFC-alignment coverage

The main follow-up work would include:

- keeping `channel-ref` CAK discovery articulation unchanged
- moving Keyframe-side CAK public-key publication from `cak_public_jwk` attribute form into Keyframe payload form
- making CAK presence explicitly payload-driven and optional
- removing Keyframe-side `cak_kid` articulation
- updating Keyframe materialization to read CAK public JWK from payload rather than attribute state
- preserving the current `envelope::recipients` articulation model
- updating that recipient-envelope model so `"<identity_uuid>"` and `"<identity_uuid>|<jwe>"` are both legal forms with conditional JWE requirements
- updating `member` attribute examples and materialization rules to use quoted UUID string values such as `member + "<uuid>"`
- introducing `crypto_profile` as the Keyframe-side profile selector
- clarifying empty-value semantics for `channel-ref` `cak_kid` and `cak_public_keys`
- updating RFC and spec examples to use the revised Keyframe shape

Layer-2 parser, renderer, and draft-model code should still be inspected for dependencies, fixtures, or assumptions exposed by test coverage or example material. However, because `envelope::recipients` is already a legal `class::key` attribute name, its encoded value is already representable as an ordinary string payload, and Keyframe payloads already support JSON JWK content under existing Layer-2 rules, no normative or functional Layer-2 implementation change is currently expected.

## Resolved Decisions

- This issue is about broader Keyframe encoding and payload shape, not only recipient-envelope naming.
- Channel Reference Artipoints remain unchanged.
- `cak_kid` and `cak_public_keys` remain correct and required on `channel-ref`.
- `envelope::recipients` is the stable Keyframe recipient-envelope attribute name.
- Each articulated `envelope::recipients` value carries exactly one encoded recipient-envelope entry.
- The explicit delimiter between `identity_uuid` and `jwe` is `|`.
- `+` is the canonical operator for recipient-envelope articulation.
- `:=` is legal as a reset form but is not the ordinary authoring pattern.
- `-` is not the ordinary removal path for channel evolution.
- Recipient removal for future channel traffic should occur through Keyframe rotation.
- When a CAK is required, the Keyframe payload should carry the CAK public JWK using the same payload pattern used by Certificate Artipoints.
- An empty Keyframe payload means the Keyframe carries no CAK.
- CAK presence is driven by Keyframe payload presence, not by `crypto_profile` alone.
- Keyframe-side `cak_kid` is redundant and should be derived from the Keyframe UUID rather than articulated.
- The intended long-term Keyframe attribute set is `crypto_profile` plus recipient-envelope state.
- `envelope::recipients` entries may be identity-only or identity-plus-JWE.
- When a Keyframe carries CAK public-key material, recipient JWE is required.
- When a Keyframe carries no CAK, recipient JWE remains optional but still meaningful when present.
- `channel-ref` `cak_kid` may be empty when there is no active CAK.
- `channel-ref` `cak_public_keys` may be empty when the channel has never had any CAKs and may remain non-empty for historical CAKs.
- Membership references use stable `member` attributes with quoted UUID string values such as `member + "<uuid>"` and `member - "<uuid>"`.
- `member` attribute UUID strings are canonicalized by the Layer-3 consumer that owns the corresponding directory or materialized view.
