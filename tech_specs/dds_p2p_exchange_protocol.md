# EchoNet DDS - Peer-to-Peer (P2P) Chunk Exchange Protocol

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 1. P2P Chunk Exchange Overview

### 1.1. Purpose
This document details the peer-to-peer (P2P) protocol used by EchoNet Distributed Data Store (DDS) nodes, primarily Standard Storage Nodes (SSNs), to discover and exchange individual data chunks or erasure-coded fragments. The primary purposes of this P2P exchange are:

*   **Data Replication:** To allow SSNs to proactively create copies of data chunks to meet target redundancy levels, ensuring data availability and durability. This includes initial replication after content submission and ongoing efforts to maintain redundancy.
*   **Data Repair (Self-Healing):** To enable SSNs to repair missing or corrupted data by fetching valid copies (chunks or fragments) from peer SSNs. This is critical for the self-healing capabilities of the DDS, triggered by events like Proof-of-Storage-Receipt (PoSR) failures or node loss.

### 1.2. Key Scenarios
The P2P chunk exchange protocol is utilized in several key scenarios:
*   **Initial Replication:** When a new content chunk is first introduced to the DDS (typically by an Origin Node storing it on a set of SSNs), those SSNs may initiate further P2P exchanges to meet the initial target replication factor.
*   **Proactive Redundancy Management:** If the number of active replicas/fragments for a chunk falls below a desired threshold (but above a critical repair threshold), SSNs holding the data may proactively seek out other SSNs to store additional copies.
*   **Reactive Repair:** When an SSN fails a PoSR challenge for a chunk, or if an SSN holding a unique replica/fragment goes offline permanently, other SSNs (or a designated repair process) will use this P2P protocol to fetch necessary data from remaining peers to reconstruct and re-replicate the lost data.

## 2. Peer & Chunk Discovery for Exchange

### 2.1. Mechanism
When an SSN (Node A, the "requestor") needs a specific chunk (`ChunkID_X`) or an erasure-coded fragment, it follows these steps:

1.  **Query Discovery Protocol:** Node A queries the EchoNet Discovery Protocol (which utilizes a Kademlia-based DHT, as per `discovery_protocol_refined.md`) for `ChunkID_X` (or `OriginalChunkID` for fragments).
2.  **Receive Peer List:** The Discovery Protocol returns a list of `NodeAnnouncementInfo` (or similar pointers) for peer SSNs (Node B, Node C, etc.) that have announced they hold `ChunkID_X` or fragments of `OriginalChunkID`. These announcements include network addresses and other relevant metadata.

### 2.2. Peer Selection Criteria
Node A selects one or more suitable peers from the list based on criteria such as:
*   **Network Proximity:** Lower latency peers might be preferred (if measurable).
*   **Geographic Diversity:** For placing new replicas, selecting peers in different geographic regions than existing holders is preferred.
*   **Peer Reputation:** Peers with higher reputation scores (indicating reliability and good behavior) are prioritized.
*   **Peer Load & Capacity:** Peers with lower current load and sufficient available storage capacity may be favored.
*   **Node Operator Diversity:** Preferring nodes run by different operators, if this information is available or inferable.

## 3. Chunk Request/Response Protocol

The P2P exchange of chunks and fragments leverages RPC methods defined within the `DDSNodeService` (see `proto/dds_specific.proto` and documented in `tech_specs/dds_protocol_structures.md`). These RPCs are expected to operate over secure libp2p streams, as outlined conceptually in `echonet_v3_communication_protocols.md`.

### 3.1. Requesting a Full Chunk (for Replication/Repair)
*   **Request Message:** `echonet.dds.v3.RequestChunkForReplicationRequest`
    *   `chunk_id` (`string`): The ID of the `DDSChunk` being requested.
    *   `requester_node_did` (`string`): The DID of Node A making the request. This allows Node B to authenticate the requestor.
    *   `nonce` (`string`, optional): A unique string to prevent replay attacks, ensuring this specific request is fresh.
*   **Signature:** The request should be implicitly signed by the secure channel (e.g., mTLS via libp2p) or an explicit signature can be added if required by the communication protocol layer, using the `requester_node_did`.

### 3.2. Responding with a Full Chunk
*   **Response Message:** `echonet.dds.v3.RequestChunkForReplicationResponse`
    *   `chunk` (`echonet_core.v3.DDSChunk`): The requested `DDSChunk` message, including its `chunk_id` and `data`. This is sent if the request is successful and the chunk is found.
    *   `success` (`bool`): `true` if the chunk is provided, `false` otherwise.
    *   `error` (`echonet_core.v3.DDSProtocolError`): If `success` is `false`, this field provides details about the error (e.g., chunk not found, permission denied).
*   **Signature:** Similar to requests, responses are secured by the channel. Explicit signing by the responder (Node B) could be added if deemed necessary beyond channel security.

### 3.3. Interaction Pattern
This is a standard RPC request/response pattern. Node A sends the `RequestChunkForReplicationRequest` to Node B. Node B processes the request and replies with `RequestChunkForReplicationResponse`.

## 4. Data Transfer Mechanism

*   **Streaming:** For the `data` field within the `echonet_core.v3.DDSChunk` message (which can be up to 1MB or more), the underlying gRPC framework (if used, as suggested in `echonet_v3_communication_protocols.md`) will handle streaming of the message or its parts over libp2p streams. This prevents loading entire chunks into memory at once on either node if the RPC framework handles it efficiently.
*   **Integrity Checks during Transfer:**
    *   The primary integrity check is performed upon full receipt of the `DDSChunk` data by verifying its hash against the `chunk_id`.
    *   The secure channel (e.g., TLS over libp2p) provides transport-layer integrity against corruption during transit.
*   **Connection Handling:** Connections are established and managed using libp2p. The `requester_node_did` is used to identify the target peer, whose network address is obtained from the Discovery Protocol.

## 5. Verification upon Receipt

Upon receiving the `RequestChunkForReplicationResponse` containing the `DDSChunk`:
1.  The requesting node (Node A) **MUST** verify that the hash of the received `chunk.data` matches the `chunk.chunk_id` (which should also match the initially requested `chunk_id`).
2.  If verification fails:
    *   The received chunk is discarded.
    *   The providing peer (Node B) may be flagged (e.g., a temporary local penalty).
    *   Persistent failures from Node B could lead to a negative impact on its reputation score within the broader EchoNet system.
3.  If verification succeeds, Node A stores the chunk and, if appropriate (e.g., if it's a new replica), announces its availability via the Discovery Protocol (using `AnnounceAvailabilityRequest`).

## 6. Erasure-Coded Fragment Exchange

The protocol for exchanging erasure-coded fragments is analogous to full chunk exchange:

*   **Request Message:** `echonet.dds.v3.RequestErasureFragmentRequest`
    *   `original_chunk_id` (`string`): The `chunk_id` of the original `DDSChunk` from which the fragment was derived.
    *   `fragment_index` (`uint32`): The specific index of the fragment being requested.
*   **Response Message:** `echonet.dds.v3.RequestErasureFragmentResponse`
    *   `fragment` (`echonet_core.v3.ErasureFragment`): The requested `ErasureFragment` message, including its data and parameters (`original_chunk_id`, `fragment_index`, `k_value`, `m_value`, `original_chunk_hash`).
    *   `success` (`bool`): `true` if the fragment is provided.
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if not successful.

**Verification upon Receipt (Fragment):**
*   The receiving node verifies the integrity of the fragment if possible (e.g., if the fragment itself has an internal checksum, though this is not standard in the current `ErasureFragment` definition).
*   The primary verification for erasure-coded data occurs when `k` fragments are collected and the original `DDSChunk` is reconstructed. At that point, the `original_chunk_hash` (stored within each `ErasureFragment`) is compared against the hash of the reconstructed data.

## 7. Flow Control & Error Handling

*   **Flow Control:**
    *   **Concurrent Requests:** SSNs should be capable of handling multiple incoming and outgoing chunk/fragment exchange requests concurrently.
    *   **Queuing:** Incoming requests may be queued if a node is at its concurrent processing limit.
    *   **Rate Limiting:** Nodes should implement rate limiting on incoming requests (per peer DID and globally) to prevent abuse and ensure fair resource allocation.
*   **Error Handling:**
    *   The `error` field in response messages (of type `echonet_core.v3.DDSProtocolError`) is used to communicate failures. Common error scenarios during P2P exchange and their potential mapping to `DDSProtocolError.code` (defined in `tech_specs/dds_protocol.md` and included in `echonet_v3_core.proto`):
        *   Peer does not have the requested chunk/fragment: `DDS_CHUNK_NOT_FOUND` or `DDS_FRAGMENT_NOT_FOUND`.
        *   Requestor is not authorized (if specific policies apply): `DDS_AUTHORIZATION_FAILED`.
        *   Peer experiences an internal error serving the request: `DDS_ERROR_UNSPECIFIED` or a more specific internal code.
        *   Network timeout during transfer: `DDS_TIMEOUT`.
    *   If a hash mismatch occurs after data transfer (as per Section 5), this is handled by the requestor; it's not an error *within the response message* itself but a post-transfer validation failure.

## 8. Security Considerations

*   **Peer Authentication:**
    *   The `requester_node_did` in requests allows the responding node to identify the caller.
    *   Communication channels are secured using libp2p's mechanisms (e.g., TLS, Noise), providing channel encryption and peer authentication based on libp2p peer IDs, which should be cryptographically linked to `node_did`s.
*   **Data Integrity:**
    *   The primary defense against malicious peers providing bad data is the cryptographic hash verification of `chunk.data` against `chunk.chunk_id` upon receipt.
    *   For erasure-coded fragments, integrity is verified upon reconstruction using the `original_chunk_hash`.
*   **Denial-of-Service (DoS) Protection:**
    *   Rate limiting on incoming requests.
    *   Reputation penalties for peers making excessive invalid requests or failing to provide valid data they claim to hold.
    *   Connection management to prevent resource exhaustion from too many open streams.
*   **Authorization (Optional Refinement):** While not explicitly detailed for P2P replication requests, a responding node could have policies to prioritize or restrict serving chunks to certain peers based on network status, trust, or incentive alignment. For MVP, availability to any network peer for replication/repair is assumed.

## 9. Protocol Versioning

This P2P Chunk Exchange Protocol, including the structure of the RPC messages (`RequestChunkForReplicationRequest/Response`, `RequestErasureFragmentRequest/Response`) and the behavior of the `DDSNodeService` methods involved, will adhere to the versioning strategy outlined in `echonet_v3_versioning_strategy.md`. Any changes to message formats or RPC semantics will be managed through this versioning process, coordinated with overall EchoNet protocol upgrades.

This protocol ensures that DDS nodes can efficiently and reliably exchange data, forming the backbone of EchoNet's data replication and self-healing capabilities.
