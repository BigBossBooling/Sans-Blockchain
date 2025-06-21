# DigiSocialBlock - Dispute Resolution Protocol Specification

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Dispute Resolution Protocol Overview

### 1.1. Purpose
The DigiSocialBlock (also referred to as EchoNet) Dispute Resolution Protocol aims to provide a structured, transparent, and decentralized mechanism for addressing and resolving specific types of on-DLI (Distributed Ledger Inspired) conflicts that may arise between participants or between participants and system processes. Its goal is to foster fairness and provide recourse within the ecosystem.

### 1.2. Scope for MVP / Initial Specification
The initial scope for this protocol will focus on a limited set of clearly definable and verifiable on-DLI dispute types. Complex off-DLI disputes, subjective interpersonal conflicts, or issues requiring extensive real-world investigation are out of scope for this initial DLI-native protocol.

**Initial Focus Dispute Types:**
*   **Content Moderation Appeals:** Appealing a specific moderation action (e.g., a "flag" as per `NexusInteractionRecordV1` of type `FLAG`) applied to a piece of content (`NexusContentObjectV1`). The dispute is about the validity of the flag, not necessarily a direct judgment on the content itself by this protocol (though the outcome might inform further moderation actions).
*   **Proof-of-Engagement (PoP) Reward Inaccuracies:** Challenging the calculation or attribution of a specific PoP reward (from `tech_specs/poe_protocol.md`) if there's evidence of a clear, verifiable error in the DLI-native process.
*   **Simple Slashing Challenges (Conceptual - Post-MVP for full implementation):** If the MVDLI implements very simple, automated penalties (slashing) for clear protocol violations (e.g., provable PoW double-signing), a mechanism to challenge an incorrectly applied penalty with clear evidence could be considered. For initial MVP of dispute resolution, this might be out of scope until slashing is more mature.

### 1.3. Core Principles
*   **Verifiability:** All dispute filings, evidence submissions, and resolutions are recorded as immutable DLI events and data structures, making the entire process auditable.
*   **Defined Process:** Disputes follow a clear, predefined lifecycle.
*   **Impartiality (Targeted):** The resolution mechanism aims for impartiality by leveraging community consensus (via governance voting for MVP) or, in the future, reputation-based adjudicators or deterministic automated rules.
*   **Transparency:** The rules of the dispute process and the records of past disputes (excluding potentially private evidence details) are transparent.

## 2. Data Structures for Dispute Resolution

These structures are defined using Protocol Buffers v3 syntax.

```protobuf
syntax = "proto3";

package digisocialblock.dli.dispute;

import "google/protobuf/timestamp.proto";

// Enum defining the types of disputes that can be raised.
enum DisputeTypeV1 {
  DISPUTE_TYPE_V1_UNSPECIFIED = 0;
  // Appeal against a content flag or simple moderation action.
  // subject_ref would be the ContentID of the flagged content, or InteractionID of the flag.
  MODERATION_ACTION_APPEAL = 1;
  // Challenge regarding a specific PoP reward calculation or distribution.
  // subject_ref would be the InteractionID that was expected to generate reward, or a reward transaction ID.
  POP_REWARD_CHALLENGE = 2;
  // Challenge against an automated slashing event (more advanced, likely post-MVP for full handling).
  // subject_ref would be the transaction ID or event ID of the slashing.
  AUTOMATED_PENALTY_CHALLENGE = 3;
  // Other specific on-DLI dispute types can be added via governance.
}

// Enum defining the status of a dispute case.
enum DisputeStatusV1 {
  DISPUTE_STATUS_V1_UNSPECIFIED = 0;
  // Dispute has been opened and is awaiting further action/evidence.
  OPEN = 1;
  // Period during which parties can submit evidence.
  EVIDENCE_GATHERING = 2;
  // Evidence period is closed, case is awaiting adjudication (e.g., community vote setup).
  AWAITING_ADJUDICATION = 3;
  // Adjudication in progress (e.g., community voting period is active).
  ADJUDICATION_IN_PROGRESS = 4;
  // Resolution reached, outcome determined in favor of the complainant.
  RESOLVED_UPHELD = 5;
  // Resolution reached, outcome determined against the complainant.
  RESOLVED_DISMISSED = 6;
  // Resolution reached, but with a custom or mixed outcome.
  RESOLVED_OTHER = 7;
  // Dispute was escalated to a higher or different resolution mechanism (future scope).
  ESCALATED = 8;
  // Dispute was withdrawn by the complainant.
  WITHDRAWN = 9;
}

// Enum defining the possible outcomes of a dispute resolution.
enum ResolutionOutcomeV1 {
  RESOLUTION_OUTCOME_V1_UNSPECIFIED = 0;
  // The complainant's claim or appeal was validated.
  COMPLAINT_UPHELD = 1;
  // The complainant's claim or appeal was not validated.
  COMPLAINT_DISMISSED = 2;
  // A mediated, split, or other specific outcome. Details in ResolutionV1.resolution_details.
  OTHER = 3;
}

// EvidenceV1 represents a piece of evidence submitted for a dispute case.
message EvidenceV1 {
  // Unique identifier for this piece of evidence (e.g., hash of submitter_did + submission_timestamp + data_ref).
  // Validation: Required.
  string evidence_id = 1;

  // DID of the party submitting the evidence.
  // Validation: Required, must be a valid did:echonet URI.
  string submitter_did = 2;

  // Timestamp (UTC) of evidence submission.
  // Validation: Required.
  google.protobuf.Timestamp submission_timestamp = 3;

  // Text description or context for the evidence.
  // Validation: Required, max length (e.g., 4096 chars).
  string description = 4;

  // Reference to the evidence data itself.
  // Could be a ContentID of a NexusContentObjectV1 (e.g., for a document, image, log snippet stored on DDS),
  // or an IPFS hash, or other DLI-verifiable data pointer.
  // Validation: Required, must be a valid DLI data reference.
  string data_ref = 5;

  // Signature by the submitter_did over (case_id from DisputeCaseV1, description, data_ref, submission_timestamp).
  // Ensures authenticity and non-repudiation of evidence.
  // Validation: Required.
  bytes submitter_signature = 6;
}

// ResolutionV1 details the outcome of a dispute case.
message ResolutionV1 {
  // The final outcome of the dispute.
  // Validation: Required.
  ResolutionOutcomeV1 outcome = 1;

  // Text description of the judgment, rationale, and any actions to be taken.
  // Validation: Required if outcome is OTHER, otherwise optional but recommended. Max length.
  string resolution_details = 2;

  // Reference to the adjudicating entity or process.
  // Examples:
  // "CommunityVote:ReferendumProposalID:<proposal_id>" (for Option A)
  // "ArbitratorPanelID:<panel_id>" (for future Option B)
  // "AutomatedRuleID:<rule_id>" (for future Option C)
  // Validation: Required.
  string adjudicator_ref = 3;

  // Timestamp (UTC) when the resolution was finalized.
  // Validation: Required.
  google.protobuf.Timestamp resolution_timestamp = 4;

  // Optional: Signature by the adjudicating entity if applicable (e.g., lead arbitrator of a panel).
  // For community votes, the WitnessProofV1 of the governance vote finalization serves as attestation.
  bytes adjudicator_signature = 5;
}

// DisputeCaseV1 defines the structure for an on-DLI dispute.
message DisputeCaseV1 {
  // Unique identifier for this dispute case.
  // Generated by hashing key initial fields like (complainant_did, subject_ref, dispute_type, opened_timestamp).
  // Validation: Required, must be a valid hash string.
  string case_id = 1;

  // Type of the dispute.
  // Validation: Required.
  DisputeTypeV1 dispute_type = 2;

  // Reference to the DLI entity or event that is the subject of the dispute.
  // e.g., ContentID of flagged content, InteractionID of a disputed PoP reward event, TransactionID of a slashing.
  // Validation: Required, format depends on dispute_type.
  string subject_ref = 3;

  // DID of the user/entity raising the dispute.
  // Validation: Required, must be a valid did:echonet URI.
  string complainant_did = 4;

  // DID(s) of the primary party/parties on the other side of the dispute.
  // e.g., DID of the user who flagged content, or a system DID if disputing an automated process.
  // Can be empty if the dispute is solely against a system state/outcome not directly attributable.
  repeated string respondent_dids = 5;

  // Text description of the dispute provided by the complainant.
  // Validation: Required, max length (e.g., 10000 chars).
  string description = 6;

  // List of evidence submissions associated with this case.
  // This would typically store ContentIDs of EvidenceV1 objects stored on DDS.
  repeated string evidence_refs = 7; // List of ContentIDs for EvidenceV1 objects

  // Current status of the dispute.
  // Validation: Required.
  DisputeStatusV1 status = 8;

  // Timestamp (UTC) when the dispute was officially opened on the DLI.
  // Set from the WitnessProofV1 of the DisputeOpen event.
  // Validation: Required.
  google.protobuf.Timestamp opened_timestamp = 9;

  // Timestamp (UTC) when the dispute was closed (status changed to RESOLVED_* or WITHDRAWN).
  // Only populated if applicable. Set from the WitnessProofV1 of the closing event.
  google.protobuf.Timestamp closed_timestamp = 10;

  // Details of the resolution, if the case is resolved.
  // Populated when status changes to RESOLVED_*.
  ResolutionV1 resolution = 11;

  // Version of this DisputeCaseV1 schema (e.g., "1.0.0").
  string record_schema_version = 12;
}
```

## 3. Dispute Lifecycle & DLI Interactions

Disputes are managed as DLI-native state, with key lifecycle events validated by EchoNet Witnesses. A conceptual "Dispute Registry" (likely a DLI state index or a set of discoverable DDS records) tracks all dispute cases and their current status.

### 3.1. Open Dispute
1.  **User Action (Complainant):** Initiates a dispute via an EchoNet client application.
2.  **`DisputeCaseV1` Construction (Initial):** Client application helps the complainant construct an initial `DisputeCaseV1` object:
    *   `dispute_type`, `subject_ref`, `complainant_did`, `respondent_dids` (if known), `description` are filled.
    *   `status` is set to `OPEN`.
    *   Initial `evidence_refs` may be included if evidence is submitted at the time of opening.
    *   The `case_id` is typically generated by hashing a selection of these initial immutable fields to ensure uniqueness.
3.  **DLI Event Submission:** A "DisputeOpen" DLI event is created. The payload of this event is the partially filled `DisputeCaseV1` object. The event must be signed by the `complainant_did`.
    *   *(Conceptual: A fee might be required to submit this event to prevent spam - see Section 4).*
4.  **Witness Validation:** Witnesses validate the "DisputeOpen" event:
    *   Signature verification using `complainant_did`'s DID Document.
    *   Validity of `complainant_did` and any initial `respondent_dids`.
    *   Validity of `subject_ref` (e.g., does the `ContentID` exist?).
    *   Adherence to `DisputeCaseV1` schema and rules for `dispute_type`.
    *   Payment of dispute fee (if applicable).
5.  **Recording:**
    *   Upon successful validation (attested by a `WitnessProofV1`), the `opened_timestamp` field in `DisputeCaseV1` is set from the `WitnessProofV1.network_timestamp`.
    *   The `DisputeCaseV1` object is stored on DDS, receiving a `ContentID`. This `ContentID` can serve as a global reference for the case object itself.
    *   The Dispute Registry is updated with the `case_id` and its current status (`OPEN`) and a pointer to its `ContentID` on DDS.

### 3.2. Submit Evidence
1.  **User Action (Complainant, Respondent, or other permitted parties):** Submits evidence relevant to an open dispute case via an EchoNet client.
2.  **`EvidenceV1` Construction:** Client constructs an `EvidenceV1` object:
    *   `submitter_did`, `submission_timestamp` (client-asserted), `description`, `data_ref` (ContentID of actual evidence data stored on DDS).
    *   The `evidence_id` is generated (e.g., hash of key fields).
    *   The `submitter_signature` is created over `(case_id, description, data_ref, submission_timestamp)`.
3.  **DLI Event Submission:** An "EvidenceSubmit" DLI event is created, containing the `case_id` it pertains to and the `EvidenceV1` object. Signed by `submitter_did`.
4.  **Witness Validation:** Witnesses validate:
    *   Signature on the event.
    *   Existence and `OPEN` or `EVIDENCE_GATHERING` status of the referenced `case_id` in the Dispute Registry.
    *   Authority of `submitter_did` to submit evidence for this case (e.g., must be complainant or respondent, or during an open evidence period).
    *   Validity of `data_ref` (e.g., is it a valid `ContentID`?).
5.  **Recording:**
    *   The `EvidenceV1` object is stored on DDS (gets its own `ContentID`).
    *   The `DisputeCaseV1` object associated with `case_id` is updated by adding the `ContentID` of this new `EvidenceV1` object to its `evidence_refs` list. This update itself is a DLI state change validated by Witnesses. (Alternatively, the Dispute Registry might maintain the list of evidence for a case).
    *   The `DisputeCaseV1.status` might transition to `EVIDENCE_GATHERING` if it was `OPEN`.

### 3.3. Adjudication/Resolution Process
After a defined evidence gathering period (managed by governable parameters or rules within the `DisputeCaseV1`'s lifecycle logic), the dispute moves to adjudication.

*   **Initial Recommended Approach (Option A - Community Vote via Governance):**
    1.  **Transition to Adjudication:** The `DisputeCaseV1.status` transitions to `AWAITING_ADJUDICATION`. This could be time-based or triggered by a party.
    2.  **Governance Proposal Creation:** A formal governance proposal is created (as per `tech_specs/governance_framework_spec.md` and a future `governance_proposals_voting_spec.md`).
        *   **Proposal Content:** Summarizes the dispute (`case_id`, `subject_ref`, a neutral summary or links to complainant/respondent statements, links to all evidence `ContentID`s).
        *   **Voting Options:** Typically "Uphold Complaint," "Dismiss Complaint," possibly "Abstain."
    3.  **Voting Period:** The proposal enters a voting period. `did:echonet` holders vote with their PoP-derived reputation/voting power. The `DisputeCaseV1.status` changes to `ADJUDICATION_IN_PROGRESS`.
    4.  **Outcome Determination:** Once the voting period ends, the `VotingManagerV1` (from governance framework) determines the outcome based on quorum and threshold rules.
*   **Future Options (Post-MVP):**
    *   **Option B (Reputation-Selected Arbitrators):** A system where DIDs with high "adjudicator_reputation" can be empaneled to review and decide on cases. This requires a separate reputation stream and arbitrator selection/staking mechanisms.
    *   **Option C (Automated Rules):** For highly specific and deterministic dispute types (e.g., clear mathematical error in a PoP reward calculation based on on-DLI data), a DLI-native "contract" might automatically verify and apply a correction.

### 3.4. Close Dispute
1.  **Resolution Finalization:**
    *   For Option A (Community Vote): The `ProposalOutcomeFinalizedEvent` from the governance module serves as the trigger.
    *   The `ResolutionV1` structure is populated:
        *   `outcome`: Based on the vote (e.g., `COMPLAINT_UPHELD`, `COMPLAINT_DISMISSED`).
        *   `resolution_details`: Summary of the vote or link to the proposal.
        *   `adjudicator_ref`: "CommunityVote:ReferendumProposalID:<proposal_id_of_the_vote>".
        *   `resolution_timestamp`: Network timestamp from the finalization event.
2.  **DLI Event Submission:** A "DisputeClose" DLI event is generated by the governance module or an authorized entity. This event contains the `case_id` and the populated `ResolutionV1` data.
3.  **Witness Validation:** Witnesses validate this closing event.
4.  **Recording:**
    *   The `DisputeCaseV1` object on DDS is updated with the `ResolutionV1` data and its `status` is changed to `RESOLVED_UPHELD`, `RESOLVED_DISMISSED`, or `RESOLVED_OTHER`. The `closed_timestamp` is set.
    *   The Dispute Registry is updated to reflect the final status of the `case_id`.

## 4. Fees & Incentives (Conceptual for Initial Specification)

*   **Dispute Opening Fee:** A nominal fee (payable in ECHO, the native token) may be required to open a dispute.
    *   **Purpose:** To deter spam and frivolous claims.
    *   **Handling:** Fee could be burned, go to a community treasury, or partially refund if the dispute is upheld.
*   **Evidence Submission Fee:** Potentially a very small fee per evidence submission to manage storage costs.
*   **Incentives for Adjudicators (Future - for Option B/C):**
    *   If arbitrators are used, they would need to be incentivized (e.g., portion of dispute fees, allocation from treasury or PoP rewards).
*   **Incentives for Voters (Community Vote - Option A):**
    *   Participation in dispute resolution referenda could contribute positively to a user's governance participation aspect of their overall `reputationScore`, aligning with general governance incentives. Direct payment per vote is usually avoided to prevent vote buying.

## 5. Error Codes Specific to Dispute Resolution

(Aligned with `echonet_v3_error_handling_strategy.md`)
*   `DISPUTE_ERROR_UNSPECIFIED (0)`
*   `DISPUTE_INVALID_CASE_ID (1)`: Referenced `case_id` does not exist or is malformed.
*   `DISPUTE_INVALID_SUBJECT_REF (2)`: The `subject_ref` for the dispute is invalid or does not exist.
*   `DISPUTE_UNAUTHORIZED_ACTION (3)`: DID attempting an action (e.g., submitting evidence, closing dispute) is not authorized.
*   `DISPUTE_INVALID_STATUS_TRANSITION (4)`: Attempt to move dispute to an invalid next state.
*   `DISPUTE_EVIDENCE_PERIOD_CLOSED (5)`: Attempt to submit evidence after the allowed period.
*   `DISPUTE_FEE_REQUIRED (6)`: Required fee for opening/submitting was not paid.
*   `DISPUTE_ALREADY_RESOLVED (7)`: Action attempted on a dispute case that is already closed.
*   `DISPUTE_RESOLUTION_FAILED (8)`: Adjudication process failed to reach a conclusion (e.g., vote did not meet quorum).
*   `DISPUTE_MAX_EVIDENCE_REACHED (9)`: Too many evidence items submitted for a case.

## 6. Security, Fairness & Transparency

*   **DLI Verifiability:** All stages of the dispute (opening, evidence, resolution) are DLI events/state changes, validated by Witnesses and providing an immutable audit trail.
*   **Spam Prevention:** Dispute opening fees and potentially reputation prerequisites for complainants can deter spam.
*   **Evidence Integrity:** Evidence is signed by submitters and stored on DDS. `data_ref` points to immutable content.
*   **Fair Process (Target):**
    *   Clearly defined periods for evidence submission by all involved parties.
    *   Resolution based on transparent community voting (for MVP) using weighted voting power (aiming to reflect engagement and reputation).
    *   Clear criteria for what constitutes a valid dispute type.
*   **Transparency of Rules:** The dispute resolution protocol itself, including types of disputes handled and resolution mechanisms, is part of the overall EchoNet specification and subject to governance.

## 7. Protocol Versioning

The Dispute Resolution Protocol, including the `DisputeCaseV1` and related data structures, DLI event types, and state transition rules, will adhere to the versioning strategy outlined in `echonet_v3_versioning_strategy.md`. Upgrades to the protocol will be managed via EchoNet governance.

---
This initial specification for the Dispute Resolution Protocol focuses on establishing a foundational, verifiable process. It prioritizes community-based resolution via the existing governance framework for the MVP, with hooks for more specialized adjudication methods in future iterations.
