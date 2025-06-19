# DigiSocialBlock - Content Hashing & Timestamping Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Content Hashing Specifications

This section details the algorithms and procedures for generating content hashes (`ContentID`s for `NexusContentObjectV1` and hashes for `DDSChunk`s), ensuring content integrity and unique identification within the DigiSocialBlock DLI system (also referred to as `EchoNet`).

### 1.A. `NexusContentObjectV1` `ContentID` Generation

The `ContentID` for a `NexusContentObjectV1` serves as its unique, immutable identifier on the DLI. It is derived from a cryptographic hash of a canonical representation of its core, immutable attributes.

*   **Hash Algorithm:** **SHA-256** (as confirmed in `echonet_v3_cryptographic_correctness_review.md`). The output is a 256-bit hash.
*   **`ContentID` Format:** The raw 32-byte SHA-256 hash, typically represented as a 64-character lowercase hexadecimal string.

*   **Pre-image Definition for `ContentID`:**
    The pre-image is a specific concatenation of selected fields from `NexusContentObjectV1` and its nested `CoreContentMetadataV1` (defined in `tech_specs/dli_core_data_structures.md`). The order of concatenation is critical.

    The following fields, in this exact order, form the pre-image:
    1.  `creator_did` (from `NexusContentObjectV1`)
    2.  `content_type` (enum value from `NexusContentObjectV1`)
    3.  `body_hash` (from `NexusContentObjectV1`)
    4.  `core_metadata_hash` (from `NexusContentObjectV1`)
    5.  `client_asserted_timestamp` (from `CoreContentMetadataV1`, which is referenced by `core_metadata_hash`)

    *Note on `core_metadata_hash`*: This field itself is a hash of the `CoreContentMetadataV1` structure, ensuring that all immutable metadata within `CoreContentMetadataV1` contributes to the `ContentID`. The generation of `core_metadata_hash` is detailed in section 1.C.
    *Note on `client_asserted_timestamp`*: While `core_metadata_hash` already includes this timestamp, its explicit inclusion *again* in the `ContentID` pre-image (if this is the design intent from `content_hash_timestamping_refined.md` for added domain separation or specific versioning logic) is noted. However, typical design would have it covered by `core_metadata_hash`. *Correction for typical design: `client_asserted_timestamp` is part of `CoreContentMetadataV1` and thus its influence is via `core_metadata_hash`. The `ContentID` pre-image should be: `creator_did`, `content_type`, `body_hash`, `core_metadata_hash`.* I will proceed with this corrected, more standard pre-image.

    **Corrected Pre-image Definition for `ContentID`:**
    1.  `creator_did` (from `NexusContentObjectV1.creator_did`)
    2.  `content_type` (enum integer value from `NexusContentObjectV1.content_type`)
    3.  `body_hash` (bytes from `NexusContentObjectV1.body_hash`)
    4.  `core_metadata_hash` (bytes from `NexusContentObjectV1.core_metadata_hash`)


*   **Canonicalization Procedure for `ContentID` Pre-image:**
    Adheres to `echonet_v3_canonical_data_formats.md`. The pre-image byte string is constructed by concatenating the canonical representations of the above fields *in the specified order*:

    1.  **`creator_did`:**
        *   Represented as its raw UTF-8 string bytes.
        *   Terminated by a single Null Byte (`\x00`).
    2.  **`content_type`:**
        *   The integer value of the `ContentTypeV1` enum.
        *   Represented as a 64-bit unsigned integer, Big Endian byte order (`uint64_be`).
    3.  **`body_hash`:**
        *   The raw bytes of the hash. (e.g., 32 bytes for SHA-256). No re-encoding.
    4.  **`core_metadata_hash`:**
        *   The raw bytes of the hash. (e.g., 32 bytes for SHA-256). No re-encoding.

    **Example (Conceptual):**
    `PRE_IMAGE_BYTES = UTF8_BYTES(creator_did) + \x00 + UINT64_BE(content_type_enum_int) + RAW_BYTES(body_hash) + RAW_BYTES(core_metadata_hash)`
    `ContentID = SHA256(PRE_IMAGE_BYTES)`

### 1.B. `DDSChunk` Hashing (`chunk_id` Generation)

*   **Hash Algorithm:** **SHA-256** (consistent with `NexusContentObjectV1` and `echonet_v3_cryptographic_correctness_review.md`).
*   **`chunk_id` Format:** The raw 32-byte SHA-256 hash, typically represented as a 64-character lowercase hexadecimal string.
*   **Pre-image:** The raw, unmodified byte sequence of the `DDSChunk.data` itself.
    `chunk_id = SHA256(DDSChunk.data)`
*   **Canonicalization:** Not applicable beyond taking the raw bytes of the chunk data.

### 1.C. `CoreContentMetadataV1` Hashing (`core_metadata_hash` Generation)

The `core_metadata_hash` is stored in `NexusContentObjectV1` and is part of the `ContentID` pre-image. It ensures the immutability of the metadata defined within `CoreContentMetadataV1`.

*   **Hash Algorithm:** **SHA-256**.
*   **`core_metadata_hash` Format:** The raw 32-byte SHA-256 hash.

*   **Pre-image Definition for `core_metadata_hash`:**
    All fields from `CoreContentMetadataV1` (defined in `tech_specs/dli_core_data_structures.md`), in their specified field number order.
    1.  `title` (string)
    2.  `description` (string)
    3.  `tags` (repeated string)
    4.  `language` (string)
    5.  `custom_attributes` (map<string, string>)
    6.  `client_asserted_timestamp` (google.protobuf.Timestamp)

*   **Canonicalization Procedure for `core_metadata_hash` Pre-image:**
    This procedure strictly follows Protocol Buffers v3 deterministic binary serialization rules for the `CoreContentMetadataV1` message.
    1.  Construct an instance of the `CoreContentMetadataV1` message.
    2.  Serialize this message to its binary wire format using standard protobuf3 deterministic serialization. This guarantees:
        *   Fields are ordered by field number.
        *   Consistent encoding for all data types (UTF-8 for strings, varints for integers, specific encoding for timestamps, length-prefixing for strings/bytes and repeated fields, etc.).
        *   Map fields (`custom_attributes`) are serialized by sorting keys lexicographically (UTF-8 byte order) and then serializing each key-value pair.
    3.  The resulting byte string from the deterministic protobuf serialization is the pre-image.

    `PRE_IMAGE_BYTES_CORE_METADATA = DeterministicProto3Serialize(CoreContentMetadataV1_instance)`
    `core_metadata_hash = SHA256(PRE_IMAGE_BYTES_CORE_METADATA)`

    *Note:* Using the direct deterministic protobuf serialization for `CoreContentMetadataV1` is chosen here because it's a well-defined, self-contained structure, and protobuf libraries offer this capability, simplifying canonicalization compared to a manual field-by-field concatenation for this specific sub-hash. This aligns with `echonet_v3_canonical_data_formats.md` which allows for protobuf's canonical binary format.

## 2. Timestamping Mechanism Specifications

EchoNet employs a dual timestamping mechanism to provide both client perspective and network-verified time for content.

### 2.A. Client-Asserted Timestamp

*   **Recording:**
    *   The `client_asserted_timestamp` is a field within the `CoreContentMetadataV1` message (which is part of `NexusContentObjectV1`).
    *   It is populated by the client application at the moment the content is finalized by the creator, before submission to the DLI.
    *   It SHOULD be a `google.protobuf.Timestamp`, representing UTC (Coordinated Universal Time).
*   **Validation by Witnesses (during `NexusContentObjectV1` validation):**
    *   **Existence:** The timestamp must be present.
    *   **Reasonableness:** It must not be unreasonably in the past (e.g., older than a configured threshold like 24 hours, unless specific content types allow historical timestamps).
    *   **Plausibility:** It must not be in the future beyond a very small clock skew tolerance (e.g., a few minutes).
    *   Violations may lead to the content object being deemed invalid by Witnesses.

### 2.B. Network-Validated Timestamp

*   **Source:**
    *   The authoritative network-validated timestamp comes from the `network_timestamp` field within the `WitnessProofV1`.
    *   This `WitnessProofV1.network_timestamp` is typically the median of the `witness_timestamp` values provided by the attesting Witnesses in their `WitnessAttestationV1` messages for the content publication event.
*   **Recording & Association:**
    *   The `NexusContentObjectV1` structure contains a field: `network_validated_timestamp` (google.protobuf.Timestamp).
    *   Once a `WitnessProofV1` is generated for the publication of a `NexusContentObjectV1` (identified by its `ContentID` as `event_id` in the proof):
        *   The DLI logic (or a client observing the proof) is responsible for populating the `network_validated_timestamp` field of the corresponding `NexusContentObjectV1` with the value from `WitnessProofV1.network_timestamp`.
        *   This updated `NexusContentObjectV1` (now including the `network_validated_timestamp`) might be re-stored/updated on the DDS if the DDS allows metadata updates, or this timestamp is primarily used by indexing services that combine the `NexusContentObjectV1` data with its `WitnessProofV1`. For simplicity and immutability of the core object, it's often preferred that indexing services link these. However, if `NexusContentObjectV1` is designed to be a record that *can* be minimally augmented with network-validated data like this timestamp, then it would be updated. Given its definition in `dli_core_data_structures.md`, it *has* this field, implying it's intended to be filled.
*   **Resolution/Precedence:**
    *   For all DLI-authoritative operations (e.g., determining content order in feeds, PoE reward sequencing, dispute resolution, official publication time), the **`network_validated_timestamp` takes precedence** over the `client_asserted_timestamp`.
    *   The `client_asserted_timestamp` remains valuable as the creator's view of time and for applications where that perspective is relevant, but it is not considered the official DLI timestamp for the event.

### 2.C. Utilization

*   **Content Discovery (`content_discovery_module_refined.md`):**
    *   `network_validated_timestamp` is primarily used for sorting content by "recently published," "trending" (when combined with interaction velocity), and for time-based filtering.
*   **Proof-of-Engagement (PoE) (`poe_rewards_refined.md`):**
    *   `network_validated_timestamp` of both content and interactions (`NexusInteractionRecordV1` will also have a network-validated timestamp via its own `WitnessProofV1`) is crucial for sequencing events, calculating decay for reputation scores, and determining eligibility windows for rewards.
*   **Versioning & History:** Timestamps help in tracking versions of content (if a higher-level versioning system is built using these immutable objects) and understanding the historical sequence of events.
*   **Auditing & Forensics:** Both timestamps provide valuable data points for auditing content history.

## 3. Format of Hashes and Timestamps

*   **Hashes (e.g., `ContentID`, `chunk_id`, `core_metadata_hash`):**
    *   When represented as strings, they SHOULD be lowercase hexadecimal characters.
    *   Example: `a1b2c3d4e5f60718293a4b5c6d7e8f90a1b2c3d4e5f60718293a4b5c6d7e8f90` (for SHA-256).
    *   Internally (e.g., in protobuf `bytes` fields), they are raw binary byte sequences (32 bytes for SHA-256).
*   **Timestamps (`google.protobuf.Timestamp`):**
    *   Internally stored as seconds and nanoseconds since Unix epoch.
    *   When represented as strings (e.g., in JSON for APIs, or logs), they SHOULD follow the RFC3339/ISO 8601 format: `YYYY-MM-DDTHH:mm:ss.sssZ` (e.g., `2023-10-28T12:34:56.789Z`).

## 4. Security & Integrity Notes

*   **Canonicalization Adherence:** Strict and unwavering adherence to the defined canonicalization procedures (both for the `ContentID` pre-image and the `core_metadata_hash` pre-image) is paramount. Any deviation will result in a different hash, making content unverifiable or leading to duplicate entries. All client implementations and DLI nodes must use the exact same procedures.
*   **Immutability:**
    *   Once a `ContentID` is generated and its corresponding `NexusContentObjectV1` is validated and associated with a `WitnessProofV1`, the content data (`body_hash`) and the core metadata (`core_metadata_hash` and its constituent fields) are cryptographically sealed and immutable. Any change to these would result in a new `ContentID`.
    *   This immutability is a cornerstone of EchoNet's content integrity and verifiability.
*   **Timestamp Integrity:** While clients assert a timestamp, the `network_validated_timestamp` (derived from Witness consensus) provides a secure and reliable timestamp that is resistant to manipulation by individual clients or Witnesses.

---
This document provides the definitive specifications for how content is uniquely identified through hashing and how its creation time is recorded and verified within the EchoNet DLI. Developers must strictly adhere to these specifications.
