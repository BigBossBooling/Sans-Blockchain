# DigiSocialBlock DLI - Core Data Structures Technical Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Introduction

This document provides precise technical specifications for the foundational core data structures used within the DigiSocialBlock Distributed Ledger Inspired (DLI) system, also referred to as `EchoNet` in broader architectural documents. These structures are critical for representing content, users, interactions, and their validation proofs on the DLI, ensuring consistency, verifiability, and interoperability across the network.

These specifications are derived from conceptual definitions in `echonet_v3_core_data_structures.md` and align with principles outlined in `echonet_v3_canonical_data_formats.md` and `echonet_v3_core_definitions.md`.

## 2. Chosen Specification Format

These structures are defined using **Protocol Buffers v3 (protobuf3)** syntax for clarity, efficiency, strong typing, and cross-language code generation. The protobuf binary format is generally canonical by default, which is crucial for hashing and signing.

For contexts requiring human readability or specific signing schemes (e.g., some JWS/JOSE applications), a **Canonical JSON representation** MUST be used. This means:
*   Fields MUST be output in the order of their field numbers.
*   Default values (e.g., 0 for integers, empty strings) MUST be emitted.
*   Timestamps will be represented as defined in `google.protobuf.Timestamp` (string format: `YYYY-MM-DDTHH:mm:ss.sssZ`).
*   Bytes fields will be Base64URL encoded.
*   Maps will be converted to JSON objects, with keys sorted lexicographically.

Adherence to these rules, as detailed in `echonet_v3_canonical_data_formats.md`, is essential for deterministic serialization when not using the protobuf binary format directly.

## 3. Core Data Structure Definitions

### 3.1. `NexusUserObjectV1`

*   **Structure Name & Version:** `NexusUserObjectV1`
*   **Purpose:** Represents core, DLI-verifiable attributes of a user, linked to their Decentralized Identifier (DID). This object serves as the on-DLI anchor for a user's presence and reputation, distinct from off-DLI mutable profile data.
*   **Field Definitions:**

```protobuf
syntax = "proto3";

package digisocialblock.dli.core;

import "google/protobuf/timestamp.proto";
import "google/protobuf/struct.proto"; // For potential future extensions if needed

// NexusUserObjectV1 represents core, DLI-verifiable attributes of a user.
message NexusUserObjectV1 {
  // The unique Decentralized Identifier (DID) for the user.
  // Format: did:echonet:<unique-identifier>
  // Validation: Required, must match specified DID pattern.
  string user_did = 1; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Timestamp of the user's registration on the DLI.
  // Validation: Required. Must be a valid UTC timestamp.
  google.protobuf.Timestamp registration_timestamp = 2;

  // DLI-computed reputation score.
  // Managed by PoE (Proof-of-Engagement) and governance logic.
  // Validation: Cannot be directly set; updated by network processes.
  int64 reputation_score = 3;

  // Bitmask representing user status flags (e.g., isWitness, isStorageProvider).
  // Enum for flags should be defined and managed globally.
  // Example flags (conceptual, actual values TBD by global enum):
  // 0x0001: IsActive
  // 0x0002: IsWitnessCandidate
  // 0x0004: IsWitnessActive
  // 0x0008: IsStorageProviderCandidate
  // 0x0010: IsStorageProviderActive
  // 0x0020: IsModerator (community elected)
  uint32 status_flags = 4;

  // Map of public key entries associated with the user's DID.
  // Key: Purpose string or Key ID (e.g., "signing", "encryption", "did-doc-key-1").
  // Value: Bytes representing the public key (e.g., DER-encoded, or raw bytes).
  // Validation: Keys must be valid public keys of supported types.
  map<string, bytes> public_key_entries = 5;

  // Version of this user object structure. Incremented for breaking changes.
  uint32 version = 6; // Default to 1 for this version

  // Optional: Pointer/reference to off-DLI extended profile data.
  // Could be a ContentID of a profile document or an external URL (if allowed by policy).
  // Validation: If present, must be a valid URI or ContentID.
  string extended_profile_ref = 7;

  // Reserved for future fields.
  // google.protobuf.Struct reserved_for_future_expansion = 99;
}
```

*   **Canonicalization Notes:** For hashing or signing, the protobuf binary serialization is preferred. If Canonical JSON is used, field order as defined by number, lexicographical sorting of `public_key_entries` keys, and emission of default values are critical.
*   **Cross-references:** Used in `WitnessProofV1` (for `attestingWitnessDIDs`), `NexusContentObjectV1` (`creatorDID`), `NexusInteractionRecordV1` (`actorDID`). User lifecycle events (creation, updates) would be validated by Witnesses.

### 3.2. `NexusContentObjectV1`

*   **Structure Name & Version:** `NexusContentObjectV1`
*   **Purpose:** Represents a piece of content on the DLI. It includes core immutable metadata that contributes to its `ContentID`, and references to the content body and potentially mutable metadata.
*   **Field Definitions:**

```protobuf
syntax = "proto3";

package digisocialblock.dli.core;

import "google/protobuf/timestamp.proto";

// Enum defining the type of content.
enum ContentTypeV1 {
  CONTENT_TYPE_V1_UNSPECIFIED = 0;
  CONTENT_TYPE_V1_ARTICLE = 1;     // Long-form text content
  CONTENT_TYPE_V1_SHORT_POST = 2;  // Microblog post, status update
  CONTENT_TYPE_V1_IMAGE = 3;
  CONTENT_TYPE_V1_VIDEO = 4;
  CONTENT_TYPE_V1_AUDIO = 5;
  CONTENT_TYPE_V1_COMMENT = 6;     // A comment itself can be content, though usually part of an InteractionRecord
  CONTENT_TYPE_V1_PROFILE = 7;     // User profile data stored as content
  CONTENT_TYPE_V1_GENERIC_FILE = 8;
}

// Core immutable metadata for content. This structure is hashed to produce core_metadata_hash.
message CoreContentMetadataV1 {
  // Title or name of the content.
  // Validation: Max length (e.g., 256 chars).
  string title = 1;

  // Brief description or abstract.
  // Validation: Max length (e.g., 1024 chars).
  string description = 2;

  // Tags or keywords associated with the content.
  // Validation: Max number of tags (e.g., 20), max length per tag (e.g., 50 chars).
  repeated string tags = 3;

  // Language of the content (e.g., "en-US", "es-ES").
  // Validation: Must be a valid IETF language tag.
  string language = 4;

  // Custom attributes or application-specific metadata.
  // Key: Attribute name. Value: Attribute value.
  // Validation: Limits on key/value length and total size.
  map<string, string> custom_attributes = 5;

  // Timestamp asserted by the client/creator at the time of creation.
  // Validation: Must be a valid UTC timestamp. Cannot be significantly in the future.
  google.protobuf.Timestamp client_asserted_timestamp = 6;
}

// NexusContentObjectV1 represents a piece of content on the DLI.
message NexusContentObjectV1 {
  // The unique identifier for this content object.
  // Typically a hash of (creator_did + client_asserted_timestamp + core_metadata_hash + body_hash).
  // Validation: Required, must be a valid hash (e.g., SHA2-256 hex encoded).
  string content_id = 1;

  // DID of the content creator.
  // Validation: Required, must be a valid did:echonet format.
  string creator_did = 2; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Type of the content.
  // Validation: Required, must be a valid ContentTypeV1 enum value.
  ContentTypeV1 content_type = 3;

  // Cryptographic hash of the canonical CoreContentMetadataV1 structure.
  // Ensures immutability of this core metadata set.
  // Validation: Required, must be a valid hash.
  bytes core_metadata_hash = 4;

  // Cryptographic hash of the entire content body (concatenated chunks or single blob).
  // Validation: Required, must be a valid hash.
  bytes body_hash = 5;

  // Ordered list of identifiers (e.g., hashes) for the chunks comprising the content body.
  // Used for reconstructing content from Distributed Data Stores (DDS).
  // Validation: If content is chunked, this is required. Max number of chunks.
  repeated string body_chunk_ids = 6;

  // Size of the original content body in bytes.
  // Validation: Required, must be >= 0.
  int64 body_size_bytes = 7;

  // Timestamp when this content object was validated and accepted by the network (Witnesses).
  // Set by the Witness network.
  // Validation: Cannot be set by client. Must be a valid UTC timestamp.
  google.protobuf.Timestamp network_validated_timestamp = 8;

  // Optional: Pointer/reference to dynamic or mutable metadata not covered by core_metadata_hash.
  // Could be a ContentID of another NexusContentObjectV1 (e.g., for versioned metadata) or an external link.
  // Validation: If present, must be a valid URI or ContentID.
  string dynamic_metadata_ref = 9;

  // Version of this content object structure.
  uint32 version = 10; // Default to 1 for this version
}
```

*   **Canonicalization Notes:** `ContentID` generation depends on the canonical serialization of its constituent parts. The `core_metadata_hash` depends on canonical serialization of `CoreContentMetadataV1`. For hashing or signing `NexusContentObjectV1` itself, protobuf binary is preferred. If Canonical JSON is used, field order by number, sorted map keys for `custom_attributes` in `CoreContentMetadataV1`, and emission of defaults are critical.
*   **Cross-references:** `ContentID` is a key identifier used throughout the system, e.g., in `NexusInteractionRecordV1`, `WitnessProofV1`. Content creation is a DLI event validated by Witnesses.

### 3.3. `NexusInteractionRecordV1`

*   **Structure Name & Version:** `NexusInteractionRecordV1`
*   **Purpose:** Represents a generic user interaction with content or another user on the DLI. This structure is versatile enough to encapsulate various types of engagement.
*   **Field Definitions:**

```protobuf
syntax = "proto3";

package digisocialblock.dli.core;

import "google/protobuf/timestamp.proto";
import "google/protobuf/any.proto"; // For type-safe specific payloads

// Enum defining the type of interaction.
enum InteractionTypeV1 {
  INTERACTION_TYPE_V1_UNSPECIFIED = 0;
  INTERACTION_TYPE_V1_COMMENT = 1;       // A textual comment on content. Payload: CommentPayloadV1
  INTERACTION_TYPE_V1_REACTION = 2;      // A predefined reaction (e.g., like, heart). Payload: ReactionPayloadV1
  INTERACTION_TYPE_V1_SHARE = 3;         // Sharing/reposting content. Payload: SharePayloadV1
  INTERACTION_TYPE_V1_FLAG = 4;          // Reporting content. Payload: FlagPayloadV1
  INTERACTION_TYPE_V1_TIP = 5;           // Sending a direct monetary tip. Payload: TipPayloadV1
  INTERACTION_TYPE_V1_VOTE = 6;          // Generic vote (e.g., on polls, governance). Payload: VotePayloadV1
  INTERACTION_TYPE_V1_USER_RELATION = 7; // E.g., Follow, Block. Payload: UserRelationPayloadV1
}

// Example Specific Payloads (define these according to application needs)
message CommentPayloadV1 {
  string text = 1; // Validation: Max length (e.g., 4096 chars), sanitization.
  // Optional: reference to content object if comment itself is rich content
  string comment_as_content_id = 2; // [(validate.rules).string.pattern = "^[a-f0-9]{64}$"]; // Example if ContentID is SHA256 hex
}

message ReactionPayloadV1 {
  string reaction_type_id = 1; // e.g., "like", "love", "dislike". Validation: Must be from a predefined set.
}

message SharePayloadV1 {
  // Optional: commentary accompanying the share.
  string commentary = 1; // Validation: Max length.
  // Optional: platform where it was shared (if off-EchoNet and recorded).
  string target_platform = 2;
}

message FlagPayloadV1 {
  string reason_category = 1; // e.g., "spam", "offensive", "copyright". Validation: Must be from a predefined set.
  string justification_notes = 2; // Validation: Max length.
}

message TipPayloadV1 {
  string asset_id = 1; // e.g., "ECHO_TOKEN" or other supported asset identifiers.
  string amount_str = 2; // String representation of amount to handle large numbers / precision. Validation: Must be positive numeric.
  string memo = 3; // Optional memo for the tip.
}


// NexusInteractionRecordV1 represents a generic user interaction.
message NexusInteractionRecordV1 {
  // Unique identifier for this interaction.
  // Typically a hash of (target_content_id (if_any) + actor_did + interaction_type + timestamp + hash(payload)).
  // Validation: Required, must be a valid hash.
  string interaction_id = 1;

  // Optional: DID of the user performing the interaction.
  // Can be empty for anonymous interactions if allowed by policy for certain types.
  // Validation: If present, must be a valid did:echonet format.
  string actor_did = 2; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Type of the interaction.
  // Validation: Required, must be a valid InteractionTypeV1 enum value.
  InteractionTypeV1 interaction_type = 3;

  // Optional: Identifier of the content being interacted with.
  // Not all interactions target content (e.g., user follow).
  // Validation: If present, must be a valid ContentID.
  string target_content_id = 4; // [(validate.rules).string.pattern = "^[a-f0-9]{64}$"]; // Example

  // Optional: Identifier of another user being interacted with (e.g., for a follow).
  // Validation: If present, must be a valid did:echonet format.
  string target_user_did = 5; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Timestamp when the interaction was recorded by the client/network.
  // Validation: Required. Must be a valid UTC timestamp.
  google.protobuf.Timestamp timestamp = 6;

  // Specific payload for the interaction type. Use google.protobuf.Any to wrap specific payload messages.
  // This allows for type safety and extensibility.
  // Example: For a COMMENT, this would contain a CommentPayloadV1.
  // Validation: Payload must match interaction_type and pass its specific validation.
  google.protobuf.Any payload = 7;

  // Optional: Identifier of the parent interaction, for threading (e.g., replies to comments).
  // Validation: If present, must be a valid InteractionID.
  string parent_interaction_id = 8; // [(validate.rules).string.pattern = "^[a-f0-9]{64}$"]; // Example

  // Version of this interaction record structure.
  uint32 version = 9; // Default to 1 for this version
}
```

*   **Canonicalization Notes:** `interaction_id` generation depends on canonical serialization of its constituent parts, including the `payload`. Protobuf binary for `NexusInteractionRecordV1` itself is preferred. For Canonical JSON, ensure `payload` (if `Any`) is correctly represented (usually with an `@type` field and the payload itself).
*   **Cross-references:** Interactions are DLI events validated by Witnesses. `actor_did` links to `NexusUserObjectV1`. `target_content_id` links to `NexusContentObjectV1`. `InteractionID` can be part of a `WitnessProofV1`.

### 3.4. `WitnessProofV1`

*   **Structure Name & Version:** `WitnessProofV1`
*   **Purpose:** Represents the collective validation of a DLI event (like content creation or an interaction) by a committee of Witnesses. It serves as an immutable receipt of network consensus on the event's validity and timestamp. (Refinement of `PoWReceipt`).
*   **Field Definitions:**

```protobuf
syntax = "proto3";

package digisocialblock.dli.core;

import "google/protobuf/timestamp.proto";

// Enum defining the type of DLI event being proven.
enum DLIEventTypeV1 {
  DLI_EVENT_TYPE_V1_UNSPECIFIED = 0;
  DLI_EVENT_TYPE_V1_CONTENT_PUBLISHED = 1;
  DLI_EVENT_TYPE_V1_INTERACTION_RECORDED = 2;
  DLI_EVENT_TYPE_V1_USER_OBJECT_CREATED = 3;
  DLI_EVENT_TYPE_V1_USER_OBJECT_UPDATED = 4; // e.g., reputation update, status flag change
  DLI_EVENT_TYPE_V1_GOVERNANCE_VOTE_CAST = 5;
  // Potentially other system-level events
}

// WitnessProofV1 is the record of validation by a Witness committee.
message WitnessProofV1 {
  // Identifier of the event being validated (e.g., ContentID, InteractionID, UserDID for user updates).
  // Validation: Required. Format depends on event_type.
  string event_id = 1;

  // Type of the DLI event that was validated.
  // Validation: Required, must be a valid DLIEventTypeV1 enum value.
  DLIEventTypeV1 event_type = 2;

  // Hash of the event data that was validated by the witnesses.
  // This ensures that the proof is tied to a specific version/state of the event object.
  // Validation: Required, must be a valid hash.
  bytes event_data_hash = 3;

  // Timestamp assigned by the Witness committee when consensus on validation was reached.
  // This is the official "network timestamp" for the event.
  // Validation: Required. Must be a valid UTC timestamp.
  google.protobuf.Timestamp network_timestamp = 4;

  // Identifier of the Witness committee responsible for this proof.
  // Could be a hash of the participating witness DIDs or a sequential ID.
  // Validation: Required.
  string witness_committee_id = 5;

  // List of DIDs of the Witnesses who attested to the event's validity.
  // These DIDs must correspond to registered and active Witnesses.
  // Validation: Required, non-empty. Each DID must be a valid did:echonet format.
  repeated string attesting_witness_dids = 6; // [(validate.rules).repeated.min_items = 1];

  // Aggregated cryptographic signature from the attesting Witnesses.
  // The specific signature scheme (e.g., BLS multi-signature) is defined by network protocol.
  // Validation: Required. Must be a valid signature verifiable against the attesting_witness_dids' public keys.
  bytes aggregated_signature = 7;

  // Optional: Additional data specific to the proof or event type.
  // e.g., For PoE, this might include the computed quality score or parameters used.
  // For content, it might include a reference to a content moderation log if applicable.
  // Use google.protobuf.Any for type safety if specific structures are defined.
  bytes specific_proof_data = 8; // Consider google.protobuf.Any if structure is needed

  // Version of this witness proof structure.
  uint32 version = 9; // Default to 1 for this version
}
```

*   **Canonicalization Notes:** The `aggregated_signature` is created over a canonical representation of the core proof details (e.g., `event_id`, `event_type`, `event_data_hash`, `network_timestamp`, `witness_committee_id`, `attesting_witness_dids`). Adherence to strict serialization rules for this pre-image is paramount. Protobuf binary for `WitnessProofV1` itself is preferred for storage/transmission.
*   **Cross-references:** This structure is fundamental to DLI integrity. `event_id` links to various other DLI objects. `attesting_witness_dids` link to `NexusUserObjectV1` of Witnesses. These proofs are stored and are discoverable, providing auditable validation for all key DLI events.

---
This completes the initial draft of the technical specifications for these core DLI data structures.
