# DigiSocialBlock - On-System Data Consent Protocol

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Data Consent Protocol Overview

### 1.1. Purpose
The DigiSocialBlock (also referred to as EchoNet) On-System Data Consent Protocol provides users with granular, explicit, verifiable, and revocable control over how their data is shared and used within the EchoNet ecosystem. This protocol is fundamental to user trust, data protection, and ethical data handling.

### 1.2. Scope
This protocol applies to data where a user (`grantor_did`) explicitly grants permissions to another entity (`grantee_did` - which could be another user, an application, a service, or a system module) for specific categories of data or data items. Examples of data requiring consent:
*   Sharing detailed personal profile information beyond the public DID Document (e.g., email, full name, location if stored).
*   Allowing specific applications or services to access user-generated content (e.g., private posts, direct messages) for purposes beyond basic rendering to the user.
*   Opting into collection and use of interaction data for analytics or personalization features.
*   Consenting to participate in targeted advertising programs (if EchoNet were to implement such features).
*   Granting access to specific `NexusContentObjectV1` instances that are otherwise access-restricted.

Publicly available data (e.g., public posts, the DID Document itself) generally does not require consent for viewing, but might for specific types of processing.

### 1.3. Core Principles
*   **Granularity:** Users should be able to grant consent for specific data categories or even individual data items, rather than broad, all-encompassing permissions.
*   **Explicitness:** Consent must be explicitly given through a clear user action. No default opt-ins for sensitive data sharing.
*   **User Revocability:** Users must be able to easily revoke consent at any time, and the system must honor such revocations promptly.
*   **Verifiability & Auditability:** Consent grants and revocations are recorded as immutable DLI events, providing an auditable trail.
*   **Transparency:** Users should be able to easily view and manage their active consent settings.
*   **Purpose Limitation:** Consent should ideally be tied to a specific purpose for which the data will be used by the grantee.

## 2. `NexusConsentRecordV1` Data Structure

This structure defines the record of a consent decision. It is stored on the DLI (specifically, on DDS after Witness validation) and its state is queryable via a "Consent Registry."

```protobuf
syntax = "proto3";

package digisocialblock.dli.consent;

import "google/protobuf/timestamp.proto";

// Enum defining the status of a consent record.
enum ConsentStatusV1 {
  CONSENT_STATUS_V1_UNSPECIFIED = 0; // Should not be used in persisted records
  GRANTED = 1;                       // Consent is actively granted
  REVOKED = 2;                       // Consent was previously granted but has been revoked
  EXPIRED = 3;                       // Consent was time-limited and has passed its expiry
}

// NexusConsentRecordV1 defines a single instance of user consent.
message NexusConsentRecordV1 {
  // Unique identifier for this consent record.
  // Typically a hash of (grantor_did, grantee_did, data_category_ref, granted_timestamp, terms_hash (if any)).
  // Validation: Required, must be a valid hash string (e.g., SHA2-256 hex).
  string consent_id = 1;

  // DID of the user granting consent.
  // Validation: Required, must be a valid did:echonet URI.
  string grantor_did = 2; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // DID of the entity/service being granted consent.
  // This could be a specific application DID, another user's DID, or a well-known system service DID
  // (e.g., "did:echonet:system:AnalyticsService", "did:echonet:system:TargetedAdsModule").
  // Validation: Required, must be a valid DID URI.
  string grantee_did = 3; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-:]+$"];

  // Reference to a defined data category that this consent applies to.
  // e.g., "profile_contact_details", "interaction_history_for_analytics", "personalized_feed_opt_in".
  // These categories are defined in the Data Categories Registry (see Section 3).
  // Validation: Required, must be a recognized data category identifier.
  string data_category_ref = 4;

  // Optional: Array of specific data pointers (e.g., ContentIDs of specific NexusContentObjectV1 items)
  // covered by this consent. If empty, consent applies to the entire data_category_ref.
  // Validation: Each entry, if present, must be a valid ContentID or other recognized data pointer.
  repeated string specific_data_pointers = 5;

  // Current status of this consent.
  // Validation: Required, must be a valid ConsentStatusV1 enum value.
  ConsentStatusV1 status = 6;

  // Timestamp (UTC) when the consent was initially granted.
  // Set by the client, verified for plausibility by Witnesses.
  // Validation: Required.
  google.protobuf.Timestamp granted_timestamp = 7;

  // Timestamp (UTC) when the consent was revoked.
  // Only populated if status is REVOKED. Set by client, verified by Witnesses.
  // Validation: Must be present if status is REVOKED; must be >= granted_timestamp.
  google.protobuf.Timestamp revoked_timestamp = 8;

  // Optional: Timestamp (UTC) when this consent automatically expires.
  // If not set, consent is indefinite until revoked.
  // Validation: If present, must be > granted_timestamp.
  google.protobuf.Timestamp expiry_timestamp = 9;

  // Optional: Cryptographic hash (e.g., SHA-256) of the specific terms or policy document version
  // under which this consent was granted. This provides an immutable link to the exact conditions.
  // Validation: If present, must be a valid hash string.
  string terms_hash = 10;

  // Digital signature by the grantor_did over the canonicalized core consent fields.
  // Pre-image for signature: (consent_id (or its constituent parts if consent_id is self-derived after signing), grantor_did, grantee_did, data_category_ref, specific_data_pointers (canonicalized list), status (at time of grant/revoke), granted_timestamp, revoked_timestamp (if revoking), expiry_timestamp, terms_hash).
  // The exact canonicalization must be strictly defined.
  // Validation: Required. Must be a valid signature verifiable with a key from grantor_did's DID Document.
  bytes grantor_signature = 11;

  // Version of this NexusConsentRecordV1 schema (e.g., "1.0.0").
  // For handling future upgrades to the record structure itself.
  string record_schema_version = 12;
}
```

**Canonicalization for `grantor_signature`:**
The pre-image for `grantor_signature` requires a strict canonical representation of the fields defining the consent's scope and state. For a grant, this would typically include: `grantor_did`, `grantee_did`, `data_category_ref`, canonicalized `specific_data_pointers`, `granted_timestamp`, `expiry_timestamp` (if any), and `terms_hash` (if any). For a revocation, it would include `consent_id` (of the record being revoked), `grantor_did`, `status=REVOKED`, and `revoked_timestamp`. This must follow rules similar to those in `tech_specs/content_hashing_timestamping_specs.md` (e.g., UTF-8 strings with null terminators, specific integer/timestamp formats, sorted & concatenated lists). The `consent_id` itself is often best generated *after* signing by hashing these core fields.

## 3. Data Categories Registry (Conceptual)

*   **Definition:** A well-defined, human-readable, and machine-interpretable registry of all data categories for which consent can be granted or revoked within EchoNet.
*   **Examples:**
    *   `profile_basic_public`: Publicly visible profile info (name, avatar, bio). Usually no consent needed beyond making profile public.
    *   `profile_contact_private`: Private contact details (email, phone) - requires explicit consent to share with specific grantees.
    *   `content_interaction_analytics_pseudonymized`: Allowing one's interaction data (likes, shares, comments - pseudonymized) to be used for global EchoNet analytics.
    *   `content_interaction_for_personalization_service_X`: Allowing a specific personalization service X to use interaction data.
    *   `direct_messaging_access_app_Y`: Allowing application Y to read/write direct messages.
*   **Management:**
    *   Initially defined as part of the core EchoNet protocol specification.
    *   New categories or modifications could be proposed and adopted via the EchoNet governance process (`echonet_v3_governance_model.md`).
    *   Each category should have a unique identifier string (`data_category_ref`) and a clear description of the data it covers and potential uses.

## 4. Consent Lifecycle & DLI Interactions

Consent records are DLI-managed state, with changes validated by Witnesses. A "Consent Registry" (a logical DLI component, likely a specialized state index or a collection of records made discoverable via the DHT) tracks the current status of consents.

### 4.1. Grant Consent
1.  **User Action:** User initiates consent grant via an EchoNet client application (e.g., checking a box in settings, approving a prompt from an app).
2.  **Record Construction:** Client application constructs a `NexusConsentRecordV1` with:
    *   `grantor_did`: User's DID.
    *   `grantee_did`: DID of the entity receiving consent.
    *   `data_category_ref`: The specific category.
    *   `specific_data_pointers` (if applicable).
    *   `status`: `GRANTED`.
    *   `granted_timestamp`: Current UTC timestamp.
    *   `expiry_timestamp` (if applicable).
    *   `terms_hash` (if applicable).
    *   `record_schema_version`.
3.  **Signing:** The client serializes the core fields (see canonicalization notes in Section 2) and requests a signature from the user's DID (via hardware keystore). The resulting `grantor_signature` is added to the record. The `consent_id` is then typically generated as a hash of these signed core fields to ensure its uniqueness and tie to the grant.
4.  **DLI Event Submission:** A "ConsentGrant" DLI event, containing the signed `NexusConsentRecordV1`, is submitted to the network.
5.  **Witness Validation:** Witnesses validate:
    *   Validity of `grantor_did` and `grantee_did`.
    *   Correctness of `grantor_signature`.
    *   Plausibility of `granted_timestamp` and `expiry_timestamp`.
    *   Recognition of `data_category_ref`.
    *   Format of `NexusConsentRecordV1`.
6.  **Recording:**
    *   Upon successful validation (via `WitnessProofV1`), the `NexusConsentRecordV1` is stored on DDS (achieving a `ContentID`).
    *   The Consent Registry is updated: an entry is created or updated for (`grantor_did`, `grantee_did`, `data_category_ref`, potentially `specific_data_pointers` hash) pointing to this `NexusConsentRecordV1`'s `ContentID` and reflecting its `GRANTED` status and `expiry_timestamp`.

### 4.2. Query Consent
Services or users needing to check consent status perform a query.
*   **RPC Definition (Conceptual):**
    ```protobuf
    service ConsentRegistryService {
      // Queries the current consent status for a specific context.
      rpc QueryConsent(QueryConsentRequest) returns (QueryConsentResponse);
    }

    message QueryConsentRequest {
      string grantor_did = 1;
      string grantee_did = 2;
      string data_category_ref = 3;
      repeated string specific_data_pointers_query = 4; // Optional: For checking consent on specific items.
    }

    message QueryConsentResponse {
      ConsentStatusV1 status = 1; // GRANTED, REVOKED, EXPIRED, or UNSPECIFIED (if no record)
      google.protobuf.Timestamp expiry_timestamp = 2; // If applicable and status is GRANTED
      string consent_record_content_id = 3; // ContentID of the latest relevant NexusConsentRecordV1
      string terms_hash = 4; // Optional
      google.protobuf.Timestamp granted_timestamp = 5;
      google.protobuf.Timestamp revoked_timestamp = 6;
    }
    ```
*   **Logic:** The Consent Registry implementation would look up the combination. It must consider `expiry_timestamp` to return `EXPIRED` if applicable, even if the stored record says `GRANTED`. It should return the most recent, authoritative record.

### 4.3. Revoke Consent
1.  **User Action:** User initiates consent revocation via an EchoNet client.
2.  **Revocation Message Construction:**
    *   Client identifies the `consent_id` of the grant to be revoked (or the combination of grantor, grantee, category).
    *   A new `NexusConsentRecordV1` can be created with `status = REVOKED`, `revoked_timestamp` set to current UTC, and referencing the original grant details (or a more lightweight `ConsentRevocationV1` message could be designed, but using `NexusConsentRecordV1` keeps it uniform).
    *   Alternatively, if `consent_id` is known, a simpler revocation message might only need `consent_id`, `grantor_did`, `status=REVOKED`, `revoked_timestamp`, and a new `grantor_signature`. For this spec, we'll assume updating the existing record's state or creating a new record that supersedes it. Let's assume creating a new record with status `REVOKED` that points to the same context (grantor, grantee, category) is cleaner for auditability.
3.  **Signing:** The user signs this revocation intent with their `grantor_did`.
4.  **DLI Event Submission:** A "ConsentRevoke" DLI event, containing the signed revocation instruction (e.g., the new `NexusConsentRecordV1` with status `REVOKED`), is submitted.
5.  **Witness Validation:** Witnesses validate the signature and the authority of the revoker for the specified consent.
6.  **Recording:**
    *   The new `NexusConsentRecordV1` (status `REVOKED`) is stored on DDS.
    *   The Consent Registry is updated for (`grantor_did`, `grantee_did`, `data_category_ref`) to point to this new record, reflecting the `REVOKED` status.

## 5. Consent Enforcement (Conceptual)

*   **Proactive Checks:** EchoNet modules, services, and third-party applications integrated with EchoNet are *obligated* to call `QueryConsent` before accessing or processing data covered by this protocol.
*   **Fail-Closed:** If consent is not `GRANTED`, or if the query fails, access should be denied or processing halted.
*   **Challenges:**
    *   Technical enforcement can be difficult for off-DLI data processing.
    *   Relies heavily on module/application compliance.
*   **Potential Consequences for Violations:**
    *   **DLI-Detectable:** If a service performs a DLI action that demonstrably requires consent it doesn't have (e.g., publishing data derived from private sources), this could potentially be challenged via DLI mechanisms.
    *   **Reputation Loss:** Services or applications found to violate consent can suffer reputation damage within the EchoNet ecosystem (`NexusUserObjectV1.reputation_score` for DID-identified services).
    *   **Community Governance:** The EchoNet governance process might establish policies and dispute resolution mechanisms for consent violations.
    *   **External Legal Frameworks:** GDPR, CCPA, etc., may apply irrespective of DLI mechanisms.

## 6. Security & Privacy Considerations

*   **Integrity & Non-Repudiation:** `grantor_signature` on `NexusConsentRecordV1` and DLI validation by Witnesses ensure that consent grants/revocations are authentic and cannot be repudiated by the grantor. The DLI provides an immutable audit trail.
*   **Privacy of Consent Data:** While the `NexusConsentRecordV1` itself (and its status in the Consent Registry) is verifiable and potentially public to the grantee and grantor (and system auditors), the actual user data it pertains to remains protected by its own separate access control and encryption mechanisms. The consent record is metadata *about* permission.
*   **Unauthorized Operations:** Only the `grantor_did` (or a DID delegated by them via DID Document capabilities) can grant or revoke consent for their data. This is enforced via signature verification.
*   **Specificity:** Clear `data_category_ref` and optional `specific_data_pointers` help prevent overly broad consent grants.
*   **Session Management:** Client applications should ensure that consent decisions are actively made by the user and not cached indefinitely without re-verification for sensitive operations.

## 7. Protocol Versioning

The Data Consent Protocol, including the `NexusConsentRecordV1` structure and the definitions in the Data Categories Registry, will adhere to the versioning strategy outlined in `echonet_v3_versioning_strategy.md`.
*   `NexusConsentRecordV1.record_schema_version` tracks the version of the record structure.
*   Changes to data categories or consent lifecycle rules will be managed as part of overall DLI protocol versioning and potentially subject to governance.

---
This specification for the On-System Data Consent Protocol aims to empower EchoNet users with meaningful control over their data, fostering a trustworthy and privacy-respecting digital environment.
