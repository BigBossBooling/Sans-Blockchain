# EchoNet DDS Protocol Specification (Final Consolidated)

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 0. Introduction

This document provides the definitive consolidated technical specification for the EchoNet Distributed Data Stores (DDS) protocol. It synthesizes information from various detailed planning and architectural documents into a single, comprehensive guide. The DDS is responsible for the decentralized storage, retrieval, replication, and long-term durability of content chunks and their metadata within the EchoNet ecosystem.

This document is intended for developers implementing DDS nodes, client applications interacting with DDS, and auditors seeking to understand the full DDS protocol.

## 1. DDS Protocol Overview

The EchoNet DDS aims to provide robust, resilient, decentralized, and cost-effective storage for all content and significant metadata, accessible via `ContentID`s (for content objects) and `ChunkID`s (for individual data chunks).

*   **Purpose & Scope:** The DDS protocol governs how content chunks are stored, retrieved, replicated, proven to be stored (Proof-of-Storage-Receipt - PoSR), and discovered across a decentralized network of storage nodes. It is essential for the persistence, availability, and integrity of content published to the EchoNet platform.
*   **Key Capabilities:**
    *   Decentralized storage of data chunks.
    *   Content-addressing using cryptographic hashes.
    *   Data replication for redundancy.
    *   Erasure coding for enhanced durability and storage efficiency.
    *   Self-healing mechanisms to repair lost or corrupted data.
    *   Proof-of-Storage/Retrievability (PoSR) to ensure nodes are faithfully storing data.
*   **Node Types:** The DDS ecosystem involves several types of nodes:
    *   **Origin Nodes (Creator Devices):** Initial source of content.
    *   **Standard Storage Nodes (SSNs):** Backbone for storing and serving data chunks/fragments, participate in PoSR.
    *   **Archival Storage Nodes (ASNs):** Specialized SSNs for long-term, low-cost storage.
    *   **Ephemeral Cache Nodes (ECNs):** Cache popular content for faster retrieval.
    *   *(Detailed descriptions of node roles and incentives can be found in `dds_refined_architecture.md`.)*

## 2. Core DDS Data Structures

The DDS protocol relies on several Protocol Buffer (protobuf v3) messages for its operations. These are defined across two primary `.proto` files:

*   **`echonet_v3_core.proto`**: Defines fundamental data units used across EchoNet, including those critical for DDS state and on-DLI records.
    *   `DDSChunk`: Represents a single data chunk with its ID and binary data.
    *   `ErasureFragment`: Represents an erasure-coded fragment of a `DDSChunk`, including parameters like `original_chunk_id`, `fragment_index`, `k_value`, `m_value` (total shards), and `original_chunk_hash`.
    *   `StorageChallengeRequest`: Used by challengers (e.g., Witnesses) to initiate a PoSR check.
    *   `StorageChallengeResponse`: Used by SSNs to respond to a PoSR challenge, including proof payload and signature.
    *   `DDSProtocolError`: A standardized structure for reporting errors within DDS operations.
*   **`proto/dds_specific.proto`**: Defines messages primarily used for DDS network communication and RPCs.
    *   `NodeAnnouncementInfo`: Used by storage nodes to announce their capabilities and held chunks to the discovery system. Includes DID, address, storage type, capacity, held chunk IDs, and a signature.
    *   `ReplicationInstruction`: Used to instruct a node to replicate or repair a specific chunk. Includes `chunk_id`, source peer hints, and reason.
    *   RPC Request/Response Messages: Specific messages for each method in `DDSNodeService` (e.g., `StoreChunkRequest`, `RetrieveChunkResponse`).

For complete definitions and detailed field explanations, refer to:
*   [`echonet_v3_core.proto`](../proto/echonet_v3_core.proto) (conceptual link)
*   [`proto/dds_specific.proto`](../proto/dds_specific.proto) (conceptual link)
*   [`tech_specs/dds_protocol_structures.md`](./dds_protocol_structures.md) (provides detailed explanations of these messages)

## 3. DDS Node RPC Service & Methods

Interactions with DDS nodes (SSNs, ASNs) are primarily conducted via the `DDSNodeService`, defined in `proto/dds_specific.proto`. This service exposes the following key RPC methods:

*   **`StoreChunk(StoreChunkRequest) returns (StoreChunkResponse)`:** Allows clients or other nodes to request an SSN to store a `DDSChunk`.
*   **`RetrieveChunk(RetrieveChunkRequest) returns (RetrieveChunkResponse)`:** Allows clients or other nodes to retrieve a `DDSChunk` (or a byte range) from an SSN/ASN.
*   **`AnnounceAvailability(AnnounceAvailabilityRequest) returns (AnnounceAvailabilityResponse)`:** Used by SSNs to announce their `NodeAnnouncementInfo` (including held chunks) to the discovery system (e.g., DHT).
*   **`HandlePoSRChallenge(echonet_core.v3.StorageChallengeRequest) returns (echonet_core.v3.StorageChallengeResponse)`:** Allows challengers (e.g., Witnesses) to issue a PoSR challenge to an SSN, which then responds with a proof.
*   **`RequestChunkForReplication(RequestChunkForReplicationRequest) returns (RequestChunkForReplicationResponse)`:** Used by an SSN to request a full `DDSChunk` from a peer SSN for replication or repair.
*   **`RequestErasureFragment(RequestErasureFragmentRequest) returns (RequestErasureFragmentResponse)`:** Used by an SSN or client to request a specific `ErasureFragment` from a peer SSN.

Detailed information on these RPC methods, including their purpose, request/response parameters, and specific behaviors, can be found in:
*   [`tech_specs/dds_rpc_api.md`](./dds_rpc_api.md) (which serves as a primary reference point)
*   [`tech_specs/dds_protocol_structures.md`](./dds_protocol_structures.md) (for detailed message descriptions)
*   [`proto/dds_specific.proto`](../proto/dds_specific.proto) (for the authoritative service and message definitions)

## 4. Peer-to-Peer Chunk Exchange Protocol

SSNs utilize a P2P protocol to exchange chunks and fragments for replication and repair. This protocol involves:
1.  **Discovery:** An SSN needing a chunk/fragment queries the EchoNet Discovery Protocol (DHT) to find peers holding the data, using the `chunk_id` or `original_chunk_id`.
2.  **Peer Selection:** The requestor selects suitable peers based on criteria like network proximity, reputation, and load.
3.  **Request/Response:** The requestor uses the `RequestChunkForReplication` or `RequestErasureFragment` RPCs to ask a peer for the data. The peer responds with the data or an error.
4.  **Data Transfer:** Data is typically streamed, especially for large chunks, over secure libp2p channels.
5.  **Verification:** Upon receipt, the requestor cryptographically verifies the integrity of the received chunk/fragment data against its known hash.

For comprehensive details on this P2P exchange mechanism, including specific message usage and security considerations, refer to:
*   [`tech_specs/dds_p2p_exchange_protocol.md`](./dds_p2p_exchange_protocol.md)

## 5. Erasure Coding Parameters & Handling

EchoNet DDS employs erasure coding to enhance data durability and storage efficiency.
*   **Scheme:** Reed-Solomon codes.
*   **Default Parameters:**
    *   `k` (Data Shards): 10
    *   `m` (Parity Shards): 4
    *   Total Shards: 14 (any 10 can reconstruct the original data).
    *   These parameters are governable via `ProtocolParametersV1`.
*   **Fragment Structure:** Defined by the `ErasureFragment` message in `echonet_v3_core.proto`, which includes `original_chunk_id`, `fragment_index`, `data`, `k_value`, `m_value` (total shards), and `original_chunk_hash`.
*   **Reconstruction Protocol:** Involves discovering and retrieving at least `k` unique fragments from different SSNs using the P2P exchange protocol, then applying Reed-Solomon decoding, and finally verifying the reconstructed data against the `original_chunk_hash`.

For a complete specification of erasure coding, including fragment identification, encoding/distribution, detailed reconstruction steps, and error conditions, refer to:
*   [`tech_specs/dds_erasure_coding_spec.md`](./dds_erasure_coding_spec.md)

## 6. DDS-Specific Error Codes

DDS operations use a standardized set of error codes, defined within the `DDSProtocolError` message structure (see `echonet_v3_core.proto`). These codes help in identifying issues related to chunk storage, retrieval, PoSR, replication, and erasure coding. Examples include `DDS_CHUNK_NOT_FOUND`, `DDS_STORAGE_FULL`, `DDS_INVALID_POSR_PROOF`, `DDS_INSUFFICIENT_FRAGMENTS`.

For a comprehensive list of DDS-specific error codes, their message templates, and suggested severities, refer to:
*   [`tech_specs/dds_security_error_codes.md`](./dds_security_error_codes.md) (Section 2)
*   [`echonet_v3_error_handling_strategy.md`](../echonet_v3_error_handling_strategy.md) (for the global error structure and philosophy)

## 7. DDS Security Considerations

Security is integral to the design and operation of the DDS. Key considerations include:
*   **Node Authentication:** DDS operations are authenticated using DIDs and secure communication channels (e.g., mTLS over libp2p).
*   **Data Integrity:** Cryptographic hash verification of chunks and reconstructed data is mandatory. Secure transport protocols protect data in transit.
*   **PoSR Integrity:** Secure challenge generation, signed proof submissions by SSNs, and robust validation by Witnesses.
*   **Secure Discovery Interaction:** Authenticated announcements of chunk availability to the DHT using signed `NodeAnnouncementInfo` messages.
*   **Access Control:** Content validation by Witnesses typically precedes DDS storage. Retrieval is generally public, with future provisions for client-side encryption for private data.
*   **Attack Protection:** Measures against storage/bandwidth exhaustion (rate limiting, incentives) and data pollution (pre-validation, hash checks).

For a detailed discussion of these security aspects and specific threat mitigations, refer to:
*   [`tech_specs/dds_security_error_codes.md`](./dds_security_error_codes.md) (Section 3)
*   Relevant sections in `tech_specs/dds_p2p_exchange_protocol.md` and `echonet_v3_communication_protocols.md`.

## 8. DDS Protocol Versioning

The DDS protocol, including its data structures (in `echonet_v3_core.proto` and `proto/dds_specific.proto`), RPC services, and operational semantics, adheres to the global EchoNet versioning strategy.
Changes to any DDS component will be versioned using Semantic Versioning (MAJOR.MINOR.PATCH). Significant or breaking changes will be managed through the EchoNet governance process.

For complete details on the versioning strategy, refer to:
*   [`echonet_v3_versioning_strategy.md`](../echonet_v3_versioning_strategy.md)

This document, along with its referenced detailed specifications, provides the complete technical guide for the EchoNet Distributed Data Stores protocol.
