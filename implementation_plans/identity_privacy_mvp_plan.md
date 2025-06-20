# EchoNet - Identity & Privacy MVP Implementation Plan

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Identity & Privacy MVP - Goals & Scope

### 1.1. Goals
The primary goal of the Identity & Privacy Minimal Viable Product (MVP) is to implement foundational self-sovereign identity (`did:echonet`) and user data consent functionalities within the EchoNet MVDLI (Minimal Viable DLI) core. This MVP will enable users to control their digital identities and basic data sharing permissions from an early stage.

The Identity & Privacy MVP must demonstrate:
*   Local generation of a `did:echonet` key pair and DID.
*   Construction and registration of a basic `NexusDIDDocumentV1` on the MVDLI.
*   Resolution of a `NexusDIDDocumentV1` from the MVDLI.
*   Creation of a minimal `NexusUserObjectV1` linked to the DID.
*   Granting of a basic data consent via a `NexusConsentRecordV1`.
*   Revocation of that consent.
*   A conceptual mechanism for a service to query the status of a consent.

### 1.2. Scope
**In Scope for Identity & Privacy MVP:**
*   Client-side library functions for `did:echonet` key generation, DID string derivation, `NexusDIDDocumentV1` construction (as per `tech_specs/did_echonet_method.md`), and data signing/verification.
*   DLI event for "DID Registration":
    *   Includes the `NexusDIDDocumentV1`.
    *   Validated by MVDLI PoW (basic schema and signature checks).
    *   `NexusDIDDocumentV1` stored on MVDLI DDS.
    *   DID resolvable via MVDLI Discovery (e.g., `did -> ContentID_of_DID_Doc` in DHT).
*   Client-side DID resolution mechanism.
*   Creation of a minimal `NexusUserObjectV1` (e.g., containing the DID, registration timestamp, and a reference to the DID Document, or vice-versa) stored on MVDLI DDS when a DID is registered.
*   DLI events for "ConsentGrant" and "ConsentRevoke":
    *   Includes a signed `NexusConsentRecordV1` (as per `tech_specs/data_consent_protocol.md`).
    *   Validated by MVDLI PoW (basic schema, signature, valid DIDs).
    *   `NexusConsentRecordV1` stored on MVDLI DDS.
*   Rudimentary "Consent Registry": A conceptual DLI-native mechanism (e.g., using DHT records or a simple on-DLI state map) to query the current status (`GRANTED`, `REVOKED`, `EXPIRED`) of a consent by `grantor_did`, `grantee_did`, and `data_category_ref`.

**Non-Goals for Identity & Privacy MVP:**
*   Complex DID key rotation or recovery mechanisms.
*   Advanced DID Document features (e.g., extensive service endpoints, complex `alsoKnownAs` relationships).
*   Full Verifiable Credential (VC) issuance or verification.
*   Sophisticated consent query language or complex policy enforcement AI.
*   Granular consent over specific data items (`specific_data_pointers` in `NexusConsentRecordV1` might be implemented but complex querying for it deferred).
*   Automated expiry of consent records (query logic should check expiry, but no active deletion process for MVP).
*   Detailed UI/UX for managing DIDs and consents (focus on CLI or programmatic access for MVP).
*   Full implementation of DID Update and Deactivate operations (conceptual outline only).

## 2. Key Implementation Tasks & Prioritization for Identity & Privacy MVP

These tasks build upon the MVDLI core functionalities.

*   **Task 2.1: `did:echonet` Library/SDK Functions (Client-Side)**
    *   **Description:** Create a core library (e.g., in Go, callable by CLI) for:
        *   Generating Ed25519 key pairs.
        *   Deriving the `did:echonet:<method-specific-identifier>` from a public key as per `tech_specs/did_echonet_method.md`.
        *   Constructing a basic `NexusDIDDocumentV1` protobuf message, including initial `verificationMethod` (using `publicKeyMultibase` format) and `authentication` entries.
        *   Signing arbitrary byte payloads using the Ed25519 private key.
        *   Verifying signatures given a public key.
    *   **Inputs:** `tech_specs/did_echonet_method.md`, `tech_specs/dli_core_data_structures.md` (for `NexusDIDDocumentV1` structure).
    *   **Priority:** **High**.
*   **Task 2.2: DID Registration on MVDLI**
    *   **Description:**
        *   Client-side: Implement logic to create a "DID Registration" DLI event. This event will contain the `NexusDIDDocumentV1` (from Task 2.1) and be signed by the initial authentication key defined within that document.
        *   MVDLI Node (PoW Witnesses): Extend MVDLI PoW validation to recognize and perform basic checks on "DID Registration" events (e.g., valid signature, well-formed DID Document).
        *   MVDLI Node (DDS Integration): Upon successful PoW validation, ensure the `NexusDIDDocumentV1` is stored on the MVDLI's rudimentary DDS, returning/deriving its `ContentID`.
        *   MVDLI Node (Discovery Integration): Publish the mapping `did:echonet:<identifier> -> ContentID_of_DID_Doc` to the MVDLI's rudimentary DHT.
    *   **Inputs:** `tech_specs/did_echonet_method.md`, MVDLI PoW/DDS/Discovery components.
    *   **Priority:** **High**.
*   **Task 2.3: DID Resolution via MVDLI**
    *   **Description:**
        *   Client-side: Implement a resolver function that takes a `did:echonet` string, queries the MVDLI DHT for the `ContentID` of its `NexusDIDDocumentV1`, and then retrieves the document from the MVDLI DDS.
    *   **Inputs:** MVDLI Discovery/DDS components.
    *   **Priority:** **High**.
*   **Task 2.4: Basic `NexusUserObjectV1` Creation (linked to DID)**
    *   **Description:**
        *   As part of the "DID Registration" DLI event processing (Task 2.2), create a minimal `NexusUserObjectV1`.
        *   This object will include the `user_did`, `registration_timestamp` (from `WitnessProofV1` of the registration event), and a reference like `did_document_ref` (the `ContentID` of the `NexusDIDDocumentV1`). The `nexus_user_object_ref` in the `NexusDIDDocumentV1` should also be populated with the `ContentID` of this `NexusUserObjectV1`.
        *   Store this `NexusUserObjectV1` on the MVDLI DDS.
    *   **Inputs:** `tech_specs/dli_core_data_structures.md` (for `NexusUserObjectV1`), MVDLI PoW/DDS components.
    *   **Priority:** **Medium**.
*   **Task 2.5: Data Consent Grant/Revoke DLI Events (Client-Side & MVDLI Basic Validation)**
    *   **Description:**
        *   Client-side: Implement functions to construct a `NexusConsentRecordV1` for granting consent (status `GRANTED`) and for revoking consent (status `REVOKED`), as per `tech_specs/data_consent_protocol.md`. This includes signing the relevant fields with the grantor's DID key.
        *   Client-side: Implement logic to submit these as "ConsentGrant" or "ConsentRevoke" DLI events.
        *   MVDLI Node (PoW Witnesses): Extend MVDLI PoW validation to recognize these event types and perform basic checks (valid signatures, valid DIDs, recognized `data_category_ref` - perhaps a fixed list for MVP).
    *   **Inputs:** `tech_specs/data_consent_protocol.md`, `tech_specs/did_echonet_method.md` (for DIDs).
    *   **Priority:** **Medium**.
*   **Task 2.6: Basic Consent Registry (MVDLI - Conceptual)**
    *   **Description:**
        *   MVDLI Node (DDS Integration): Upon successful PoW validation of consent events, store the `NexusConsentRecordV1` on the MVDLI DDS.
        *   MVDLI Node (Discovery/State): Implement a rudimentary mechanism to make the current status of consent queryable. This could be achieved by:
            *   Storing a key like `hash(grantor_did, grantee_did, data_category_ref) -> current_status_and_expiry_and_ContentID_of_record` in the MVDLI DHT.
            *   Or, if the MVDLI supports a simple DLI-native state map, updating it there.
        *   Client-side: Implement a basic `QueryConsent` function that uses this rudimentary registry.
    *   **Inputs:** `tech_specs/data_consent_protocol.md`, MVDLI PoW/DDS/Discovery components.
    *   **Priority:** **Medium**.
*   **Task 2.7: Simple DID Update/Deactivate (Conceptual Outline - Lower Priority for MVP)**
    *   **Description:** Document the intended flow for DID Update (submit new DID Doc version, signed by old key, update DHT) and Deactivate (submit deactivation event, signed, tombstone in DHT) as per `tech_specs/did_echonet_method.md`. Full implementation is deferred post-MVP.
    *   **Inputs:** `tech_specs/did_echonet_method.md`.
    *   **Priority:** **Low** (for MVP implementation, high for conceptual completeness).

## 3. Dependencies on MVDLI Core (Module 1)

The successful implementation of the Identity & Privacy MVP heavily relies on the prior or concurrent completion of core MVDLI functionalities:
*   **Functioning P2P Network:** For event gossip and direct node communication (from MVDLI Task 3).
*   **Event Gossip Mechanism:** A basic pub/sub system for propagating DID and Consent DLI events (from MVDLI Task 3).
*   **Rudimentary PoW Validation:** The MVDLI's simplified PoW mechanism must be able to process and validate new DLI event types for DIDs and Consents (from MVDLI Task 5).
*   **Basic DDS Storage:** The MVDLI's DDS must be able to store and retrieve `NexusDIDDocumentV1` and `NexusConsentRecordV1` objects by their `ContentID`s (from MVDLI Task 6).
*   **Basic Discovery (DHT):** The MVDLI's DHT must support `PUT` and `GET` operations for resolving DIDs to DID Document `ContentID`s and for the conceptual Consent Registry lookups (from MVDLI Task 7).
*   **Core Data Structures & Canonicalization:** Base data structures and hashing/signing utilities from MVDLI Task 2 must be available.

## 4. Development & Testing Approach for Identity & Privacy MVP

*   **Extend MVDLI Test Client/CLI:** Add commands to the MVDLI CLI tool (from MVDLI Task 8) to:
    *   Generate and display a new `did:echonet` DID and its initial (conceptual) document.
    *   Register a DID with the MVDLI.
    *   Resolve a DID and display its document.
    *   Grant a consent for a predefined category.
    *   Query the status of that consent.
    *   Revoke the consent.
*   **Unit Tests:** Comprehensive unit tests for all `did:echonet` library functions (key generation, DID derivation, document construction, signing, verification) and for `NexusConsentRecordV1` creation and signing.
*   **Integration Tests:** Within the MVDLI simulated network environment, test the end-to-end flows for DID registration, resolution, consent granting, querying, and revocation. Verify state changes in the rudimentary DDS and Discovery/Consent Registry.

## 5. Next Steps After Identity & Privacy MVP

Upon successful completion and testing of the Identity & Privacy MVP:
1.  **Full DID Update and Deactivate Implementation:** Implement the complete lifecycle operations for DIDs.
2.  **More Robust Consent Registry:** Design and implement a more scalable and efficient Consent Registry, potentially with more advanced query capabilities.
3.  **Refine Key Management:** Improve client-side key management, potentially integrating with hardware keystores for mobile (as per `tech_specs/mobile_node_specifications.md`).
4.  **Begin Verifiable Credential (VC) Exploration:** Investigate how `did:echonet` can be used for issuing and verifying VCs.
5.  **Integration with Other Modules:** Start integrating these identity and consent mechanisms with other developing EchoNet modules, such as Content Creation (associating content with DIDs), user profiles, and any features requiring data access consent.
6.  **Security and Privacy Hardening:** Conduct thorough reviews and implement more robust security and privacy protection measures.

This plan provides a clear path for building the initial layer of user-controlled identity and data consent in EchoNet, working in tandem with the MVDLI core development.
