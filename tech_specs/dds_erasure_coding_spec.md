# EchoNet DDS - Erasure Coding Specification

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 1. Introduction

This document details the erasure coding scheme, parameters, and handling mechanisms used within the EchoNet Distributed Data Stores (DDS). Erasure coding is a critical component for ensuring data durability and storage efficiency in the DDS layer.

This specification consolidates information from various architectural and protocol documents, including `dds_refined_architecture.md`, `replication_redundancy_refined.md`, `echonet_v3_core.proto` (for message definitions), and `tech_specs/dds_p2p_exchange_protocol.md` (for P2P exchange).

## 2. Erasure Coding Scheme and Parameters

### 2.1. Chosen Scheme
*   **Scheme:** Reed-Solomon codes.
*   **Rationale:** Reed-Solomon codes are widely used and well-understood, offering optimal data recovery capabilities for a given amount of redundancy (Maximum Distance Separable codes).

### 2.2. Default Parameters
The EchoNet DDS employs default parameters for erasure coding, which can be subject to adjustment via the `ProtocolParametersV1` governance mechanism.
*   **`k` (Data Shards):** The number of data shards the original `DDSChunk` is split into.
    *   **Default Value:** `10`
*   **`m` (Parity Shards):** The number of additional parity shards generated.
    *   **Default Value:** `4`
*   **Total Shards:** The total number of shards generated for each original `DDSChunk` is `k + m`.
    *   **Default Total:** `14` (i.e., 10 data + 4 parity shards).
*   **Reconstruction Property:** Any `k` out of the `k + m` total shards are sufficient to reconstruct the original `DDSChunk`.
    *   **Default:** Any 10 out of 14 shards can reconstruct the data.

*Note: Other documents like `replication_redundancy_refined.md` have mentioned alternative example schemes (e.g., k=10, m=6 parity for a total of 16 shards). The default values specified here (k=10, m=4 parity) align with earlier explicit protocol considerations in `tech_specs/dds_protocol.md`. The `ErasureFragment` message itself carries `k_value` and `m_value` (where `m_value` in the message should represent total shards `k+m`), allowing for flexibility if different schemes are used for different data or if parameters are updated via governance.*

## 3. Erasure Fragment Structure and Identification

### 3.1. `ErasureFragment` Message
The structure for an erasure-coded fragment is defined as the `ErasureFragment` message in `echonet_v3_core.proto`.

```protobuf
message ErasureFragment {
  string original_chunk_id = 1;    // Hash of the original DDSChunk
  uint32 fragment_index = 2;       // Index of this specific fragment (0 to m_value-1)
  bytes data = 3;                  // The actual data of this fragment
  uint32 k_value = 4;              // The 'k' parameter of the Reed-Solomon scheme used
  uint32 m_value = 5;              // The total number of shards (k+m) for this scheme
  bytes original_chunk_hash = 6;   // Hash of the original DDSChunk data (for verification after reconstruction)
}
```

### 3.2. Fragment Identification
Each `ErasureFragment` is uniquely identified by the combination of:
*   `original_chunk_id`: The hash of the original `DDSChunk` it belongs to.
*   `fragment_index`: Its specific index within the set of `m_value` total fragments.

Storage Nodes (SSNs) holding erasure fragments will announce their availability based on the `original_chunk_id`.

## 4. Fragment Encoding, Storage, and Distribution

1.  **Encoding:** When a `DDSChunk` is designated for erasure coding:
    *   The Origin Node or an SSN responsible for initial processing splits the `DDSChunk.data` into `k` data shards of equal size (padding the last shard if necessary).
    *   It then applies the Reed-Solomon encoding algorithm to generate `m` parity shards from these `k` data shards.
2.  **Fragment Creation:** Each of these `k+m` shards becomes the `data` field of an `ErasureFragment` message.
    *   `original_chunk_id` is set to the `chunk_id` of the source `DDSChunk`.
    *   `fragment_index` is assigned from `0` to `(k+m)-1`.
    *   `k_value` and `m_value` (total shards) are set according to the scheme used.
    *   `original_chunk_hash` is set to the `chunk_id` of the source `DDSChunk` (which is the hash of its data).
3.  **Distribution and Storage:** Each of the `k+m` unique `ErasureFragment`s is then distributed and stored on different SSNs, following the data placement strategies outlined in `replication_redundancy_refined.md` to maximize geographic and operational diversity.

## 5. Data Reconstruction from Fragments

A node (client or another SSN performing repair) can reconstruct the original `DDSChunk` if it can retrieve any `k` distinct `ErasureFragment`s belonging to the same `original_chunk_id`.

### 5.1. Steps for Reconstruction:
1.  **Identify Need:** The node determines it needs to reconstruct `OriginalChunkID_X`.
2.  **Discover Fragment Holders:** The node queries the EchoNet Discovery Protocol (DHT) for peers announcing availability for `OriginalChunkID_X`. The discovery system should ideally allow nodes to indicate they hold fragments rather than the full chunk, or this can be inferred if full chunk holders are not found but fragment holders are.
3.  **Select Peers:** From the list of potential holders, the node selects at least `k` distinct peers (ideally more, for redundancy in requests). Peer selection criteria are detailed in `tech_specs/dds_p2p_exchange_protocol.md`.
4.  **Request Fragments:** The node sends `RequestErasureFragmentRequest` messages to the selected peers. Each request specifies the `original_chunk_id` and a unique `fragment_index` it needs. This process is repeated until `k` distinct fragments are successfully retrieved.
    *   Refer to `tech_specs/dds_p2p_exchange_protocol.md` (Section 6) for the P2P fragment exchange protocol.
5.  **Collect Fragments:** The node collects the `data` payloads from the `RequestErasureFragmentResponse` messages. It must receive at least `k` fragments with distinct `fragment_index` values, all sharing the same `original_chunk_id`, `k_value`, and `m_value`.
6.  **Verify Fragment Parameters:** Ensure all collected fragments have consistent `k_value` and `m_value`.
7.  **Apply Reed-Solomon Decoding:** Using the collected `k` fragment data payloads and their respective `fragment_index` values, the node applies the Reed-Solomon decoding algorithm to reconstruct the original data shards. These are then combined to form the original `DDSChunk.data`.
8.  **Verify Reconstructed Data:** The node calculates the hash of the reconstructed `DDSChunk.data`. This calculated hash **MUST** match the `original_chunk_hash` field present in each of the received `ErasureFragment`s (which should also be identical to the `original_chunk_id`).
    *   If verification fails, the reconstruction has failed, possibly due to corrupted fragments or incorrect peer responses. The node may need to discard the attempt and try retrieving fragments from different peers.
9.  **Assemble `DDSChunk`:** If verification succeeds, the node assembles a new `DDSChunk` message using the `original_chunk_id` and the reconstructed data.

## 6. Specific Error Conditions

Beyond general P2P or DDS errors (defined in `DDSProtocolError` within `echonet_v3_core.proto`), specific conditions related to erasure coding include:

*   **`DDS_INSUFFICIENT_FRAGMENTS` (from `tech_specs/dds_protocol.md` Error Codes, maps to `DDSProtocolError`):** Not enough unique fragments could be retrieved from the network to meet the required `k` threshold for reconstruction.
*   **`DDS_ERASURE_CODING_ERROR` (from `tech_specs/dds_protocol.md` Error Codes, maps to `DDSProtocolError`):**
    *   This can occur if the Reed-Solomon decoding process itself fails even with `k` fragments (e.g., due to internal library errors, or if fragments were subtly corrupted in a way not caught by transport).
    *   This can also be used if the post-reconstruction hash verification (Step 8 above) fails, indicating that the reconstructed data does not match the expected `original_chunk_hash`.
*   **Inconsistent Fragment Parameters:** If retrieved fragments for the same `original_chunk_id` report different `k_value` or `m_value`, indicating a severe network inconsistency or malicious/faulty peers. This should lead to a reconstruction failure.

## 7. Protocol Versioning

The erasure coding scheme, parameters (`k`, `m`), and the `ErasureFragment` message structure are subject to the overall versioning strategy defined in `echonet_v3_versioning_strategy.md`. Changes to these, especially the scheme or fundamental parameters, would likely require a protocol version update managed via governance. The `ErasureFragment` message itself contains `k_value` and `m_value` to support potential future flexibility or multiple concurrent schemes if ever introduced.

This specification ensures a clear understanding of how erasure coding is implemented and utilized within EchoNet DDS for data durability and efficiency.
