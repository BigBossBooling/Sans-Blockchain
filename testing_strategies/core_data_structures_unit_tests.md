# DigiSocialBlock - Core Data Structures Unit Testing Strategy

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Introduction

### 1.1. Purpose
This document outlines the unit testing strategy for the core Data Ledger Inspired (DLI) Go data structures within the EchoNet (also referred to as DigiSocialBlock) project. These structures are generated from Protocol Buffer (`.proto`) definitions, primarily `echonet_v3_core.proto`. The strategy focuses on verifying:
1.  Correctness of Protobuf serialization and deserialization.
2.  Thorough validation logic implemented in the `Validate() error` method for each structure, as specified in `tech_specs/data_structure_validation_logic_spec.md`.
3.  Correct functioning of any custom helper methods defined in `tech_specs/go_struct_helper_methods_conceptual.md`.

This strategy aims to ensure the reliability, integrity, and consistency of these foundational data types.

### 1.2. Guiding Principles
*   **Thoroughness:** Aim for comprehensive test coverage, especially for validation logic, including success paths, all defined error conditions, and edge cases.
*   **Isolation:** Unit tests should test data structures and their methods in isolation, mocking external dependencies where necessary (though minimal for these types).
*   **Automation:** All unit tests will be automated and integrated into the CI/CD pipeline.
*   **Clarity:** Test cases should be well-named, easy to understand, and clearly document the specific behavior or rule they are verifying.
*   **TDD Preference:** Development of `Validate()` methods and helper functions should ideally follow Test-Driven Development principles.

## 2. Unit Testing Goals for Data Structures

*   Verify correct and idempotent Protobuf serialization (binary marshaling/unmarshaling) and, where applicable, JSON marshaling/unmarshaling for debugging.
*   Verify the correct behavior of all custom helper methods defined for the Go structs.
*   Exhaustively test all validation paths within each struct's `Validate()` method, ensuring:
    *   Valid instances pass validation (return `nil`).
    *   Each specific validation rule correctly identifies invalid data.
    *   The precise error type (as defined in `echonet_v3_error_handling_strategy.md`) is returned for each distinct validation failure.
    *   Recursive validation of nested messages is performed correctly, and errors are propagated.
*   Achieve and maintain very high unit test code coverage for the packages containing these core data structures and their associated logic.

## 3. General Testing Approach & Environment

*   **Language & Framework:** Go's standard `testing` package will be used. Libraries like `testify/assert` and `testify/require` will be used for clear and concise assertions.
*   **Table-Driven Tests:** This will be the preferred method for testing `Validate()` methods and other functions with multiple input variations and expected outcomes. Each table entry will represent a distinct test case (e.g., a specific way a field can be invalid).
*   **Test Data Generation:**
    *   **Valid Instances:** Helper functions (factories) will be created to produce valid, fully populated instances of each core struct. These can be used as a baseline for creating invalid instances.
    *   **Invalid Instances:** For each validation rule, test cases will systematically create instances that violate that specific rule while keeping other fields valid (as much as possible) to isolate the test.
*   **Error Verification:** Tests for `Validate()` methods must check not only that an error is returned but also that the **specific type** of error (e.g., `ErrDataValidationFieldRequired`, `ErrDataValidationInvalidFormat`) and relevant error details (like the field name) are correct.

## 4. Test Case Categories for `Validate()` Methods

The following outlines test case categories for the `Validate()` method of each core data structure defined in `echonet_v3_core.proto` and detailed in `tech_specs/data_structure_validation_logic_spec.md`.

*(Note: For brevity, only a few representative examples are shown per struct. The actual test suite must cover all validation rules specified in `tech_specs/data_structure_validation_logic_spec.md` for every field.)*

### 4.1. `NexusUserObjectV1.Validate()`
*   **Valid Instance:**
    *   Test with a fully valid `NexusUserObjectV1`; expect `nil` error.
*   **Field: `user_did` (`string`)**
    *   Missing: `user_did` is empty; expect `ErrDataValidationFieldRequired(field="user_did")`.
    *   Invalid Format: `user_did` is "did:invalid:123"; expect `ErrDataValidationInvalidFormat(field="user_did", ...)`.
*   **Field: `registration_timestamp` (`google.protobuf.Timestamp`)**
    *   Missing: `registration_timestamp` is `nil`; expect `ErrDataValidationFieldRequired(field="registration_timestamp")`.
    *   Invalid Value: `registration_timestamp.Seconds <= 0`; expect `ErrDataValidationInvalidValue(field="registration_timestamp", ...)`.
*   **Field: `public_key_entries` (`map<string, bytes>`)**
    *   Invalid Key: Map key is empty; expect `ErrDataValidationInvalidFormat(field="public_key_entries.key", ...)`.
    *   Invalid Value: Map value (bytes) is empty or invalid length for key type; expect `ErrDataValidationInvalidFormat(field="public_key_entries.value", ...)`.
*   **Field: `version` (`uint32`)**
    *   Invalid Value: `version` is 0; expect `ErrDataValidationInvalidValue(field="version", ...)`.

### 4.2. `CoreContentMetadataV1.Validate()`
*   **Valid Instance:**
    *   Test with a fully valid `CoreContentMetadataV1`; expect `nil` error.
*   **Field: `title` (`string`)**
    *   Too Long: `title` exceeds max length; expect `ErrDataValidationTooLong(field="title", ...)`.
*   **Field: `tags` (`repeated string`)**
    *   Too Many: Number of tags > max allowed; expect `ErrDataValidationTooManyElements(field="tags", ...)`.
    *   Invalid Tag: A tag is empty or too long; expect `ErrDataValidationInvalidFormat(field="tags[i]", ...)`.
*   **Field: `client_asserted_timestamp` (`google.protobuf.Timestamp`)**
    *   Missing: `client_asserted_timestamp` is `nil`; expect `ErrDataValidationFieldRequired(field="client_asserted_timestamp")`.
    *   Invalid Value: Timestamp is non-positive or out of plausible range; expect `ErrDataValidationInvalidValue` or `ErrDataValidationOutOfRange`.

### 4.3. `NexusContentObjectV1.Validate()`
*   **Valid Instance:**
    *   Test with a fully valid `NexusContentObjectV1` (including valid nested `CoreContentMetadataV1` conceptually, though `core_metadata_hash` is validated here); expect `nil` error.
*   **Field: `content_id` (`string`)**
    *   Missing: `content_id` is empty; expect `ErrDataValidationFieldRequired(field="content_id")`.
    *   Invalid Format: `content_id` is not a valid SHA256 hex; expect `ErrDataValidationInvalidFormat(field="content_id", ...)`.
*   **Field: `creator_did` (`string`)**
    *   (As per `NexusUserObjectV1.user_did` tests)
*   **Field: `content_type` (`ContentTypeV1` enum)**
    *   Unspecified: `content_type` is `CONTENT_TYPE_V1_UNSPECIFIED`; expect `ErrDataValidationInvalidValue(field="content_type", ...)`.
    *   Invalid Integer: `content_type` is an integer not corresponding to any defined enum value; expect `ErrDataValidationInvalidEnumValue(field="content_type")`.
*   **Field: `core_metadata_hash` (`bytes`)**
    *   Missing: `core_metadata_hash` is empty; expect `ErrDataValidationFieldRequired(field="core_metadata_hash")`.
    *   Invalid Length: Length is not 32 bytes (for SHA-256); expect `ErrDataValidationInvalidLength(field="core_metadata_hash", ...)`.
*   **(Recursive Validation):**
    *   If `CoreContentMetadataV1` was directly nested (it's not, its hash is), one would test that `core_metadata.Validate()` is called. (Not directly applicable here, but principle stands for other nested messages).

### 4.4. Interaction Payloads (e.g., `CommentPayloadV1.Validate()`)
*   **Valid Instance:** Test with valid payload; expect `nil`.
*   **Field: `CommentPayloadV1.text` (`string`)**
    *   Missing: `text` is empty; expect `ErrDataValidationFieldRequired(field="text")`.
    *   Too Long: `text` exceeds max length; expect `ErrDataValidationTooLong(field="text", ...)`.
*   **Field: `ReactionPayloadV1.reaction_type_id` (`string`)**
    *   Missing: `reaction_type_id` empty; expect `ErrDataValidationFieldRequired(field="reaction_type_id")`.
    *   Invalid Value: `reaction_type_id` not in predefined set; expect `ErrDataValidationInvalidValue(field="reaction_type_id", ...)`.
*   *(Similar specific tests for `SharePayloadV1`, `FlagPayloadV1`, `TipPayloadV1`, `UserRelationPayloadV1` based on their rules).*

### 4.5. `NexusInteractionRecordV1.Validate()`
*   **Valid Instance:** Test with valid record and valid, unmarshaled specific payload; expect `nil`.
*   **Field: `interaction_id` (`string`)**
    *   (Similar to `NexusContentObjectV1.content_id` tests)
*   **Field: `actor_did` (`string`)**
    *   (Similar to `NexusUserObjectV1.user_did` tests)
*   **Field: `interaction_type` (`InteractionTypeV1` enum)**
    *   Unspecified or invalid integer value checks.
*   **Field: `payload` (`google.protobuf.Any`)**
    *   Missing: `payload` is `nil` when `interaction_type` requires it; expect `ErrDataValidationFieldRequired(field="payload")`.
    *   Type Mismatch: `payload.TypeUrl` does not correspond to expected message type for `interaction_type`; expect `ErrDataValidationInvalidType(field="payload", ...)`.
    *   Recursive Validation Failure: `payload` unmarshals to correct type, but the specific payload's `Validate()` method returns an error; expect that error to be propagated.
        *   E.g., `interaction_type` is `COMMENT`, payload unmarshals to `CommentPayloadV1`, but `CommentPayloadV1.text` is empty. The error from `CommentPayloadV1.Validate()` should be returned.

### 4.6. `WitnessProofV1.Validate()`
*   **Valid Instance:** Test with valid proof; expect `nil`.
*   **Field: `event_id` (`string`)**
    *   Missing or invalid format based on `event_type`.
*   **Field: `event_type` (`DLIEventTypeV1` enum)**
    *   Unspecified or invalid integer value checks.
*   **Field: `event_data_hash` (`bytes`)**
    *   Missing or invalid length.
*   **Field: `attesting_witness_dids` (`repeated string`)**
    *   Empty List: List is empty; expect `ErrDataValidationMinElementsRequired(field="attesting_witness_dids", ...)`.
    *   Invalid DID Format: An item in the list is not a valid DID.
    *   Duplicate DIDs: The list contains duplicate DIDs. Expect `ErrDataValidationDuplicateElements(field="attesting_witness_dids")`.
*   **Field: `aggregated_signature` (`bytes`)**
    *   Missing or invalid length based on signature scheme.

## 5. Test Case Categories for Serialization/Deserialization

*   **Round-Trip Testing (Binary):**
    *   For each core struct:
        1.  Create a valid, populated instance.
        2.  Marshal to binary bytes using `proto.Marshal()`.
        3.  Unmarshal the bytes back into a new instance using `proto.Unmarshal()`.
        4.  Compare the original and new instances for equality using `proto.Equal()`. Expect true.
*   **Round-Trip Testing (JSON - for debugging/logging helpers):**
    *   If `ToPrettyJSON()` and `FromJSON()` helpers are implemented:
        1.  Create a valid instance.
        2.  Call `ToPrettyJSON()`.
        3.  Call `FromJSON()` (or standard `protojson.Unmarshal`) on the output JSON.
        4.  Compare original and new instances using `proto.Equal()`. Expect true.
*   **Malformed/Corrupted Bytes Unmarshaling:**
    *   Attempt to unmarshal various forms of invalid/incomplete binary byte slices into each struct type.
    *   Expect appropriate errors from the `proto.Unmarshal()` function (typically `protoiface.UnmarshalOutput. มิได้`).
*   **Unknown Fields:** Test unmarshaling data that contains unknown fields (from a future version of the proto); ensure this succeeds if standard proto3 behavior is desired (unknown fields are ignored).

## 6. Test Case Categories for Custom Helper Methods

Refer to `tech_specs/go_struct_helper_methods_conceptual.md`. For each conceptual helper method implemented:
*   **Valid Inputs:** Test with typical, valid inputs and verify expected outputs.
*   **Edge Case Inputs:** Test with boundary values, empty slices/maps, nil pointers (if applicable for receivers or arguments) to ensure robust behavior.
*   **Error Conditions:** If the helper method can return an error, test scenarios that should trigger those errors and verify the correct error type/message.
*   **Example for `NexusInteractionRecordV1.AsCommentPayload()`:**
    *   Valid: `interaction_type` is `COMMENT`, `payload` contains `CommentPayloadV1`; expect correct `CommentPayloadV1` and `nil` error.
    *   Invalid Type: `interaction_type` is `REACTION`; expect `nil` payload and an appropriate error (e.g., `ErrDataValidationInvalidType`).
    *   Unmarshal Error: `payload` is malformed `Any` for `CommentPayloadV1`; expect `nil` payload and unmarshaling error.

## 7. Test Data Management

*   **Test Fixtures:** Create reusable test fixtures (instantiated structs with predefined data) for common valid and invalid scenarios to reduce boilerplate in tests.
*   **Factory Functions:** Develop helper functions (factories) in test packages to easily create instances of structs, e.g., `newValidNexusContentObject()`, `newContentObjectWithEmptyID()`, `newContentObjectWithInvalidDIDFormat()`. These factories can take parameters to vary specific fields for targeted testing.
*   **Descriptive Names:** Test functions and table entries should have clear, descriptive names indicating what they are testing (e.g., `TestNexusContentObjectV1_Validate_ContentID_Missing`, `TestNexusUserObjectV1_IsWitnessActive_ReturnsTrueWhenFlagSet`).

## 8. Mocking

*   For unit testing the data structures' `Validate()` methods and standard Protobuf serialization, mocking of external dependencies is generally **not required**. These tests should focus on the intrinsic properties and logic of the structs themselves.
*   If custom helper methods are added that *do* introduce external dependencies (e.g., a helper that resolves a DID online – though this should be avoided in data structure methods), those dependencies would need to be mocked for unit testing the helper.

## 9. Integration with CI/CD

*   All unit tests for core data structures MUST be part of the automated test suite executed by the CI/CD pipeline (as detailed in `roadmap/overall_mvp_implementation_roadmap.md`, Section 8.1).
*   Builds and Pull Request merges MUST fail if any of these unit tests fail.
*   Code coverage for these foundational packages will be generated by CI tools and monitored. A very high coverage target (e.g., >95%) is desirable for `Validate()` methods and critical helper functions due to their fundamental role in data integrity.

---
This unit testing strategy provides a comprehensive approach to verifying the foundational data structures of EchoNet, ensuring they are reliable, correct, and adhere to their specifications.
