# DigiSocialBlock DDS - Network Protocol Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. DDS Protocol Overview

### 1.1. Purpose and Scope
This document specifies the network protocol for the Distributed Data Stores (DDS) within the DigiSocialBlock DLI system (also referred to as `EchoNet`). The DDS protocol governs how content chunks are stored, retrieved, replicated, proven to be stored (Proof-of-Storage-Receipts - PoSR), and discovered across a decentralized network of storage nodes.

This protocol is essential for the persistence, availability, and integrity of content published to the EchoNet platform.

### 1.2. Key Capabilities
*   **Decentralized Storage:** Content data (chunks) is stored across numerous independent storage nodes.
*   **Content-Addressing:** Chunks are identified and retrieved using their cryptographic hashes (e.g., `ChunkID`, often a component of a `ContentID` from `tech_specs/dli_core_data_structures.md`).
*   **Replication:** The protocol facilitates the creation and maintenance of multiple copies of data chunks across different nodes for redundancy and availability.
*   **Erasure Coding:** Supports storing data as erasure-coded fragments to enhance durability with storage efficiency.
*   **Self-Healing:** Mechanisms to detect lost or corrupted replicas/fragments and initiate repair processes.
*   **Proof-of-Storage (PoSR):** Nodes must periodically prove they are still storing the data they claim to hold.

### 1.3. Types of Nodes Involved (as per `dds_refined_architecture.md`)
*   **Standard Storage Nodes (SSNs):** The primary workhorses of the DDS. They store content chunks, participate in replication, respond to retrieval requests, and engage in PoSR challenges.
*   **Archival Storage Nodes (ASNs):** Specialized SSNs focused on long-term, durable storage, potentially with different incentive structures. They follow the same core DDS protocol but may have higher PoSR requirements.
*   **Ephemeral Cache Nodes (ECNs):** Nodes that temporarily cache popular content for faster retrieval. They might use a subset of the DDS protocol (primarily `RetrieveChunk` and discovery).
*   **Origin Nodes (Content Creator's Client):** The initial source of content chunks. They use the `StoreChunk` RPC to introduce content to the DDS network.
*   **Clients (Content Consumer's Client):** Retrieve content chunks from DDS nodes using `RetrieveChunk`.
*   **Witnesses:** While not directly storing content for general access, Witnesses may interact with the DDS protocol to:
    *   Verify the initial availability of content chunks during content publication validation.
    *   Challenge SSNs/ASNs as part of the PoSR mechanism.
    *   Potentially retrieve chunks for content moderation or validation purposes.
*   **Discovery System Nodes (DHT Nodes):** Nodes participating in the Discovery Protocol (e.g., Kademlia DHT as per `discovery_protocol_refined.md`) that help map `ChunkID`s to SSNs holding them.

## 2. Core Data Structures specific to DDS Protocol

These structures are defined using Protocol Buffers v3 syntax. They complement structures found in `tech_specs/dli_core_data_structures.md`.

```protobuf
syntax = "proto3";

package digisocialblock.dds.protocol;

import "google/protobuf/timestamp.proto";
// Assuming NexusContentObjectV1 and other core types are accessible via import
// e.g., import "digisocialblock/dli/core/nexus_content_object.proto";
// For simplicity here, we'll redefine simple IDs if needed, but in practice, use imports.

// Represents a single chunk of data to be stored or retrieved.
message DDSChunk {
  // Identifier of the chunk, typically a cryptographic hash of its content.
  // Validation: Required, must be a valid hash string (e.g., SHA2-256 hex).
  string chunk_id = 1;

  // The actual binary data of the chunk.
  // Validation: Required. Size limits may apply (e.g., max 1MB or 4MB per chunk).
  bytes data = 2;

  // Optional: Index of this chunk if it's part of a sequence for a larger content object.
  uint32 sequence_index = 3;

  // Optional: Total number of chunks for the parent content object, if known.
  uint32 total_chunks = 4;
}

// Represents an erasure-coded fragment of a DDSChunk.
message ErasureFragment {
  // Identifier of the original chunk this fragment belongs to.
  string original_chunk_id = 1;

  // Index of this fragment (0 to m-1, where m is the total number of fragments).
  uint32 fragment_index = 2;

  // The actual binary data of the fragment.
  bytes data = 3;

  // Erasure coding parameters used (k out of m).
  uint32 k_value = 4; // Number of data fragments
  uint32 m_value = 5; // Total number of fragments (data + parity)

  // Hash of the original chunk, to verify reconstruction.
  bytes original_chunk_hash = 6;
}

// Request for a Proof-of-Storage-Receipt (PoSR) challenge.
message StorageChallengeRequest {
  // Unique ID for this challenge instance.
  string challenge_id = 1;

  // Identifier of the chunk being challenged.
  string chunk_id = 2;

  // Nonce or other challenge data specific to the PoSR scheme.
  // e.g., specific byte ranges to hash, or seed for a Merkle proof.
  bytes challenge_payload = 3;

  // Timestamp by when the response is expected.
  google.protobuf.Timestamp deadline = 4;
}

// Response to a PoSR challenge.
message StorageChallengeResponse {
  string challenge_id = 1;
  string chunk_id = 2;

  // The generated proof data (e.g., Merkle proof, hash of challenged ranges).
  bytes proof_payload = 3;

  // DID of the SSN providing the proof.
  string node_did = 4;

  // Signature of (challenge_id + chunk_id + proof_payload) by the node_did.
  bytes node_signature = 5;
}

// Instruction for a node to replicate/repair a chunk.
message ReplicationInstruction {
  // Identifier of the chunk to be replicated or repaired.
  string chunk_id = 1;

  // List of known peer DIDs currently holding this chunk (optional).
  // If empty, the recipient node should use the Discovery system.
  repeated string source_peer_dids = 2;

  // Target replication factor (if applicable).
  uint32 target_replication_factor = 3;

  // Reason for instruction (e.g., "INITIAL_REPLICATION", "REPAIR_LOST_REPLICA").
  string reason = 4;
}

// Information about a storage node's capabilities and stored content.
message NodeAnnouncementInfo {
  // DID of the announcing node.
  string node_did = 1;

  // Network address of the node (e.g., multiaddr format).
  string address = 2;

  // Type of storage node (e.g., "SSN_GENERIC", "ASN_HIGH_DURABILITY").
  string storage_type = 3;

  // Geographic region hint (optional).
  string region_hint = 4;

  // Total storage capacity in bytes (optional).
  uint64 total_capacity_bytes = 5;

  // Available storage capacity in bytes (optional).
  uint64 available_capacity_bytes = 6;

  // List of chunk_ids this node is announcing it holds.
  // For bulk announcements, this can be paginated or use probabilistic structures like Bloom filters.
  repeated string held_chunk_ids = 7;
}

```

## 3. RPC Service Definition for DDS Nodes (`DDSNodeService`)

This service defines the RPC calls supported by DDS storage nodes (SSNs, ASNs). Defined using protobuf3 service syntax.

```protobuf
syntax = "proto3";

package digisocialblock.dds.protocol;

// Assuming DDSChunk, StorageChallengeRequest, etc. are defined in this file or imported.

service DDSNodeService {
  // Client/Origin Node requests an SSN to store a chunk.
  // Authorization: May require proof of ContentObject validation by Witnesses, or be signed by creator DID.
  rpc StoreChunk(StoreChunkRequest) returns (StoreChunkResponse);

  // Client/ECN/SSN requests to retrieve a chunk from an SSN/ASN.
  rpc RetrieveChunk(RetrieveChunkRequest) returns (RetrieveChunkResponse);

  // SSN/ASN announces to the Discovery Protocol (e.g., DHT) that it holds certain chunks.
  // This is typically not a direct peer-to-peer RPC but an interaction with the discovery system.
  // For conceptual clarity, it's listed here; actual mechanism depends on DHT specifics.
  rpc AnnounceAvailability(AnnounceAvailabilityRequest) returns (AnnounceAvailabilityResponse);

  // Witness or auditing node challenges an SSN/ASN to prove storage.
  rpc HandlePoSRChallenge(StorageChallengeRequest) returns (StorageChallengeResponse);

  // SSN requests a chunk from another SSN for replication or repair.
  rpc RequestChunkForReplication(RequestChunkForReplicationRequest) returns (RequestChunkForReplicationResponse);

  // Client or SSN requests an erasure-coded fragment from an SSN.
  rpc RequestErasureFragment(RequestErasureFragmentRequest) returns (RequestErasureFragmentResponse);

  // SSN requests to reconstruct a full chunk from erasure-coded fragments it has collected.
  // This might be a local operation or involve a helper service if fragments are from diverse sources.
  // For now, assume local reconstruction if an SSN has enough fragments.
  // If it needs to fetch more fragments, it uses RequestErasureFragment.
}

// ---- StoreChunk RPC ----
message StoreChunkRequest {
  DDSChunk chunk = 1; // Contains chunk_id and data.

  // Optional: Desired initial replication factor. The network will aim for this.
  uint32 desired_initial_replication = 2;

  // Optional: Signature of chunk_id by the original content creator's DID.
  // Helps SSNs verify authenticity and prioritize storage.
  bytes origin_signature = 3;

  // Optional: ContentID of the parent object this chunk belongs to.
  string parent_content_id = 4;
}

message StoreChunkResponse {
  // Unique ID for this storage transaction on this node.
  string storage_receipt_id = 1;

  // True if successfully stored locally (replication initiated separately).
  bool success = 2;

  // Error details if success is false.
  DDSProtocolError error = 3;
}

// ---- RetrieveChunk RPC ----
message RetrieveChunkRequest {
  string chunk_id = 1;

  // Optional: Specify a byte range (e.g., "bytes=0-1023").
  // If not provided, the whole chunk is returned.
  string byte_range = 2; // RFC 7233 format
}

message RetrieveChunkResponse {
  DDSChunk chunk = 1; // Contains chunk_id and requested data (full or partial).

  // True if data is present and retrieved.
  bool success = 2;

  // Error details if success is false.
  DDSProtocolError error = 3;
}

// ---- AnnounceAvailability RPC ----
// Actual interaction with DHT, but showing request/response structure.
message AnnounceAvailabilityRequest {
  NodeAnnouncementInfo announcement = 1;
}

message AnnounceAvailabilityResponse {
  bool success = 1; // True if announcement accepted by local DHT client/interface.
  DDSProtocolError error = 2;
}

// ---- HandlePoSRChallenge is defined by StorageChallengeRequest/Response above ----

// ---- RequestChunkForReplication RPC ----
message RequestChunkForReplicationRequest {
  string chunk_id = 1;

  // DID of the node requesting the chunk.
  string requester_node_did = 2;

  // Optional: Nonce to prevent replay attacks for this request.
  string nonce = 3;
}

message RequestChunkForReplicationResponse {
  DDSChunk chunk = 1;
  bool success = 2;
  DDSProtocolError error = 3;
}

// ---- RequestErasureFragment RPC ----
message RequestErasureFragmentRequest {
  string original_chunk_id = 1;
  uint32 fragment_index = 2;
}

message RequestErasureFragmentResponse {
  ErasureFragment fragment = 1;
  bool success = 2;
  DDSProtocolError error = 3;
}

// ---- DDSProtocolError Structure ----
message DDSProtocolError {
  // Numeric error code, specific to DDS Protocol.
  int32 code = 1; // Aligns with values in Section 6.
  // Human-readable error message.
  string message = 2;
  // Optional: Additional context for the error.
  string details = 3;
}

```

## 4. Peer-to-Peer Chunk Exchange Protocol (for replication and repair)

1.  **Discovery:**
    *   An SSN (`NodeA`) identifies a need to replicate or repair `ChunkX` (e.g., due to low replica count or a failed PoSR).
    *   `NodeA` queries the Discovery System (DHT, as per `discovery_protocol_refined.md`) for peers storing `ChunkX`. The DHT returns a list of `NodeAnnouncementInfo` for SSNs (e.g., `NodeB`, `NodeC`) holding the chunk.
2.  **Peer Selection:** `NodeA` selects a suitable peer (`NodeB`) from the list based on factors like proximity, load, or reputation (if available).
3.  **Request:** `NodeA` calls `RequestChunkForReplication` on `NodeB`, specifying `chunk_id = ChunkX`.
    *   `NodeA` (Requester) -> `NodeB` (Provider): `RequestChunkForReplicationRequest { chunk_id: "ChunkX", requester_node_did: "did:echonet:NodeA" }`
4.  **Response & Data Transfer:**
    *   If `NodeB` has `ChunkX` and agrees to serve it, it responds with `RequestChunkForReplicationResponse` containing the `DDSChunk`.
    *   `NodeB` -> `NodeA`: `RequestChunkForReplicationResponse { chunk: { chunk_id: "ChunkX", data: <bytes> }, success: true }`
    *   Data transfer can be part of this RPC response for smaller chunks. For larger chunks, the response might initiate a separate streaming connection (e.g., using a protocol like gRPC streaming, HTTP/2 streaming, or QUIC). The initial RPC would then confirm availability and provide connection details.
5.  **Verification:** `NodeA` verifies the received chunk's hash against `ChunkX`.
6.  **Storage & Announcement:** `NodeA` stores `ChunkX` locally and then calls `AnnounceAvailability` to update the Discovery System.

This process is similar for fetching `ErasureFragment`s, using the `RequestErasureFragment` RPC.

## 5. Erasure Coding Parameters & Handling

*   **Default Parameters (Conceptual - subject to change via governance/research):**
    *   Reed-Solomon codes will be used.
    *   **`k` (data shards):** 10
    *   **`m` (total shards - data + parity):** 14 (meaning 4 parity shards)
    *   This provides durability against the loss of any 4 shards out of 14.
*   **Fragmentation:**
    *   When a `DDSChunk` is erasure-coded, it's split into `k` data fragments. `m-k` parity fragments are then computed.
    *   Each fragment is stored with an `ErasureFragment` structure, which includes `original_chunk_id`, `fragment_index`, `k`, `m`, and `original_chunk_hash`.
    *   SSNs store individual fragments and announce their availability using `AnnounceAvailability` (potentially specifying they hold a fragment, not the full chunk, or the DHT handles this distinction).
*   **Reconstruction:**
    *   To reconstruct the original `DDSChunk`, a client or SSN needs to retrieve any `k` distinct fragments (by `fragment_index`) for that `original_chunk_id`.
    *   The `RequestErasureFragment` RPC is used to fetch these fragments from different SSNs.
    *   Once `k` fragments are retrieved, the Reed-Solomon decoding algorithm is applied to reconstruct the original data. The `original_chunk_hash` in `ErasureFragment` is used to verify the integrity of the reconstructed chunk.

## 6. Error Codes Specific to DDS Protocol

These error codes align with the `DDSProtocolError` structure and the philosophy of `echonet_v3_error_handling_strategy.md`.

*   `DDS_ERROR_UNSPECIFIED (0)`: Generic error.
*   `DDS_CHUNK_NOT_FOUND (1)`: Requested chunk ID is not found on the node.
*   `DDS_STORAGE_FULL (2)`: Node cannot store the chunk due to insufficient capacity.
*   `DDS_INVALID_CHUNK_HASH (3)`: Provided chunk data does not match the `chunk_id` hash.
*   `DDS_INVALID_BYTE_RANGE (4)`: Retrieve request specified an invalid or unsatisfiable byte range.
*   `DDS_AUTHORIZATION_FAILED (5)`: Node is not authorized to store/retrieve the chunk or perform the action.
*   `DDS_NODE_UNAVAILABLE (6)`: Target node for replication or retrieval is currently unreachable.
*   `DDS_REPLICATION_FAILED (7)`: Could not successfully replicate the chunk to required peers.
*   `DDS_REPAIR_FAILED (8)`: Chunk repair process failed.
*   `DDS_INVALID_POSR_CHALLENGE (9)`: The PoSR challenge received is malformed or invalid.
*   `DDS_INVALID_POSR_PROOF (10)`: The PoSR proof provided by a node is invalid or failed verification.
*   `DDS_TIMEOUT (11)`: Operation timed out.
*   `DDS_PROTOCOL_VIOLATION (12)`: Peer did not follow protocol correctly.
*   `DDS_ERASURE_CODING_ERROR (13)`: Error during erasure coding (encoding or decoding).
*   `DDS_FRAGMENT_NOT_FOUND (14)`: Requested erasure fragment not found.
*   `DDS_INSUFFICIENT_FRAGMENTS (15)`: Not enough erasure fragments available for reconstruction.

## 7. Security Considerations

*   **Node Authentication:** All DDS nodes (SSNs, ASNs) must have a DID (`NexusUserObjectV1`) and use it to sign relevant messages (e.g., PoSR responses, announcements if required by DHT). RPCs should occur over mutually authenticated TLS channels (using DID-linked certificates or similar).
*   **Authorization for `StoreChunk`:**
    *   SSNs should verify that the `parent_content_id` (if provided in `StoreChunkRequest`) corresponds to a `NexusContentObjectV1` that has been validated by Witnesses (proof discoverable via DLI).
    *   Alternatively, the `origin_signature` (signature of `chunk_id` by creator's DID) can be used to prove authenticity.
    *   Policies may restrict anonymous storage or require pre-authorization via a token/capability system.
*   **Integrity of Transferred Data:**
    *   All `DDSChunk` data is verified against its `chunk_id` (hash) upon receipt.
    *   Erasure fragments are verified against the `original_chunk_hash` after reconstruction.
    *   Secure channels (TLS) protect data in transit.
*   **PoSR Integrity:** The `StorageChallengeRequest` and `StorageChallengeResponse` mechanism, including nonces and signatures, is designed to prevent replay attacks and ensure proof freshness and authenticity.
*   **Access Control for Retrieval:** While most content is public, future extensions might allow encrypted chunks or restricted access based on DID capabilities, managed at a higher layer but enforced by SSNs.
*   **Discovery System Security:** The security of the DHT (Sybil resistance, ensuring correct `NodeAnnouncementInfo`) is crucial for the DDS and is detailed in `discovery_protocol_refined.md`.

## 8. Versioning

The DDS protocol (including RPC definitions and data structures) will be versioned according to the principles outlined in `echonet_v3_versioning_strategy.md`.
*   Protobuf package names can include a version (e.g., `digisocialblock.dds.protocol.v1`).
*   RPC calls can include version negotiation, or service endpoints can be versioned.
*   Node capabilities (advertised via `NodeAnnouncementInfo`) can include supported protocol versions.
*   Breaking changes will require a new major version. Non-breaking additions can be handled with minor versions.

This ensures that the DDS network can evolve over time while maintaining compatibility or allowing for graceful upgrades.
---
This document provides the foundational network protocol for DDS operations. Further details on specific PoSR algorithms or advanced DHT interactions will be in their respective dedicated documents.
