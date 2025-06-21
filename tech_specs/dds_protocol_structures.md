# DigiSocialBlock DDS - Specific Protocol Data Structures

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 1. Introduction

This document specifies supplementary Protocol Buffer (protobuf3) data structures that are specific to the Distributed Data Stores (DDS) network protocol within the EchoNet (also referred to as DigiSocialBlock) DLI ecosystem. These structures are primarily used for peer-to-peer network communication between DDS nodes, including RPC service definitions and their associated request/response messages.

Core DDS data units that are fundamental to DLI state or may be directly referenced in DLI records (like `WitnessProofV1` for PoSR) are defined in `echonet_v3_core.proto`. This includes:
*   `DDSChunk`
*   `ErasureFragment`
*   `StorageChallengeRequest`
*   `StorageChallengeResponse`
*   `DDSProtocolError`

The structures defined herein are typically transient, facilitating DDS operations, and are defined in `proto/dds_specific.proto`.

## 2. DDS-Specific Data Structure Definitions

These messages are defined in `proto/dds_specific.proto`.

### 2.1. `NodeAnnouncementInfo`
*   **Purpose:** Used by DDS storage nodes (SSNs, ASNs, potentially ECNs) to announce their presence, capabilities, and the content chunks they currently hold to the discovery system (e.g., a DHT).
*   **Fields:**
    *   `node_did` (`string`): The Decentralized Identifier (DID) of the announcing node. Must be a valid `did:echonet` URI. This DID is used to establish trust and verify the signature.
    *   `address` (`string`): The network address (e.g., libp2p multiaddr format) at which the node can be reached for DDS operations.
    *   `storage_type` (`string`): Specifies the type of storage node (e.g., "SSN_GENERIC", "ASN_HIGH_DURABILITY", "ECN_CACHE"). This helps clients or other nodes select appropriate peers.
    *   `region_hint` (`string`, optional): A hint about the geographic region of the node, which can be used for latency-aware peer selection.
    *   `total_capacity_bytes` (`uint64`, optional): The total storage capacity the node offers to the network.
    *   `available_capacity_bytes` (`uint64`, optional): The currently available storage capacity on the node.
    *   `held_chunk_ids` (`repeated string`): A list of `chunk_id`s (hashes of `DDSChunk`s) that the node currently stores and makes available. For scalability, this might be paginated in requests or use probabilistic structures in a full implementation, but is a simple list for MVP.
    *   `announcement_timestamp` (`google.protobuf.Timestamp`): The UTC timestamp when this announcement was generated.
    *   `signature` (`bytes`): A cryptographic signature by the `node_did` over the canonicalized representation of the other fields in the announcement. This verifies the authenticity and integrity of the announcement.

### 2.2. `ReplicationInstruction`
*   **Purpose:** Sent to a DDS node to instruct it to replicate or repair a specific data chunk. This is a key message for the self-healing mechanism of the DDS.
*   **Fields:**
    *   `chunk_id` (`string`): The identifier (hash) of the `DDSChunk` that needs to be replicated or repaired.
    *   `source_peer_dids` (`repeated string`, optional): A list of DIDs of peer nodes known to currently hold a valid copy of the `chunk_id`. If empty, the recipient node is expected to use the discovery system to find sources.
    *   `target_replication_factor` (`uint32`, optional): If applicable (e.g., for simple replication), the desired number of active replicas to ensure for this chunk.
    *   `reason` (`string`): A human-readable string indicating the reason for the instruction (e.g., "INITIAL_REPLICATION", "REPAIR_LOST_REPLICA", "INCREASE_REDUNDANCY").
    *   `instruction_id` (`string`): A unique identifier for this specific instruction, useful for tracking and idempotency.
    *   `issued_timestamp` (`google.protobuf.Timestamp`): The UTC timestamp when this instruction was issued.

## 3. RPC Service: `DDSNodeService` and Messages

The following messages are request and response types for the `DDSNodeService` RPCs, facilitating direct interactions with DDS nodes.

### 3.1. `StoreChunkRequest` / `StoreChunkResponse`
*   **Purpose:** Used by clients or other nodes to request a DDS node to store a `DDSChunk`.
*   **`StoreChunkRequest` Fields:**
    *   `chunk` (`echonet_core.v3.DDSChunk`): The actual chunk data and its ID.
    *   `desired_initial_replication` (`uint32`, optional): Hint for the initial replication factor the storing node might try to achieve.
    *   `origin_signature` (`bytes`, optional): Signature of `chunk.chunk_id` by the original content creator's DID, for authenticity.
    *   `parent_content_id` (`string`, optional): The `ContentID` of the larger content object this chunk belongs to.
*   **`StoreChunkResponse` Fields:**
    *   `storage_receipt_id` (`string`): A unique identifier for this storage transaction on the responding node.
    *   `success` (`bool`): True if the chunk was successfully accepted for local storage. Replication is typically handled asynchronously.
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if `success` is false.

### 3.2. `RetrieveChunkRequest` / `RetrieveChunkResponse`
*   **Purpose:** Used by clients or other nodes to retrieve a `DDSChunk` from a DDS node.
*   **`RetrieveChunkRequest` Fields:**
    *   `chunk_id` (`string`): The ID of the chunk to retrieve.
    *   `byte_range` (`string`, optional): Specifies a byte range to retrieve (RFC 7233 format, e.g., "bytes=0-1023"). If empty, the whole chunk is returned.
*   **`RetrieveChunkResponse` Fields:**
    *   `chunk` (`echonet_core.v3.DDSChunk`): The requested chunk data (full or partial).
    *   `success` (`bool`): True if the chunk (or byte range) was successfully retrieved.
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if `success` is false.

### 3.3. `AnnounceAvailabilityRequest` / `AnnounceAvailabilityResponse`
*   **Purpose:** Used by a DDS node to announce its information (including held chunks) to the discovery system.
*   **`AnnounceAvailabilityRequest` Fields:**
    *   `announcement` (`NodeAnnouncementInfo`): The detailed announcement from the node.
*   **`AnnounceAvailabilityResponse` Fields:**
    *   `success` (`bool`): True if the announcement was successfully processed by the receiving peer (e.g., local DHT client accepted it).
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if `success` is false.

### 3.4. `RequestChunkForReplicationRequest` / `RequestChunkForReplicationResponse`
*   **Purpose:** Used by a DDS node to request a specific chunk from another peer node, typically for replication or repair purposes.
*   **`RequestChunkForReplicationRequest` Fields:**
    *   `chunk_id` (`string`): The ID of the chunk being requested.
    *   `requester_node_did` (`string`): The DID of the node making the request.
    *   `nonce` (`string`, optional): A nonce to prevent replay attacks for this specific request.
*   **`RequestChunkForReplicationResponse` Fields:**
    *   `chunk` (`echonet_core.v3.DDSChunk`): The requested chunk data.
    *   `success` (`bool`): True if the chunk is available and provided.
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if `success` is false.

### 3.5. `RequestErasureFragmentRequest` / `RequestErasureFragmentResponse`
*   **Purpose:** Used by a client or DDS node to request a specific erasure-coded fragment of an original chunk from a peer.
*   **`RequestErasureFragmentRequest` Fields:**
    *   `original_chunk_id` (`string`): The ID of the original `DDSChunk` from which the fragment was derived.
    *   `fragment_index` (`uint32`): The index of the specific fragment being requested.
*   **`RequestErasureFragmentResponse` Fields:**
    *   `fragment` (`echonet_core.v3.ErasureFragment`): The requested erasure fragment data.
    *   `success` (`bool`): True if the fragment is available and provided.
    *   `error` (`echonet_core.v3.DDSProtocolError`): Error details if `success` is false.

## 4. Full `proto/dds_specific.proto` Content

```protobuf
syntax = "proto3";

package echonet.dds.v3; // New package for DDS specific messages

import "echonet_core.v3"; // Imports types like DDSChunk, StorageChallengeRequest etc.
import "google/protobuf/timestamp.proto";

option go_package = "github.com/your_org/digisocialblock/internal/dds/proto/v3;ddspbv3";

// Information about a storage node's capabilities and stored content.
// (From tech_specs/dds_protocol.md)
message NodeAnnouncementInfo {
  // DID of the announcing node.
  string node_did = 1; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Network address of the node (e.g., multiaddr format).
  string address = 2;

  // Type of storage node (e.g., "SSN_GENERIC", "ASN_HIGH_DURABILITY", "ECN_CACHE").
  string storage_type = 3;

  // Geographic region hint (optional).
  string region_hint = 4;

  // Total storage capacity in bytes (optional).
  uint64 total_capacity_bytes = 5;

  // Available storage capacity in bytes (optional).
  uint64 available_capacity_bytes = 6;

  // List of chunk_ids this node is announcing it holds.
  // For bulk announcements, this can be paginated or use probabilistic structures like Bloom filters.
  // For MVP, a simple list.
  repeated string held_chunk_ids = 7;

  // Timestamp of the announcement.
  google.protobuf.Timestamp announcement_timestamp = 8;

  // Signature by node_did over (address, storage_type, region_hint, total_capacity_bytes, available_capacity_bytes, sorted(held_chunk_ids), announcement_timestamp)
  bytes signature = 9;
}

// Instruction for a node to replicate/repair a chunk.
// (From tech_specs/dds_protocol.md)
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

  // Unique ID for this instruction, can be used for tracking.
  string instruction_id = 5;

  // Timestamp when the instruction was issued.
  google.protobuf.Timestamp issued_timestamp = 6;
}

// --- RPC Service Definition for DDS Nodes (DDSNodeService) ---
// (From tech_specs/dds_protocol.md)

// ---- StoreChunk RPC Messages ----
message StoreChunkRequest {
  echonet_core.v3.DDSChunk chunk = 1; // Contains chunk_id and data.

  // Optional: Desired initial replication factor. The network will aim for this.
  uint32 desired_initial_replication = 2;

  // Optional: Signature of chunk_id by the original content creator's DID.
  // Helps SSNs verify authenticity and prioritize storage.
  bytes origin_signature = 3;

  // Optional: ContentID of the parent object this chunk belongs to (from echonet_core.v3)
  string parent_content_id = 4;
}

message StoreChunkResponse {
  // Unique ID for this storage transaction on this node.
  string storage_receipt_id = 1;

  // True if successfully stored locally (replication initiated separately).
  bool success = 2;

  // Error details if success is false.
  echonet_core.v3.DDSProtocolError error = 3;
}

// ---- RetrieveChunk RPC Messages ----
message RetrieveChunkRequest {
  string chunk_id = 1;

  // Optional: Specify a byte range (e.g., "bytes=0-1023").
  // If not provided, the whole chunk is returned.
  string byte_range = 2; // RFC 7233 format
}

message RetrieveChunkResponse {
  echonet_core.v3.DDSChunk chunk = 1; // Contains chunk_id and requested data (full or partial).

  // True if data is present and retrieved.
  bool success = 2;

  // Error details if success is false.
  echonet_core.v3.DDSProtocolError error = 3;
}

// ---- AnnounceAvailability RPC Messages ----
// Actual interaction with DHT, but showing request/response structure.
message AnnounceAvailabilityRequest {
  NodeAnnouncementInfo announcement = 1;
}

message AnnounceAvailabilityResponse {
  bool success = 1; // True if announcement accepted by local DHT client/interface.
  echonet_core.v3.DDSProtocolError error = 2;
}

// ---- RequestChunkForReplication RPC Messages ----
message RequestChunkForReplicationRequest {
  string chunk_id = 1;

  // DID of the node requesting the chunk.
  string requester_node_did = 2;

  // Optional: Nonce to prevent replay attacks for this request.
  string nonce = 3;
}

message RequestChunkForReplicationResponse {
  echonet_core.v3.DDSChunk chunk = 1;
  bool success = 2;
  echonet_core.v3.DDSProtocolError error = 3;
}

// ---- RequestErasureFragment RPC Messages ----
message RequestErasureFragmentRequest {
  string original_chunk_id = 1;
  uint32 fragment_index = 2;
}

message RequestErasureFragmentResponse {
  echonet_core.v3.ErasureFragment fragment = 1;
  bool success = 2;
  echonet_core.v3.DDSProtocolError error = 3;
}

// DDSNodeService defines the RPC calls supported by DDS storage nodes.
service DDSNodeService {
  // Client/Origin Node requests an SSN to store a chunk.
  rpc StoreChunk(StoreChunkRequest) returns (StoreChunkResponse);

  // Client/ECN/SSN requests to retrieve a chunk from an SSN/ASN.
  rpc RetrieveChunk(RetrieveChunkRequest) returns (RetrieveChunkResponse);

  // SSN/ASN announces to the Discovery Protocol (e.g., DHT) that it holds certain chunks.
  rpc AnnounceAvailability(AnnounceAvailabilityRequest) returns (AnnounceAvailabilityResponse);

  // Witness or auditing node challenges an SSN/ASN to prove storage.
  // Uses messages from echonet_core.v3
  rpc HandlePoSRChallenge(echonet_core.v3.StorageChallengeRequest) returns (echonet_core.v3.StorageChallengeResponse);

  // SSN requests a chunk from another SSN for replication or repair.
  rpc RequestChunkForReplication(RequestChunkForReplicationRequest) returns (RequestChunkForReplicationResponse);

  // Client or SSN requests an erasure-coded fragment from an SSN.
  rpc RequestErasureFragment(RequestErasureFragmentRequest) returns (RequestErasureFragmentResponse);
}
```
This markdown document clarifies the separation of DDS structures and provides detailed explanations for those specific to DDS network communications, now residing in `proto/dds_specific.proto`.
