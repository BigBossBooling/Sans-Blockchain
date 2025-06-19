# DigiSocialBlock - `did:echonet` Method Specification

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Introduction to `did:echonet`

### 1.1. Purpose
The `did:echonet` Decentralized Identifier (DID) method provides a cryptographically verifiable, self-sovereign identity layer for users and entities within the EchoNet (also referred to as DigiSocialBlock) DLI ecosystem. It allows participants to create and manage their digital identities without reliance on centralized authorities, fostering user control and data ownership.

### 1.2. Alignment with W3C DID Core
The `did:echonet` method is designed to be conformant with the W3C DID Core Specification 1.0 and related W3C standards. This ensures interoperability and a common understanding of DID structure and operations.

## 2. `did:echonet` Method Syntax

The generic DID syntax is `did:method-name:method-specific-identifier`.
For EchoNet, the syntax is:

**`did:echonet:<method-specific-identifier>`**

### 2.1. Definition of `<method-specific-identifier>`
The `<method-specific-identifier>` for `did:echonet` is designed to be globally unique, collision-resistant, and cryptographically derived from the initial public key used to create the DID. This provides an inherent link between the identifier and the controlling key pair.

*   **Generation Process:**
    1.  Generate an initial cryptographic key pair (e.g., Ed25519, see Section 3). Let the public key be `pk_initial`.
    2.  Take the raw bytes of `pk_initial`.
    3.  Compute `hash_output = SHA-256(pk_initial_bytes)`.
    4.  The `<method-specific-identifier>` is the **Base58BTC encoding** of the first 16 bytes (128 bits) of `hash_output`. This provides a reasonably short, URL-friendly, and globally unique identifier. (Alternative: full hash, but shorter is often preferred for DIDs).

*   **Example (Conceptual):**
    `pk_initial_bytes = <raw bytes of an Ed25519 public key>`
    `hash_output = SHA256(pk_initial_bytes)`
    `identifier_bytes = hash_output[0:15]` (first 16 bytes)
    `method_specific_identifier = Base58BTCEncode(identifier_bytes)`
    Resulting DID: `did:echonet:2Pfnk26MSHefb4YLB6i3n7Z1` (Example identifier)

*   **Justification:**
    *   **Global Uniqueness:** SHA-256 hash of a public key provides high collision resistance.
    *   **Decentralization:** No central registry needed to check for identifier availability during creation.
    *   **Implicit Key Binding (Initial):** While the DID Document is the source of truth for current keys, deriving the identifier from the initial key provides a root of trust.

## 3. Cryptographic Suite

The `did:echonet` method primarily relies on widely adopted and secure cryptographic algorithms.

*   **Key Types for Verification Methods:**
    *   **`Ed25519VerificationKey2020` (Primary):** Ed25519 keys are used for creating digital signatures and verifying authenticity. Public keys are typically represented in Multibase format (e.g., starting with `z` for Base58BTC encoded raw public key bytes).
    *   **`X25519KeyAgreementKey2020` (Optional):** X25519 keys derived from Ed25519 keys (or generated separately) can be included for key agreement protocols, enabling encrypted communication or data exchange.
*   **Signature Algorithm:**
    *   **EdDSA** using the **Ed25519** curve (as specified in RFC 8032).
*   **Hashing Algorithm (for DID identifier generation and general data integrity):**
    *   **SHA-256** (as specified in FIPS PUB 180-4).

These choices align with `echonet_v3_cryptographic_correctness_review.md`.

## 4. DID Document (`NexusDIDDocumentV1`)

The `did:echonet` DID Document describes the DID subject, including its cryptographic keys, authentication methods, and service endpoints. It is structured based on the W3C DID Core data model.

*   **Structure (Defined using Protocol Buffers v3):**
    The `@context` field, while not explicitly in Protobuf, is implicitly `["https://www.w3.org/ns/did/v1", "https://ns.echonet.org/did/v1"]` (the second URL is an example for EchoNet specific extensions, if any, when represented in JSON-LD).

```protobuf
syntax = "proto3";

package digisocialblock.dli.identity;

import "google/protobuf/timestamp.proto";

// NexusDIDDocumentV1 represents the DID Document for a did:echonet DID.
// It aligns with the W3C DID Core data model.
message NexusDIDDocumentV1 {
  // The DID string itself.
  // Validation: Required, must be a valid did:echonet URI.
  string id = 1;

  // Optional: List of DIDs that are controllers of this DID.
  // If not present, the controller is the DID subject itself (this 'id' field).
  repeated string controller = 2;

  // Optional: Previous version ID, for tracking update history.
  // Could be the ContentID of the previous version of this DID Document.
  string previous_version_id = 3;

  // List of verification methods associated with the DID.
  // These define cryptographic public keys or other mechanisms to verify proofs.
  repeated VerificationMethod verification_method = 4;

  // Verification methods specifically designated for authentication.
  // References 'id' from one of the VerificationMethod entries.
  repeated string authentication = 5;

  // Verification methods specifically designated for creating assertions (e.g., Verifiable Credentials).
  // References 'id' from one of the VerificationMethod entries.
  repeated string assertion_method = 6;

  // Verification methods specifically designated for key agreement (encryption).
  // References 'id' from one of the VerificationMethod entries.
  repeated string key_agreement = 7;

  // Optional: Service endpoints associated with the DID.
  // e.g., pointers to user profiles, social media services, communication addresses.
  repeated Service service = 8;

  // Timestamp of when this version of the DID Document was created on the DLI.
  // Set by the network (from WitnessProofV1).
  google.protobuf.Timestamp created = 9;

  // Timestamp of when this version of the DID Document was last updated on the DLI.
  // Set by the network (from WitnessProofV1).
  google.protobuf.Timestamp updated = 10;

  // Version identifier for this specific DID Document instance.
  // Could be a sequential number, a timestamp, or the ContentID of this document itself.
  string version_id = 11;

  // Reference to the associated NexusUserObjectV1 on DDS.
  // This links the DID to its on-DLI reputation, status flags, etc.
  // Stored as the ContentID of the NexusUserObjectV1.
  // Validation: Must be a valid ContentID string.
  string nexus_user_object_ref = 12;
}

// VerificationMethod defines a public key or other verification mechanism.
message VerificationMethod {
  // URI identifying the verification method, typically DID#key-id.
  // e.g., "did:echonet:2Pfnk26MSHefb4YLB6i3n7Z1#key-1"
  // Validation: Required. Must be a valid URI fragment relative to the DID id.
  string id = 1;

  // Type of the verification method.
  // e.g., "Ed25519VerificationKey2020", "X25519KeyAgreementKey2020"
  // Validation: Required. Must be a registered verification method type.
  string type = 2;

  // DID of the controller of this verification method.
  // Usually the DID subject itself.
  // Validation: Required. Must be a valid DID string.
  string controller = 3;

  // Public key material, encoded in Multibase format (e.g., Base58BTC for 'z' prefix).
  // e.g., "z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH"
  // Validation: Required. Must be a valid public key of the specified 'type'.
  string public_key_multibase = 4;
}

// Service defines a service endpoint associated with a DID.
message Service {
  // URI identifying the service endpoint, typically DID#service-id.
  // e.g., "did:echonet:2Pfnk26MSHefb4YLB6i3n7Z1#profile"
  // Validation: Required. Must be a valid URI fragment relative to the DID id.
  string id = 1;

  // Type of the service.
  // e.g., "EchoNetUserProfile", "EncryptedMessagingService"
  // Validation: Required.
  string type = 2;

  // The service endpoint URI or other identifier.
  // e.g., "https://echonet.social/u/2Pfnk26MSHefb4YLB6i3n7Z1/profile.json", "contentID://<hash_of_profile_doc>"
  // Validation: Required. Must be a valid URI.
  string service_endpoint = 3;
}

```
*Note: `Authentication`, `AssertionMethod`, and `KeyAgreement` in `NexusDIDDocumentV1` are `repeated string` fields. These strings directly reference the `id` of a `VerificationMethod` entry (e.g., "did:echonet:xyz#key-1"). This is a common pattern in DID Documents.*

*   **Relationship to `NexusUserObjectV1`:**
    *   The `NexusDIDDocumentV1` is primarily for describing verification material and service endpoints, adhering to the W3C standard.
    *   The `NexusUserObjectV1` (defined in `tech_specs/dli_core_data_structures.md`) stores EchoNet-specific DLI state about the user, such as `reputation_score`, `status_flags` (e.g., isWitness), and a map of `public_key_entries` that *could* mirror some keys from the DID Document for direct DLI use.
    *   The `nexus_user_object_ref` field in `NexusDIDDocumentV1` provides a direct link (e.g., via `ContentID`) to the version of the `NexusUserObjectV1` associated with this DID Document version. This allows resolvers to fetch the user's DLI-specific attributes.
    *   Alternatively, critical keys from `NexusUserObjectV1.public_key_entries` might be duplicated in `NexusDIDDocumentV1.verification_method` for broader interoperability. The design choice here is that `NexusDIDDocumentV1` is the primary source for verification methods, and `NexusUserObjectV1` might cache/use specific keys for its internal DLI operations.

## 5. CRUD Operations (Create, Read, Update, Deactivate) & DLI Integration

All DID lifecycle operations are DLI events, validated by EchoNet Witnesses and recorded immutably.

### 5.1. Create (Register)
1.  **Key Generation:** User's client application generates one or more cryptographic key pairs (e.g., an initial Ed25519 pair for authentication and assertion).
2.  **DID Calculation:** The `<method-specific-identifier>` is calculated from the initial primary public key as per Section 2.1. The full `did:echonet:<identifier>` is formed.
3.  **DID Document Construction:** The client constructs the initial `NexusDIDDocumentV1`.
    *   `id` is set to the new DID.
    *   `verificationMethod` includes the initial public key(s).
    *   `authentication` and `assertionMethod` reference the ID of the appropriate verification method(s).
    *   `created` and `updated` are initially unset (will be set by network upon validation).
    *   `version_id` can be an initial value (e.g., "1" or a hash of the initial document).
    *   A corresponding `NexusUserObjectV1` is also prepared, including relevant public keys.
4.  **Submission Event:**
    *   The user's client creates a "DID Registration Event" (a DLI transaction). This event includes:
        *   The proposed `NexusDIDDocumentV1`.
        *   The initial `NexusUserObjectV1` (or its core data).
    *   The event MUST be signed by the private key corresponding to a public key listed in the `authentication` section of the submitted `NexusDIDDocumentV1`.
5.  **Validation & Recording:**
    *   The event is gossiped and picked up by EchoNet Witnesses.
    *   Witnesses validate:
        *   Correctness of the DID calculation against the public key.
        *   Format and consistency of `NexusDIDDocumentV1` and `NexusUserObjectV1`.
        *   Validity of the signature on the registration event.
    *   If valid, a `WitnessProofV1` is generated. The `network_timestamp` from this proof populates `created` and `updated` in the DID Document.
    *   The `NexusDIDDocumentV1` and `NexusUserObjectV1` are stored on DDS, each receiving a `ContentID`. The `nexus_user_object_ref` in the DID Doc is set to the `ContentID` of the `NexusUserObjectV1`. The `version_id` of the DID Doc could be its own `ContentID`.
6.  **Publication to Discovery System:**
    *   The mapping `did:echonet:<identifier> -> ContentID_of_NexusDIDDocumentV1` is published to the Discovery Protocol (e.g., DHT or a dedicated DLI-native DID Registry contract/ledger).

### 5.2. Read (Resolve)
1.  **Query Discovery System:** A resolver queries the Discovery System (DHT/Registry) with the `did:echonet:<identifier>`.
2.  **Retrieve Pointer:** The Discovery System returns the `ContentID` of the latest valid `NexusDIDDocumentV1` associated with that DID.
3.  **Fetch DID Document:** The resolver retrieves the `NexusDIDDocumentV1` from DDS using its `ContentID`.
4.  **Verification (Optional but Recommended):** The resolver should verify the integrity of the DID Document (e.g., if its `ContentID` matches what was expected, or if the document itself contains internal consistency checks).

### 5.3. Update
1.  **Construct New DID Document:** The DID controller (user) creates a new version of the `NexusDIDDocumentV1`. Changes might include adding/removing/rotating keys, updating service endpoints. The `previous_version_id` field SHOULD be set to the `version_id` (or `ContentID`) of the document being updated.
2.  **Submission Event:**
    *   Client creates a "DID Update Event" including the new `NexusDIDDocumentV1`.
    *   This event MUST be signed by a private key whose public key is listed in an `authentication` method (or a dedicated `capabilityInvocation` method if defined for updates) of the *currently valid* version of the DID Document being updated. This proves control.
3.  **Validation & Recording:**
    *   Witnesses validate the update event, including the signature against the old DID Document.
    *   If valid, a `WitnessProofV1` is generated. The `network_timestamp` from this proof populates `updated` in the new DID Document (and `created` if this versioning scheme treats each version as a new creation time for that version).
    *   The new `NexusDIDDocumentV1` is stored on DDS, getting a new `ContentID`. This `ContentID` becomes its `version_id`.
4.  **Update Discovery System:** The mapping in the Discovery System for `did:echonet:<identifier>` is updated to point to the `ContentID` of the new `NexusDIDDocumentV1`.

### 5.4. Deactivate (Revoke - Conceptual)
Deactivation makes a DID permanently unusable for future operations.
1.  **Construct Deactivation Event:** The DID controller creates a "DID Deactivation Event".
    *   This event specifies the DID to be deactivated.
    *   It MUST be signed by a key authorized for deactivation (typically an `authentication` key or a specific `capabilityInvocation` key) in the last valid version of the DID Document.
2.  **Validation & Recording:**
    *   Witnesses validate the deactivation event and signature.
    *   If valid, a `WitnessProofV1` is generated.
3.  **Update Discovery System:**
    *   The entry for `did:echonet:<identifier>` in the Discovery System is marked as "deactivated" or replaced with a pointer to a "tombstone" record (a minimal DID Document indicating deactivation and the deactivation time).
    *   No further updates to this DID are permitted.

## 6. Security & Privacy Considerations

*   **User Key Management:** Users are solely responsible for the security of their private keys. Loss of keys authorized for control means loss of control over the DID. EchoNet client software MUST provide secure key generation and storage (e.g., using hardware-backed keystores as per `tech_specs/mobile_node_specifications.md`).
*   **DID Document Privacy:** DID Documents are generally considered public information necessary for others to verify signatures and discover service endpoints. Sensitive personal data should NOT be stored directly in the DID Document. It should be stored off-DLI, potentially encrypted, with service endpoints in the DID Document pointing to how to access it (with user consent).
*   **Unauthorized Operations:** Requiring updates and deactivations to be signed by keys authorized in the current DID Document version prevents unauthorized changes.
*   **Discovery System Security:** The integrity and censorship-resistance of the Discovery System (DHT/Registry) are crucial for reliable DID resolution. This is addressed in `discovery_protocol_refined.md`.
*   **Rotation of Keys:** The update mechanism allows users to rotate their keys regularly for enhanced security.

## 7. Interoperability

*   **W3C Compliance:** Adherence to DID Core helps ensure `did:echonet` can be understood by generic DID resolvers and libraries.
*   **Verifiable Credentials (VCs):** `did:echonet` can be used as the `issuer` and `holder` identifier in Verifiable Credentials, enabling participation in broader digital trust ecosystems. (Future scope for detailed VC integration).
*   **Service Endpoint Standards:** Using common service `type` definitions can improve interoperability for discovering and interacting with associated services.

---
This specification defines the `did:echonet` method, providing a robust foundation for self-sovereign identity within the EchoNet ecosystem. It integrates with DLI components like Witnesses, DDS, and the Discovery System for its lifecycle management.
