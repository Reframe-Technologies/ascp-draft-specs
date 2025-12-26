# ASCP Channels: Secure Distribution Layer Specification

**Layer 1 of the ASCP Protocol Stack**

**Public Comment Draft -** *Request for community review and collaboration*

Version: 0.57 — Informational (Pre-RFC Working Draft)  
December 2025

**Editors:** Jeffrey Szczepanski, Reframe Technologies, Inc.; contributors

# **1. Status of This Document**

This document specifies the **ASCP Channels: Secure Distribution Layer**, defining the **Layer-1 Channel Encoder and Channel Decoder (Channel Codec)** for the Agents Shared Cognition Protocol (ASCP).

This document is published as a working draft for technical review and implementation feedback. It has not undergone IETF review and has no formal standing within the IETF standards process.

The key words **“MUST”**, **“MUST NOT”**, **“REQUIRED”**, **“SHALL”**, **“SHALL NOT”**, **“SHOULD”**, **“SHOULD NOT”**, **“RECOMMENDED”**, **“MAY”**, and **“OPTIONAL”** in this document are to be interpreted as described in **RFC 2119** and **RFC 8174**.

Normative requirements in this document apply **only to Layer-1 behavior**: the production and validation of **Channel Envelopes** and the resulting **Artipoint Records** appended to a **Channel Log**. This document does not define Layer-0 log synchronization behavior, Layer-2 grammar semantics, or Layer-3 meaning, governance, or trust evaluation.

# **2. Abstract**

The ASCP Channels specification defines the **Layer-1 Secure Distribution Layer (Channel Codec)**. It specifies how a **Channel Encoder** transforms a validated **Articulation Sequence** (Layer-2) into a **Channel Envelope** (JOSE **JWS**, optionally wrapped in **JWE**) and then into an **Artipoint Record** suitable for append to a **Channel Log**, and how a **Channel Decoder** performs the inverse transformation.

Layer-1 Channels provide cryptographic integrity and verifiable authorship, and **MAY** provide confidentiality to enforce **visibility scope**. Layer-1 operates only over provisioned key material and other **external configuration inputs**, and does not define membership, governance rules, or semantic interpretation of articulated content.

All ordering, replication, and convergence properties are delegated to other protocol layers and are treated here solely as external inputs.

# **3. Introduction**

## **3.1 Motivation: Secure Distribution of Articulated Context**

The Agents Shared Cognition Protocol (ASCP) requires a secure and deterministic mechanism for distributing articulated context between humans and agents. Articulation Sequences must be conveyed in a manner that preserves authorship integrity, enforces visibility constraints, and produces durable records suitable for later replay and interpretation.

The ASCP Channels layer provides this capability at **Layer-1** by defining the cryptographic encoding and decoding of articulated coordination. Channels ensure that articulated contributions can be securely packaged, transmitted, and verified without embedding or interpreting semantic meaning. This allows higher layers to reason over a stable, tamper-evident record of articulation while remaining decoupled from transport and cryptographic mechanics.

## **3.2 Position of Channels in the ASCP Layer Model**

A Channel is a **semantic construct defined at Layer-3** (specifically, a **Distribution Construct**) whose semantics are realized mechanically at Layer-1 through cryptographic encoding.

At Layer-1, Channels are implemented as an **append-only sequence of Artipoint Records** in a **Channel Log**. Each Artipoint Record encapsulates a serialized **Articulation Sequence** within a **Channel Envelope** and applies:

- cryptographic signatures to preserve authorship integrity, and
- optional encryption to enforce **visibility scope**.

Layer-1 Channels do not interpret the contents of Articulation Sequences, nor do they evaluate membership, governance, or semantic meaning. Any such parameters are treated here solely as **external configuration inputs**.

This strict separation ensures that cryptographic enforcement remains mechanical and verifiable, without coupling Layer-1 to semantic interpretation, governance models, or application behavior.

## **3.3 Scope of This Specification**

This specification defines the **Layer-1 Secure Distribution Layer** of ASCP, including:

- the Channel Encoder and Channel Decoder responsibilities;
- the cryptographic model used to sign and optionally encrypt Articulation Sequences;
- the normative structure of Channel envelopes and Artipoint Records;
- the mechanical handling of Keyframes as cryptographic configuration inputs;
- sender and receiver processing rules for encoding, verifying, decrypting, and decoding records.

This document specifies **how** articulated context is cryptographically realized for distribution, not **why** particular Channel configurations or policies exist.

## **3.4 Out of Scope**

This specification explicitly does **not** define:

- Channel semantics, audience meaning, or visibility intent;
- membership, participation, authority, or governance rules;
- identity trust roots, PKI models, or endorsement semantics;
- interpretation or materialization of Artipoint semantics;
- application-level behavior, user experience, or workflow logic;
- log ordering, replication, convergence, or synchronization behavior.

Keyframes, while expressed as Artipoints at Layer-2 and interpreted semantically at Layer-3, are consumed here **only** for their cryptographic effects. The legitimacy, intent, or governance rationale for Keyframes is outside the scope of this document.

# **4. Terminology**

The following terms are used normatively in this specification. Terminology follows the **ASCP Terminology Primer**; definitions here are scoped explicitly to **Layer-1 responsibilities**.

## **4.1 Channel**

A **Channel** is a **semantic construct** (specifically, a **Distribution Construct**) defined at Layer-3 whose semantics are realized at Layer-1 through cryptographic encoding and decoding.

At Layer-1, a Channel is realized through the processing of **Channel Envelopes** into **Artipoint Records** associated with a **Channel Log** by the **Channel Encoder** and **Channel Decoder**.

## **4.2 Channel Envelope**

A **Channel Envelope** (or **Envelope**) is the **JOSE-encoded container** produced and consumed by the Channel Encoder and Channel Decoder.

An Envelope encapsulates a serialized Articulation Sequence and applies:

- **JWS** for authorship integrity, and
- **optional JWE** for payload confidentiality and visibility enforcement.

Channel Envelopes are the only objects submitted for append and replicated within a Channel Log; once appended, they constitute **Artipoint Records**.

This term refers exclusively to the JOSE-encoded record format handled at Layer-1. It does not include governance artifacts, Keyframe Artipoints, or other Layer-2 structures used to provision cryptographic material.

## **4.3 Articulation Sequence**

An **Articulation Sequence** is an ordered collection of one or more Articulation Statements authored together by a single identity.

At Layer-1, the Articulation Sequence is treated as an **opaque cleartext payload**. It is serialized, signed, optionally encrypted, and later decoded, but **never parsed or interpreted** by this layer.

> *See: ASCP Artipoint Grammar Specification.*

## **4.4 Artipoint Record**

An **Artipoint Record** is the durable, cryptographically secured materialization of an Articulation Sequence produced by the Channel Encoder and appended to a Channel Log.

An Artipoint Record is defined strictly as the persisted form of a Channel Envelope once appended to a Channel Log.

The terms “Channel Envelope” and “Artipoint Record” refer to the same object at different stages of its lifecycle: pre-append (Envelope) and post-append (Record).

## **4.5 Replica**

A **Replica** is a local instance that stores and processes a Channel Log.

Replicas exchange Channel Envelopes using the **ASCP Log-Sync Protocol (ALSP)**. Layer-1 does not define replication topology, ordering guarantees, or convergence behavior.

## **4.6 Keyframe**

A **Keyframe** is a Layer-2 encoded Artipoint whose semantic interpretation defines cryptographic configuration parameters for a Channel (e.g., key versions, algorithms, recipient sets).

Layer-1 **does not parse or evaluate Keyframes**. Instead, it consumes cryptographic material and parameters provisioned by higher layers as a result of Keyframe evaluation.

## **4.7 Key Identifier (kid)**

A **Key Identifier (kid)** is a structured, Layer-1–interpreted identifier used in JOSE headers to deterministically select cryptographic key material.

The `kid` value is parsed and validated by the Channel Codec according to the encoding rules defined in Section 6.4. While its structure is interpreted for key lookup, it carries no semantic meaning regarding membership, authority, or governance.

## **4.8 Channel AES Key**

The **Channel AES Key** is the symmetric encryption key used to encrypt and decrypt JWE-protected Channel Envelopes for a specific key version.

This key is provisioned to Layer-1 as the result of higher-layer evaluation and is not derived from payload content or inferred by this specification. When used with AES-GCM, the Channel AES Key MUST be combined with a fresh Initialization Vector (IV) for each encryption operation.

## **4.9 Channel Access Key (CAK)**

The **Channel Access Key (CAK)** is a transport-level credential used to authorize participation in Channel Log replication.

The CAK governs **replication eligibility only**. It does not convey semantic membership, authorship authority, or governance rights.

## **4.10 Author**

The **Author** is the **Layer-3 semantic identity** whose cryptographic key is used to produce the JWS signature over the Articulation Sequence contained within an Envelope.

At Layer-1, authorship is enforced **cryptographically** via verification of a JWS signature produced using key material bound to the Author’s identity. Layer-1 verifies signatures but does not evaluate authorship legitimacy or authority.

## **4.11 Sender**

The **Sender** is the **Layer-1 operational role** that constructs a Channel Envelope.

The Sender:

- serializes an Articulation Sequence,
- applies a JWS signature using the Author’s key material,
- optionally encrypts the payload using the Channel AES Key, and
- submits the resulting record to the Channel Log.

The Sender MAY be the Author or a delegated agent acting on the Author’s behalf. Sender identity has no semantic significance at Layer-1 beyond correct cryptographic processing.

## **4.12 Recipient (Recipient Set)**

A **Recipient** is an identity for which a decryptable JWE envelope is produced.

Recipients are supplied to Layer-1 as part of cryptographic provisioning derived from higher-layer evaluation. All Recipients associated with a given key version share access to the same Channel AES Key.

Layer-1 does not determine Recipient membership; it enforces encryption strictly for the provided Recipient Set.

## **4.13 ASCP Log-Sync Protocol (ALSP)**

The **ASCP Log-Sync Protocol (ALSP)** is the **Layer-0 protocol** responsible for durable, ordered replication of Channel Logs between Replicas.

ALSP does not interpret Channel Envelopes and does not perform cryptographic validation.

## **4.14 Channel Log**

A **Channel Log** is a **Coordination Log** scoped to a single Channel: the append-only sequence of **Artipoint Records** maintained for that Channel.

Layer-1 appends records to the Channel Log but does not define ordering guarantees, convergence properties, or replication mechanics.

## **4.15 JOSE (JWS and JWE)**

**JOSE** refers to the JSON Object Signing and Encryption standards used by ASCP Channels.

- **JWS** provides cryptographic integrity and authorship verification.
- **JWE** provides optional confidentiality and visibility enforcement.

Algorithm requirements and constraints are defined in Section 6.

## **4.16 Initialization Vector (IV)**

An **Initialization Vector (IV)** is a cryptographic nonce used as an input to authenticated encryption algorithms, specifically **AES-GCM** as used by ASCP Channel Envelopes.

At Layer-1, the IV is:

- generated by the Channel Encoder during JWE construction,
- carried explicitly as a component of the JWE Compact Serialization,
- validated by the Channel Decoder during decryption.

IVs are **not secret** but **MUST be unique per encryption operation** when used with the same symmetric key. IV reuse under the same Channel AES Key is cryptographically unsafe and violates this specification.

IV generation, size, and validation requirements are defined normatively in Sections **6.3.2** and **8.3.4**.

# **5. Architectural Overview**

This section describes the **architectural role and layer boundaries** of ASCP Channels. It is informational and intended to clarify how the Layer-1 Channel Encoder and Channel Decoder relate to adjacent layers.

## **5.1 Channels as the Layer-1 Secure Distribution Mechanism**

At Layer-1, ASCP Channels provide the **cryptographic realization of secure distribution** for articulated context.

A Channel is realized mechanically as an **append-only sequence of Artipoint Records** in a **Channel Log**. Each record encapsulates a **Channel Envelope** that applies JOSE **JWS** signatures and **MAY** apply JOSE **JWE** confidentiality.

Layer-1 Channels are intentionally minimal. They treat articulated content as opaque payload and are concerned solely with cryptographic correctness, envelope admissibility, and durable recording of envelopes. Channels do not interpret, validate, or transform the semantic content of payloads.

## **5.2 Relationship to Layer-0 (ALSP)**

Layer-1 Channels operate above the **ASCP Log-Sync Protocol (ALSP)**.

ALSP is responsible for durable, ordered replication of **Artipoint Records** in **Channel Logs** between Replicas. ALSP does not interpret **Channel Envelopes** and does not perform cryptographic processing or validation of articulated content. It defines message exchange, history synchronization, and convergence behavior, but performs no cryptographic processing or validation of the articulated content.

Layer-1 relies on ALSP as an external service that delivers and persists Channel Envelopes. This specification does not define or constrain ALSP behavior beyond assuming that envelopes delivered to the Channel Decoder originate from an append-only log.

## **5.3 Relationship to Layer-2 (Artipoint Grammar)**

The payload carried within a Channel Envelope is an **Articulation Sequence** expressed using the ASCP Artipoint Grammar.

Layer-1 does not parse, validate, or interpret this grammar. From Layer-1’s perspective, the payload is an opaque **serialized octet sequence** representing an Articulation Sequence, which is signed, optionally encrypted, and later verified and decoded.

All articulation structure, syntax, and semantic meaning are defined at Layer-2 and above.

## **5.4 Relationship to Layer-3 (Semantic Evaluation)**

Layer-3 is responsible for interpreting articulated content and deriving meaning, governance state, and effective collaboration semantics.

Layer-1 does not participate in this evaluation. Instead, it is **parameterized by the outputs of Layer-3**, such as which cryptographic keys to use and which identities must be able to decrypt a given envelope.

This separation ensures that the secure distribution mechanism operates deterministically and independently of semantic models, governance rules, and application behavior.

## **5.5 Channels Do Not Interpret Semantic Governance**

Channels maintain a strict separation from semantic governance.

The Channel Encoder and Channel Decoder:

- MUST NOT parse governance attributes,
- MUST NOT evaluate membership, roles, or authority,
- MUST NOT reject envelopes based on semantic meaning or legitimacy,
- MUST NOT derive or infer policy decisions.

Their responsibilities are limited to cryptographic validation, optional encryption, and correct handling of Channel Envelopes as records.

All determinations of meaning, authorization, and consequence occur outside Layer-1.

## **5.6 Immutability and Multi-Writer Operation**

Layer-1 MUST treat all Channel Envelopes delivered from the Channel Log as immutable inputs and MUST NOT modify, rewrite, or remove them.

Layer-1 supports **multi-writer operation** by processing any envelope that satisfies the cryptographic requirements defined in this specification. Decisions about *who ought to produce envelopes* are made at higher layers; Layer-1 enforces only the cryptographic consequences of those decisions.

Layer-1 does not define ordering guarantees, conflict resolution, or convergence behavior. It processes envelopes in the order delivered by the underlying log replication mechanism and preserves them without reordering or modification.

# **6. Cryptographic Foundations**

This section specifies the **Layer-1 cryptographic codec** for ASCP Channels. It defines how the Channel Encoder and Channel Decoder use JOSE (JWS/JWE) to produce and validate Channel Envelopes, including **required algorithms**, **header requirements**, and the **ASCP** `kid` **encoding** used for deterministic key selection.

## **6.1 JWS for Authorship Signatures**

Every envelope written to a Channel **MUST** be signed using **JWS Compact Serialization** as defined in **RFC 7515 §3.1**.

The JWS signature provides integrity protection over the JOSE protected header and the payload, and enables cryptographic verification using key material selected via `kid`.

Layer-1 **MUST**:

- parse the JWS Compact Serialization form and validate it has exactly three base64url segments (`header.payload.signature`),
- base64url-decode the protected header and validate it is valid JSON,
- require the protected header to include at least `alg`, `kid`, and `typ`,
- require `typ` to indicate a JWS-bearing ASCP envelope (see §6.3.4),
- select the verification key using the JOSE `kid` value (see §6.4),
- verify the JWS signature according to the `alg` header,
- reject any envelope with a missing, malformed, or unresolvable `kid`,
- treat the JWS payload as **opaque byte content** (Layer-1 MUST NOT parse or interpret payload grammar),
- reject envelopes using signature algorithms not permitted by §6.3.1.

Layer-1 key selection for JWS verification is a codec operation. Implementations **MUST** maintain a key table that resolves `kid` values to the appropriate verification material required by the selected `alg`.

**Note (informative):** The provenance of keys (e.g., which higher-layer objects asserted them) is not evaluated by Layer-1. Layer-1’s obligation is deterministic selection and cryptographic verification based on provisioned key tables.

## **6.2 JWE for Optional Payload Confidentiality**

Channels **MAY** encrypt signed payloads using **JWE Compact Serialization** as defined in **RFC 7516 §3.1**.

When used, JWE encryption **MUST** follow the **sign-then-encrypt** pattern:

1. Construct a JWS envelope (per §6.1).
2. Use the resulting JWS Compact Serialization string as the plaintext input to JWE.
3. Encrypt using the Channel AES Key selected by `kid`.

Layer-1 **MUST**:

- parse the JWE Compact Serialization form and validate it has exactly five base64url segments (`header.encrypted-key.iv.ciphertext.tag`),
- base64url-decode the protected header and validate it is valid JSON,
- require the protected header to include at least `alg`, `enc`, `kid`, and `typ`,
- require `typ` to indicate a JWE-wrapped ASCP envelope (see §6.3.4),
- require `enc` to be a permitted content encryption algorithm (§6.3.2),
- require `alg` to be permitted for the specific envelope profile in use (see below),
- select the decryption key using `kid` (see §6.4),
- validate IV requirements (see §6.3.2 and §8.3.4),
- perform authenticated decryption and reject the envelope if authentication fails,
- require the decrypted plaintext to be a valid JWS Compact Serialization string and then apply §6.1 verification,
- reject any envelope that applies encryption without a valid JWS layer.

### JWE Profiles Used by ASCP Channels

ASCP uses JWE in two distinct contexts:

1. **Envelope Payload Confidentiality (group confidentiality)**
   - Encrypts the **JWS Compact Serialization** produced for a given Channel using the currently provisioned **Keyframe-defined cryptographic state**.
   - Uses **direct** symmetric key mode (`alg = "dir"`), with `kid` selecting the appropriate key material for payload confidentiality.
2. **Key Distribution / Wrapping (per-recipient key envelopes)**
   - Uses JWE key agreement / wrapping to deliver Channel key material to Recipients.
   - This profile constrains `alg` to permitted key agreement / wrapping algorithms and uses `kid` to select recipient-specific public key material.

The concrete envelope structures and header profiles are specified below; this section defines the cryptographic constraints and permitted algorithms.

## **6.3 Supported Algorithms and Header Requirements**

This section defines the permitted algorithm set and JOSE header constraints for ASCP Channels. Implementations **MUST NOT** rely on runtime algorithm negotiation; the algorithm set is fixed by this specification for interoperability.

### **6.3.1 JWS Signature Algorithms (Required)**

Every Channel envelope **MUST** be signed using one of the following JWS signature algorithms:

**Required:**

- `ES256` (ECDSA using P-256 and SHA-256), as defined in **RFC 7518**
- `EdDSA` (Ed25519), as defined in **RFC 8037**

**Optional:**

- `ES384` (ECDSA using P-384 and SHA-384), as defined in **RFC 7518**

Envelopes using algorithms not listed above **MUST** be rejected.

### **6.3.2 JWE Content Encryption Algorithms**

When encrypting Channel Envelopes using JWE, the content encryption algorithm **MUST** be:

- `A256GCM` (AES-256-GCM), as defined in **RFC 7518**

AES-GCM encryption as used by ASCP Channel Envelopes **MUST** be performed with an explicit **Initialization Vector (IV)** as defined in Section **4.16**.

The Channel Encoder **MUST** generate a fresh, cryptographically random IV for each encryption operation performed with a given Channel AES Key. Reuse of an IV with the same Channel AES Key **MUST NOT** occur.

The Channel Decoder **MUST** validate that an IV is present and correctly formed as part of the JWE Compact Serialization and **MUST** reject any envelope for which authenticated decryption fails.

### **6.3.3 Key Agreement / Wrapping Algorithms**

Key agreement and key wrapping algorithms are used **only** for per-recipient key distribution / wrapping envelopes. They **MUST NOT** be used directly to encrypt articulated payloads (i.e., the JWS payload layer).

**Required:**

- `ECDH-ES` (Elliptic Curve Diffie-Hellman Ephemeral-Static), as defined in **RFC 7518 §4.6**

**Optional:**

- `ECDH-ES+A256KW` (ECDH-ES with AES-256 Key Wrap), as defined in **RFC 7518**

Envelopes specifying unsupported or incompatible algorithms **MUST** be rejected.

### **6.3.4 JOSE Header Requirements**

All ASCP Channel envelopes **MUST** conform to the following JOSE header requirements (per **RFC 7515**, **RFC 7516**, and **RFC 7517**):

- `kid` **MUST** be present.
- `alg` **MUST** be present.
- `typ` **SHOULD** be present and, if present, MUST be one of:
  - `"ascp+jws"` for JWS-only envelopes,
  - `"ascp+jws+jwe"` for JWE-wrapped JWS envelopes,
  - `"ascp+jwe"` for JWE key-envelope objects when used (if applicable to implementation profiles). This value MUST NOT be used for Articulation Sequence payloads.

For JWE envelopes, `enc` **MUST** be present.

Header validation errors **MUST** result in envelope rejection for Layer-2 handoff. (Layer-0 log retention is outside this section.)

### **6.3.5 Summary of JOSE Usage in ASCP**

| Use Case                             | Protocol | Format                | Key Material Selected via `kid`                           |
| ------------------------------------ | -------- | --------------------- | --------------------------------------------------------- |
| Sign Articulation Sequence payload   | JWS      | Compact Serialization | `ascp:cert:<uuid>` → author signing/verification material |
| Encrypt signed payload (JWS layer)   | JWE      | Compact Serialization | `ascp:keyframe:<uuid>` → payload confidentiality key      |
| Wrap/Distribute Channel key material | JWE      | Compact Serialization | `ascp:cert:<uuid>` → recipient key agreement/wrapping key |
| Key Representation                   | JWK      | JSON                  | As defined in **RFC 7517**                                |

## 6.4 `kid` Format and Interpretation

ASCP defines a structured `kid` namespace for **deterministic key selection** by the Layer-1 codec. The `kid` value is **not opaque** to Layer-1: Layer-1 MUST parse and validate the `kid` string format defined in this section and use it to select cryptographic material from provisioned key tables.

While `kid` is optional in JOSE specifications, ASCP **MUST** include `kid` in all Channel Envelopes to ensure deterministic key selection during key rotation and multi-writer operation.

Layer-1 uses `kid` **only** for key selection and compatibility checks required by cryptographic processing. Layer-1 MUST NOT interpret `kid` as expressing membership, authorization, governance intent, or trust semantics.

### **6.4.1** `kid` **Format Specification**

The use of `kid` conforms to **RFC 7515 §4.1.4**.

The `kid` value **MUST** be a string formatted according to one of the following forms:

```
ascp:<type>:<uuid>
ascp:<type>:<index>
```

Where:

- `ascp` identifies this as an ASCP key identifier namespace,
- `<type>` indicates the class of cryptographic material being referenced,
- `<uuid>` is a UUID string (RFC 4122 textual form),
- `<index>` is a zero-based integer index into a bootstrap key package.

Layer-1 **MUST** reject any `kid` that does not match these forms exactly.

### **6.4.2 Supported** `kid` **Types**

`ascp:cert:<uuid>` **— Certificate Reference**

- Selects verification material for JWS signature verification, and key agreement/wrapping material for per-recipient key envelopes.
- The key type and curve requirements (if any) are implied by the chosen `alg` and JOSE processing rules.

`ascp:keyframe:<uuid>` **— Keyframe Reference**

- Selects the key material used for JWE payload confidentiality and decryption for envelopes associated with that Keyframe-defined cryptographic state.
- Used with `alg = "dir"` and `enc = "A256GCM"` for envelope payload confidentiality.

`ascp:bkp:<index>` **— Bootstrap Key Package Reference**

- Selects a key from a bootstrap key package indexed by `keys[]`.
- When this form is used, provisioning MUST include a bootstrap key table for the channel context such that lookup by `<index>` is deterministic.

### **6.4.3 JOSE Header Visibility Note**

JOSE protected headers are integrity-protected but not encrypted. The `kid` value is therefore visible prior to decryption and enables deterministic key selection without exposing payload contents.

### **6.4.4 Usage Examples**

**JWS Signing**

```json
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "ascp+jws"
}
```

**JWE Channel Envelope Encryption**

```json
{
  "alg": "dir",
  "enc": "A256GCM",
  "typ": "ascp+jws+jwe",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440002"
}
```

**JWE Key Wrapping / Distribution**

```json
{
  "alg": "ECDH-ES+A256KW",
  "enc": "A256GCM",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003",
  "typ": "ascp+jwe"
}
```

### **6.4.5 Lookup Process (Normative)**

For any JOSE object processed by Layer-1, Layer-1 **MUST**:

1. parse the `kid` string and validate it conforms to §6.4.1,
2. select the correct key table based on `kid` type (certificate table, keyframe table, or bootstrap table),
3. locate provisioned cryptographic material corresponding to the parsed identifier (UUID or index),
4. validate that the selected material is compatible with the intended cryptographic operation implied by the envelope type and JOSE headers,
5. apply the key material to the cryptographic operation.

Failure at any step **MUST** result in envelope rejection for Layer-2 handoff.

## **6.5 Channel AES Key Requirements**

For each active **Keyframe-state**, Layer-1 uses a symmetric **Channel AES Key** to encrypt and decrypt Articulation Sequence payloads (JWE `alg = "dir"`).

The Channel AES Key:

- **MUST** be generated using a cryptographically secure pseudorandom number generator (CSPRNG),
- **MUST** be 256 bits in length when used with `A256GCM`,
- **MUST** be used exclusively for payload confidentiality via JWE encryption,
- **MUST NOT** be derived from payload content, **envelope headers**, or other protocol-visible material,
- **MUST** be retrievable by Layer-1 via `kid` resolution (`ascp:keyframe:<uuid>`), and
- **MUST** be combined with a fresh Initialization Vector (IV) for each encryption operation, as defined in Section **4.16**.

Provisioning MUST ensure that Layer-1 can retain any historical Channel AES Keys required to decrypt previously written envelopes, as those envelopes remain encoded under the `kid` in effect at time of creation.

## **6.6 Integrity, Authenticity, and Replay Protection**

Channels combine JOSE cryptographic mechanisms with append-only log behavior to provide durable, verifiable history for articulated context.

- **Integrity** is provided by JWS signatures. Any modification to the protected header or payload invalidates the envelope.
- **Authenticity (cryptographic)** is provided by successful verification using the key selected via `kid`.
- **Confidentiality**, when enabled, is provided by JWE using `A256GCM`, the Channel AES Key selected via `kid`, and a fresh Initialization Vector (IV) per encryption operation.
- **Replay and duplication** do not alter the cryptographic validity of envelopes; replay and deduplication policy are not defined by this section. Layer-1’s obligation is deterministic decode/verify based on envelope content and provisioned key tables.
- **Keyframe/Epoch transitions** are expressed in Layer-1 solely through changes in the `kid` used for JWE payload encryption. Envelopes encrypted under a given `kid` remain decryptable only with the corresponding Channel AES Key.

Channels intentionally do not provide per-recipient payload confidentiality for articulated content. Per-recipient confidentiality applies only to **key distribution / wrapping envelopes**, not to the articulated payload envelopes appended to the Channel Log.

# **7. Channel Identity and Layer Interfaces**

This section defines how a **Layer-1 Channel Codec instance** is bound to a specific Channel and how it interfaces with adjacent layers. It clarifies the structural role of Layer-1 within the ASCP stack without restating cryptographic rules (Section 6) or log replication behavior (Layer-0 / ALSP).

## **7.1 Channel Identity (Artipoint Reference)**

A Channel is identified by the UUID of its **Channel Declaration Artipoint**, as defined and interpreted at Layer-3 and represented at Layer-2.

This UUID serves as the **sole identifier** for Channel binding across layers:

- **Layer-1** uses the Channel UUID to select the correct cryptographic state, including key tables, active Keyframes, and historical key material required for verification and decryption.
- **Layer-0** uses the same UUID to associate envelopes with the corresponding append-only Channel Log.

Layer-1 **MUST NOT** derive Channel identity from JOSE headers, payload content, governance attributes, or any other metadata. The Channel UUID is treated as an external binding parameter supplied as part of codec configuration.

## **7.2 Interface to Layer-0 (Log Replication)**

Layer-1 does not own, manage, or interpret append-only logs. Log storage, immutability, ordering, and replication are the exclusive responsibility of **Layer-0 (ALSP)**.

The Layer-1 Channel Codec interacts with Layer-0 solely through the following abstract interfaces:

- **Append Interface:**  
  Layer-1 submits newly constructed Channel Envelopes to Layer-0 for append to the Channel Log.
- **Delivery Interface:**  
  Layer-0 delivers appended Channel Envelopes to Layer-1 for verification and decoding.

Layer-1 **MUST** treat all envelopes received from Layer-0 as immutable inputs and **MUST NOT** modify, reorder, or suppress log entries. Layer-1 processes envelopes strictly in the order delivered by Layer-0.

## **7.3 Interface to Layer-2 (Articulation Grammar)**

Layer-1 serves as the cryptographic boundary between log replication and semantic representation.

- **Upward Interface:**  
  After successful verification (and decryption, if applicable), Layer-1 passes the resulting cleartext **Articulation Sequence** to Layer-2 for grammatical validation and semantic interpretation.
- **Downward Interface:**  
  Layer-1 receives serialized Articulation Sequences from Layer-2 as input to the Channel Encoder when constructing new Channel Envelopes.

Layer-1 **MUST NOT** parse, interpret, or modify Articulation Sequences beyond cryptographic processing. All semantic meaning and grammatical validation are handled exclusively by higher layers.

## **7.4 Channel Access Key (CAK) — Clarification**

The **Channel Access Key (CAK)** is a credential used by **Layer-0** to authorize participation in Channel Log replication.

The CAK:

- is enforced exclusively by Layer-0,
- has no cryptographic role in Channel Envelopes,
- does not grant decryption capability,
- does not grant authorship capability,
- and carries no semantic meaning at Layer-1.

Layer-1 **MUST NOT** consume, validate, or interpret the CAK. It is described here solely to avoid confusion with cryptographic keys used by the Channel Codec.

# **8. Channel Envelope Format**

This section defines the **normative structure and serialization rules** for ASCP Channel Envelopes as processed by the Layer-1 Channel Codec.

Channel Envelopes are realized using **JOSE Compact Serialization** and represent the sole wire format accepted and emitted by Layer-1. This section specifies envelope forms, header fields, field ordering, and structural validation rules. It does **not** redefine cryptographic requirements, algorithm constraints, or key-selection rules, which are defined normatively in Section 6.

The formats defined here apply to:

- JWS-only Channel Envelopes providing integrity protection, and
- JWE-wrapped Channel Envelopes providing confidentiality protection.

Operational processing rules are defined in Section 11.

## **8.1 Envelope Encoding Overview**

Channel Envelopes follow a strict **two-stage encoding model**:

1. **JWS Signature (required)**  
   The Articulation Sequence is serialized and signed using JWS Compact Serialization, producing a three-part structure.
2. **JWE Encryption (optional)**  
   The resulting JWS string MAY be wrapped using JWE Compact Serialization to provide confidentiality.

Both stages MUST use Compact Serialization as defined in RFC 7515 and RFC 7516. No other JOSE serialization forms are permitted for Channel Envelopes.

## **8.2 JWS Envelope Structure**

Every Channel Envelope MUST contain a JWS layer.

### **8.2.1 JWS Compact Serialization**

A JWS Channel Envelope uses the following structure:

```
<header>.<payload>.<signature>
```

Where each component is base64url-encoded.

- **header**: JOSE protected header (JSON)
- **payload**: base64url-encoded serialized Articulation Sequence octets (typically UTF-8 text)
- **signature**: JWS signature computed over header and payload

### **8.2.2 Required JWS Header Fields**

The JWS protected header MUST include:

- `alg`: a permitted JWS algorithm (Section 6.3.1)
- `kid`: key identifier selecting Author verification material (Section 6.4)
- `typ`: `"ascp+jws"`

### **8.2.3 Payload Handling**

The JWS payload MUST be the complete Articulation Sequence encoded as UTF-8 and base64url-encoded. Layer-1 MUST treat the payload as opaque and MUST NOT alter or interpret its contents.

## **8.3 JWE Envelope Structure**

When confidentiality is enabled, the JWS string MUST be encrypted using JWE Compact Serialization.

### **8.3.1 JWE Compact Serialization**

A JWE-wrapped Channel Envelope uses the following structure:

```
<header>.<encrypted-key>.<iv>.<ciphertext>.<tag>
```

Where each component is base64url-encoded.

For **envelope payload confidentiality**, ASCP mandates **direct symmetric key mode** (`alg = "dir"`). In this mode, the `<encrypted-key>` component is empty, resulting in two consecutive period characters as defined in RFC 7516 §7.1.

### **8.3.2 Required JWE Header Fields**

The JWE protected header MUST include:

- `alg`: `"dir"`
- `enc`: `"A256GCM"`
- `kid`: key identifier selecting the Channel AES Key (Section 6.4)
- `typ`: `"ascp+jws+jwe"`

Optional JOSE parameters MAY appear and MUST be preserved if unrecognized.

### **8.3.3 Initialization Vector Placement**

The Initialization Vector (IV) is carried explicitly as the third component of the JWE Compact Serialization. IV requirements are defined normatively in Section 4.16 and Section 6.3.2. This section specifies only its serialized placement.

## **8.4 Header Field Summary**

The following JOSE header fields are used by ASCP Channel Envelopes:

### **8.4.1 Common Fields**

- `kid`: REQUIRED. Parsed and interpreted by Layer-1 for deterministic key selection.
- `typ`: REQUIRED. Identifies the envelope profile.

### **8.4.2 JWS-Specific Fields**

- `alg`: REQUIRED. Signature algorithm identifier.

### **8.4.3 JWE-Specific Fields**

- `alg`: REQUIRED. Must be `"dir"`.
- `enc`: REQUIRED. Must be `"A256GCM"`.
- `zip`: OPTIONAL. Indicates payload compression if present (see Appendix B).

Layer-1 MUST reject envelopes missing required fields or containing structurally invalid headers.

## **8.5 Structural Validation Rules**

Layer-1 MUST validate envelope structure prior to cryptographic processing.

### **8.5.1 JWS Validation**

A JWS envelope is structurally valid only if:

- it contains exactly three base64url segments,
- the protected header decodes to valid JSON,
- required header fields are present.

### **8.5.2 JWE Validation**

A JWE envelope is structurally valid only if:

- it contains exactly five base64url segments,
- the protected header decodes to valid JSON,
- required header fields are present,
- the `encrypted-key` segment is empty when `alg = "dir"`.

Structural validity does not imply cryptographic validity; cryptographic verification is defined in Sections 6 and 11.

## **8.6 Envelope Admissibility**

Only envelopes that satisfy all structural requirements in this section and all cryptographic requirements in Section 6 are admissible for Layer-2 handoff.

Envelopes that fail structural validation MUST be rejected by Layer-1 for semantic processing but MUST NOT be modified or removed from the Channel Log.

# **9. Keyframes and Versioned Cryptographic State**

This section defines how **Layer-1** applies **versioned cryptographic state** during envelope encoding and decoding.

Keyframes themselves are **Layer-2 Artipoints** whose semantic interpretation (Layer-3) yields cryptographic configuration. Layer-1 **does not parse or evaluate Keyframes**; it consumes the resulting provisioned configuration inputs and applies that state deterministically as described in this section.

## **9.1 Keyframe-State Model**

A **Keyframe-state** is the provisioned, versioned set of cryptographic parameters active for a Channel for the purpose of encoding and decoding Channel Envelopes.

Each Keyframe-state is identified by a unique **Keyframe identifier** and is referenced for deterministic key selection during envelope processing (e.g., via `kid` values of the form `ascp:keyframe:<uuid>`).

The following properties define the Keyframe-state model:

- Keyframe-states are **monotonic**: once a Keyframe-state becomes active, it is never replaced or modified.
- Keyframe-states are **forward-only**: newer Keyframe-states may supersede older ones for encoding, but older Keyframe-states remain valid for decoding historical envelopes.
- Keyframe-states are **non-retroactive**: envelopes encoded under a given Keyframe-state are always interpreted using the cryptographic state of that Keyframe-state.
- Multiple Keyframe-states **MAY** be simultaneously required at Layer-1 to support decoding envelopes written under different `kid` values.

Layer-1 **MUST** retain all cryptographic material necessary to process any envelope present in the Channel Log.

## **9.2 Provisioned Cryptographic State**

For each **Keyframe-state**, Layer-1 is provisioned with a set of cryptographic material derived from evaluated Keyframes. This material constitutes the complete cryptographic state required by the Channel Codec.

Provisioned cryptographic state is supplied to Layer-1 via an implementation-defined provisioning interface from higher layers. Layer-1 consumes only cryptographic material and identifiers; it does not receive Grammar structures or governance metadata.

For a given **Keyframe-state**, provisioned state **MUST** include:

- the **Channel AES Key** used for JWE payload encryption and decryption,
- the set of **public verification keys** used to verify JWS signatures referenced by `kid` values,
- any additional key material required for JWE key agreement or wrapping, if applicable.

Provisioned cryptographic state is indexed and selected **exclusively** by `kid` values. Layer-1 **MUST NOT** infer epoch membership or key applicability from envelope content, Grammar attributes, or governance semantics.

### **9.2.1 JWK Representation of Provisioned State (Informative)**

Provisioned cryptographic material is represented to Layer-1 using **JSON Web Keys (JWK)** as defined in **RFC 7517**. The following examples illustrate typical JWK forms corresponding to evaluated Keyframe state.

These examples are **informative** and illustrate the shape of cryptographic material provisioned to the Channel Codec. They do not imply that Layer-1 parses or validates Keyframes themselves.

#### **Channel AES Key (Symmetric Encryption Key)**

```json
{
  "kty": "oct",
  "k": "<base64url AES-256 key>",
  "alg": "A256GCM",
  "kid": "ascp:keyframe:550e8400-e29b-41d4-a716-446655440000",
  "use": "enc"
}
```

This key is selected by Layer-1 when processing JWE Channel Envelopes that reference the corresponding `kid`.

#### **Author Verification Key (Asymmetric Signature Key)**

```json
{
  "kty": "EC",
  "crv": "P-256",
  "x": "<base64url X-coordinate>",
  "y": "<base64url Y-coordinate>",
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "use": "sig"
}
```

Such keys are used by Layer-1 to verify JWS signatures on Channel Envelopes that reference the corresponding `kid`.

#### **Recipient Key Agreement / Wrapping Key**

```json
{
  "kty": "EC",
  "crv": "P-256",
  "x": "<base64url X-coordinate>",
  "y": "<base64url Y-coordinate>",
  "alg": "ECDH-ES",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440002"
}
```

Recipient public keys are selected via `kid` and used for JWE key agreement or wrapping as defined in Section 6. Layer-1 applies these keys solely for cryptographic operations.

## **9.3 Keyframe-State Selection and** `kid` **Usage**

The `kid` value present in a Channel Envelope uniquely determines the **Keyframe-state** used for processing that envelope.

### **9.3.1 Encoding Behavior**

When constructing new Channel Envelopes, the Channel Encoder:

- **MUST** select the currently active **Keyframe-state** for the Channel,
- **MUST** reference that **Keyframe-state** by including its corresponding `kid` value in the JOSE protected header,
- **MUST NOT** reference deprecated or inactive **Keyframe-state** when encoding new envelopes.

### **9.3.2 Decoding Behavior**

When processing received Channel Envelopes, the Channel Decoder:

- **MUST** extract the `kid` value from the JOSE protected header,
- **MUST** select cryptographic material corresponding to that `kid`,
- **MUST** apply verification and decryption using the selected **Keyframe-state** state.

If cryptographic material for a referenced `kid` is unavailable, the envelope is **undecryptable** at Layer-1. Such envelopes MUST be retained in the Channel Log but are not admissible for Layer-2 handoff until the required cryptographic state becomes available.

## **9.4 Keyframe-State Advancement (Key Rotation)**

Key rotation in ASCP Channels is realized at Layer-1 as **Keyframe-state advancement**.

When a new **Keyframe-state** is provisioned:

- the new Keyframe-state becomes active for subsequent envelope encoding,
- previously active Keyframe-states remain available for envelope decoding,
- no existing envelopes are re-encoded or re-interpreted.

Layer-1 treats Keyframe-state advancement as a change in **available cryptographic state**, not as a modification of existing Channel history.

The selection of when and why a new Keyframe-state is introduced is outside the scope of Layer-1 and is determined by higher-layer semantics. Layer-1 behavior is fully determined by the provisioned Keyframe-state and the `kid` values present in Channel Envelopes.

# **10. Provisioning Interface (Informative)**

This section describes the **provisioning boundary** between Layer-1 and higher layers in the ASCP architecture. It clarifies the form and nature of the inputs supplied to the Layer-1 Channel Codec without specifying or constraining how those inputs are produced.

Layer-1 does not parse Artipoints, evaluate Grammar, or interpret governance semantics. Instead, it consumes **provisioned cryptographic state** derived from higher-layer evaluation and applies that state deterministically as defined in Sections 6 through 9.

## **10.1 Provisioned Inputs**

Layer-1 expects the following categories of inputs to be provisioned by higher layers:

- **Keyframe Cryptographic key material**, including:
  - Channel AES Keys for payload encryption and decryption,
  - public verification keys for JWS signature validation,
  - additional key material required for JWE key agreement or wrapping, if applicable.
- **Key identifiers (**`kid`**)** used to index and select cryptographic material.
- **Epoch activation state**, indicating which **Keyframe-state** is currently active for envelope encoding.
- **Replication credentials** (e.g., Channel Access Keys), consumed exclusively by Layer-0.

The representation and transport of these inputs to Layer-1 are implementation-defined and outside the scope of this specification.

## **10.2 Relationship to Higher-Layer Specifications**

The cryptographic material provisioned to Layer-1 is derived from higher-layer constructs, including but not limited to:

- Keyframe Artipoints,
- Certificate Artipoints,
- Governance and trust evaluation.

The structure, semantics, and lifecycle of these constructs are specified in other ASCP documents. This specification defines only how their **evaluated cryptographic consequences** are applied by Layer-1.

# **11. Envelope Processing Model**

This section operationalizes the cryptographic requirements and defines the normative processing steps for **Channel Envelopes**, including the conditions under which cleartext payloads are emitted upward to Layer-2.

The processing model enforces a clear separation of responsibilities: Layer-1 operates only on **Channel Envelopes** and their payload octets. Layer-1 treats payloads as **opaque serialized octet sequences** (typically UTF-8 text) and **MUST NOT** alter, canonicalize, or interpret them.

Envelope processing operates strictly on an envelope-by-envelope basis. Layer-1 **MUST NOT** depend on surrounding log entries when processing any individual envelope.

## **11.1 Producer Workflow**

This workflow describes how a Sender constructs a Channel envelope prior to handing it to Layer-0 for append.

### **11.1.1 Construct Payload (Layer-2 → Layer-1 Input)**

Layer-2 produces an **Articulation Sequence** as a serialized octet sequence (typically UTF-8 text). Layer-1 **MUST** treat this payload as opaque and **MUST NOT** adjust whitespace, ordering, or grammar structure.

### **11.1.2 Sign with Identity (JWS)**

Layer-1 MUST create a JWS Compact Serialization envelope (`<header>.<payload>.<signature>`) as defined in RFC 7515 §3.1.

**Requirements:**

- `alg` MUST be compatible with the provisioned verification key referenced by `kid`.
- `kid` MUST reference provisioned Author verification material (`ascp:cert:<uuid>`).
- `typ` MUST be `"ascp+jws"`.
- The payload MUST be the exact UTF-8 Articulation Sequence, base64url-encoded.

If JWS construction fails, the Sender MUST NOT continue to encryption and MUST NOT hand the envelope to Layer-0.

### **11.1.3 Optionally Encrypt (JWE)**

If the Channel’s configuration indicates encryption is enabled, Layer-1 MUST encrypt the JWS output using JWE Compact Serialization (RFC 7516 §3.1).

**Requirements:**

- JWS MUST be constructed first (sign-then-encrypt).
- Protected header MUST include:
  - `alg`: `"dir"`
  - `enc`: `"A256GCM"`
  - `kid`: `"ascp:keyframe:<uuid>"`
  - `zip`: `"DEF"` (optional, if compression is applied)
- Initialization Vector (IV) generation and validation MUST conform to Section 6.3.2.
- Encryption MUST use the active Channel AES Key selected via `kid`.

### **11.1.4 Append to ALSP Replica (Layer-1 → Layer-0)**

Upon successful JWS (and optional JWE) construction:

- Layer-1 hands the envelope downward to Layer-0.
- Layer-1 MUST NOT write directly into any log.
- Layer-0 appends the envelope verbatim and handles all replication.

## **11.2 Consumer Workflow**

This workflow describes how envelopes delivered by Layer-0 are processed by Layer-1 before being passed upward to Layer-2.

### **11.2.1 Receive Envelope from ALSP (Layer-0 → Layer-1)**

Layer-0 delivers envelopes in log order. Layer-1 MUST accept each envelope for cryptographic processing regardless of decryptability or provisioning state.

### **11.2.2 If Encrypted, Decrypt with Channel AES Key**

For an `"ascp+jws+jwe"` envelope:

1. Parse the JWE Compact Serialization form.
2. Inspect the protected header:
   - `kid` MUST reference provisioned cryptographic material.
   - `alg` and `enc` MUST be supported.
3. Select the AES key associated with the referenced `kid`.
4. Validate IV presence and structure as defined in Sections 6 and 8.
5. Attempt authenticated decryption.

If decryption fails:

- Layer-1 MUST NOT release any cleartext to Layer-2.
- The envelope remains retained in the Channel Log.

If decryption succeeds, the resulting plaintext MUST be a valid JWS Compact Serialization string.

### **11.2.3 Verify Signature (Always Required)**

Layer-1 MUST validate JWS signatures for all envelopes, whether encrypted or cleartext.

**Requirements:**

- `kid` MUST reference provisioned verification material.
- `alg` MUST be compatible with the selected verification key.
- Signature MUST verify over the protected header and payload.

If signature verification fails, cleartext MUST NOT be emitted to Layer-2.

### **11.2.4 Emit Cleartext to Grammar Layer (Layer-1 → Layer-2)**

If and only if:

1. (For encrypted envelopes) JWE decryption succeeds, and
2. JWS signature verification succeeds,

Layer-1 MUST emit the cleartext Articulation Sequence to Layer-2. Layer-1 MUST NOT normalize, modify, or reserialize the payload.

### **11.2.5 No Modification or Rewriting Allowed**

Layer-1 MUST NOT:

- rewrite JOSE headers,
- rewrite payloads,
- canonicalize grammar or whitespace,
- reorder statements,
- perform semantic evaluation.

Layer-1 is purely a cryptographic envelope processor.

## **11.3 Local-First Replication and Envelope Independence**

Layer-1 processes each envelope atomically and independently. It makes no assumptions about the order in which envelopes or provisioning updates arrive.

Envelopes that reference cryptographic material not yet provisioned are considered undecryptable at that time and MUST be retained without modification. Once required cryptographic state becomes available, such envelopes MAY become admissible for Layer-2 handoff.

Layer-1 imposes no additional constraints related to replication topology, partial history, or convergence behavior.

## **11.4 Error Handling Summary**

This section summarizes operational consequences of errors encountered during envelope processing.

### **11.4.1 Conditions Requiring Rejection for Semantic Handoff**

Layer-1 MUST reject an envelope for delivery to Layer-2 when:

- JWE decryption fails,
- the referenced `kid` cannot be resolved,
- JWS signature verification fails,
- required JOSE fields are missing,
- Compact Serialization is structurally invalid,
- the `typ` header is inconsistent with the envelope structure.

In all such cases:

- cleartext MUST NOT be emitted to Layer-2,
- the envelope MUST remain in the Channel Log.

### **11.4.2 Conditions That Are Non-Fatal**

Layer-1 SHOULD continue processing when:

- optional JOSE header fields are unrecognized,
- additional header parameters do not violate JOSE rules,
- compression metadata is absent or unused.

### **11.4.3 Logging and Diagnostics**

Layer-1 SHOULD log:

- signature verification failures,
- decryption failures,
- unresolved `kid` values,
- unsupported algorithms,
- malformed serialization structures.

Logs SHOULD reference identifiers supplied by the underlying log interface and MUST NOT include decrypted payloads or sensitive key material.

### **11.4.4 Notes on Immutability**

Layer-1 MUST NOT request deletion, rewriting, or suppression of envelopes. Layer-0 MUST preserve all envelopes exactly as appended.

# **12. Security Considerations**

This section describes the security properties provided by the ASCP Channels Layer (Layer-1), the assumptions under which those properties hold, and the limitations that follow from the Layer-1 scope. It does not define governance policy, trust evaluation, or semantic authorization, all of which are specified elsewhere in the ASCP architecture.

Layer-1 provides cryptographic protection for articulated content as it is recorded and distributed. Its guarantees are mechanical and deterministic: they arise from correct application of JOSE cryptography to opaque payloads and from the immutable nature of the underlying Channel Log. All higher-order meaning, authorization, and policy enforcement are explicitly outside the scope of this layer.

## **12.1 Confidentiality**

When encryption is enabled for a Channel, confidentiality is provided by JWE using authenticated encryption with AES-GCM. Payloads within encrypted Channel Envelopes are opaque to any party that does not possess the corresponding Channel AES Key for the referenced cryptographic epoch.

Layer-1 enforces confidentiality strictly at the cryptographic level. It does not inspect, parse, or partially reveal encrypted payloads. Successful decryption is possible only when the correct symmetric key material has been provisioned and selected via the `kid` value in the JOSE header.

Confidentiality is scoped to the cryptographic epoch represented by a Keyframe. All Recipients provisioned with the same Channel AES Key for a given epoch have equivalent decryption capability. Decisions about which identities receive that key are made outside Layer-1.

## **12.2 Authenticity and Integrity**

Authenticity and integrity are provided by JWS signatures. Every Channel Envelope is signed, and Layer-1 verifies the signature of every envelope it processes, whether encrypted or unencrypted.

A valid signature cryptographically binds the payload and protected header to the signing key selected via `kid`. Any modification to the payload or protected header invalidates the signature and results in rejection for Layer-2 handoff.

Layer-1 does not determine whether a signing key *should* be trusted; it verifies signatures mechanically using provisioned verification material. The correctness of identity binding and certificate legitimacy is assumed.

## **12.3 Replay and Duplication**

Layer-1 does not implement replay detection, deduplication, or semantic uniqueness checks. Envelopes are processed as independent records.

Resistance to replay at the system level is provided by the immutable, append-only properties of the Channel Log maintained by Layer-0. Replayed or duplicated envelopes do not compromise cryptographic integrity; their semantic interpretation is the responsibility of higher layers.

## **12.4 Key Compromise and Epoch Transitions**

If cryptographic keys are compromised, the security properties of the Channel degrade accordingly:

- Compromise of a Channel AES Key allows decryption of all envelopes encrypted under the corresponding **Keyframe-state**.
- Compromise of an Author signing key allows forgery of envelopes that will pass Layer-1 verification.
- Compromise of replication credentials may expose encrypted envelopes but does not grant decryption capability.

Layer-1 supports mitigation through **Keyframe-state** advancement. New **Keyframe-states** apply only to newly encoded envelopes; historical envelopes remain bound to the **Keyframe-state** in effect at the time of encoding. Layer-1 does not retroactively revoke access to previously decryptable content.

## **12.5 Provisioning Assumptions**

Layer-1 assumes that cryptographic material provisioned from higher layers is correct, complete, and authoritative. It does not validate the semantic rationale for key issuance, epoch activation, or recipient selection.

If incorrect or unauthorized key material is provisioned, Layer-1 will apply it faithfully. Such failures are considered violations of higher-layer assumptions rather than flaws in the Channel Codec.

## **12.6 Channel Access Key (CAK) Considerations**

The Channel Access Key (CAK) governs replication participation at Layer-0. It has no role in envelope cryptography and does not grant authorship or decryption capability.

Compromise of the CAK may allow unauthorized parties to replicate the Channel Log, but encrypted payloads remain protected unless the corresponding Channel AES Key is also compromised. Rotation of the CAK limits replication exposure but does not affect cryptographic validity of existing envelopes.

## **12.7 Side-Channel and Implementation Risks**

Implementations should take care to avoid side-channel leakage during signature verification, decryption, and key lookup. Differences in timing or error behavior may reveal information about key presence or envelope structure.

Diagnostic logging should avoid recording decrypted payloads, private keys, raw IVs, or other sensitive material. Logs should be sufficient for operational diagnosis without weakening cryptographic protections.

Correct generation of randomness for Initialization Vectors and keys is critical. Weak randomness or IV reuse violates the cryptographic assumptions of AES-GCM and undermines confidentiality and integrity guarantees.

## **12.8 Scope Limitations**

Layer-1 does not provide:





- semantic authorization or access control,
- per-recipient payload confidentiality for articulated payload envelopes,
- sender anonymity,
- encrypted metadata beyond JOSE payload protection,
- retroactive revocation of access,
- forward secrecy beyond Keyframe-state boundaries.

These limitations are intentional and reflect the strict layering of the ASCP architecture.

## **12.9 Trust Boundaries**

Layer-1 forms a cryptographic boundary between log replication and semantic interpretation. Its guarantees depend on the correct operation of adjacent layers but do not extend beyond its defined responsibilities.

When used as specified, Layer-1 ensures that articulated content recorded in a Channel Log is cryptographically authentic, integrity-protected, and confidential when encryption is enabled. All further interpretation and enforcement of meaning occur outside this layer.

# **13. Implementation Considerations**

This section provides **non-normative implementation guidance** for ASCP Channels. It highlights practical considerations that complement the normative requirements defined elsewhere in this document, with emphasis on correct layering, cryptographic state handling, observability, and forward compatibility.

Nothing in this section introduces additional protocol requirements.

## **13.1 Layer Separation and Provisioning Boundaries**

Implementations are encouraged to maintain a strict separation between the responsibilities of Layer-1 (Channels) and those of adjacent layers. Layer-1 is responsible solely for constructing and validating JWS and JWE Channel Envelopes and for applying provisioned cryptographic state.

Layer-1 implementations should avoid embedding any Grammar parsing, governance evaluation, membership reasoning, or policy logic. All cryptographic material—including verification keys, Channel AES Keys, Channel Access Keys, permitted algorithms, and Keyframe activation state—is supplied through an external provisioning interface.

Clear separation at this boundary reduces coupling between layers, simplifies reasoning about correctness, and supports long-term maintainability.

## **13.2 Key Table Management**

Channel implementations typically maintain multiple concurrent cryptographic keys, including current and historical Channel AES Keys, verification keys, and replication credentials. Maintaining a structured key table indexed by JOSE `kid` values can simplify deterministic key selection and reduce error-prone conditional logic.

Updates to provisioned cryptographic state are ideally applied atomically to avoid transient inconsistencies during **Keyframe-state** transitions. Historical keys may remain necessary for decrypting older envelopes and should not be discarded unless explicitly directed by higher layers.

Careful key table management supports stable historical decryptability and predictable behavior across replicas.

## **13.3 Diagnostics and Observability**

Operational debugging and deployment troubleshooting benefit from clear diagnostics at the Channel layer. Implementations may emit structured diagnostic information for conditions such as unresolved `kid` values, signature verification failures, decryption errors, or malformed envelope structures.

Diagnostic output should avoid exposing sensitive material, including decrypted payloads, private keys, raw Initialization Vectors, or symmetric key material. Logging identifiers supplied by the underlying log interface, failure classes, or algorithm identifiers is generally sufficient to aid diagnosis without weakening security properties.

Layer-1 implementations should not assume that adjacent layers provide complete visibility into cryptographic or provisioning failures.

## **13.4 Forward Compatibility**

To support interoperability across evolving ASCP deployments, implementations are encouraged to tolerate extensions and unrecognized elements. In particular, implementations may safely ignore unknown JOSE header parameters, avoid rigid assumptions about algorithm sets beyond those required by this specification, and design provisioning interfaces that can accommodate future cryptographic epochs or algorithm choices.

A forward-compatible posture allows Channels to evolve incrementally without requiring simultaneous upgrades across all replicas and helps preserve long-term stability within heterogeneous environments.

# **14. IANA Considerations**

This document defines no new IANA registries and makes no requests of IANA at this time.

Future versions of ASCP may define media types, JOSE header extensions, or algorithm identifiers that require IANA registration.

## 14.1 Media Types

If future media types are introduced (e.g., "application/ascp+jws" or "application/ascp+jws+jwe"), they will be registered through the standard IANA Media Type registration procedures and documented in subsequent revisions of this specification.

## 14.2 JOSE Header Parameters

ASCP leverages established internet standards to ensure broad compatibility.

The following IETF registrations likely will be reserved for ASCP-specific JOSE usage:

- **JOSE JWS Header**: `{"typ": "ascp+jws"}` with MIME type: `application/ascp+jws`
- **JOSE JWE Header**: `{"typ": "ascp+jws+jwe"}` with MIME type: `application/ascp+jws+jwe`

These registrations would ensure that ASCP cryptographic envelopes are properly identified and handled by standards-compliant JOSE implementations while maintaining interoperability with existing cryptographic infrastructure.

# **15. References**

## **15.1 Normative References**

**RFC 7515**

Jones, M., Bradley, J., and N. Sakimura, *JSON Web Signature (JWS)*, RFC 7515, May 2015.

**RFC 7516**

Jones, M., Rescorla, E., *JSON Web Encryption (JWE)*, RFC 7516, May 2015.

**RFC 7518**

Jones, M., *JSON Web Algorithms (JWA)*, RFC 7518, May 2015.

**RFC 7517**

Jones, M., *JSON Web Key (JWK)*, RFC 7517, May 2015.

**RFC 4648**

Josefsson, S., *The Base16, Base32, and Base64 Data Encodings*, RFC 4648, October 2006.

## **15.2 Non-Normative References**

**ALSP Specification**

Reframe Systems, *ASCP Log-Sync Protocol (ALSP)*, current version.

(Referenced for Layer-0 replication semantics and immutability model.)

**ASCP Trust and Identity Architecture**

Reframe Systems, *ASCP Trust & Identity Specification*, current version.

(Referenced for identity certificates and governance semantics.)

**ASCP Governance and Access Control**

Reframe Systems, *ASCP Governance Specification*, current version.

(Referenced for membership evaluation and policy semantics.)

**ASCP Grammar Specification**

Reframe Systems, *ASCP Artipoint Grammar & Structure*, current version.

(Referenced for Channel, Keyframe, and envelope attribute types.)

# **Appendix A — Examples (Informative)**

(TBD)

# **Appendix B. Compression Considerations**

This appendix provides **informative guidance** on the use of payload compression in ASCP Channel Envelopes. Compression is not required for protocol correctness and is not part of the core cryptographic or structural requirements of the Channel Codec.

All normative envelope format, cryptographic, and processing requirements are defined in Sections 6, 8, and 11.

## **B.1 Compression Overview**

ASCP Channel Envelopes MAY apply payload compression using JOSE-supported mechanisms prior to cryptographic processing. Compression is intended as an **optional optimization** to reduce envelope size in environments where bandwidth, storage, or replication cost is a concern.

Compression, when used, is applied to the **payload of the JWS object** prior to signing. The resulting compressed payload is then cryptographically protected in the same manner as an uncompressed payload.

Compression is orthogonal to encryption. When both compression and encryption are used, the processing order is:

1. Serialize the Articulation Sequence
2. Compress the serialized payload (optional)
3. Sign the payload using JWS
4. Encrypt the signed payload using JWE (optional)

## **B.2 Supported Compression Mechanism**

When compression is used, the JOSE `"zip"` header parameter is used as defined in **RFC 7516 §4.1.3**.

The only compression algorithm currently defined for ASCP Channel Envelopes is:

- `"DEF"` — DEFLATE compression, as defined in **RFC 1951**

Other compression algorithms are not defined by this specification.

## **B.3 When Compression May Be Appropriate**

Compression MAY be beneficial when:

- Articulation Sequences are large relative to typical envelope size,
- Channel Logs are replicated frequently over constrained links,
- Storage or replication overhead is a concern.

Compression is generally **not beneficial** for small payloads, where compression overhead may exceed the size savings.

Implementations are encouraged to consider compression heuristics appropriate to their environment and usage patterns.

## **B.4 Compression Thresholds**

Some implementations may choose to apply compression only when the serialized payload exceeds a certain size threshold.

Earlier drafts of this specification referenced a **200-byte threshold** as an illustrative heuristic. This value is **not normative** and is provided solely as an example based on empirical observation.

Implementations MAY:

- use different thresholds,
- make compression configurable,
- apply compression unconditionally,
- or disable compression entirely.

No specific threshold is required or recommended by this specification.

## **B.5 Security Considerations**

Compression introduces potential security considerations, particularly when combined with encryption.

Implementations SHOULD be aware of known classes of attacks that exploit compression side channels (e.g., CRIME, BREACH-style attacks). While these attacks are typically associated with interactive protocols and attacker-controlled plaintext, implementers should evaluate their threat model carefully when enabling compression.

Because ASCP Channel payloads consist of structured articulation data rather than attacker-supplied interactive content, the practical risk profile may differ from web-based protocols. Nonetheless, compression SHOULD be treated as an optional optimization rather than a default requirement.

## **B.6 Interoperability Notes**

Compression support is optional. Implementations that do not support compression MUST reject envelopes that specify an unsupported `"zip"` value.

Implementations that support compression MUST correctly process both compressed and uncompressed envelopes.

# **Appendix C — Rationale & Design Notes**

This informative only appendix provides additional context on several architectural choices made in the ASCP Channels design. These notes are non-normative and are intended to aid reviewers, implementers, and designers in understanding the motivations behind the Layer-1 protocol structure.

## **C.1 Why Channels Do Not Parse Grammar**

Layer-1 intentionally treats all payloads as opaque and refrains from parsing the ASCP Artipoint Grammar. This separation is essential for stability, forward compatibility, and security. Grammar definitions evolve over time as ASCP’s semantic layer matures, and tying cryptographic envelope handling to any particular grammar structure would bind Channel implementations to specific syntax, schema, or semantic conventions.

Furthermore, Grammar-level constructs (e.g., membership declarations, Keyframe attributes, RACI roles, or rich semantic annotations) are evaluated by Layer-3 governance logic. Channels must not interpret these constructs, and attempting to do so would risk embedding policy or authorization logic into the cryptographic transport layer. By restricting Layer-1 to JOSE envelope handling and externally provisioned key material, Channels remain independent of semantic evolution, promote interoperability across heterogeneous deployments, and keep security reasoning centered around JOSE, identity certificates, and Keyframes.

## **C.2 Why Keyframes Are Expressed in Grammar**

Keyframes are expressed as Layer-2 Artipoints rather than as Layer-1 protocol messages for two primary reasons: **auditability** and **composability**.

First, Keyframes must be durable, inspectable objects in the shared DAG of Artipoints. Treating them as Grammar-level constructs ensures that key lifecycle events (initialization, rotation, supersession) are recorded in the same immutable, link-addressable structure as all other collaborative context. This fits the Cortex Layer’s design goal of treating all contributions—human or agentic—as addressable knowledge objects.

Second, expressing Keyframes at the Grammar layer allows them to be composed with other Artipoint types: governance policy, membership changes, Channel declarations, and other semantic structures. Layer-3 evaluators can then compute the effective cryptographic configuration by walking the DAG and determining the active Keyframe in context. From this evaluation, Layer-3 provisions Layer-1 with the key table, active Keyframe kid, and permitted algorithms. This separation ensures that cryptographic actions remain driven by semantic meaning, while Layer-1 continues to operate purely on JOSE payloads and provisioned keys.

## **C.3 Notes on Future Extensions**

The Channels layer is designed to accommodate future cryptographic and structural enhancements without requiring a redesign of the protocol. Possible areas of extension include:

### **HPKE and Per-Recipient Encryption**

Today, Channels use symmetric AES keys per Keyframe, resulting in group-wide confidentiality boundaries. Future revisions may adopt HPKE or hybrid modes that allow per-recipient encryption while maintaining compatibility with the Channel and Keyframe model. Such extensions would belong at Layer-1 and would require enriched Keyframe representations.

### **Alternative JOSE Serializations**

While Channels currently mandate Compact Serialization for operational envelopes, future deployments may benefit from JSON or JWE General Serialization for cases requiring explicit multi-recipient support, debug-friendly structures, or richer JOSE metadata. Any such evolution must preserve backward compatibility with the Compact format.

### **Algorithm Agility and PQC Transition**

The ASCP stack anticipates long-term upgrades to post-quantum signature and key establishment schemes. Keyframes, with their versioned UUID structure and logical placement in the Grammar, are an ideal mechanism for introducing new algorithm families without altering Envelope Format or Channel processing semantics.

### **Multi-Key or Multi-Layer Channels**

Some use cases may require layered or nested encryption, dual-signature models, or specialized Channels for compliance-driven workflows. These can be added as additional Channel types or envelope profiles while retaining the core Layer-1 machinery.

## **C.4 Comparison to Related Protocols**

ASCP Channels share design DNA with several existing secure messaging and transport systems but differ in key architectural dimensions:

### **TLS / DTLS**

Unlike TLS, Channels provide no session concept, no negotiation handshake, and no in-band identity exchange. Channels secure discrete artifacts in an immutable log rather than streams of transport-layer data.

### **MLS**

MLS provides tree-structured group keying with strong forward secrecy and per-epoch cryptographic state; Channels provide simpler Keyframe-scoped symmetric keys aligned to the shared DAG. MLS couples membership and key schedule tightly, while ASCP deliberately separates membership semantics from cryptographic state.

### **DIDComm**

DIDComm supports per-message encryption to specific recipients but lacks a shared immutable log as the coordination substrate. Channels invert this relationship: the immutable DAG is primary, and envelopes secure contributions into that DAG.

### **ActivityPub / Other DAG-like Systems**

ASCP differs by treating all shared state as cryptographically verifiable and by binding contributions to identity certificates and Keyframe-driven cryptographic state, rather than relying on server-level assurance.

These comparisons highlight ASCP’s unique goals: to provide cryptographically verifiable shared cognition across humans and agents, anchored in an immutable coordination substrate rather than a messaging protocol or transport stream.