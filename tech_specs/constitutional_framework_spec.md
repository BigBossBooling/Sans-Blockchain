# DigiSocialBlock - "Constitutional" Framework and Core Protocol Parameters Specification

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. "Constitutional" Framework Overview

### 1.1. Purpose
This document specifies the technical framework for defining, protecting, and managing both the fundamental, enduring principles ("Constitutional Rules") and the adaptable operational settings ("Core Protocol Parameters") of the EchoNet (also referred to as DigiSocialBlock) DLI ecosystem.

The framework aims to:
*   Safeguard the core vision and values of EchoNet.
*   Provide a transparent and secure mechanism for the evolution of adaptable protocol parameters through decentralized governance.
*   Ensure platform stability and predictability while allowing for necessary adjustments.

### 1.2. Distinction
*   **"Constitutional Rules":** These are the foundational tenets and ethical boundaries of the EchoNet platform. They are designed to be extremely difficult to change, requiring extraordinary consensus, or may even be immutable aspects of the core protocol design. They represent the long-term social contract of the ecosystem.
*   **"Core Protocol Parameters":** These are configurable settings that affect the day-to-day operation of the DLI and its modules (e.g., PoW, DDS, PoE, fees). While critical, they are designed to be adaptable and updatable via the standard decentralized governance process to allow the platform to evolve and respond to changing needs and conditions.

## 2. Core Protocol Parameter Management

This section details how operational parameters of the EchoNet DLI are defined, stored, and updated.

### 2.1. `ProtocolParametersV1` Data Structure
A singular, versioned Protocol Buffers message will define the schema for all governable core protocol parameters. This structure serves as the single source of truth for these settings at any given point in the DLI's history.

```protobuf
syntax = "proto3";

package digisocialblock.dli.parameters;

// ProtocolParametersV1 defines the set of governable parameters for the EchoNet DLI.
// All parameters here are examples and would be expanded based on detailed module specifications.
message ProtocolParametersV1 {
  // --- Proof-of-Witness (PoW) Protocol Parameters ---
  // Target number of witnesses in a committee.
  int32 target_witness_committee_size_n = 1;
  // Minimum number of attestations required for event validation.
  int32 required_attestations_m = 2;
  // Minimum stake required for a node to register as a Witness candidate (in smallest token unit).
  int64 witness_registration_stake_min = 3;
  // Duration for which a Witness's stake is locked after de-registration.
  int64 witness_deregistration_lockup_seconds = 4;
  // Maximum number of events a witness committee can validate in a batch or epoch.
  int32 witness_max_events_per_epoch = 5;

  // --- Distributed Data Stores (DDS) Protocol Parameters ---
  // Default number of replicas for each DDS chunk.
  int32 default_dds_replication_factor_r = 11;
  // Default k value for Reed-Solomon erasure coding (number of data shards).
  int32 default_dds_erasure_k = 12;
  // Default m value for Reed-Solomon erasure coding (total data + parity shards).
  int32 default_dds_erasure_m = 13;
  // Fee for storing 1KB of data for 1 year on DDS (conceptual, in smallest token unit).
  int64 dds_storage_fee_per_kb_year = 14;

  // --- Proof-of-Engagement (PoE) Protocol Parameters ---
  // ContentID or version string of the active PoEScoreFactorsV1 configuration.
  // This allows PoEScoreFactorsV1 to be a separate, governable object stored on DDS.
  string active_poe_score_factors_ref = 21;
  // Total allocation from the main treasury or emission to the PoE reward pool per day/period.
  int64 poe_reward_pool_period_allocation = 22; // In smallest token unit
  // Minimum reputation score for an actor to earn PoE rewards.
  int64 poe_min_actor_reputation_for_reward = 23;
  // Minimum PoE quality score an interaction must achieve to be eligible for rewards.
  double poe_min_interaction_score_for_reward = 24;

  // --- Governance Protocol Parameters ---
  // Deposit required to submit a governance proposal (in smallest token unit).
  int64 proposal_submission_deposit = 31;
  // Standard duration for the voting period of a governance proposal.
  int32 proposal_voting_period_duration_seconds = 32;
  // Quorum threshold for a proposal vote to be valid (e.g., percentage of total possible voting power).
  double proposal_quorum_threshold_percentage = 33; // e.g., 0.10 for 10%
  // Approval threshold for a proposal to pass (e.g., percentage of votes cast).
  double proposal_approval_threshold_percentage = 34; // e.g., 0.66 for 66% (supermajority)

  // --- Fee & Economic Parameters (Conceptual) ---
  // Base fee for a simple DLI transaction (e.g., DID update, basic interaction).
  int64 base_dli_transaction_fee = 41; // In smallest token unit
  // Percentage of transaction fees to be burned.
  double fee_burn_percentage = 42; // e.g., 0.25 for 25%
  // Percentage of transaction fees allocated to Witness rewards.
  double fee_to_witnesses_percentage = 43; // e.g., 0.50 for 50%

  // --- Dispute Resolution Parameters ---
  // Fee to open a dispute (in smallest token unit).
  int64 dispute_opening_fee = 51;
  // Duration of the evidence gathering period for disputes.
  int32 dispute_evidence_period_seconds = 52;

  // Version of this ProtocolParametersV1 structure itself.
  // Allows for adding/deprecating parameters in a structured way.
  string parameters_schema_version = 100; // e.g., "1.0.0"
}
```

### 2.2. Storage on DLI
*   The currently active `ProtocolParametersV1` object is stored as a unique, well-known DLI state object.
*   This could be:
    *   A specific key in a global key-value store managed by the DLI state.
    *   A singleton instance managed by the `ParameterManagerV1` DLI-native logic unit.
*   Its integrity is ensured by the DLI consensus (Witness validation). All DLI nodes have access to the current parameters to enforce protocol rules consistently.

### 2.3. Update Mechanism
Parameters are updated exclusively through the decentralized governance process:
1.  **Proposal:** A change to one or more parameters is initiated by submitting a specific type of governance proposal (e.g., "ParameterChangeProposal"). This proposal details the parameter(s) to be changed, their proposed new value(s), and often a rationale. (Proposal process detailed in `tech_specs/governance_framework_spec.md` and a future `governance_proposals_voting_spec.md`).
2.  **Validation & Voting:** The proposal undergoes validation (e.g., are the new values within sane ranges?) and a community voting period as per the governance rules.
3.  **Execution:** If the proposal is approved:
    *   The `ParameterManagerV1` DLI-native logic unit is invoked with the details of the approved changes.
    *   The `ParameterManagerV1` validates the authorization (i.e., that the update instruction comes from a legitimate, finalized governance vote).
    *   It then atomically updates the global `ProtocolParametersV1` DLI state object with the new values.
    *   An immutable log of this change (including old values, new values, and the `WitnessProofV1` of the governance vote) is effectively created by the DLI event and state update.
4.  **Event Emission:** Upon successful update, a `ParameterUpdatedEvent` is emitted by the `ParameterManagerV1`.
    *   `ParameterUpdatedEvent(parameter_name_key, old_value_representation, new_value_representation, effective_from_block_or_timestamp, governance_proposal_id)`

### 2.4. Access by Modules
*   All DLI modules (Witnesses, DDS nodes, PoE logic, DID management logic, etc.) that require access to these parameters will read them from the canonical DLI state object.
*   Access is typically "read-only" for most modules; only the `ParameterManagerV1` can write, and only when authorized by governance.
*   Modules may cache parameter values locally for an epoch or specific period but should be designed to reload them if a `ParameterUpdatedEvent` indicates a change relevant to their operation.

## 3. Conceptual "Constitutional" Rules

These are fundamental principles that underpin EchoNet's operation and are intended to be highly resistant to change, ensuring the long-term character and commitments of the platform.

### 3.1. Definition
*   **Immutable Rules:** Certain rules may be deeply embedded in the core protocol design and are practically immutable without a hard fork that creates an entirely new version of EchoNet. These often relate to fundamental cryptographic methods or data structures.
*   **Extraordinary Majority Rules:** Other constitutional rules might be amendable but only through a process requiring a significantly higher consensus threshold (e.g., >75% or >90% supermajority of voting power/reputation) and potentially longer deliberation/voting periods than standard parameter changes.

### 3.2. Examples of Potential Constitutional Rules for EchoNet (Conceptual)
1.  **Right to Identity:** The fundamental ability for any entity to generate a `did:echonet` DID and control it via their private keys, as per the `did:echonet` method specification, shall not be infringed by the protocol.
2.  **Primacy of User Consent:** User data managed under the scope of the Data Consent Protocol (`tech_specs/data_consent_protocol.md`) cannot be accessed or processed by any entity (including system services) without explicit, revocable user consent recorded on the DLI.
3.  **Core Proof-of-Engagement Principles:** The PoE system (`tech_specs/poe_protocol.md`) shall always aim to reward demonstrable quality of engagement and contribution, and its core mechanisms shall not be altered to arbitrarily favor specific parties without broad consensus on the definition of "quality."
4.  **Supremacy of Decentralized Governance:** The authority to upgrade the protocol, change its core parameters (as defined in `ProtocolParametersV1`), and manage community resources (like a future treasury) rests with the decentralized governance process. No single entity shall have unilateral control to make these changes.
5.  **Freedom of Expression (with caveats):** Content published to EchoNet shall be resistant to censorship by any single party or small group. Content removal or moderation actions must follow transparent, community-defined policies and dispute resolution mechanisms, and must respect applicable legal frameworks. *(This is highly complex and requires careful, nuanced definition if formally included).*
6.  **Openness & Transparency of Core Protocol:** The core EchoNet DLI protocol specifications, including its governance and economic models, shall remain open and publicly accessible.

### 3.3. Representation & Storage
*   **Implicit in Design:** Some constitutional rules are implicitly enforced by the very architecture of the DLI, its cryptographic foundations, and core data structures (e.g., the nature of DIDs).
*   **"Founding Charter" / "Constitution Document":**
    *   A human-readable document (e.g., Markdown) explicitly outlining these constitutional rules could be created.
    *   The cryptographic hash (e.g., SHA-256) of a specific, agreed-upon version of this Charter document could be immutably stored as a DLI constant (e.g., in the genesis state or a special DLI record).
    *   This provides an on-DLI anchor for the agreed-upon principles.
    *   Amendments to this Charter would constitute a "constitutional amendment" requiring an extraordinary governance process.

### 3.4. Enforcement
*   **Protocol Design:** The primary enforcement mechanism is the DLI protocol itself. Actions violating deeply embedded rules would be rejected as invalid by Witnesses and nodes.
*   **Governance:** For rules requiring extraordinary majority, the governance framework (`VotingManagerV1`, `ProposalLifecycleManagerV1`) would enforce the higher voting thresholds and longer periods for constitutional amendment proposals.
*   **Community & Social Consensus:** Ultimately, the broader EchoNet community's commitment to these principles plays a role in their continued enforcement and interpretation.

## 4. Security & Stability

*   **Atomic Parameter Updates:** The `ParameterManagerV1` must ensure that updates to the `ProtocolParametersV1` object are atomic. If a governance proposal approves multiple parameter changes, they should all apply simultaneously or none at all to prevent inconsistent states.
*   **Protection of Constitutional Rules:**
    *   Immutable rules are protected by the difficulty of hard-forking the network.
    *   Rules amendable by extraordinary majority are protected by the high consensus thresholds defined in the governance framework.
*   **Parameter Validation & Sanity Checks:**
    *   The `ParameterManagerV1` (and the governance proposal system) SHOULD include logic for basic sanity checks on proposed new parameter values (e.g., ensuring `required_attestations_M` is less than or equal to `target_witness_committee_size_N`, ensuring percentages are within 0-100, ensuring numerical values are within reasonable operational bounds).
    *   This prevents governance from accidentally or maliciously setting parameters to values that could destabilize the network (e.g., setting transaction fees to an absurdly high number, or replication factor to zero). Governance can override these sanity checks with a specific "unsafe_set_parameter" proposal type if absolutely necessary, but this would be highly flagged.
*   **Event Ordering:** The DLI's consensus mechanism ensures a canonical ordering of governance events, including parameter updates, preventing conflicts.

## 5. Protocol Versioning

*   **`ProtocolParametersV1` Structure:** The `parameters_schema_version` field within the `ProtocolParametersV1` message allows the structure itself to be versioned. Adding new parameters is typically non-breaking if old software can ignore them. Changing or removing existing parameters is a breaking change requiring a major schema version increment and a coordinated DLI protocol upgrade.
*   **"Constitutional" Framework / Charter:** If a "Founding Charter" document is used, it would also be versioned. Changes to the Charter (via extraordinary governance) would result in a new version with a new hash stored on the DLI.
*   All versioning aligns with the overall strategy in `echonet_v3_versioning_strategy.md`. Protocol upgrades that change parameters or constitutional rules must be managed carefully.

---
This specification provides the high-level framework for managing the core rules and operational settings of EchoNet, aiming for a balance between fundamental stability and governable adaptability. More detailed specifications for the governance proposal system and voting mechanics will build upon this foundation.
