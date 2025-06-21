# EchoNet DDS - Security Considerations and Error Codes

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 1. Introduction

This document outlines critical security considerations for the EchoNet Distributed Data Stores (DDS) and provides a consolidated list of DDS-specific error codes. Ensuring the security and integrity of the DDS is paramount for user trust and the overall reliability of the EchoNet platform. Error codes are aligned with the `echonet_v3_error_handling_strategy.md`.

## 2. DDS-Specific Error Codes

DDS operations utilize the `DDSProtocolError` message structure defined in `echonet_v3_core.proto` for returning error information. The symbolic error code names (which would map to specific integer codes in an implementation) are listed below.

*   **`DDS_ERROR_UNSPECIFIED`**
    *   Message Template: "An unspecified DDS error occurred. Details: {details}"
    *   Severity: `FatalSystemError` (typically)
    *   Context: `{"details": "string"}`
*   **`DDS_CHUNK_NOT_FOUND`**
    *   Message Template: "Requested chunk with ID '{chunk_id}' was not found on this node or any queried peers."
    *   Severity: `RecoverableError` (client may try other peers or discover)
    *   Context: `{"chunk_id": "string"}`
*   **`DDS_STORAGE_FULL`**
    *   Message Template: "Node '{node_did}' cannot store chunk '{chunk_id}' due to insufficient available storage capacity."
    *   Severity: `RecoverableError` (try other storage nodes)
    *   Context: `{"node_did": "string", "chunk_id": "string", "required_bytes": "uint64"}`
*   **`DDS_INVALID_CHUNK_HASH`**
    *   Message Template: "Provided chunk data for ID '{chunk_id}' does not match the expected hash. Data integrity violated."
    *   Severity: `FatalUserError` (if client-provided) / `Warning` (if from peer, leading to rejection)
    *   Context: `{"chunk_id": "string", "expected_hash": "string", "provided_hash": "string"}`
*   **`DDS_INVALID_BYTE_RANGE`**
    *   Message Template: "Retrieve request for chunk '{chunk_id}' specified an invalid or unsatisfiable byte range: {byte_range}."
    *   Severity: `FatalUserError`
    *   Context: `{"chunk_id": "string", "byte_range": "string"}`
*   **`DDS_AUTHORIZATION_FAILED`**
    *   Message Template: "Node '{accessor_did}' is not authorized to perform operation '{operation}' on chunk/resource '{resource_id}'."
    *   Severity: `FatalUserError`
    *   Context: `{"accessor_did": "string", "operation": "string", "resource_id": "string"}`
*   **`DDS_NODE_UNAVAILABLE`**
    *   Message Template: "Target DDS node '{target_node_did}' is currently unavailable or unresponsive."
    *   Severity: `RecoverableError`
    *   Context: `{"target_node_did": "string"}`
*   **`DDS_REPLICATION_FAILED`**
    *   Message Template: "Failed to replicate chunk '{chunk_id}' to the required number of peers or meet redundancy criteria."
    *   Severity: `RecoverableError` (self-healing mechanisms should address)
    *   Context: `{"chunk_id": "string", "target_replicas": "uint32", "achieved_replicas": "uint32"}`
*   **`DDS_REPAIR_FAILED`**
    *   Message Template: "Chunk repair process failed for original chunk ID '{original_chunk_id}'. Reason: {reason}"
    *   Severity: `RecoverableError` (self-healing may retry)
    *   Context: `{"original_chunk_id": "string", "reason": "string"}`
*   **`DDS_INVALID_POSR_CHALLENGE`**
    *   Message Template: "The PoSR challenge for chunk '{chunk_id}' received by node '{node_did}' is malformed or invalid. Details: {details}"
    *   Severity: `Warning` (for the challenged node, may indicate an issue with the challenger)
    *   Context: `{"node_did": "string", "chunk_id": "string", "challenge_id": "string", "details": "string"}`
*   **`DDS_INVALID_POSR_PROOF`**
    *   Message Template: "The PoSR proof provided by node '{node_did}' for chunk '{chunk_id}' (challenge '{challenge_id}') is invalid or failed verification."
    *   Severity: `Warning` (for the network, may trigger repair and penalize the node)
    *   Context: `{"node_did": "string", "chunk_id": "string", "challenge_id": "string"}`
*   **`DDS_TIMEOUT`**
    *   Message Template: "DDS operation '{operation}' timed out after {duration_ms} ms."
    *   Severity: `RecoverableError`
    *   Context: `{"operation": "string", "duration_ms": "uint64"}`
*   **`DDS_PROTOCOL_VIOLATION`**
    *   Message Template: "Peer '{peer_node_did}' violated DDS protocol during operation '{operation}'. Details: {details}"
    *   Severity: `Warning` (may lead to disconnecting/penalizing the peer)
    *   Context: `{"peer_node_did": "string", "operation": "string", "details": "string"}`
*   **`DDS_ERASURE_CODING_ERROR`**
    *   Message Template: "Erasure coding operation '{ec_operation}' failed for chunk '{original_chunk_id}'. Reason: {reason}"
    *   Severity: `RecoverableError` (if decoding, try other fragments) / `FatalSystemError` (if encoding consistently fails)
    *   Context: `{"ec_operation": "encode/decode", "original_chunk_id": "string", "reason": "string"}`
*   **`DDS_FRAGMENT_NOT_FOUND`**
    *   Message Template: "Requested erasure fragment index {fragment_index} for original chunk ID '{original_chunk_id}' was not found."
    *   Severity: `RecoverableError` (try other peers or discover)
    *   Context: `{"original_chunk_id": "string", "fragment_index": "uint32"}`
*   **`DDS_INSUFFICIENT_FRAGMENTS`**
    *   Message Template: "Insufficient unique fragments available/retrieved to reconstruct original chunk ID '{original_chunk_id}'. Required: {k_value}, Available: {retrieved_count}."
    *   Severity: `RecoverableError` (self-healing or client needs to find more fragments)
    *   Context: `{"original_chunk_id": "string", "k_value": "uint32", "retrieved_count": "uint32"}`

## 3. DDS Security Considerations

### 3.1. Node Authentication for DDS Operations
*   **Mechanism:** All DDS nodes (SSNs, ASNs, ECNs) and clients interacting with DDS services MUST possess an EchoNet Decentralized Identifier (`did:echonet`).
*   **RPC Authentication:** As outlined in `echonet_v3_communication_protocols.md`, inter-node RPC calls (including all `DDSNodeService` methods) are authenticated. This is achieved by:
    *   **Secure Channels:** Establishing secure communication channels (e.g., mTLS over libp2p streams) where peers authenticate each other using cryptographic keys linked to their DIDs.
    *   **Signed Requests (Conceptual):** While channel security provides significant protection, critical operations or those requiring non-repudiation at the application layer can embed signatures within the request messages themselves (e.g., `NodeAnnouncementInfo.signature`, `StorageChallengeResponse.node_signature`). The `origin_signature` in `StoreChunkRequest` is an example for authenticating the original creator.
*   **Witness Authentication:** Witnesses challenging SSNs for PoSR must also authenticate using their DIDs.

### 3.2. Data Integrity During Transfer (P2P Exchange)
*   **Hash Verification:** Upon receiving a `DDSChunk` (e.g., via `RequestChunkForReplicationResponse`), the recipient node **MUST** calculate the SHA-256 hash of the `chunk.data` and verify it against the provided `chunk.chunk_id`. If there's a mismatch, the data is discarded, as detailed in `tech_specs/dds_p2p_exchange_protocol.md`.
*   **Erasure Fragment Verification:** For `ErasureFragment`s, the integrity of the reconstructed data from `k` fragments is verified against the `original_chunk_hash` included in each fragment message.
*   **Secure Transport:** The use of TLS or equivalent secure channel protocols (e.g., Noise via libp2p) protects data against tampering and eavesdropping during P2P transfer between nodes.

### 3.3. Proof-of-Storage/Retrievability (PoSR) Integrity
*   **Challenge Generation:**
    *   Challenges (`StorageChallengeRequest.challenge_payload`) issued by Witnesses (or automated challengers) must include unpredictable nonces or select random parts of the data to audit. This prevents SSNs from pre-calculating proofs for data they no longer store.
    *   The `challenge_id` must be unique to prevent replay attacks of old proofs.
*   **Proof Submission (`StorageChallengeResponse`):**
    *   The `proof_payload` must be a cryptographic proof (e.g., Merkle proof, hash of challenged data ranges) that demonstrably proves possession of the specific `chunk_id`.
    *   The entire response, including `challenge_id`, `chunk_id`, and `proof_payload`, MUST be signed by the `node_did` of the SSN, using `node_signature`.
*   **Proof Validation:** Witnesses validating PoSR proofs must:
    *   Verify the `node_signature`.
    *   Verify the `proof_payload` against the original `challenge_payload` and the known hash of the `chunk_id`.
    *   Ensure the `challenge_id` is recent and not replayed.
*   **Preventing Fake Storage:**
    *   Regular and unpredictable PoSR challenges are key.
    *   Proof-of-Replication (PoRep) schemes, if implemented, can further ensure that an SSN is storing a unique physical copy and not just simulating storage through access to other copies. (This is an advanced topic beyond basic PoSR).
    *   Slashing conditions for failed PoSR (as per `dds_refined_architecture.md`) disincentivize cheating.

### 3.4. Secure Interaction with Discovery System (DHT)
*   **Authenticated Announcements (`NodeAnnouncementInfo`):**
    *   When an SSN announces the availability of chunks/fragments it holds using `NodeAnnouncementInfo` (via `AnnounceAvailabilityRequest`), this announcement is signed by the SSN's `node_did`.
    *   Peers receiving this information (e.g., DHT nodes storing the announcement, or clients resolving it) can verify the signature to ensure the announcement's authenticity and integrity, preventing spoofed availability records.
*   **DHT Security (General):** While the full security of the Kademlia-based DHT is a broader topic (covered in `discovery_protocol_refined.md`), measures such as requiring stake or PoW for DHT participation, and robust peer ID verification, help protect against Sybil attacks and pollution of routing tables, thus improving the reliability of discovering legitimate SSN addresses.
*   **Client Verification:** Clients resolving chunk locations via the DHT should ideally verify the `NodeAnnouncementInfo.signature` of the returned SSN records.

### 3.5. Access Control for DDS Operations
*   **Storing Data (`StoreChunk` RPC):**
    *   **Content Validation Pre-requisite:** Generally, content chunks are stored on DDS *after* the corresponding `NexusContentObjectV1` has been validated by EchoNet Witnesses. This prevents the DDS from being filled with spam or illicit content that hasn't passed initial DLI validation.
    *   **Authorization (Optional):** SSNs might implement policies to only accept `StoreChunk` requests if they include a valid `origin_signature` (from the content creator) or if the `parent_content_id` points to a Witness-validated `NexusContentObjectV1`.
*   **Retrieving Data (`RetrieveChunk` RPC):**
    *   **Public Content:** For most content on EchoNet, chunks are publicly retrievable by any node or client that knows their `chunk_id`.
    *   **Future Private/Encrypted Content:** If EchoNet supports encrypted or access-restricted content, the DDS itself would store opaque, encrypted blobs. Access control (decryption keys, permissions) would be managed at a higher application layer or via client-side mechanisms (e.g., sharing decryption keys out-of-band). The DDS would not be responsible for enforcing access control on the encrypted blobs themselves, only for serving them if requested by `chunk_id`.

### 3.6. Protection Against DDS-Specific Attacks
*   **Storage Exhaustion Attacks:**
    *   **Mitigation:** SSNs manage their own storage. They can refuse `StoreChunk` requests if they are full (`DDS_STORAGE_FULL`).
    *   **Content Validation:** The requirement for content to be validated by Witnesses before widespread DDS replication limits the ability of attackers to trivially fill the network with garbage.
    *   **Incentives & Staking:** SSNs are incentivized to store valid, demanded content. Staking requirements can make it costly for an SSN to maliciously fill its own space with useless data if that prevents it from earning rewards for legitimate storage.
*   **Bandwidth Exhaustion Attacks (Excessive Retrieval):**
    *   **Mitigation:** SSNs should implement rate limiting on `RetrieveChunk` and `RequestChunkForReplication` requests per source IP/DID.
    *   **Retrieval Fees (Conceptual):** As mentioned in `dds_refined_architecture.md`, small retrieval fees can disincentivize bulk, low-value retrievals.
    *   **ECNs:** Ephemeral Cache Nodes can absorb load for popular content.
*   **Data Pollution/Corruption Attacks:**
    *   **Mitigation:** Content validation by Witnesses before storage. Hash verification of all chunks/fragments upon receipt during P2P exchange. PoSR challenges detect silent corruption or data loss on SSNs.
*   **Eclipse Attacks on SSNs:**
    *   **Mitigation:** SSNs should connect to a diverse set of peers in the DHT and for gossip, reducing the chance of being isolated by malicious neighbors. This is a general P2P network security concern.

By implementing these security measures and robust error handling, the EchoNet DDS aims to provide a resilient, trustworthy, and secure decentralized storage layer. Continuous monitoring and adaptation will be necessary to address evolving threats.
