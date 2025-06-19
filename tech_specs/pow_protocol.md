# DigiSocialBlock PoW - Proof-of-Witness Protocol Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. PoW Protocol Overview

### 1.1. Purpose
The Proof-of-Witness (PoW) protocol is the lightweight, fast, and secure consensus mechanism for the DigiSocialBlock Distributed Ledger Inspired (DLI) system (also referred to as `EchoNet`). It provides a decentralized way to validate and timestamp events occurring on the DLI, such as content publication, user interactions, and other significant state changes, without the computational overhead of traditional Proof-of-Work blockchains.

### 1.2. Key Operations
*   **Event Validation:** Witnesses scrutinize submitted DLI events against defined validity rules.
*   **Attestation:** Eligible Witnesses cryptographically attest to the validity (or invalidity) of an event.
*   **`WitnessProofV1` Generation:** Aggregation of a sufficient number of attestations into a single, verifiable proof that an event has been accepted by the network.

### 1.3. Core Principles
*   **M-of-N Consensus:** An event is considered validated if `M` out of `N` selected Witnesses in a committee attest to its validity.
*   **Rotating Committees:** Witness committees for events are transient and selected pseudo-randomly (e.g., via Verifiable Random Function - VRF) to enhance security and distribute workload.
*   **Stake/Reputation for Eligibility:** Nodes must meet certain criteria (e.g., staked ECHO tokens, minimum reputation score as per `NexusUserObjectV1`) to be eligible for Witness selection, deterring malicious participation.
*   **Lightweight & Fast:** Designed for low latency in event confirmation, suitable for social engagement and content platforms.

## 2. Witness Eligibility & Registration Protocol

### 2.1. Eligibility Criteria
To become an active Witness, a node operator (represented by their `NexusUserObjectV1` and associated DID) must meet:
*   **Minimum Stake:** A predefined amount of ECHO tokens must be staked. This stake is subject to slashing. The amount is determined by governance.
*   **Minimum Reputation Score:** A minimum `reputation_score` on their `NexusUserObjectV1` (see `tech_specs/dli_core_data_structures.md`). This score is influenced by positive contributions (e.g., successful witnessing, PoE activity) and can be reduced by penalties.
*   **Technical Requirements:** Meet specified uptime requirements (e.g., >99% availability over a recent period, monitored by network health checks or peer attestations) and possess adequate computational resources (CPU, bandwidth for event processing and gossip).
*   **Registration:** Explicitly register their intent to be a Witness.

Verification of these criteria is done via on-DLI data: stake amounts are recorded in staking "contracts" or ledgers, reputation scores are part of the `NexusUserObjectV1`, and uptime might be assessed through peer monitoring services or self-reported attestations subject to challenges.

### 2.2. Registration & De-registration
*   **Registration Transaction/RPC:**
    *   A node operator submits a DLI transaction (e.g., `WitnessRegistrationRequestV1`) signed by their DID.
    *   Payload includes: `node_did`, `proof_of_stake_transaction_id`, `network_address_for_witnessing`, `public_keys_for_witnessing_signatures`.
    *   This transaction is validated by the DLI (e.g., by existing Witnesses). If valid, the `status_flags` in the node's `NexusUserObjectV1` is updated to include `IsWitnessCandidate`. After a probation period and further checks, it can be promoted to `IsWitnessActive`.
*   **De-registration Transaction/RPC:**
    *   A Witness can voluntarily de-register by submitting a `WitnessDeRegistrationRequestV1`.
    *   Stake is typically unlocked after a cool-down period to prevent "nothing-at-stake" attacks immediately after malicious behavior.
*   **Automated De-registration (due to inactivity/slashing):** If a Witness is slashed below the minimum stake or their reputation drops too low, or they are consistently unavailable, the DLI can automatically change their status from `IsWitnessActive`.

### 2.3. Active Witness Set Maintenance & Discovery
*   The set of all `NexusUserObjectV1` entries with the `IsWitnessActive` flag constitutes the current pool of eligible Witnesses.
*   This set (or a cryptographic commitment to it, like a Merkle root of active Witness DIDs) is maintained as part of the DLI state.
*   Nodes can discover active Witnesses by querying the DLI for this set or by subscribing to updates. This list is crucial for VRF threshold calculations and for verifying attestations.

## 3. Witness Committee Selection Protocol

### 3.1. Chosen Mechanism: VRF-based Self-Selection
For each DLI event requiring validation, a unique committee of `N` Witnesses is chosen from the active Witness pool. Selection is achieved via a Verifiable Random Function (VRF) where eligible Witnesses can self-select based on the VRF output.

### 3.2. Detailed Steps
1.  **Event Identifier (`event_digest`):** When a new DLI event `E` (e.g., a `NexusContentObjectV1` submission, `NexusInteractionRecordV1`) is created, a stable cryptographic hash of its core data is computed: `event_digest = hash(canonical_event_data)`.
2.  **Epoch Identifier (`epoch_id`):** A slowly changing epoch identifier (e.g., based on DLI block height ranges or time periods like 1 hour) is used to ensure committee rotation even for events that might have similar hashes or occur closely in time. `epoch_id` should be globally agreed upon.
3.  **Role Identifier (`role_id` - for future use):** For potential future scenarios where different roles within a committee might exist (e.g., "proposer", "validator", "aggregator"), a role identifier can be included. For now, assume a single role: "attester".
4.  **VRF Input:** Each active Witness `W` computes `vrf_input = event_digest || epoch_id || role_id`.
5.  **VRF Computation:** Witness `W` uses its secret VRF key `sk_W` to compute `vrf_hash = VRF_hash(sk_W, vrf_input)` and `vrf_proof = VRF_proof(sk_W, vrf_input)`.
6.  **Selection Threshold (`threshold_N`):**
    *   The network aims for a target committee size `N` (e.g., N=150).
    *   The `threshold_N` is dynamically calculated based on the total number of active Witnesses (`TotalActiveWitnesses`) such that approximately `N` Witnesses will have their `vrf_hash` fall below this threshold.
    *   `threshold_N = (N / TotalActiveWitnesses) * MAX_VRF_HASH_VALUE`.
    *   This calculation requires `TotalActiveWitnesses` to be a known DLI state parameter.
7.  **Self-Selection:** If `vrf_hash_W < threshold_N`, Witness `W` is selected for the committee for event `E` during `epoch_id`. `W` includes `vrf_hash_W` and `vrf_proof_W` in its attestations to prove its legitimate selection.

### 3.3. Parameters
*   **Target Committee Size (`N`):** e.g., 150. Governed parameter.
*   **Required Attestations (`M`):** e.g., `ceil(2/3 * N)`. An event is confirmed if at least `M` committee members attest to its validity. (e.g., for N=150, M=101). Governed parameter.

## 4. Event Validation & Attestation Protocol

### 4.1. Event Propagation
*   New DLI events submitted by users or other nodes (e.g., `NexusContentObjectV1` from an Origin Node, `NexusInteractionRecordV1` from a client) are gossiped through the EchoNet peer-to-peer network.
*   Witnesses (especially those selected for a committee for that event type/epoch) subscribe to relevant event streams or actively listen for new events.

### 4.2. Validation Checks (by each committee Witness)
Each selected Witness independently validates the received event. Examples:
*   **For `NexusContentObjectV1`:**
    *   Verify `creator_did` signature on the object (if applicable, or on a wrapper transaction).
    *   Check `content_id` correctly computed from `creator_did`, `client_asserted_timestamp` (in `CoreContentMetadataV1`), `core_metadata_hash`, `body_hash`.
    *   Validate `core_metadata_hash` against the provided `CoreContentMetadataV1` structure.
    *   Ensure `client_asserted_timestamp` is recent and not in the future.
    *   Check `contentType` is valid.
    *   Verify `body_hash` format (actual body chunk availability might be a separate DDS check).
    *   Check for compliance with content policies (e.g., no spam, size limits, based on `echonet_v3_input_validation_strategy.md`).
*   **For `NexusInteractionRecordV1`:**
    *   Verify `actor_did` signature (if applicable for the interaction type).
    *   Check `interaction_id` computation.
    *   Validate `target_content_id` or `target_user_did` exist (may require DLI lookup).
    *   Ensure `timestamp` is recent.
    *   Validate the `payload` based on `interaction_type` (e.g., comment length, valid reaction type).
*   **For `NexusUserObjectV1` (Registration/Update):**
    *   Verify DID format and ownership proofs for `public_key_entries`.
    *   Validate claims for stake or other prerequisites.
*   **General Checks:** Schema validation, signature verification, timestamp validity, consistency with DLI state, adherence to `echonet_v3_input_validation_strategy.md`.

### 4.3. Attestation Message Format (Protobuf3)

```protobuf
syntax = "proto3";

package digisocialblock.pow.protocol;

import "google/protobuf/timestamp.proto";
// Assuming DLIEventTypeV1 is imported from tech_specs/dli_core_data_structures.proto or similar

enum DLIEventTypeV1 { // Duplicated here for standalone clarity, ideally imported
  DLI_EVENT_TYPE_V1_UNSPECIFIED = 0;
  DLI_EVENT_TYPE_V1_CONTENT_PUBLISHED = 1;
  DLI_EVENT_TYPE_V1_INTERACTION_RECORDED = 2;
  DLI_EVENT_TYPE_V1_USER_OBJECT_CREATED = 3;
  DLI_EVENT_TYPE_V1_USER_OBJECT_UPDATED = 4;
  DLI_EVENT_TYPE_V1_GOVERNANCE_VOTE_CAST = 5;
  DLI_EVENT_TYPE_V1_DDS_POSR_PROOF = 6; // Attesting to validity of a PoSR proof itself
}

message WitnessAttestationV1 {
  // Hash digest of the event data being attested.
  bytes event_digest = 1;

  // Type of the DLI event.
  DLIEventTypeV1 event_type = 2;

  // True if the witness deems the event valid according to network rules.
  bool is_valid = 3;

  // DID of the attesting witness.
  string witness_did = 4; // [(validate.rules).string.pattern = "^did:echonet:[a-zA-Z0-9._%-]+$"];

  // Timestamp from the witness when validation was completed.
  google.protobuf.Timestamp witness_timestamp = 5;

  // Cryptographic signature by witness_did over (event_digest, event_type, is_valid, witness_timestamp, poe_quality_score, validation_error_code, vrf_hash).
  bytes witness_signature = 6;

  // Optional: PoE Quality Score, if the event is an interaction subject to PoE.
  int64 poe_quality_score = 7;

  // Optional: Error code string if isValid is false, explaining reason for invalidity.
  // References error codes from echonet_v3_error_handling_strategy.md or PoW specific ones.
  string validation_error_code = 8;

  // VRF hash proving this witness was selected for the committee.
  bytes vrf_hash = 9;

  // VRF proof corresponding to vrf_hash.
  bytes vrf_proof = 10;
}
```

### 4.4. Gossip of Attestations
*   Once a Witness generates its `WitnessAttestationV1`, it gossips this message to its peers, particularly other known members of the committee (if discoverable) or to a broader set of network nodes/aggregators.
*   Gossip can use standard pub/sub mechanisms over the EchoNet P2P layer.

## 5. `WitnessProofV1` Aggregation & Dissemination

### 5.1. Aggregation Logic
*   Any node on the network (including Witnesses, or dedicated aggregator services) can listen for `WitnessAttestationV1` messages for a given `event_digest`.
*   An aggregator collects attestations. To be considered for a `WitnessProofV1`:
    1.  The `WitnessAttestationV1` must be for the same `event_digest` and `event_type`.
    2.  The `witness_signature` must be valid against the `witness_did`'s public key.
    3.  The `vrf_hash` and `vrf_proof` must be valid for the `witness_did` and the event/epoch, and `vrf_hash` must be below `threshold_N`.
    4.  The `is_valid` flag must be true. (Aggregators only collect positive attestations for a valid proof).
*   Once `M` such valid, positive attestations are collected from distinct Witnesses, a `WitnessProofV1` can be constructed.

### 5.2. `WitnessProofV1` Structure (Consistency with `tech_specs/dli_core_data_structures.md`)
The `WitnessProofV1` structure is defined in `tech_specs/dli_core_data_structures.md`. Key fields populated as follows:
*   **`event_id`**: The `event_digest` (hash of the validated event data).
*   **`event_type`**: From the attestations.
*   **`event_data_hash`**: This IS the `event_digest`. (Clarification: `event_id` in `WitnessProofV1` should be the primary ID of the object, e.g. `ContentID`, `InteractionID`. `event_data_hash` is the hash of the specific data snapshot that was validated). *Correction: `event_id` in `WitnessProofV1` should be the primary identifier of the event object itself (e.g. a `ContentID` or `InteractionID`), while `event_data_hash` would be the hash of the specific data that was attested to, if there's a distinction (e.g. if the object has mutable parts not covered by the ID).* For simplicity, if `event_id` is already a hash of the core data, `event_data_hash` might be redundant or a hash of a larger encompassing structure. Let's assume `event_id` is the primary identifier (e.g. `ContentID`), and `event_data_hash` is the hash of the data presented to witnesses for validation.
*   **`network_timestamp`**: Typically the median of `witness_timestamp` values from the `M` collected valid attestations to prevent bias from any single Witness.
*   **`witness_committee_id`**: Can be derived deterministically, e.g., `hash(sorted_list_of_attesting_witness_dids || event_digest || epoch_id)`.
*   **`attesting_witness_dids`**: List of `witness_did` from the `M` collected attestations.
*   **`aggregated_signature`**:
    *   Option 1 (Simple): A list of the individual `witness_signature` fields from the `M` attestations. Verifiers would check each one.
    *   Option 2 (Preferred for efficiency): A true aggregate signature if a scheme like BLS (Boneh-Lynn-Shacham) is used. All `M` Witnesses would sign the same message (e.g., `event_digest || network_timestamp`), and their signatures can be aggregated into a single compact signature. This requires BLS-compatible keys.
    *   *(Decision from `echonet_v3_cryptographic_correctness_review.md` on BLS adoption would dictate this)*. For now, assume it *could* be an aggregate BLS signature.
*   **`specific_proof_data`**: Optional. Could include the average `poe_quality_score` if applicable, or other metadata derived from attestations.

### 5.3. Dissemination
*   Once a `WitnessProofV1` is generated, it's gossiped to the network.
*   It's stored persistently, typically on the DDS itself, retrievable via its own hash or linked from the original event data (e.g., the event object might have a field `witness_proof_id`).
*   The hash of the `WitnessProofV1` (or its `ContentID` if stored as such) is published to the Discovery System (DHT), allowing clients to look up proof of an event's validation.

## 6. Slashing Conditions & Protocol (Conceptual)

Slashing serves as a deterrent against Witness misbehavior.
### 6.1. Provable Offenses
*   **Double-Signing / Conflicting Attestations:** A Witness attesting differently (e.g., `isValid=true` and `isValid=false`, or different `poe_quality_score`s for the same PoE event) for the exact same `event_digest` and `event_type` within the same context (e.g., epoch). Evidence would be the two conflicting `WitnessAttestationV1` messages signed by the same Witness.
*   **Attesting to a Provably Invalid Event:** A Witness attests `isValid=true` for an event that is demonstrably invalid based on DLI rules (e.g., event signature is wrong, data is malformed in a way that any honest node can verify, violates fundamental DLI invariants). Evidence requires the event itself and the Witness's attestation.
*   **Prolonged Unavailability:** Consistent failure to participate in committees when selected (evidenced by lack of attestations for events where the Witness's VRF proof would have qualified them). This might be tracked by network monitoring services.
*   **Failed PoSR Responses (if Witness also an SSN):** As per DDS protocol.
*   **Complicity in Invalid PoSR Validation:** If a Witness attests to a `DDS_POSR_PROOF` event (`DLIEventTypeV1_DDS_POSR_PROOF`) which is later found to be fraudulent, and the fraudulence of the PoSR proof was deterministically verifiable at the time of attestation.

### 6.2. Evidence Submission
*   A "slashing request" is submitted as a special DLI transaction.
*   This request must contain cryptographic proof of the offense (e.g., the two conflicting attestations, the invalid event + attestation).
*   The DLI (potentially a special governance committee or automated contract logic) validates this evidence.

### 6.3. Slashing Execution
*   If evidence is validated, penalties are applied:
    *   A portion of the Witness's staked ECHO is burned or reallocated to a community fund.
    *   `reputation_score` in `NexusUserObjectV1` is significantly reduced.
    *   The Witness may be temporarily or permanently banned from the active Witness set (e.g., `IsWitnessActive` flag removed).
*   Slashing logic is governed by rules defined and updatable via the EchoNet governance process.

## 7. Error Codes Specific to PoW Protocol

Aligned with `echonet_v3_error_handling_strategy.md` and `DDSProtocolError`-like structure.
*   `POW_ERROR_UNSPECIFIED (0)`
*   `POW_INVALID_ATTESTATION_SIGNATURE (1)`: Witness signature on attestation is invalid.
*   `POW_INVALID_VRF_PROOF (2)`: Witness VRF proof for committee selection is invalid.
*   `POW_WITNESS_NOT_IN_COMMITTEE (3)`: Attestation received from a Witness not selected for the committee.
*   `POW_EVENT_VALIDATION_FAILED_BY_WITNESS (4)`: Witness explicitly marked event as invalid (used in `validation_error_code` of attestation).
*   `POW_INSUFFICIENT_ATTESTATIONS (5)`: Not enough valid attestations received to form `WitnessProofV1`.
*   `POW_CONFLICTING_ATTESTATIONS_FOR_PROOF (6)`: Aggregator found conflicting `is_valid` flags from different witnesses for the same event proof.
*   `POW_INVALID_EVENT_DATA_HASH (7)`: Event data hash in attestation doesn't match expected.
*   `POW_STALE_EPOCH (8)`: Attestation is for a past/invalid epoch for the given event.
*   `POW_AGGREGATED_SIGNATURE_FAILURE (9)`: Failed to create or verify the aggregated signature for `WitnessProofV1`.
*   `POW_SLASHING_CONDITION_NOT_MET (10)`: Submitted slashing request lacks valid proof of offense.

## 8. Security & Liveness Considerations

*   **Sybil Attacks:** Minimum stake and reputation for Witness eligibility are primary defenses. VRF-based selection makes it harder for an attacker to control a specific committee unless they control a significant fraction of the *total eligible stake/reputation*.
*   **Liveness:**
    *   Dynamic adjustment of `threshold_N` for VRF selection helps ensure committees are usually formed.
    *   If too few attestations are received, the event is not considered validated. Clients may need to resubmit or the event may be re-evaluated in a later epoch.
    *   Graceful degradation if total active Witnesses drops significantly (e.g., network might halt or increase `M` temporarily if security is paramount).
*   **Collusion Resistance:**
    *   Random committee selection per event/epoch makes long-term collusion difficult.
    *   Transparency of attestations and `WitnessProofV1` allows for public scrutiny.
    *   Slashing for attesting to provably invalid events disincentivizes collusion on malicious validation.
*   **VRF Security:** Relies on the unbiasability and unpredictability of the chosen VRF scheme. Witness secret VRF keys must be kept secure.
*   **Timestamp Manipulation:** Using the median of Witness timestamps for `network_timestamp` in `WitnessProofV1` mitigates impact of individual Witnesses with incorrect clocks.

## 9. Protocol Versioning

The PoW protocol (including VRF inputs, attestation formats, `WitnessProofV1` construction logic) will adhere to the versioning strategy in `echonet_v3_versioning_strategy.md`.
*   Protobuf message types will be versioned (e.g., `WitnessAttestationV1`, `WitnessAttestationV2`).
*   DLI state will record supported PoW protocol versions.
*   Upgrades to the PoW protocol will be significant DLI upgrades, managed by EchoNet governance.

---
This document provides the specification for the Proof-of-Witness validation protocol. It forms a critical part of ensuring data integrity and consensus within the EchoNet DLI.## 9. Protocol Versioning

The PoW protocol (including VRF inputs, attestation formats, `WitnessProofV1` construction logic) will adhere to the versioning strategy in `echonet_v3_versioning_strategy.md`.
*   Protobuf message types will be versioned (e.g., `WitnessAttestationV1`, `WitnessAttestationV2`).
*   DLI state will record supported PoW protocol versions.
*   Upgrades to the PoW protocol will be significant DLI upgrades, managed by EchoNet governance.

---
This document provides the specification for the Proof-of-Witness validation protocol. It forms a critical part of ensuring data integrity and consensus within the EchoNet DLI.
