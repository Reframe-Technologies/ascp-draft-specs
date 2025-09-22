# **ASCP LogSync Protocol (ALSP)**

**Layer 0 - The ASCP Logging and Synchronization Layer**

Version 0.2 - September 2025

# **Protocol Overview**

ALSP (ASCP LogSync Protocol) defines a transport-agnostic protocol for synchronizing append-only, ordered logs of ASCP Channel articulation sequences between authorized replicas in a distributed system. Layer 0 treats these articulation sequences as semantically opaque payloads that happen to use JOSE compact serialization format, enabling transport optimizations while delegating all content interpretation to Layer 1.

The primary goals of ALSP are:

- Deterministic convergence of logs across replicas
- Local-first, eventually consistent sync
- Minimal semantic assumptions such that the Layer 1 channel content is regarded as semantically opaque
- Strong encryption, authentication and authorization

ALSP is architecturally agnostic, supporting both centralized server-based deployments and fully distributed peer-to-peer networks. The protocol's symmetric design allows any authorized replica to serve sync requests from other replicas, enabling flexible network topologies based on deployment requirements.

Within ALSP, each ASCP Channel maps to a logical append-only log of opaque articulation sequences. Within a repository's organizational construct, ALSP ensures that all authorized replicas eventually converge to identical content and ordering for each channel log. Replicas then merge messages from all accessible channels into a unified, causally ordered stream for application-layer processing.

# **Glossary**

**ALSP -** ASCP LogSync Protocol - The Layer 0 of ASCP responsible for transport-agnostic synchronization of append-only logs (Channels) using deterministic ordering.

**Articulation -** An immutable, signed, and optionally encrypted statement in ASCP representing structured shared cognition. Passed down as JOSE JWS/JWE compact serialized "records" from Layer 1 to Layer 0 and vice versa.

**ASCP -** Agents Shared Cognition Protocol – a four-layer protocol stack enabling persistent, structured collaboration between humans and AI agents.

**bootstrap log -** The initial Channel log fetched by replicas containing references to known channels and organizational key material that serves as the root of trust and context.

**canonical order -** The deterministic message ordering rule in ALSP: (lamport\_time, message\_id).

**Channel -** The fundamental unit of distribution and access in ASCP. Each channel maintains an append-only log of semantically opaque articulation sequences from Layer 1 and has a universally unique ID.

**Channel Access Key (CAK) -** An Ed25519 keypair used for channel authorization at Layer 0. The public key is distributed via a bootstrap process, while the private key is used by authorized clients to create JWS signatures for channel access proofs.

**Channel Log -** An ordered sequence of semantically opaque articulation sequences for a specific channel, maintained per replica and synchronized via ALSP.

**channel manifest -** Bootstrap-provided data containing channel metadata, access permissions, and CAK public keys for Layer 0 channel authorization.

**digest hash exchange -** A method for comparing message log consistency between replicas using the SHA-256 digest algoritm. See `log_digest`.

**Dot Preserving Binary (DPB) -** A binary encoding format that transforms JOSE compact serialization into a more efficient wire representation by converting Base64url segments to raw binary while preserving structural dot separators. Used by Layer 0 to optimize transport of Layer 0 and Layer 1 JOSE streams with perfect reversibility.

**hello message -** The initial ALSP negotiation message exchanged between replicas to establish identity, capabilities, and sync configuration using JWS based mutual authentication.

**Identity Key**: The JWK-encoded elliptic curve key pair used by ASCP participants (humans and AI agents) for cryptographic identity, signing, and key agreement. Unlike certificates from centralized CAs, ASCP identity keys are typically self-generated and exchanged in JOSE-compliant JWK format, with key binding proven through JWT-based identity claim bundles that establish the connection between identity (like an email address or agent URN) and the public key.

**JWK (JSON Web Key)**: The JOSE standard format used to represent cryptographic keys, including Channel symmetric keys and participant certificates.

**JWS (JSON Web Signature)**: The format used for all ALSP message authentication, providing standardized cryptographic proof of message authenticity and session binding.

**JWT (JSON Web Token)**: The format used for ASCP identity establishment and session authentication, enabling secure claims exchange between identity providers and ASCP peers.

**JOSE (JavaScript Object Signing and Encryption)**: The family of internet standards (JWS, JWE, JWK) that ASCP leverages for all cryptographic operations.

**kid -** Key identifier used in JWS headers to reference cryptographic material (certificates or CAKs) resolved through the bootstrap process.

**Lamport Clock -** A monotonically increasing logical clock used to provide deterministic ordering of messages across distributed systems.

**lamport\_max -** The highest Lamport clock value known to a replica, propagated during sync to ensure consistency across peers.

**log\_digest -** A SHA-256 hash of message IDs up to a given Lamport time, used to verify log integrity and detect divergence.

**message\_id -** A universally unique identifier assigned by ALSP at Layer 0 to each opaque Layer 1 articulation statement payload to ensure idempotent replication and ordering.

**node\_id -** A universally unique identifier representing the origin node (replica) of a message.

**payload -** The Dot Preserving Binary (DPB) encoded content of an articulation message from Layer 1. While ALSP understands these payloads use JOSE compact serialization format for transport optimization, the contents of the JOSE JWS/JWE stream are not interpreted by ALSP, with all semantic processing handled at Layer 1.

**pull sync -** A model where a replica explicitly requests Channel Log content by sending sync\_request messages.

**push mode -** A sync model where a replica sends new Channel Log content asynchronously to the peer after the initial sync is established via pull mode operation.

**replica -** An authorized node or instance that stores and synchronizes ASCP channel logs. Each replica maintains its own Lamport clock that is reconciled during synchronization. 

**session\_nonce -** A randomly generated 32-character value unique to each ALSP session, used in JWS authentication to prevent cross-session replay attacks and ensure message authenticity.

**sync\_request -** An ALSP message used to request synchronization of messages for a specific channel

**sync\_response -** ALSP message(s) responding to a sync\_request, containing a list of channel log content in transport format.

**sync\_update -** An optional ALSP message used for real-time push of new Channel Log content to connected peers in live sync mode.

# **Layer Separation and Security Model**

A critical architectural principle underlies ALSP: **everything from Layer 1 is treated as completely semantically opaque to Layer 0**. ALSP knows nothing about the contents, structure, or meaning of Layer 1 payloads. While Layer 0 understands that these payloads use JOSE compact serialization format (enabling the DPB transport optimization), it remains semantically agnostic to the protected content within these structures. As such, it treats them purely as base64url payloads that need secure, identical replication across authorized replicas.

This separation serves distinct security purposes across layers:

**Layer 0 (ALSP) Security**: Protects the integrity of the replication transport itself:

- **Authenticate peers** (who is allowed to sync)
- **Authorize channel access** (which logs can this peer replicate)
- **Ensure identical replication** (same payload bytes in same order across all replicas)
- **Prevent tampering/replay** at the transport layer

**Layer 1 (ASCP Message Layer) Security**: Protects the meaning and content of articulation statements through JWS signatures, JWE encryption, keyframes, and certificate chains. While Layer 0 understands the JOSE compact serialization format for transport optimization, it remains semantically agnostic to the protected content within these structures.

This architectural separation means Layer 0 ALSP operates as a **secure replication and synchronization mechanism** that can efficiently encode and transmit Layer 1's JOSE-formatted messages while Layer 1 handles all semantic security. The two security models are complementary, not redundant: Layer 0 ensures that Layer 1's cryptographically secured articulation statements reach all authorized replicas with identical content and ordering, while Layer 1 ensures that the meaning and content itself remains cryptographically protected regardless of transport encoding optimizations.

# **ALSP Relationship to ASCP Channels**

Building on this separation principle, ASCP operates as a four-layer protocol stack with ALSP serving as Layer 0:

**Layer 0 (ALSP Transport Layer)**: Handles channel log synchronization, ordering, and distribution across replicas. This layer is transport-agnostic and provides deterministic convergence using Lamport clocks, ensuring all authorized replicas eventually hold identical, canonically ordered logs per channel. Layer 0 supports selective replication—nodes only replicate channels they are authorized to participate in or monitor.

**Layer 1 (ASCP Message Layer)**: Contains the actual articulation content—signed, optionally encrypted articulation statements that represent structured shared cognition. Layer 1 sends individual articulation sequences down to Layer 0 for storage and distribution as semantically opaque payloads. ALSP understands these payloads use JOSE compact serialization format, enabling transport encoding optimizations through DPB, while delegating all semantic interpretation of the protected content to higher application layers.

## **Channel Model**

Channels serve as the fundamental unit of both layers:

- **Channel ID**: A universally unique identifier for associated Channel which also defines the scope of both distribution and access
- **Channel Log**: A linear causal sequence of Layer 1 messages, maintained per channel with universally deterministic Layer 0 ordering across all Channels and replicas.
- **Multicast Distribution**: Channels act as secure multicast groups rather than logical partitions—authorized members receive all messages for their accessible channels

During synchronization, Layer 0 transmits received messages up to Layer 1 in clear causal order but not necessarily exact time order. The Layer 1 timestamp can be used for semantic indication of actual message order within the accuracy of each node's wall clock time.

This architecture enables ALSP to synchronize structured articulation content while remaining semantically agnostic, with each channel log transmitted independently but merged into a universally consistent timeline at the application layer.

# **Articulation Sequence Encoding**

This section defines how the opaque JOSE compact serialized encodings of Articulation Sequences are represented in the wire protocol and provides normative guidance for local channel log storage as well. Layer 1 articulation sequences arrive at Layer 0 as JOSE compact serialized strings, encoded into DPB (Dot Preserving Binary) encoded payloads, which are then wrapped in a CBOR-encoded Layer 0 envelope containing ordering and identity metadata. Each channel log is usually maintained locally as a sorted list of these wrapped messages, with the same basic format typically used both on the wire and in local storage.

## **Layer 0 Wrapper of Layer 1 Articulation Sequences**

This section defines the **binary** Layer-0 wrapper used to store and carry semantically opaque Layer-1 articulation sequences over ALSP. The wrapper is encoded with **CBOR** (IETF RFC 8949) using **deterministic CBOR** rules. Layer-1 payloads are **opaque bytes** containing the JOSE compact serialization encoded via **Dot-Preserving Binary (DPB)**. Layer-0 does **not** interpret payloads.

> Channel identifiers and other ALSP sync metadata are conveyed by the surrounding ALSP messages and **do not** appear in this wrapper.

### **Data Model (Single Entry)**

A Layer-0 entry wraps exactly one Layer-1 payload together with ordering and identity metadata.

**CDDL (normative)**

```
; Integer-keyed, compact map
L0-Entry = {
  0: lamport-time,       ; uint / uint64
  1: bstr .size 16,      ; message_id (UUID, raw 16 bytes per RFC 4122)
  2: bstr                ; payload (DPB bytes; Layer-1 opaque)
}

lamport-time = uint      ; producers/consumers MUST support up to 64-bit
```

**Field Semantics**

- **0 — lamport\_time (uint)**
- Lamport clock tick for deterministic ordering. MUST be treated as an **unsigned 64-bit** value (even if many entries fit in 32 bits).
- **1 — message\_id (bstr .size 16)**
- Message UUID encoded as **16 raw bytes** (RFC 4122 canonical byte order). **Textual UUIDs are not permitted** in this wrapper.
- **2 — payload (bstr)**
- Opaque DPB bytes (the exact Layer-1 JOSE compact sequence encoded with DPB). Layer-0 MUST NOT transform or re-serialize this value.

### **Deterministic CBOR Requirements (normative)**

To ensure signatures and hashes are stable:

1. **Key ordering:** Map keys MUST be encoded in ascending order (0, 1, 2).
2. **Shortest form:** Integers MUST use the shortest-length CBOR major type encodings that represent the value.
3. **Byte strings:** MUST use definite-length encoding with exact length.
4. **No extras:** No duplicate keys; no semantic tags; no indefinite-length items.

### **Insertion & Ordering Rules**

- **Canonical ordering key:**(lamport\_time, message\_id) with message\_id compared as raw 16-byte lexicographic tie-breaker.
- **Idempotence:** Receivers MUST de-duplicate **per channel** by (message\_id).
- **Atomicity:** Each L0-Entry is self-contained; batch boundaries (below) have no semantics.

### **Example (informative)**

Given:

- lamport\_time = 1\_345\_678
- message\_id = 550e8400-e29b-41d4-a716-446655440000 (raw 16 bytes)
- payload = h'65794a2e686247382e736967' (example DPB)

**Diagnostic notation:**

```
{
 0: 1345678,
 1: h'550e8400e29b41d4a716446655440000',
 2: h'65794a2e686247382e736967'
}
```

**Wire bytes (hex):**

```
A3                                  # map(3)
  00                                # key: 0
  1A 00 14 88 8E                    # uint32 1,345,678  (shortest form)
  01                                # key: 1
  50                                # bstr, length 16
     55 0E 84 00 E2 9B 41 D4
     A7 16 44 66 55 44 00 00
  02                                # key: 2
  4C                                # bstr, length 12
     65 79 4A 2E 68 62 47 38 2E 73 69 67
```

### **Error Handling (normative)**

Receivers MUST reject (and not insert) an entry if any of the following hold:

- The map is not exactly 3 keys {0,1,2} in ascending order.
- lamport\_time is not an unsigned integer or exceeds 64-bit range supported by the implementation.
- message\_id is not a 16-byte bstr.
- payload is not a definite-length bstr (zero length is permitted but discouraged).
- Any indefinite-length item or semantic tag is present.

Rejected entries MUST NOT affect Lamport counters or ordering. Implementations SHOULD surface diagnostics.

## ASCP Dot Preserving Binary (DPB) Format

JOSE compact serialization represents JWS and JWE tokens as sequences of Base64url-encoded segments separated by ASCII dot characters (0x2E). To optimize Layer 0 transport efficiency while maintaining perfect reversibility, ALSP introduces the **ASCP Dot Preserving Binary (DPB)** format for encoding Layer 1 JOSE streams.

The DPB format transforms JOSE compact serialization into a more compact binary representation by converting Base64url segments to raw binary while preserving structural dot separators. This eliminates the 33% Base64url overhead while maintaining perfect bidirectional conversion and exact semantic boundaries. The format uses self-delimiting ULEB128 length encoding for efficiency and employs the UTF-8 safe 0x1F escape sequence that cannot appear in valid JOSE compact serialization.

### **Encoding Rules**

The transformation follows these principles:

1. **Dot preservation**: Literal dot characters (0x2E) remain unchanged to preserve segment boundaries
2. **Base64url segment conversion**: Each Base64url segment is converted to a binary block consisting of:
   - An escape header (0x1F)
   - The segment length encoded in ULEB128 format
   - The raw decoded bytes from the Base64url segment
3. **Empty segment handling**: Empty segments between consecutive dots require no encoding (dots remain consecutive)
4. **Raw ASCII segments**: Raw ASCII middle segments as defined in RFC 7797 remain as literal text without binary block encoding

### **Key Benefits**

- Eliminates 33% Base64url encoding overhead
- Maintains perfect reversibility to original JOSE compact serialization
- Preserves exact segment boundaries for JOSE parsing
- Self-delimiting format enables efficient streaming

### **ULEB128 Length Encoding**

The segment length uses ULEB128 (Unsigned Little Endian Base 128), a variable-length integer encoding widely adopted in binary protocols including DWARF debugging information, WebAssembly, and Protocol Buffers. ULEB128 represents integers by breaking them into 7-bit chunks, with each byte's most significant bit serving as a continuation flag: 1 indicates more bytes follow, while 0 marks the final byte. The encoding reads little-endian, with the lowest 7 bits transmitted first.

For example, the value 1 encodes as a single byte 0x01, while 128 requires two bytes: 0x80 0x01 (the first byte's high bit indicates continuation, its low 7 bits represent zero, and the second byte contains the remaining value 1). The value 624485 encodes as 0xE5 0x8E 0x26, demonstrating the format's efficiency for larger integers.

### **Example of DPB**

Consider a JOSE JWS with the following compact serialization:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibm
FtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.SflKx
wRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

This breaks down into three segments:

- **Header**: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
- **Payload**: eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWR taW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0
- **Signature**: SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV\_adQssw5c

**Decoded lengths (bytes)**

- Header = **27**, Payload = **68**, Signature = **32**

**DPB bytes (hex)**

```
// Header segment: 1f 1b (escape + ULEB128(27)) + 27 bytes
1f 1b
7b 22 61 6c 67 22 3a 22 48 53 32 35 36 22 2c 22
74 79 70 22 3a 22 4a 57 54 22 7d

// Dot separator
2e

// Payload segment: 1f 44 (escape + ULEB128(68)) + 68 bytes  
1f 44
7b 22 73 75 62 22 3a 22 31 32 33 34 35 36 37 38
39 30 22 2c 22 6e 61 6d 65 22 3a 22 4a 6f 68 6e
20 44 6f 65 22 2c 22 61 64 6d 69 6e 22 3a 74 72
75 65 2c 22 69 61 74 22 3a 31 35 31 36 32 33 39
30 32 32 7d

// Dot separator
2e

// Signature segment: 1f 20 (escape + ULEB128(32)) + 32 bytes
1f 20
49 f9 4a c7 04 49 48 c7 8a 28 5d 90 4f 87 f0 a4
c7 89 7f 7e 8f 3a 4e b2 25 5f da 75 0b 2c c3 97
```

**Notes:**

- This example shows a JWS with header, payload and signature in JOSE compact serialization form, but DPB encoding is format-agnostic—any sequence of dot-separated Base64url encoded streams can be represented in DPB format
- `1f 1b` → binary block opener (0x1F) + ULEB128(27) indicating 27-byte segment length
- Dots (0x2E) remain as literal separators preserving JOSE structure
- All Base64url segments round-trip perfectly via raw bytes and canonical Base64url re-encoding (no padding required)

### **Reversibility Process**

1. Scan for 0x1F escape sequences and literal dots
2. For each binary block: decode ULEB128 length, extract raw bytes, re-encode as Base64url
3. Reconstruct JOSE compact serialization with original dot separators
4. Result is bit-for-bit identical to original JOSE string

# **ALSP Global Synchronization Model**

ALSP uses **Lamport clocks** for global message ordering across all channel logs. This approach provides:

- **Deterministic ordering** across all replicas
- **Causal safety** ensuring messages appear after all referenced messages
- **Offline robustness** without requiring synchronized wall clocks
- **Minimal overhead** through simple metadata requirements

## **Canonical Ordering Rule**

All logs use the tuple `(lamport_time, message_id)` for total ordering:

- `lamport_time`: Unsigned integer (numerical sort)
- `message_id`: Lexicographic string comparison (UTF-8 byte order)

This guarantees a stable, causally compatible total order across all channels with common lamports grouped and then by the message\_id for tie breaking.

## **Global Counter Design**

ALSP uses a single, replica-wide Lamport counter that spans all channels, rather than maintaining separate counters per channel log. This design addresses a specific ordering challenge in ASCP's partial replication model.

Replicas may later gain access to channels containing messages from earlier logical times. While this doesn't cause retroactive causal violations, it can significantly impact linear wall clock time ordering. The global counter approach optimizes for better cross-log linear time ordering by ensuring all messages from a replica use values from the same logical timeline.

## **Lamport Max Propagation**

To maximize linear time ordering consistency, ALSP includes a `lamport_max` field in all sync messages (both push and pull). This field advertises the sender's current local `lamport_time` counter value, which reflects the highest logical time known across all channels and peers by that sender.

When a replica receives sync messages, it must update its local counter to the maximum of:

- Its current counter value
- Any received `lamport_max` values
- Any Lamport values from received messages

This propagation mechanism provides several benefits:

1. **Proactive clock advancement**: Replicas can advance their Lamport clock before sync completion, reducing reliance on tie-breakers in high-concurrency scenarios
2. **Global time consistency**: Prevents new local messages from having outdated clock values
3. **Network-wide coordination**: Allows replicas to stay synchronized with logical activity across the entire network

**While not strictly required for correctness, the** `lamport_max` **field serves as an effective optimization in real-world deployments.** It significantly improves log ordering quality under conditions of partial replication or high concurrency by enabling proactive counter synchronization based on network-wide logical activity.

## **Normative Implementation Guidance**

Each replica maintains a ***single*** Lamport counter that applies globally across all channel logs.

**On startup**, initialize the local lamport clock counter to the maximum Lamport value present across all locally stored messages, or, ideally, any previously received lamport\_max value, whichever is higher. This preserved `lamport_max` is more important for replicas that operate offline frequently while replicas that immediately resync with peers will quickly discover the current network `lamport_max` automatically.

When creating a new message locally for any channel:

- Increment the Lamport counter by 1.
- Assign the incremented value to the new message.

When receiving messages or sync metadata from peers:

- Identify the highest Lamport value among any received message's Lamport value, any advertised lamport\_max field in a sync header, and the local counter itself.
- Update the local counter to this maximum value.

**During sync**, all messages must be inserted into local logs in canonical order:

```
lamport_time → message_id
```

This preserves deterministic and convergent ordering across all replicas. All newly created messages are guaranteed to appear after all previously known events in the global ordering.

**Note:** It is normal for messages from different authors to share the same Lamport clock value. This does not indicate a conflict. Deterministic ordering is preserved via the message\_id tie-breaker.

Lamport values are strictly monotonic per replica and never reused, ensuring global consistency and causal safety even in the presence of concurrent authorship or parallel sync operations.

Design Note: Wall-clock timestamps are not suitable for canonical ordering due to potential clock skew — especially in offline or partially connected environments. This can lead to messages being timestamped earlier than those they reference, resulting in **causal violations**. For this reason, timestamps (though preserved in Layer 1 for semantic purposes) are excluded from transport-level ordering.

Vector clocks provide explicit causality tracking and concurrency detection, but at the cost of significant metadata overhead and complexity. Since ASCP already enforces causality structurally through immutable DAG references between Artipoints, ALSP does not require clocks to infer causality. Lamport clocks strike the right balance, ensuring valid causal order without added complexity.

# ALSP Protocol Messages

Each replica maintains a current Lamport clock and an ordered log per channel. This section defines the ALSP protocol messages used to connect and synchronize with peers. While ALSP operates over standard network transports such as WebSockets with TLS, it implements its own authentication and authorization mechanisms independent of the underlying transport layer.

As part of the replica's state, Layer 0 must maintain configuration information about which channels it is maintaining locally and push local updates to the ALSP peer with minimal added latency. However, it should try to push as many messages as possible in a single sync batch for all locally available messages for a given channel.

Sync operations are channel-based. For the efficiency of connected peers, one should push messages in Lamport clock order, although this is not strictly required as long as lamport\_max always represents the highest clock value known to the replica.

## ALSP Auth Request

This message is sent by the connecting peer to the far end peer or server. It begins the session Authentication process. In effect, this message is stating the connecting peer's intent to operate the session using the identity\_cert key pair for this session along with the user\_identity also provided. The far end peer will then confirm identity based on information it already has from it's own bootstrapping log information OR it will respond with an ALSP Auth Challenge to force a proof of identity. Important: This message is signed using the sender's own nonce provided in the message and therefore is basically only used as a proof they are in possession of the private key that goes with the included identity certificate and that they selected that specific nonce with a current timestamp.

```json
{
    "alsp_msg_type": "auth_request",    // An explicit ALSP Auth Request
    "auth_mode": "direct / provisioned" // Idenity key send or received
    "timestamp": "<timestamp>",         // Timestamp of this message
    "session_nonce": "utf-8-string",    // Generated randomly for session 
    "identity_cert": "<JWK+JWS>",       // Signed Public Key Package
    "recovery_cert": "<JWK+JWS>",       // Signed Public Key Package
    "user_identity": "utf-8-string",    // An email address or URN
    "node_id": "uuid",                  // The sender's replica node uuid
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "auth\_request")
- **auth\_mode**: Indicates whether we are performing authorotization and key binding using direct mode, provisioned mode. See session authentication section and ASCP Trust and Identity for more information about the two modes.
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **session\_nonce**: This is a session specific randomly generated nonce value consisting of 32 random characters. This value then must be used by the receiving peer to secure all future messages sent back to the sending peer. See the section on authortization for more details.
- **identity\_cert**: The JWS signed JWK key package for the securing all of the sender's messages of this session. This key must be the sender's key that is or will be bound to the sender's identity.
- **recovery\_cert**: The JWS signed JWK key package for the securing only this message. This key is more critically used to secure the actual client identity key pair into the auth\_challenge reply. This is optional and is encoded instead of identity\_cert where the server generates the client identity key pair instead of the client itself so it can be escrowed by the server safely.
- **user\_identity**: The identity that the sender is known by or will be known by if the authentication succeeds.
- **node\_id**: The sender's replica node UUID that uniquely identifies this node in the network

## ALSP Auth Challenge

This message is sent in response to the ALSP Auth Request when the passed identity sent in the ALSP Auth Request is being secured with a key package that is not recognized, or when the node\_id is new to the contacted peer. A server or peer MAY send this challenge even if the credentials are known valid so as to force a new identity binding process to occur, but it should only do this if there is a reason to be suspisous of the connect attempt. For example, perhaps the connection is from an untrusted IP address or unexpected geographic jurisdiction, etc.

```json
{
    "alsp_msg_type": "auth_challenge",  // An explicit ALSP Auth challenge
    "timestamp": "<timestamp>",         // Timestamp of this message
    "session_nonce": "utf-8-string",    // Generated randomly for session 
    "client_key_env": "<JWE-encoded UKE>" // JWE of user-key-envelope
    "identity_cert": "<JWK+JWS>",       // Signed Public Key Package
    "user_identity": "utf-8-string",    // An email address or URN
    "node_id": "uuid",                  // The sender's replica node uuid
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "auth\_challenge")
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **session\_nonce**: This is a session specific randomly generated nonce value consisting of 32 random characters. This nonce is the **challenge nonce** for JWT encoding purposes as described in the ASCP Trust and Indentity Architecture specification document.
- **client\_key\_env**: In provisioned mode auth, this is the JWE encoded **user-key-envelope** for the client to use for it's own identity. It is encrypted using the JWK `recovery_cert` public key provided in the auth\_request. See session authentication section and **ASCP Trust and Identity** for more information about the modes when this is used.
- **identity\_cert**: The JWS signed JWK key package for the securing all of the sender's messages of this session. This key must be the sender's key that is bound to the sender's identity in the ASCP bootstrap log.
- **user\_identity**: The identity that the sender is known by according to the ASCP bootstrap log.
- **node\_id**: The sender's replica node UUID that uniquely identifies this node in the network

Note: The receiver of an ALSP Auth Challenge MUST always validate the credentials provided by the challenger, though this validation may only be possible after the local replica is established and/or fully updated. If at any point the receiver determines that the credentials are invalid per the ASCP bootstrap log, the connection must be dropped and the replica should be returned to initial state or considered suspect due to suspected far end impersonation attempt.

## **ALSP Hello Message**

The **ALSP hello** message completes the authentication process and negotiates session properties after the initial Auth\_Request/Auth\_Challenge exchange. The authentication flow follows one of two patterns:

**Standard Flow**: When Auth\_Request credentials are validated immediately, the far end peer responds directly with its own hello message, and the connecting peer then responds with its hello message to complete mutual authentication.

**Challenge Flow**: When new key binding is required, the far end peer responds to the Auth\_Request with an Auth\_Challenge. The connecting peer then sends its hello message first (including the Identity Token Package to establish identity binding), and the far end peer completes the sequence with its own hello message.

This mutual hello exchange establishes the foundation for the ALSP session by:

- Completing authentication of both parties through JWS signatures
- Negotiating maximum message lengths and buffer capabilities
- Synchronizing Lamport clock values via lamport\_max propagation
- Establishing node identities with friendly descriptions for the session
- Confirming compatibility and authorization to proceed with channel synchronization

The hello negotiation must complete successfully before any sync operations can begin, ensuring both nodes are properly authenticated and configured for secure, efficient log synchronization.

```json
{
    "alsp_msg_type": "hello",           // An explicit ALSP hello message
    "timestamp": "<timestamp>",         // Timestamp of this message
    "session_nonce": "utf-8-string",    // Generated randomly for session.
    "boot_keys_jwe": "<JWE of BCK>",    // JWE encoded boot channel keys
    "lamport_max": 16569909,            // Local lamport global max value  
    "node_id": "uuid",                  // The sender's replica node uuid
    "push_enabled": true,               // True for push mode to be enabled
    "node_description": "utf-8-string", // Friendly info about sender
    "max_alsp_length": 262144,          // Maximum receive length in bytes
    "user_auth_cert": "cert",           // The sender's certificate
    "user_identity": "utf-8-string",    // Usually an email address
    "status_message": "utf-8-string"    // Information message / error
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "hello")
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **session\_nonce**: This is a session specific randomly generated nonce value consisting of 32 random characters. Each of the connecting peers inserts their own unique value in the hello message and must be the same nonce used as part of auth\_request and auth\_challenge. This value then must be used to secure all future messages sent to the other peer. See the section on session authortization for more details.
- **lamport\_max**: Local lamport clock global max value representing the highest logical time known across all channels and peers
- **boot\_keys\_jwe**: These are the @Bootstrap channel symmetrical encryption keys when the bootstrap channel is run encrypted. Encrypting the Bootstrap channel is optional but recommended.
- **node\_id**: The sender's replica node UUID that uniquely identifies this node in the network
- **push\_enabled**: When set to true, the sender is requesting that push mode be enabled. In the response message, this will be confirmed or denied based on the true or false value. When push mode is enabled, after a pull request has been satisfied, the associated channel will flip into async mode and further updates will come in sync\_update messages.
- **node\_description**: Friendly information about the sender (e.g., "Jeff Mac Laptop") for debugging and identification purposes
- **max\_alsp\_length**: An unsigned 32-bit value defining the maximum receive length in bytes that this node can handle for individual incoming ALSP messages. All nodes must implement a receive buffer size of 32K bytes or larger, but should implement at least 128K bytes. If omitted, 128K bytes is assumed by the peer. Senders must never exceed the specified size and should declare an error in the hello response if an explicit value of 32K bytes or smaller is indicated. Note: The advertised value must also respect any limitations imposed by the underlying network transport protocol.
- **user\_auth\_cert**: For auth\_challenge response cases, this contains the JWS signed Identity Token Package used to establish identity binding. In all other cases, this is the `kid` pointing to the certificate associated with the session for the sending peer. This must match the identity provided in the user\_identity field. See the session authentication process section for more details.
- **user\_identity**: The identity of the user associated with this session and replica. Usually this is an email address identifying the user associated with this node (e.g., "<jeff@reframe.com>"), however it can be a URI/URL identifier of an intelligent agent. Either way, it must match the identity in the associated user\_auth\_cert. If this is omitted, then the identity in the cert is assumed.
- **status\_message**: UTF-8 string containing information messages or error details about the connection status. This field is optional, but should be provided if the peer's hello message is being rejected (e.g., "The provided credentials are not valid"). If the status\_message indicates an explicit error, the sender will also send a following ALSP error message documented later in this specification.

Note: If there is a problem meeting the parameters of the hello message, the response should contain an appropriate status\_message string and/or be followed by an **ALSP error** message. If a hello message is rejected due to authentication or protocol constraints, the receiver MUST respond with an ALSP error message with `disconnect: true` before closing the connection.

## **Sync Pull Request (Explicit Poll)**

This message is an explicit polling request to indicate which Channel log you are requesting messages for. When the ALSP session is being used in push mode, the ALSP sync\_request message simultaneously serves as a subscription request for the associated channel and once the session is caught up to real-time, further updates will come as push updates.

```json
{
    "alsp_msg_type": "sync_request",    // An explicit ALSP poll message
    "credentials": "eyJaiJFZERTQSJ9..." // Channel access proof (JWS)
    "timestamp": "<timestamp>",         // Timestamp of this message
    "lamport_max": 16569909,            // Local lamport global max value
    "from_lamport": 10500,              // Lower bound (inclusive)
    "to_lamport": 11000,                // Upper bound (inclusive)
    "node_id": "uuid",                  // Our replica node uuid
    "channel_id": "uuid",               // Channel requested
    "log_digest": "sha256:abcd...",     // Digest of Message ID in channel log
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "sync\_request")
- **credentials**: JWS compact serialization containing the channel access proof. The JWS payload contains `channel_id`, `peer_session_nonce`, and `timestamp`, signed using the Channel Access Key (CAK) private key to prove authorization to access the requested channel.
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **lamport\_max**: The requesting replica's current Lamport clock global maximum value, used for counter synchronization across the network. This represents the highest Lamport value seen from any source and should match the local node's current Lamport counter per global synchronization requirements.
- **from\_lamport**: Lower bound of the desired Lamport clock range (inclusive). If there are gaps in the current channel's log, those gaps may be considered "filled" if the replica has messages from other channels with Lamport values that cover the missing range. This allows the replica to treat its local state as contiguous even when some channel-specific messages are missing. The receiving node uses this value to determine which messages from its own channel log need to be sent in response.
- **to\_lamport**: Upper bound of the desired range (inclusive, optional). For scenarios requiring gap recovery, audit replays, or partial rehydration of historical logs, this field limits the sync response to a specific Lamport time window.
- **node\_id**: The universally unique identifier of the replica node making the sync request. This excludes messages the local replica originated, preventing redundant transmission. If omitted, all possible messages are sent in the response. This is useful for reconstructing lost replicas or populating new nodes.
- **channel\_id**: The universally unique identifier of the specific channel being requested for synchronization. In sync mode, this should be treated as channel subscription initiation as well.
- **log\_digest**: Optional - SHA-256 digest of all Layer 0 `message_ids` in the channel log up to (but not including) the specified `from_lamport` value. The digest MUST be computed using SHA-256 on concatenated UTF-8 message\_id values in canonical log order. See the Channel Health Check and Recovery section for usage details.

**Usage:**

- This message MAY be sent by a replica to enable sync mode negotiation.
- Servers MAY fulfill the range with a single sync\_response for each relevant channel or in batches using more: true.
- Message results MUST still be deduplicated by `message_id` and inserted in canonical order.

## **Sync Pull Response (Explicit Response)**

This message is the reponse to a ALSP sync\_request message. There may be multiple response messages to a single request as indicated by the more field. The sending should send as many messages as possible at at time while respecting the maximum total response length agreed to in the ALSP hello.

```json
{
    "alsp_msg_type": "sync_response",   // Response(s) to a sync_request
    "timestamp": "<timestamp>",         // Timestamp of this message
    "lamport_max": 16569909,            // Peer's lamport global max value
    "channel_id": "uuid",               // Channel being supplied 
    "more": false                       // More responses to follow?
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "sync\_response")
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **lamport\_max**: The responding replica's current Lamport clock global maximum value, used for counter synchronization across the network
- **channel\_id**: The universally unique identifier of the specific channel being supplied in the response
- **more**: Boolean flag indicating whether additional messages remain to be sent (false if this is the final response batch).

The **messages** associated with the Sync Pull Response go in the "alsp\_payload" section of the top level message header in Transport Layer Message Format containing the requested channel log entries. These should already be sorted although they may not all go into the same location in the local log and most be inserted into the local log in proper order regardless of the order they are presented in.

**Important:** A sender must send separate sync reposnse for each `channel_id`. Messages from different channels cannot be combined into a single sync\_response message.

## **Sync Live Update  (Push Model - Optional)**

This ALSP message is used for asynchronous real-time sharing of new local messages. This mechanism is only used if explicitly allowed as a result of the ALSP hello negoiation process otherwise the synchronous mechanism must be used. Note, as discussed in regards to the ALSP hello message, the push model must be used for persistent connections to prevent unnecessary polling. Once a channel log is operating in live sync update mode, the sender must pass on new log entries for subscribed channels with minimum latency.

```json
{
    "alsp_msg_type": "sync_update",  // Indication of an async live update 
    "timestamp": "<timestamp>",      // Timestamp of this message
    "lamport_max": 16569909,         // Peer's lamport global max value
    "channel_id": "uuid",            // Channel being supplied   
}
```

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "sync\_update")
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **lamport\_max**: The peer's current Lamport clock global maximum value, used for counter synchronization across the network
- **channel\_id**: The universally unique identifier of the specific channel being supplied in the update. This field is optional when the only purpose of the sync\_update is to globally distribute an updated lamport\_max value. To prevent a storm of lamport\_max-only sync\_updates, this type of sync\_update must only be sent when your lamport\_max has advanced beyond the value last exchanged with this peer.

The **messages** associated with the Sync Live Update go in the "alsp\_payload" section list of top level message header in Transport Layer Message Format containing the latest available channel log entries. These should already be sorted although they may not all go into the same location in the local log and must be inserted into the local log in proper order regardless of the order they are presented in.

The sync peer will send as many messages as allowed in each update batch, respecting the maximum message length negotiated during the hello exchange. Unlike sync responses, no `more` field is required because updates are sent continuously as new messages become available at the sending peer.

**Important:** A sender must send separate sync updates for each `channel_id`. Messages from different channels cannot be combined into a single sync\_update message.

# Push vs. Pull Mode Operation

ALSP operates in one of two synchronization modes:

**Pull Mode (Default)**: Updates must be explicitly requested via sync\_request messages. The peer responds with available messages but does not send automatic updates.

**Push Mode (Optional)**: After initial synchronization, the peer automatically sends sync\_update messages when new messages become available for subscribed channels.

## Negotiating Push Mode

During the hello exchange, peers negotiate push capabilities:

- Connecting peer sets `push_enabled: true` to request push mode
- Receiving peer confirms or denies based on its own `push_enabled` response
- Server nodes MUST implement push mode; client/distributed peers SHOULD implement it for efficiency

## Push Mode Operation

When push mode is negotiated successfully:

1. **Channel Subscription**: Each sync\_request acts as a subscription request for that channel\_id
2. **State Transition**: Once the sync\_response completes (bringing the peer up to date), that channel enters push mode for the remainder of the session
3. **Automatic Updates**: The peer automatically sends sync\_update messages whenever new messages become available for subscribed channels, without requiring further sync\_requests

This eliminates polling overhead and reduces latency for real-time synchronization scenarios.

**Important**: Push subscriptions are unidirectional. If both peers want to receive automatic updates for the same channel, each peer must send its own sync\_request to subscribe. Otherwise, push updates will only flow in one direction.

## Channel State Transitions

Channel Sync States are maintained per peer connection for nodes that handle multiple inbound connections:

- **Unsubscribed**: No sync relationship established for this channel
- **Pull Sync**: Channel actively being synchronized via sync\_request/sync\_response cycles
- **Push Sync**: Channel subscribed for real-time sync\_update delivery
- **Error State**: Channel temporarily unavailable due to auth/transport failures

**State Transition Rules:**

1. A channel enters **Pull Sync** when a sync\_request is received and authorized
2. A channel transitions to **Push Sync** when:
   - The initial sync\_request has been completely fulfilled (more = false in final sync\_response)
   - Both peers negotiated push\_enabled = true during hello
3. A channel returns to **Pull Sync** if push delivery fails after 3 consecutive attempts

**Replica State Tracking:**  
Each replica MUST maintain a per-peer, per-channel subscription table tracking which remote peers subscribed for push updates for which specific channels. This table is populated when:

- A sync\_request completes successfully
- Explicitly cleared when a connection closes or authentication fails

## Push Delivery Failure Handling

**Failure Detection:**

- Transport-level failures (WebSocket disconnect, timeout)
- Protocol-level failures (invalid JWS, authorization expired)

**Fallback Behavior:**

1. On push delivery failure, mark the channel as **Error State** for that peer
2. The failed peer MUST re-initiate sync via sync\_request to resume
3. Senders SHOULD NOT automatically retry push delivery to avoid message storms
4. Optionally, send an ALSP error message with error\_code "push\_failed" and suggested\_action "Re-sync via pull request"

**Graceful Degradation:**  
Replicas operating in unreliable network conditions MAY disable push mode entirely and rely on periodic pull sync to ensure eventual consistency.

# ALSP Node Topology

ALSP's symmetric protocol design enables flexible deployment topologies ranging from centralized client-server architectures to distributed peer-to-peer networks. The choice of architecture is orthogonal to the protocol itself—any replica can serve sync requests to any other authorized replica regardless of network topology.

## Deployment Patterns

**Centralized Architecture**: The most common deployment pattern involves clients connecting to a centralized "server" that acts as both a data backup repository and primary replication partner. This server maintains authoritative copies of all channel logs and coordinates synchronization across multiple client replicas.

**Distributed Architecture**: ALSP also supports loosely distributed sets of nodes where peers connect to peers without requiring a central authority. The specific peer discovery and connection configuration mechanisms are deployment-specific and not defined by this protocol specification.

## Connection Roles

**Servers**: Must accept inbound connections and serve sync requests from authorized clients. Servers never initiate outbound connections to clients, operating purely in a responsive mode for connection establishment. To classify as a server in the protocol, a server MUST take connections from multiple nodes. Servers MUST refuse to authenticate a session with a node\_id replica that already has an established connection.

**Clients/Peers**: Typically initiate outbound connections to their configured sync partners. Clients may optionally accept inbound connections from other peers, but this is not required for basic operation. Given the full-duplex and symmetrical nature of the protocol, the topology MUST be constructed such that peers only maintain a single point-to-point connection with each other. For peer-to-peer operation, a peer MUST NOT attempt to connect to another node that already has a connection established regardless of who established it.

## Server Synchronization Behavior

By definition, servers not only respond to sync requests and send channel updates to clients, but also actively request live updates from connected clients. When a client supports push mode, the server automatically receives new content via sync\_update messages. However, if a client does not support push mode, the server must periodically issue sync\_request messages to collect new client content and maintain up-to-date channel logs.

This bidirectional synchronization ensures that servers maintain comprehensive, current views of all channel activity across their connected client base, regardless of individual client capabilities.

# Error Handling and Disconnects

ALSP defines a structured error message to handle protocol violations, authentication failures, and capability mismatches between replicas. These messages should be considered informational.

## Error Message Format

```json
{
  "alsp_msg_type": "error",           // Error message type identifier  
  "timestamp": "rfc-3339-timestamp>", // Timestamp of this message
  "error_code": "invalid_auth",       // Machine-readable error code
  "reason": "utf-8-string",           // Human-readable error explanation
  "suggested_action": "utf-8-string", // Corrective action guidance
  "disconnect": true                  // Whether session should be closed
}
```

### Fields

- **alsp\_msg\_type**: Message type identifier for ALSP protocol messages (e.g., "error")
- **timestamp**: This contains the RFC 3339 UTC timestamp of when the message was generated. This is used as part of the message authorization.
- **code**: A short, machine-readable identifier for the error type - codes are listed below
- **reason**: Human-readable explanation of the error (e.g., "certificate does not match claimed identity")
- **suggested\_action**: Optional guidance for corrective action (e.g., "Reconnect with a valid identity certificate")
- **disconnect**: Boolean flag indicating whether the sender intends to close the ALSP session immediately after sending this message

## Common Error Codes

| Code                 | Description                                                                                                                   |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| invalid\_auth        | Certificate does not match claimed identity or is untrusted.                                                                  |
| unauthorized         | Replica is not permitted to access the requested channel.                                                                     |
| hash\_mismatch       | A validation SHA-256 hash mismatch has occurred and this implies log divergence.                                              |
| stale\_timestamp     | The message was rejected due to the timestamp being out-of -range. Replay attack is suspected.                                |
| protocol\_violation  | Message structure or sequence violated ALSP semantics.                                                                        |
| unsupported\_version | Peer sent an incompatible message format or spec version.                                                                     |
| payload\_too\_large  | Message exceeds max\_alsp\_length or violates buffer constraints.                                                             |
| internal\_error      | Sender encountered an unexpected failure.                                                                                     |
| re-auth-required     | This error can be generated to force the termination of a sync session. The disconnect parameter MUST be true for this error. |

## Usage

- Error messages may be sent in response to any ALSP message.
- A hello response with an error code is a valid rejection of the handshake.
- The receiver of an error message should log or report the error and must honor the disconnect directive if set to true.

# **Consistency Guarantees Summary**

ALSP guarantees **eventual consistency** across authorized replicas through a hierarchical model of immutability and deterministic ordering.

**At the articulation staement level**, each articulation is globally unique and immutable with verifiable authorship and timestamps. The protocol prevents duplication or mutation once messages are replicated, ensuring message integrity across the network.

**At the log level**, each channel maintains an append-only stream where deletions and overwrites are prohibited. Messages are stored and transmitted in canonical lamport clock order using the message\_id for tie-breaking the order, providing deterministic sequencing within each channel and also globaly across logs.

**For cross-channel coordination**, while messages are transported per-channel, all valid messages received by a client are merged into a single global timeline. This unified timeline serves as the foundation for DAG construction and user-facing coordination history, where channel membership determines visibility rather than creating logical isolation between channels.

This consistency model enables multi-writer safety without conflicts, full offline write and sync capabilities are replayable, globally deterministic views for every participant in the network.

# **Transport Bindings**

While ALSP is designed to be transport-agnostic, this specification defines a WebSocket-based binding as the primary implementation. WebSockets provide the optimal characteristics for log synchronization: persistent bidirectional connections that support both push and pull sync models, low-latency message delivery, and efficient batching of articulation entries. Other transport bindings such as HTTPS and gRPC may be considered as future options.

## **ALSP over WebSockets with TLS**

The WebSocket binding leverages the full-duplex, persistent nature of WebSocket connections combined with Dot Preserving Binary (DPB) encoding to support high-throughput, resilient synchronization even in the face of large backlogs or offline replay scenarios.

### **Transport Characteristics**

WebSockets provide optimal support for ALSP's bidirectional synchronization requirements:

- Full-duplex communication allows both client and server to initiate sync operations without polling
- Persistent connections reduce overhead compared to repeated HTTPS requests
- Native support for live updates and push-based delivery

### **TLS Connection Setup**

This section normatively specifies the default ALSP transport over TLS. All ALSP connections establish secure communication through a standardized process that combines transport-level security via TLS 1.3 WebSocket connections with protocol-level authentication through the ALSP auth\_request, auth\_challenge, and hello message exchange. The following requirements define the specific connection establishment procedures:

1. **Endpoint and scheme.** For a given domain, the canonical ALSP endpoint **SHOULD** be a WebSocket Secure (WSS) origin at port **443**, e.g., wss\://ascp.\<domain>. Clients **MUST** use the wss scheme and TCP port 443 unless explicitly configured otherwise.
2. **TLS and ALPN.** The client **MUST** initiate a TLS handshake to the origin on TCP/443. During the handshake, the client **MUST** offer **ALPN** protocols h2 and http/1.1; the server **MUST** select one. Implementations **MUST**support TLS 1.3; support for TLS 1.2 is **OPTIONAL**.
3. **WebSocket establishment.**
   - If ALPN selects **HTTP/2**, the client and server **MUST** establish the WebSocket using the **Extended CONNECT** mechanism per RFC 8441 with :protocol = "websocket".
   - If ALPN selects **HTTP/1.1**, the client and server **MUST** establish the WebSocket using the standard HTTP/1.1 **Upgrade** handshake.
4. **Connection model.** After the WebSocket is established, ALSP **MUST** operate over a **persistent, full-duplex** TCP connection. Application data **MUST** be carried as **WebSocket binary messages**.
5. **Framing and message unit.** Each WebSocket **binary message** **MUST** contain exactly one ALSP envelope (the deterministic-CBOR Layer‑0 envelope, that contains the DPB JWS payload as defined elsewhere in this specification). WebSocket fragmentation is transport-internal and **MUST NOT** be observable at the ALSP layer.
6. **Compression.** Implementations **SHOULD NOT** negotiate per-message compression extensions (e.g., per-message-deflate) for ALSP frames by default, as Layer 1 payloads already employ JWE compression where appropriate, and compression can introduce security vulnerabilities like CRIME and BREACH attacks.
7. **Keepalive and liveness.** Endpoints **SHOULD** use WebSocket **Ping/Pong** frames to detect half-open connections and to maintain NAT/firewall mappings at reasonable intervals.
8. **Subprotocol identification.** Servers **SHOULD** advertise, and clients **SHOULD** request, the WebSocket subprotocol token `ascp.alsp.v1` (where v1 corresponds to ALSP version 0.x for backward compatibility) via Sec-WebSocket-Protocol to enable policy and routing at intermediaries. If the subprotocol is not accepted, the connection **MAY** proceed without it.
9. **Proxy traversal.** When traversing explicit HTTP proxies, clients **MAY** use HTTP CONNECT tunneling prior to TLS, per HTTP semantics. Once tunneled, the requirements in (2) and (3) apply.
10. **Error handling and reconnect.** On transport failure (TLS or WebSocket), clients **SHOULD** immediately re-establish a new WSS connection and resume ALSP synchronization per the ALSP state machine (hello/auth and channel resubscription rules defined elsewhere in this specification).
11. **WebSocket close handling.** When ALSP protocol errors occur, implementations **SHOULD** close the WebSocket with code 1002 (Protocol Error) and include the ALSP error\_code in the close reason where possible.

### Message Encoding and Batching

This section specifies the on‑wire encoding for ALSP messages. It defines a compact CBOR envelope that batches Layer‑0 entries while carrying a flexible message header as a JSON blob. The entire CBOR envelope is ultimately integrity‑protected via a JWS signature.

- **Envelope:** The ALSP envelope is encoded as **deterministic CBOR**.
- **Header:** alsp\_msg\_header is a **CBOR byte string** whose contents are **UTF‑8 JSON**. The CBOR layer treats it as opaque.
- **Payload:** alsp\_msg\_payload is an optional **CBOR array** of Layer‑0 entries for efficient batching.

This design yields a compact, canonical, and extensible envelope while preserving maximum flexibility for header fields.

**CDDL (normative)**

```cddl
; Deterministic CBOR is required: ascending map keys, shortest
; integer encodings, definite lengths, and no semantic tags
; or indefinite-length items.

ALSP-Envelope = {
  0: tstr,            ; alsp_version (e.g., "0.1")
  1: bstr,            ; alsp_msg_header: UTF-8 JSON bytes (opaque)
  ?2: [* L0-Entry]    ; alsp_msg_payload: batch of Layer-0 entries
}

; L0-Entry is defined elsewhere and remains CBOR (not repeated here).
; It includes the lamport time, message_id (UUID bytes), and DPB-encoded Layer-1 JOSE payload.

```

**Deterministic CBOR requirements (normative)**

1. Map keys **MUST** be encoded in ascending order (for ALSP-Envelope: 0, 1, 2).
2. Integers **MUST** use the shortest valid encodings.
3. Arrays and byte strings **MUST** use **definite lengths**.
4. Implementations **MUST NOT** use semantic tags or indefinite-length items in the envelope.

**alsp\_msg\_header (JSON-in-bytes)**

- The value of alsp\_msg\_header **MUST** be a valid **UTF‑8** encoded **JSON** document (RFC 8259). I‑JSON (RFC 7493) is RECOMMENDED.
- To ensure stable signatures across implementations, producers **SHOULD** serialize this JSON in **canonical form**(e.g., JCS / RFC 8785).
- Receivers **MUST** treat the header as an **opaque byte string** at the CBOR layer and **MUST NOT** re‑serialize it when verifying signatures.
- The schema of the JSON header is intentionally flexible. Typical fields include, but are not limited to, alsp\_msg\_type, timestamp (RFC 3339 UTC), lamport\_max, channel\_id, node\_id, more, and any additional routing or metadata fields required by the implementation.

**Example (CBOR diagnostic notation)**

```cbor-diag
{
  0: "0.1",
  1: h'7B22616C73705F6D73675F74797065223A2273796E635F726573706F6E7
       365222C2274696D657374616D70223A22323032352D30392D3035543132
       3A33343A35365A222C226C616D706F72745F6D6178223A3136353639393
       0392C226D6F7265223A66616C73657D',
  2: [
    { 0: 1345678,
      1: h'550e8400e29b41d4a716446655440000',
      2: h'...DPB bytes...' }
  ]
}

# Field-by-field breakdown
# a3                          : map(3)
#   00                        : key 0 (uint 0)
#     63 30 2e 31             : tstr len=3  → "0.1"
#   01                        : key 1 (uint 1)
#     58 68                   : bstr len=0x68=104 (header JSON)
#       7b .. 7d              : 104 bytes UTF-8 JSON
#   02                        : key 2 (uint 2)
#     81                      : array(1)
#       a3                    : map(3) — L0 entry
#         00                  : key 0 (uint 0)
#           1a 00 14 88 8e    : uint(1345678)
#         01                  : key 1 (uint 1)
#           50                : bstr len=16 (UUID)
#             55 0e 84 00 e2 9b 41 d4 a7 16 44 66 55 44 00 00
#                             : 16 bytes UUID
#         02                  : key 2 (uint 2)
#           44                : bstr len=4 (DPB bytes)
#             de ad be ef     : example DPB payload bytes
```

All multi-byte integers in CBOR use network byte order (big-endian) as per RFC 8949. The value at key 1 above is a **bstr** whose hex decodes to this JSON (UTF‑8):

```json
{
  "alsp_msg_type":"sync_response",
  "timestamp":"2025-09-05T12:34:56Z",
  "lamport_max":16569909,"more":false
}
```

**Normative Summary**

1. ALSP envelopes **MUST** be encoded as deterministic CBOR with integer keys 0, 1, and optional 2.
2. alsp\_msg\_header **MUST** be a CBOR bstr containing valid UTF‑8 JSON; producers **SHOULD** use canonical JSON
3. alsp\_msg\_payload **MAY** be present and, when present, **MUST** be a CBOR array of Layer‑0 wrapped Layer-1 DPB encoded entries.
4. The **raw CBOR bytes** of the envelope are the **JWS payload**. JOSE headers carry algorithm, key id, and content‑type hints as needed.
5. alsp\_msg\_payload enables efficient transfer of many Layer‑0 entries in a single envelope while preserving discrete message boundaries and canonical ordering via lamport times.
6. Each L0 entry encapsulates a DPB‑encoded Layer‑1 JOSE payload. DPB applies to Layer‑1 and JOSE streams; the CBOR envelope itself is not DPB‑encoded.

### **Sync Operations**

The WebSocket binding supports both pull- and push-based synchronization flows:

- **Pull**: A replica sends a sync\_request with a channel\_id  to request messages it is missing. The server replies with a sync\_response containing a batch of messages, with batch sizes controlled by the `max_alsp_length` negotiated during hello and segmented via the `more: true` flag when needed.
- **Push**: A replica that has just appended message(s) to its local log can immediately send an sync\_update message to connected peers, propagating updates proactively. These updates respect the same batching constraints and may contain multiple Layer 1 messages when syncing bursts of activity.

This hybrid model ensures high throughput without sacrificing structure or extensibility. It allows replicas to stay in sync even over unreliable or intermittent connections, and provides a clean foundation for building resumable, incremental sync protocols in future ALSP evolutions.

### **Two-Level Authentication Model**

All transport endpoints MUST enforce TLS 1.3 or higher for secure connection establishment and ALSP-level peer session authentication. ALSP implements a dual authentication approach:

1. **Transport Level (TLS)**: Implementations MUST establish a strong cipher based TLS connection to provide end-to-end encryption with a unique session key. For client-server configurations operating on public internet or corporate infrastructure, implementations SHOULD validate server certificates against PKI infrastructure to authenticate the server domain. For peer-to-peer configurations, implementations MAY use alternative authentication techniques, though such mechanisms are outside the scope of ASCP. 
2. **Protocol Level (ALSP)**: Once the secure transport connection is established, ALSP auth\_request, auth\_challenge (as needed), and hello message exchange provide the mandatory peer authentication and authorization that applies at the channel replica level. This sequence establishes the session and forms the basis for ongoing message authentication.

**Access Control**: Clients can access their own user articulation channel and additional channels based on explicit membership, with all channel access verified through Channel Access Key (CAK) authorization.

Certificates used for ALSP may be self-signed, org-issued, or PKI-rooted per ASCP trust model.

# **Session Authentication**

ALSP implements protocol-level authentication that operates above the transport layer (typically TLS 1.3) to authenticate peer identities and authorize synchronization operations to procee. Session authentication achieves four key objectives:

1. **Identity Authentication**: Reliably authenticate the connecting peer and bind that identity to the session
2. **Session Authorization**: Authorize access to channel specific processes so that replica establishment and/or synchronization can proceed.
3. **Replay Protection**: Prevent message reuse across different sessions through session nonces
4. **Transport Agnostic**: Maintain security independent of underlying transport mechanisms

The complete identity establishment and trust model is detailed in the **ASCP Trust and Identity** specification. This section focuses on the specific ALSP messages that implement the authentication flows described in that specification.

Session authentication follows one of two paths using a **unified dual-nonce model**:

- **Challenge Flow**: auth\_request → auth\_challenge → hello exchange (for new identity binding or reauth challenges)
- **Immediate Flow**: auth\_request → hello exchange (for known, bound identities)

Both flows use session-specific nonces for replay protection and establish the cryptographic foundation for all subsequent message authentication.

## ALSP Authentication Modes

ALSP supports two ways for clients to authenticate and bind themselves to an identity during session setup:

### **Direct Mode**

In direct mode, the client presents a long-term identity and identity key when connecting. This identity key may already exist — or it may be established for the first time during the session.

Direct mode supports two main scenarios:

- Continuing an existing identity (e.g., signing in from a known device and bound key)
- Establishing a **new identity** by self-generating a key and binding it to the presented identity — all while keeping the private key secret from any intermediary

> Use this mode when:
> You already have your identity key, or you want to generate and bind a new one without anyone else ever seeing it.

### **Provisioned Mode**

In provisioned mode, the client does not yet have an identity key and requests one to be provisioned or recovered. Instead of presenting an identity key, the client provides its public recovery key. The server responds by securely issuing an identity key in the required auth\_challenge response. If the client's identity has an existing key on record, the server provides the recovered key; otherwise, it generates a new identity key for first-time binding. Either way, the key presented in the auth\_challenge is a JWE secured **user-key-envelope** that has been secured by the recovery\_key presented in the auth\_request.

> Use this mode when:
> You’re setting up a new device, recovering access, or receiving your identity from a trusted issuer.

### **Initial Identity Bootstrap (First-Time Only)**

New participants establish verifiable identity by generating a self-sovereign ECDSA P-256 key pair, responding to an ASCP challenge nonce, and authenticating with a trusted identity provider. This creates an IdentityClaimBundle that cryptographically links their public key to their verified identity using nested JWS signatures. The ASCP peer validates and logs this binding as an immutable Identity Artipoint. See the **ASCP Trust and Identity** specification for more details of the Identity bundles and binding of a public key to a specific identity via an Identity provider.

> **Why dual nonces?** Each side controls one nonce. Messages you send must carry *the other side’s* nonce to prove they were created **for this session**. This prevents cross‑session and cross‑peer replay, even on connectionless transports where you know you picked the nonce to be used for validation.

### **Challenge Flow (New Identity Binding)**

This flow is used for first-time connections where new identity binding is required, or when the server needs to re-authenticate the connecting peer's identity.

- Client sends auth\_request message with their chosen session nonce, identity information, and JWK for their public identity key (direct mode) or their public recovery key (provisioned mode). The JWS protected header uses the client's own nonce and typically omits the `kid` field since no binding exists yet.
- Server responds with auth\_challenge containing the challenge nonce, which becomes the server's session nonce for the client to use in all future messages.
- For provisioned mode, the generated or recovered client identity is sent as the **client\_key\_env** JWE encrypted using the `recovery_cert` send in the auth\_request. That is the entire user-key-envelope is itself JWE encoded using the recovery\_cert to make sure nothing about keys is visible to anyone except for the far end receiver.
- Client sends hello immediately after receiving auth\_challenge, using the server's nonce in the JWS protected header and including the JWS-signed Identity Token Package in the `user_auth_cert` field to establish identity binding.
- Server responds with hello to complete mutual authentication, or sends an error if identity binding fails.

### **Immediate Flow (Known Identity)**

This is the standard authentication process when peers already have established, trusted identity-key bindings.

- Client sends auth\_request with their session nonce, identity, and JWK for their public identity key. The JWS protected header includes both the client's nonce and the `kid` for their signing certificate.
- Server validates the key-identity binding immediately and responds directly with hello, skipping the challenge step.
- Server's hello uses the client's nonce in the JWS protected header and includes the server's own session nonce in the message payload.
- Client responds with their own hello, using the server's nonce in the protected header.

# **Message Authentication**

After authentication completes, every message's JWS protected header must carry the peer's session nonce. All messages are signed with the sender's private key and verified using their registered public key, providing cryptographic proof of identity and session binding.

## **Benefits of This Model**

- **Self-sovereign keys**: No third party controls user private keys after bootstrap
- **Cryptographic integrity**: Every message is signed and verifiable
- **Provider flexibility**: Works with any JWT-capable identity provider
- **Replay resistance**: Challenge nonces prevent replay attacks
- **Immutable audit trail**: All identity bindings are logged permanently

## Session Lifecycle & Resilience

- **Fresh nonces per connection.** Nonces MUST be unique per session and **never reused** across reconnections. On reconnect, the flow restarts at auth\_request → auth\_challenge → hello(s).
- **Idle/expiry.** Implementations MAY expire sessions after inactivity or token expiry, returning re-auth-requiredwith disconnect: true to force a clean rebind.
- **Key caching.** Session‑cached verification keys MAY be retained for the session lifetime. Caches MUST be cleared on session teardown. If a CAK rotates mid‑session, send error with code: "re-auth-required" and disconnect: true.

## Errors

Use the standard ALSP error message. Guidance

- invalid\_auth: Bad signature, unknown kid, or cert mismatch.
- unauthorized: Token or CAK proof rejected.
- stale\_timestamp: Outside allowed skew window.
- protocol\_violation: Wrong typ/nonce usage or message order.
- re-auth-required: Force a full restart of the auth sequence.

## JWS-Based Message Authentication

All ALSP messages are secured using JWS compact serialization, where the complete message structure (including headers and payload array) is cryptographically signed and encoded into the ALSP Dot Preserving Binart (DPB).

Each ALSP session begins with authentication using one of two flows: Auth\_Request/Hello exchange for known peers, or Auth\_Request/Auth\_Challenge/Hello sequence for new identity binding. During establishment, both peers generate unique 32-character hexadecimal `session_nonce` values that provide replay protection for the entire session.

### **Nonce Usage Pattern**

- **Auth Request Messages**: Use your own `session_nonce` in the JWS protected header as we are just trying to prove possession of the private key that goes with the JWS signature on the request itself.
- **All other messages**: Use the peer's `session_nonce` in the JWS protected header to prove session binding to the current session.

This nonce pattern is core to proving both identity authentication during session establishment and replay protection for all subsequent messages.

### JWS Structure

**Protected Header:**

```clike
{
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "alsp",        // or "alsp+auth" for auth request messages
  "nonce": "session_nonce_value"  // Peer's nonce except for Auth Request
}
```

**alg**: The cryptographic algorithm used for JWS signature generation, typically "ES256" for ECDSA using P-256 curve and SHA-256.

**kid**: The kid field is empty or omitted for auth request messages and hello messages before a proper identity binding exists, otherwise it refers to the certificate that is being used for ongoing message authentication through the session.

**typ**: The JWS application type for the particular signed message. This is "alsp" for all messages except for the auth\_request message and hello message sent by a peer before a proper identity binding exists.

**nonce**: The session nonce value used for replay protection. For auth\_request messages, this is the sender's own session\_nonce. For all other messages, this is the peer's session\_nonce to prove session binding.

**JWS Payload:**  
The JWS payload contains the complete ALSP envelope encoded as deterministic CBOR bytes, as defined in the Transport Bindings section. This envelope includes the ALSP version, message header (containing alsp\_msg\_type, timestamp, channel\_id, lamport\_max, etc. as UTF-8 JSON bytes), and optional payload array of Layer-0 wrapped Layer-1 articulation statement entries. This ensures cryptographic integrity protection for both routing metadata and articulation content at the transport layer. The `timestamp` field within the ALSP message header must be populated with the current time in RFC 3339 UTC format at message generation for replay protection.

**Wire Format:**

```clike
<DPB_encoded_JWS_bytes>
```

Where the DPB-encoded bytes represent the JWS compact serialization: `<JWSprotectedHeader>.<JWSpayload>.<JWSsignature>`

### Validation Process

Upon receiving any ALSP message, the receiving peer must validate the JWS in the `alsp_msg` field:

1. **Decode DPB and Parse the JWS**: First decode the DPB-encoded bytes to recover the JWS compact serialization, then extract protected header, payload, and signature components
2. **Validate the protected header**: Ensure `alg` matches expected algorithm, `kid` references a known certificate, and `nonce` matches the expected value (sender's nonce for hello, receiver's nonce for others)
3. **Verify the signature**: Using the sender's public key from the hello exchange
4. **Validate the payload**: Decode JSON and ensure `timestamp` is within acceptable bounds

This process ensures message authenticity, non-repudiation, session binding, and header integrity protection. If validation fails, the peer must reject the message and send an appropriate error response.

### Replay Attack Protection

To prevent replay attacks, receiving peers may reject messages whose timestamps are out of sync with the local clock by more than one minute. This temporal validation provides protection against both accidental message duplication and malicious replay attempts.

Implementations should account for reasonable clock skew between peers while maintaining security. The one-minute window provides a balance between security and operational flexibility, accommodating minor clock differences and network latency while preventing significant temporal manipulation.

Peers operating in environments with known clock synchronization challenges may choose to implement more lenient temporal validation, but should never accept messages with timestamps more than several minutes out of sync, as this significantly increases replay attack vulnerability. When authentication is refused as a result of timestamp violation a specific error message indicating this should be sent.

# Channel Permissions: **Proof via Signing Key**

To support secure, verifiable access control to channel logs at Layer 0—without requiring Layer 1 payload inspection—ALSP introduces a **channel-level authentication mechanism** called **Channel Access Proof**.

This mechanism enables a client to prove possession of a **channel-specific signing key** (sk), without revealing the key itself. The server validates this proof using a public key (pk) that is published at channel creation time via the channel manifest.

**Note:** Channel access authorization operates independently from message authentication. While all ALSP messages use JWS signatures via the `alsp_msg` field for session-level authentication, channel access requires an additional authorization proof using channel-specific credentials. This dual-layer approach ensures that authenticated users can only sync channels they are explicitly authorized to access.

## **Channel Access Key (CAK)**

Each ASCP Channel includes a **Channel Access Key (CAK)** defined at creation time. The CAK is an Ed25519 keypair:

- **Public Key (pk)** — Published and distributed during the bootstrap process for verification
- **Private Key (sk)** — Used by authorized clients to create JWS signatures for channel access proofs; must remain secret and only be available to authorized channel members

The CAK public key is referenced via the `kid` field in JWS headers (e.g., `"kid": "ascp:cak:550e8400-e29b-41d4-a716-446655440002"`) and must be discoverable through the bootstrap process.

This key authorizes access to the channel log at Layer 0. It is conceptually distinct from any content encryption keys used at higher layers.

The associated private key is distributed to authorized participants through secure out-of-band mechanisms during channel membership establishment.

The CAK public key must be established at channel creation time and distributed via the bootstrap process, ensuring trust in the origin of the public key.

## **Channel Sync Request Auth Proof**

A client that wishes to synchronize a channel log protected by a CAK must include a `credentials` field in its sync\_request message:

```json
"credentials": "<JWSprotectedHeader>.<payload>.<JWSsignature>"
```

**JWS Protected Header:**

```json
{
  "alg": "EdDSA",
  "kid": "ascp:cak:550e8400-e29b-41d4-a716-446655440002",  
  "typ": "alsp+cak"
}
```

**JWS Payload:**

```json
{
  "channel_id": "550e8400-e29b-41d4-a716-446655440002",
  "nonce": "abcd1234567890abcdef1234567890ab",
  "timestamp": "2024-01-15T10:30:45.123Z"
}
```

The `kid` field references the Channel Access Key public key, and the payload contains the same three security components as before.

The indicated `peer_session_nonce` is the peer's nonce for session and `timestamp` is the timestamp from the sync\_request message itself.

## **Security Rationale for Channel Access Proof**

The Channel Access Proof signature includes three critical components that each serve a specific security purpose:

- `channel_id`: Ensures the proof is bound to the specific channel being requested, preventing authorization confusion between channels
- `peer_session_nonce`: Cryptographically binds the proof to the current session context, preventing cross-session replay attacks where a captured message from one session could be replayed in another session
- `timestamp`: Provides temporal replay protection within the same session context

This three-part signature is essential for **transport-agnostic security**. In connectionless transport modes, ALSP cannot rely on persistent session state to detect replays, so each message must be self-authenticating against all forms of replay attack. Even in connection-oriented transports like WebSockets, this provides defense-in-depth against sophisticated attackers who might capture and replay messages across different sessions or time periods.

Without the `peer_session_nonce` binding, an attacker could capture a valid channel access proof from one session and replay it in another session during the timestamp validity window, gaining unauthorized access to channel logs.

### **Server Verification Procedure**

1. **Parse the JWS**: Extract protected header, payload, and signature from the `credentials` field
2. **Validate the header**: Ensure `alg` is "EdDSA" and `typ` is "alsp+cak"
3. **Look up the public key**: Use the `kid` field to locate the Channel Access Key public key from the channel manifest
4. **Verify the JWS signature**: Using standard JOSE JWS verification with the Ed25519 public key
5. **Validate the payload**: Ensure:
   - `channel_id` matches the requested channel
   - `nonce` matches this session's peer nonce
   - `timestamp` is within acceptable window (±60 seconds)
6. If verification passes, allow sync to proceed; else, return an ALSP error\_code of "unauthorized" with a reason of "Channel credentials invalid for channel\_id"

Note how the server **never needs to know the private key,** but clients unable to produce a valid JWS signature **cannot sync the channel**.

### **Channel Access Key (CAK) Declaration**

This mechanism relies on the CAK being declared as metadata associated with the channel. This is surfaced in the channel\_manifest, which is known to Layer 0 via the bootstrapping process. This manifest must be signed by the channel creator. Once published, the access key is used to verify future sync\_request proofs.

# **Bootstrap Channel Key Encoding**

The Bootstrap channel requires its own symmetric keys for encrypting all payloads at rest. These keys are distributed to authorized peers via the ALSP hello mechanism. This section defines how the array of Bootstrap keys is encoded for storage and transport. These are always encrypted using the recipient public identity key associated with the authenticated session.

## **Key Array Structure**

The Bootstrap channel maintains an **array of AES symmetric keys**, each represented as a JSON Web Key (JWK) object. These JWKs use the **same encoding format** already defined for channel AES keys in the channel\_key\_envelope specification. Each entry in the array represents one active or historical Bootstrap encryption key, enabling clients to decrypt messages from any epoch of the channel.

A Bootstrap key JWK has the following properties:

```json
{
    "kty": "oct",
    "k": "<base64url-encoded-key-bytes>",
    "alg": "A256GCM",
    "use": "enc",
    "kid": "<alsp:bck:index>"
}
```

Where:

- kty = “oct” (octet sequence)
- k = base64url encoding of the raw AES-256 key bytes
- alg = “A256GCM” (AES-256-GCM content encryption)
- use = “enc”
- kid = unique key identifier

### **Encapsulation in JWE**

The entire array of JWK objects is serialized as a JSON array and then encrypted using JSON Web Encryption (JWE) **compact serialization**. The JWE wrapping uses the same algorithms and key agreement patterns defined for channel key envelopes:

- **Key Agreement:**&#x45;CDH-ES+A256KW with the recipient’s public key
- **Content Encryption:**&#x41;256GCM

The result is a JWE compact string containing the encrypted key array as the ciphertext payload.

### **Example Structure**

```
BASE64URL(protected_header) "." BASE64URL(iv) "." BASE64URL(ciphertext) "." BASE64URL(tag)
```

Where ciphertext is the JSON array of JWK-formatted Bootstrap keys, encrypted under the agreed content encryption key.

### **Distribution and Rotation**

- **Distribution:** During ALSP hello, an authorized peer receives the JWE containing the Bootstrap key array. This enables the peer to decrypt historical and current Bootstrap channel messages.
- **Rotation:** New keys are appended to the array when a Bootstrap key rotation occurs. Older keys are retained to ensure historical decryptability, consistent with the immutable log.
- **Key Selection:** The kid field in each Bootstrap channel message’s outer JWE header indicates which key from the array is used for decryption.

By referencing the channel\_key\_envelope format for the AES key package and applying it to an array of Bootstrap keys, this approach maintains uniformity across the protocol while allowing the Bootstrap channel to support multi-epoch keying without special-case handling.

# **ALSP Key Identifier (kid) Resolution**

ALSP uses structured `kid` values in JWS headers to reference cryptographic material obtained through the bootstrap process. The `kid` format follows the same pattern as Layer 1 but resolves through different mechanisms appropriate to Layer 0's requirements.

**Format:**

```
ascp:<type>:<uuid>
```

**Supported Types at Layer 0:**

- `ascp:cert:<uuid>` - References an ASCP authentication certificate for message authentication
  - Used in JWS protected headers for peer authentication
  - Resolved from the bootstrap process certificate directory
  - Contains the public key corresponding to the `user_auth_cert` exchanged during hello
- `ascp:cak:<uuid>` - References a Channel Access Key for channel authorization
  - Used in JWS protected headers for channel access proofs
  - Resolved from the channel manifest obtained during bootstrap
  - Contains the Ed25519 public key for the specific channel

**Key Material Format:**

Unlike Layer 1 where keys are embedded in articulation statements, Layer 0 key material is distributed via the bootstrap process and stored in **JWK format** for consistency.

### EC Public Key JWK Format:

```json
{
  "kty": "EC",
  "crv": "P-256",
  "x": "<base64url X-coordinate>",
  "y": "<base64url Y-coordinate>",
  "alg": "ECDH-ES",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440003"
}

```

- `kty`: Key type (Elliptic Curve)
- `crv`: Curve (NIST P-256)
- `x`, `y`: Coordinates of the EC public key
- `alg`: Suggested algorithm (ECDH-ES)
- `kid`: Key identifier used for JWE or JWS headers and directory lookup

**Note:** The `use` field is intentionally omitted. This allows the same key to be used for both signature verification in JWS or key agreement in JWE.

This is compliant with RFC 7517 §4.2, which permits keys to omit the use field to avoid unnecessary restriction.

### Ed25519 Public Key JWK Format:

```json
{
  "kty": "OKP",
  "crv": "Ed25519",  
  "x": "<base64url-encoded-public-key>",
  "alg": "EdDSA",
  "kid": "ascp:cak:550e8400-e29b-41d4-a716-446655440002"
}
```

**Resolution Process:**

1. Parse the `kid` to extract type and UUID
2. Look up the key material in the appropriate bootstrap-provided directory
3. Extract the public key from the JWK representation
4. Use for JWS signature verification

This approach maintains consistency with Layer 1's key referencing model while operating within Layer 0's bootstrap-based key distribution mechanism.

## **Key Lifecycle and Resolution at Layer 0**

While Layer 1 manages all key lifecycle operations (rotation, expiration, revocation), Layer 0 has specific responsibilities for key resolution and caching:

**Static Key-ID Binding**: Each `kid` represents an immutable key. If a key needs to change, it must receive a new `kid`. This ensures that Layer 0 can trust that any given `kid` always resolves to the same cryptographic material.

**Session-Based Key Resolution**:

- Layer 0 implementations should resolve JWK keys from the bootstrap process/channel manifest at the time of use
- All cached keys should be purged at the start of each new ALSP session
- Keys should be resolved fresh from the bootstrap data when first encountered in a session

**Security-Based Cache Management**: For security best practices, any cached keys should be purged if they haven't been referenced for 10 minutes, ensuring that Layer 0 operates with fresh key material and reducing the window for potential key compromise.

**JWK Validation**: When resolving a key via `kid`, Layer 0 must verify that the `kid` field in the JWS header matches the `kid` field in the resolved JWK object to prevent key confusion attacks.

**Trust Model**: Layer 0 operates under a **log-anchored trust model** similar to Layer 1 - key material from the bootstrap process and channel manifests is trusted as it existed at the time those artifacts were created and signed. Layer 0 does not perform live PKI validation or revocation checking; instead, it trusts the cryptographic relationships established when the bootstrap document and channel manifests were originally articulated into the log. This ensures that historical sync operations remain verifiable using the credentials that were valid when those log entries were created, regardless of subsequent key lifecycle events handled at higher layers.

**Active session handling during CAK rotation:** Active sessions may continue with their session-cached keys until natural termination, at which point fresh sessions will resolve updated keys. If a peer wishes to force revalidation of a rotated CAK, it can send a 're-auth-required' error message. After sending this error, push syncs should cease and further sync\_requests must be refused for the remainder of this session. Note that in pull mode, each sync\_request is individually authorized against current credentials, while push mode relies on session-cached authorization from the initial subscription.

# Normative Implementation Guidance

## Message Deduplication

Upon receiving messages, replicas MUST deduplicate based on message\_id before inserting into local logs.

If a message with the same message\_id already exists in the local log, it MUST NOT be reinserted or affect the Lamport counter. However, the replica MUST still process any lamport\_max values from the sync message envelope for counter synchronization.

This ensures idempotent sync and prevents duplication during retries, reconnects, or when receiving the same message from multiple peers.

## Channel Message Scope Constraint

All messages included in a single sync\_response or sync\_update MUST belong to the same channel\_id as declared in the message envelope.

Messages from different channels MUST NOT be intermixed in the same ALSP sync message. Implementations MUST send separate sync messages for each channel, even when responding to multiple concurrent sync requests.

## Push Mode Lamport Monotonicity

In push mode, replicas MUST always transmit sync\_update messages when they have new locally generated (or received) messages that have advanced their local clock beyond the `lamport_max` value last transmitted or received from that specific peer in any previous ALSP message. This sync\_update must go out even when the receiving peer is not monitoring the associated channel(s).

This ensures:

- Proper advancement of the receiver's Lamport counter to the highest observed value.
- Prevents rollback or reordering anomalies
- Maintains causal consistency under concurrent sync conditions

## Lamport Counter Persistence

Replicas SHOULD persist their current Lamport counter value to stable storage periodically and SHOULD persist it during graceful shutdowns as well.

Upon restart, replicas MUST initialize their counter to the maximum of:

- The persisted counter value
- The highest Lamport value across all locally stored messages
- Any previously received lamport\_max value from peers (if preserved)

**Startup Sync Optimization**: To maximize wall-clock time ordering quality, replicas SHOULD attempt to sync with at least one peer and process any received `lamport_max` values before encoding new local articulations into the Layer 0 channel logs. This ensures that new messages receive Lamport values that are temporally consistent with current network activity rather than artificially low values that would cause them to appear out of chronological sequence in the global timeline.

This prevents counter regression and maintains the best possible wall clock sorted global ordering consistency across restarts.

## Message Ordering and Insertion

When inserting received messages into local logs, replicas MUST maintain canonical ordering using the tuple (lamport\_time, message\_id) regardless of the order in which messages were received or transmitted.

Messages MAY be received out of canonical order due to network conditions or batching optimizations. The `more: true` field in sync responses provides natural flow control, allowing senders to segment large message sets across multiple responses while respecting the `max_alsp_length` negotiated during hello. Implementations MUST sort and insert them in the correct positions within the local log structure.

Messages MAY be received out of canonical order due to network conditions or batching optimizations. Implementations MUST sort and insert them in the correct positions within the local log structure.

## Error Handling Requirements

Implementations MUST handle malformed or invalid messages gracefully without terminating the ALSP session unless explicitly required by an error message with `disconnect: true`.

When encountering invalid messages, replicas SHOULD log the error details and continue processing any remaining valid messages in the same sync batch.

Replicas MUST validate that all required fields are present and correctly typed before processing any sync message, and SHOULD send appropriate error responses for protocol violations.

**Recovery model:** All error recovery follows the same pattern as normal sync operations - establish connection, negotiate capabilities via hello, and request missing data via sync\_request. The protocol's idempotent design ensures that reconnection after any failure is equivalent to resuming sync after offline operation.

## Connection Retry and Backoff

When ALSP connections fail or are rejected, replicas SHOULD implement exponential backoff for successive connection attempts to prevent overwhelming servers during outages.

**Recommended backoff strategy:**

- Initial retry delay: 1 second
- Maximum retry delay: 300 seconds (5 minutes)
- Backoff multiplier: 2.0 with jitter

Replicas MAY implement more aggressive retry behavior for user-initiated sync operations, but automated background sync SHOULD respect these limits.

# Example ALSP Message Format (Illustrative)

This example shows a fully-formed sync\_response message using the standardized JSON structure. It demonstrates how multiple messages from a single channel are returned in a sync response.

**JWS Protected Header:**

```json
BASE64URL(UTF8({
  "alg": "ES256",
  "kid": "ascp:cert:550e8400-e29b-41d4-a716-446655440001",
  "typ": "alsp",
  "nonce": "peer_session_nonce_here"
}))
```

**JWS Payload (the complete ALSP message structure):**

```json
BASE64URL({
  "alsp_version": "0.1",
  "alsp_msg_header": {
    "alsp_msg_type": "sync_response",
    "timestamp": "2024-01-15T10:30:45.123Z",
    "channel_id": "550e8400-e29b-41d4-a716-446655440002",
    "lamport_max": 23456,
    "more": false
  },
  "alsp_msg_payload": [
    {
      "lamport_time": 23451,
      "message_id": "850e8400-e29b-41d4-a716-212554400020",
      "payload": "<DPB_encoded_bytes>"
    },
    {
      "lamport_time": 23452,
      "message_id": "50e8400-f29b-41d4-b716-446655448010",
      "payload": "<DPB_encoded_bytes>"
    }
  ]
})
```

**Final WebSocket Message:**

The complete JWS compact serialization (header.payload.signature) is sent over the WebSocket in DPB format.

This representation applies equally to sync\_update messages. The `alsp_msg_payload` field is only present for messages that carry articulation data. Note that payloads in the message array are treated as opaque binary by ALSP and interpreted only at Layer 1.

# **ALSP Session Lifecycle (Illustrative)**

The typical ALSP session progresses through the following phases:

```asciidoc
Client                       Server
  |                            |
  | -- auth_request ---------> |
  |                            |
  | <------ auth_challenge --- |
  |                            |
  | -- hello (nonce=S) ------> |
  |                            |
  | <------ hello (nonce=C) -- |
  |                            |
  | --------- sync_request --> |
  |                            |
  | <--- sync_response ------  |
  |                            |
  | -- sync_update --------->  | (if using push model)
  |                            |
  | <--- sync_update --------  | (peer pushes back updates)
  |                            |
  | -- error ----------------> |  (optional)
  | <---------- error -------- |  (if needed)
  |                            |
  |       connection close     |
```

### **Notes**

- auth\_request is always required and initiated by connecting party
- auth\_challenge only happens if/as needed
- hello is always required and a symmetric to complete mutual authentication
- sync\_request initiates pull of sync information.
- sync\_update is optional and used for live push.
- error may be sent at any phase to signal a problem.
- Sessions may close cleanly or due to protocol violations.

# **Channel Log Health Check and Recovery**

While ALSP ensures deterministic, convergent log synchronization under normal operation, real-world systems may encounter channel divergence due to implementation bugs, storage faults, or partial sync failures. This section defines normative and recommended practices for detecting and recovering from such divergence scenarios.

## **Health Check Mechanisms**

### **1. Digest Hash Exchange (Recommended)**

Implementations MAY periodically compute a deterministic hash of the message log for each channel based on the canonical ordering of message IDs. This digest can be validated with any peer via any sync\_request message. This is done via the optional log\_digest parameter of the sync\_request message.

- Hash inputs MUST include all logged message\_id values in canonical order.
- Hashing only the Layer 0 message\_id metadata (not payloads) is sufficient for detection.
- A mismatch signals possible divergence or incomplete sync.
- One should only perform this check when one presumed that the peer has the specified range of messages available to it therefore this is best done once one believes the log is already up to date.

### **2. Sync Auditing (Future Extension)**

During sync\_request or sync\_response, replicas MAY include a summary of:

- Number of log entries
- First and last lamport\_times in the log.

Peers can use this to identify range inconsistencies or unexpected log truncation.

### **3. Merkle Tree Snapshots (Future Extension)**

A Merkle tree structure per channel log MAY be implemented to support efficient range comparison, similar to Git or Hypercore. This enables detection of localized divergence with minimal hash exchange.

## **Divergence Recovery**

If divergence is detected, replicas MAY perform the following recovery options:

### **1. Targeted Rehydration**

- Re-request a specific Lamport range via sync\_request with from\_lamport and to\_lamport set.

### **2. Full Re-Sync**

- Issue a sync\_request from from\_lamport = 0 to rehydrate the complete channel log.
- Validate canonical ordering and deduplicate on receipt.

### **3. Replica Rollback (Manual or Audited)**

- A faulty replica MAY delete and rebuild the affected channel log from scratch.
- Operators SHOULD log the rollback event and verify post-recovery state with digest checks.

### **Prevention Guidelines**

- Always persist Lamport counters and last-known lamport\_max
- Perform message insertion atomically and in canonical order
- Never insert messages into a log without full canonical sort metadata

While divergence is expected to be rare, these tools ensure that detection and repair can be handled gracefully without compromising global convergence.

# **Future Extensions (non-normative)**

- Merkle tree summary per log for integrity verification
- Log compression and chunking
- Gossip-based peer-to-peer sync overlay
- Snapshotting and checkpointing

