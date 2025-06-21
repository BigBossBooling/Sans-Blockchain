# DigiSocialBlock - Data Structure Validation Logic Specification

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Introduction

### 1.1. Purpose
This document specifies the exact validation logic that should be implemented in the `Validate() error` method for each core Data Ledger Inspired (DLI) Go struct generated from the Protocol Buffer (`.proto`) definitions (primarily from `echonet_v3_core.proto`). These validation methods are crucial for ensuring data integrity, enforcing format constraints, and maintaining the overall consistency of the EchoNet (also referred to as DigiSocialBlock) DLI.

The `Validate()` methods are intended to be called:
*   By client applications before submitting data to the DLI.
*   By DLI nodes (especially Witnesses) upon receiving DLI events or data objects to ensure their structural and semantic correctness before further processing or consensus.

### 1.2. Error Handling
All `Validate()` methods should adhere to the error handling strategy defined in `echonet_v3_error_handling_strategy.md`. Specifically:
*   If validation passes, the method shall return `nil`.
*   If validation fails, the method shall return a specific, typed error from the defined error hierarchy (e.g., `ErrDataValidationFieldRequired`, `ErrDataValidationInvalidFormat`).
*   Methods **MUST NOT** panic on validation failures.

### 1.3. General Order of Validation
Within a single `Validate()` method, checks should generally proceed in the following order for efficiency and clarity:
1.  Presence checks for required fields (including `nil` checks for message types).
2.  Basic format or value checks for individual fields (e.g., regex, length, range, valid enum value).
3.  Recursive calls to `Validate()` for nested message types.
4.  Conditional validation based on other field values.
5.  Simple cross-field consistency checks (if applicable at this level).

---

## 2. Validation Logic for Core Data Structures

The following sections detail the validation logic for each core Go struct generated from `echonet_v3_core.proto`. Placeholder regex patterns and specific length/range values are illustrative and should be finalized based on implementation needs and global constants.

### Structure: `NexusUserObjectV1.Validate()`

*   **`user_did` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="user_did")`
    *   Rule: Must be a valid `did:echonet` string matching the defined pattern (e.g., regex `^did:echonet:[a-zA-Z0-9._%-]+$`).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="user_did", details="Invalid DID format.")`
*   **`registration_timestamp` (`google.protobuf.Timestamp` message):**
    *   Rule: Must not be `nil`.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="registration_timestamp")`
    *   Rule: Must represent a valid, non-zero UTC timestamp. `seconds` part must be > 0.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="registration_timestamp", details="Timestamp must be positive.")`
    *   Rule: (Conceptual) Timestamp should not be unreasonably in the future or too far in the past (e.g., within a few minutes of current network time for new registrations). This might be context-dependent and handled by higher-level validation.
*   **`reputation_score` (`int64`):**
    *   Rule: No specific validation at this level if it's purely network-managed. If it can be part of an initial object, then perhaps "must be >= 0".
    *   ErrorIfFails: `ErrDataValidationOutOfRange(field="reputation_score", details="Reputation score cannot be negative.")` (If applicable)
*   **`status_flags` (`uint32`):**
    *   Rule: No specific validation other than being a valid `uint32`. Specific flag combinations might be validated at a higher level if necessary.
*   **`public_key_entries` (`map<string, bytes>`):**
    *   Rule: For each entry:
        *   Key (`string` "purpose"): Must not be empty. Max length (e.g., 64 chars).
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="public_key_entries.key", details="Key purpose must not be empty and within length limits.")`
        *   Value (`bytes` "public_key"): Must not be empty. Must be a valid public key of a supported type (e.g., Ed25519, X25519 - actual key validation is complex and might involve crypto libraries). Length check (e.g., 32 bytes for Ed25519).
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="public_key_entries.value", details="Public key bytes invalid or unsupported type/length.")`
*   **`version` (`uint32`):**
    *   Rule: Must be > 0 (e.g., starting from 1).
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="version", details="Version must be positive.")`
*   **`extended_profile_ref` (`string`):**
    *   Rule: If present and not empty, must be a valid URI or `ContentID` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="extended_profile_ref", details="Invalid URI or ContentID format.")`

---
### Structure: `CoreContentMetadataV1.Validate()`

*   **`title` (`string`):**
    *   Rule: If present, must not exceed maximum length (e.g., 256 chars). (Being present vs. empty might depend on content type rules set by higher layers).
    *   ErrorIfFails: `ErrDataValidationTooLong(field="title", max_len=256)`
*   **`description` (`string`):**
    *   Rule: If present, must not exceed maximum length (e.g., 1024 chars).
    *   ErrorIfFails: `ErrDataValidationTooLong(field="description", max_len=1024)`
*   **`tags` (`repeated string`):**
    *   Rule: If present, list must not exceed maximum count (e.g., 20 tags).
    *   ErrorIfFails: `ErrDataValidationTooManyElements(field="tags", max_elements=20)`
    *   Rule: Each tag in the list must not be empty and not exceed maximum length (e.g., 50 chars).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="tags[i]", details="Tag invalid format or length.")`
*   **`language` (`string`):**
    *   Rule: If present and not empty, must be a valid IETF language tag (e.g., regex validation for common patterns like `en-US`, `fr`).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="language", details="Invalid IETF language tag format.")`
*   **`custom_attributes` (`map<string, string>`):**
    *   Rule: If present, map must not exceed maximum number of entries (e.g., 10).
    *   ErrorIfFails: `ErrDataValidationTooManyElements(field="custom_attributes", max_elements=10)`
    *   Rule: For each entry:
        *   Key: Must not be empty, max length (e.g., 64 chars).
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="custom_attributes.key", details="Key invalid format or length.")`
        *   Value: Max length (e.g., 256 chars).
        *   ErrorIfFails: `ErrDataValidationTooLong(field="custom_attributes.value", max_len=256)`
*   **`client_asserted_timestamp` (`google.protobuf.Timestamp` message):**
    *   Rule: Must not be `nil`.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="client_asserted_timestamp")`
    *   Rule: Must represent a valid, non-zero UTC timestamp. `seconds` part must be > 0.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="client_asserted_timestamp", details="Timestamp must be positive.")`
    *   Rule: (Conceptual) Timestamp should not be unreasonably in the future (e.g., > 5 mins from current network time) or too far in the past (e.g., > 24 hours from current network time, unless specific content type allows). This is context-dependent.
    *   ErrorIfFails: `ErrDataValidationOutOfRange(field="client_asserted_timestamp", details="Timestamp is out of acceptable range.")`

---
### Structure: `NexusContentObjectV1.Validate()`

*   **`content_id` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="content_id")`
    *   Rule: Must be a valid SHA-256 hex string (64 lowercase hex characters).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="content_id", details="Invalid SHA256 hex format.")`
    *   Rule (Higher-level consistency, might be separate): `content_id` should match the hash derived from other fields as per `tech_specs/content_hashing_timestamping_specs.md`.
    *   ErrorIfFails: `ErrDataValidationConsistency(field="content_id", details="ContentID does not match derived hash.")`
*   **`creator_did` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="creator_did")`
    *   Rule: Must be a valid `did:echonet` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="creator_did", details="Invalid DID format.")`
*   **`content_type` (`ContentTypeV1` enum):**
    *   Rule: Must be a defined value from `ContentTypeV1` enum.
    *   ErrorIfFails: `ErrDataValidationInvalidEnumValue(field="content_type")`
    *   Rule: Must not be `CONTENT_TYPE_V1_UNSPECIFIED`.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="content_type", details="ContentType cannot be UNSPECIFIED.")`
*   **`core_metadata_hash` (`bytes`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="core_metadata_hash")`
    *   Rule: Must be of the correct length for the hash algorithm used (e.g., 32 bytes for SHA-256).
    *   ErrorIfFails: `ErrDataValidationInvalidLength(field="core_metadata_hash", expected_len=32)`
    *   Rule (Higher-level consistency): `core_metadata_hash` should match the hash of the `CoreContentMetadataV1` instance (if provided alongside for validation).
    *   ErrorIfFails: `ErrDataValidationConsistency(field="core_metadata_hash", details="core_metadata_hash does not match CoreContentMetadataV1.")`
*   **`body_hash` (`bytes`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="body_hash")`
    *   Rule: Must be of the correct length for the hash algorithm used (e.g., 32 bytes for SHA-256).
    *   ErrorIfFails: `ErrDataValidationInvalidLength(field="body_hash", expected_len=32)`
*   **`body_chunk_ids` (`repeated string`):**
    *   Rule: If `body_size_bytes` > 0 and content is chunked, this list should not be empty. (This logic might be too complex for basic Validate and belong to higher-level validation).
    *   Rule: Each `chunk_id` in the list must be a valid hash string (e.g., SHA-256 hex).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="body_chunk_ids[i]", details="Invalid chunk_id format.")`
*   **`body_size_bytes` (`int64`):**
    *   Rule: Must be >= 0.
    *   ErrorIfFails: `ErrDataValidationOutOfRange(field="body_size_bytes", details="Body size cannot be negative.")`
*   **`network_validated_timestamp` (`google.protobuf.Timestamp` message):**
    *   Rule: If not `nil` (i.e., if it has been set), must represent a valid, non-zero UTC timestamp.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="network_validated_timestamp", details="Timestamp must be positive if set.")`
*   **`dynamic_metadata_ref` (`string`):**
    *   Rule: If present and not empty, must be a valid URI or `ContentID` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="dynamic_metadata_ref", details="Invalid URI or ContentID format.")`
*   **`version` (`uint32`):**
    *   Rule: Must be > 0.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="version", details="Version must be positive.")`

---
### Structure: Interaction Payloads (`CommentPayloadV1`, `ReactionPayloadV1`, etc.)

These are validated in the context of `NexusInteractionRecordV1.payload` based on `interaction_type`.

*   **`CommentPayloadV1.Validate()`:**
    *   **`text` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="text")`
        *   Rule: Must not exceed maximum length (e.g., 4096 chars).
        *   ErrorIfFails: `ErrDataValidationTooLong(field="text", max_len=4096)`
    *   **`comment_as_content_id` (`string`):**
        *   Rule: If present and not empty, must be a valid SHA-256 hex string.
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="comment_as_content_id", details="Invalid ContentID format.")`
*   **`ReactionPayloadV1.Validate()`:**
    *   **`reaction_type_id` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="reaction_type_id")`
        *   Rule: Must be from a predefined set of reaction types (governed list).
        *   ErrorIfFails: `ErrDataValidationInvalidValue(field="reaction_type_id", details="Unrecognized reaction type.")`
*   **`SharePayloadV1.Validate()`:** (Fewer intrinsic rules, more contextual)
    *   **`commentary` (`string`):**
        *   Rule: If present, must not exceed maximum length (e.g., 1024 chars).
        *   ErrorIfFails: `ErrDataValidationTooLong(field="commentary", max_len=1024)`
*   **`FlagPayloadV1.Validate()`:**
    *   **`reason_category` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="reason_category")`
        *   Rule: Must be from a predefined set of flag reasons (governed list).
        *   ErrorIfFails: `ErrDataValidationInvalidValue(field="reason_category", details="Unrecognized flag reason.")`
    *   **`justification_notes` (`string`):**
        *   Rule: If present, must not exceed maximum length (e.g., 2048 chars).
        *   ErrorIfFails: `ErrDataValidationTooLong(field="justification_notes", max_len=2048)`
*   **`TipPayloadV1.Validate()`:**
    *   **`asset_id` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="asset_id")`
        *   Rule: Must be a recognized asset identifier on EchoNet (e.g., "ECHO_TOKEN").
        *   ErrorIfFails: `ErrDataValidationInvalidValue(field="asset_id", details="Unrecognized asset ID.")`
    *   **`amount_str` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="amount_str")`
        *   Rule: Must represent a positive numeric value. (Actual parsing and >0 check).
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="amount_str", details="Amount must be a positive number string.")`
    *   **`memo` (`string`):**
        *   Rule: If present, must not exceed maximum length (e.g., 256 chars).
        *   ErrorIfFails: `ErrDataValidationTooLong(field="memo", max_len=256)`
*   **`UserRelationPayloadV1.Validate()`:**
    *   **`relation_type` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="relation_type")`
        *   Rule: Must be from a predefined set (e.g., "FOLLOW", "BLOCK").
        *   ErrorIfFails: `ErrDataValidationInvalidValue(field="relation_type", details="Unrecognized relation type.")`
    *   **`target_user_did` (`string`):**
        *   Rule: Must not be empty.
        *   ErrorIfFails: `ErrDataValidationFieldRequired(field="target_user_did")`
        *   Rule: Must be a valid `did:echonet` string.
        *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="target_user_did", details="Invalid DID format.")`

---
### Structure: `NexusInteractionRecordV1.Validate()`

*   **`interaction_id` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="interaction_id")`
    *   Rule: Must be a valid SHA-256 hex string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="interaction_id", details="Invalid SHA256 hex format.")`
*   **`actor_did` (`string`):**
    *   Rule: Must not be empty (unless specific interaction types allow anonymous, which should be documented per type). For MVP, assume required.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="actor_did")`
    *   Rule: Must be a valid `did:echonet` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="actor_did", details="Invalid DID format.")`
*   **`interaction_type` (`InteractionTypeV1` enum):**
    *   Rule: Must be a defined value from `InteractionTypeV1` enum.
    *   ErrorIfFails: `ErrDataValidationInvalidEnumValue(field="interaction_type")`
    *   Rule: Must not be `INTERACTION_TYPE_V1_UNSPECIFIED`.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="interaction_type", details="InteractionType cannot be UNSPECIFIED.")`
*   **`target_content_id` (`string`):**
    *   Rule: If interaction type implies a content target (e.g., COMMENT, REACTION, FLAG on content), this must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="target_content_id", details="Required for this interaction type.")` (Conditional)
    *   Rule: If present and not empty, must be a valid SHA-256 hex string (`ContentID`).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="target_content_id", details="Invalid ContentID format.")`
*   **`target_user_did` (`string`):**
    *   Rule: If interaction type implies a user target (e.g., FOLLOW, BLOCK, if not handled in payload), this must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="target_user_did", details="Required for this interaction type.")` (Conditional)
    *   Rule: If present and not empty, must be a valid `did:echonet` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="target_user_did", details="Invalid DID format.")`
*   **`timestamp` (`google.protobuf.Timestamp` message):**
    *   Rule: Must not be `nil`.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="timestamp")`
    *   Rule: Must represent a valid, non-zero UTC timestamp.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="timestamp", details="Timestamp must be positive.")`
    *   Rule: (Conceptual) Timestamp should be recent and plausible.
    *   ErrorIfFails: `ErrDataValidationOutOfRange(field="timestamp", details="Timestamp is out of acceptable range.")`
*   **`payload` (`google.protobuf.Any` message):**
    *   Rule: Must not be `nil` if the `interaction_type` requires a payload (most do).
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="payload")` (Conditional)
    *   Rule: Must successfully unmarshal to the specific payload type indicated by `interaction_type` (e.g., if `COMMENT`, then `payload` must unmarshal to `CommentPayloadV1`).
    *   ErrorIfFails: `ErrDataValidationInvalidType(field="payload", details="Payload type does not match interaction_type.")`
    *   Rule: Call `Validate()` on the unmarshaled specific payload message and propagate its error if any.
    *   ErrorIfFails: (Error from specific payload's `Validate()` method)
*   **`parent_interaction_id` (`string`):**
    *   Rule: If present and not empty, must be a valid SHA-256 hex string (`InteractionID`).
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="parent_interaction_id", details="Invalid InteractionID format.")`
*   **`version` (`uint32`):**
    *   Rule: Must be > 0.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="version", details="Version must be positive.")`

---
### Structure: `WitnessProofV1.Validate()`

*   **`event_id` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="event_id")`
    *   Rule: Format depends on `event_type` (e.g., SHA256 hex for `ContentID`/`InteractionID`, DID string for user updates). Must match expected format.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="event_id", details="Format does not match event_type.")`
*   **`event_type` (`DLIEventTypeV1` enum):**
    *   Rule: Must be a defined value from `DLIEventTypeV1` enum.
    *   ErrorIfFails: `ErrDataValidationInvalidEnumValue(field="event_type")`
    *   Rule: Must not be `DLI_EVENT_TYPE_V1_UNSPECIFIED`.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="event_type", details="DLIEventType cannot be UNSPECIFIED.")`
*   **`event_data_hash` (`bytes`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="event_data_hash")`
    *   Rule: Must be of the correct length for the hash algorithm used (e.g., 32 bytes for SHA-256).
    *   ErrorIfFails: `ErrDataValidationInvalidLength(field="event_data_hash", expected_len=32)`
*   **`network_timestamp` (`google.protobuf.Timestamp` message):**
    *   Rule: Must not be `nil`.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="network_timestamp")`
    *   Rule: Must represent a valid, non-zero UTC timestamp.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="network_timestamp", details="Timestamp must be positive.")`
*   **`witness_committee_id` (`string`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="witness_committee_id")`
    *   Rule: Max length check (e.g., 128 chars).
    *   ErrorIfFails: `ErrDataValidationTooLong(field="witness_committee_id", max_len=128)`
*   **`attesting_witness_dids` (`repeated string`):**
    *   Rule: Must not be empty (minimum number of attestors, e.g., M from PoW spec).
    *   ErrorIfFails: `ErrDataValidationMinElementsRequired(field="attesting_witness_dids", min_elements=1)` // Min should be M
    *   Rule: Each DID in the list must be a valid `did:echonet` string.
    *   ErrorIfFails: `ErrDataValidationInvalidFormat(field="attesting_witness_dids[i]", details="Invalid DID format.")`
    *   Rule: DIDs in the list must be unique.
    *   ErrorIfFails: `ErrDataValidationDuplicateElements(field="attesting_witness_dids")`
*   **`aggregated_signature` (`bytes`):**
    *   Rule: Must not be empty.
    *   ErrorIfFails: `ErrDataValidationFieldRequired(field="aggregated_signature")`
    *   Rule: Length check if signature scheme has fixed length (e.g., BLS signature). (Actual signature verification is a separate cryptographic step, not part of basic `Validate()`).
    *   ErrorIfFails: `ErrDataValidationInvalidLength(field="aggregated_signature", details="Invalid signature length.")`
*   **`specific_proof_data` (`bytes`):**
    *   Rule: No specific validation at this level, as its structure is context-dependent (defined by `event_type`). Higher-level logic will parse and validate it. Max length check might be appropriate (e.g., 1KB).
    *   ErrorIfFails: `ErrDataValidationTooLong(field="specific_proof_data", max_len=1024)`
*   **`version` (`uint32`):**
    *   Rule: Must be > 0.
    *   ErrorIfFails: `ErrDataValidationInvalidValue(field="version", details="Version must be positive.")`

---
This document provides the initial set of validation rules. These will be refined and expanded as implementation progresses and specific constraints for each module are finalized. Each `Validate()` method should aim to be comprehensive for the structural integrity of the message it validates.
