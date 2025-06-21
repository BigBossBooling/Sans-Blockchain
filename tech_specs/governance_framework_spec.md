# DigiSocialBlock - Decentralized Governance Framework Specification

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Governance Framework Overview

### 1.1. Purpose
This document specifies the overarching technical framework for the Decentralized Governance module within the EchoNet (also referred to as DigiSocialBlock) DLI ecosystem. Its purpose is to establish the foundational architecture for on-DLI governance operations, enabling community-driven decision-making for protocol upgrades, parameter changes, and other governable aspects of the platform.

This framework sets the stage for more detailed specifications of individual governance components like proposal systems, voting mechanisms, and potential treasury management.

### 1.2. Core Principles
The design of the EchoNet governance framework adheres to the following core principles:
*   **Transparency:** All governance processes, proposals, votes, and outcomes are recorded on the DLI and publicly verifiable.
*   **Verifiability:** The integrity of governance actions (e.g., vote counting, proposal execution) is ensured through cryptographic methods and DLI consensus (Witness validation).
*   **Community Control:** The framework aims to empower the community of `did:echonet` holders, particularly those actively engaged (as measured by the Proof-of-Engagement - PoP system), to influence the platform's evolution.
*   **Modularity:** The architecture is designed to be modular, allowing for the future introduction or evolution of governance components and mechanisms.
*   **Clarity:** Governance rules and processes should be as clear and understandable as possible to facilitate broad participation.

## 2. Architectural Model for Governance Module(s)

A hybrid architectural model is proposed for implementing governance functionalities directly on the EchoNet DLI.

### 2.1. Core Governance State
Critical governance state information will be managed as specific, versioned DLI state objects. These objects are immutable records or stateful entries within the DLI's overall state, validated and maintained by the Witness network. Examples include:
*   The current set of globally governable DLI parameters (e.g., `PoEScoreFactorsV1`, staking thresholds).
*   The state of active and past governance proposals.
*   Aggregated vote tallies for ongoing referenda.

### 2.2. DLI-Native "Contracts" / Logic Units
Specific, well-defined DLI-native logic units (conceptually similar to smart contracts or pallets in other decentralized systems) will encapsulate the rules and processes for different governance functions. These are not general-purpose smart contracts but specialized state-transition functions executed by DLI nodes (and validated by Witnesses) in response to specific DLI events (governance transactions).

Key conceptual logic units include:

*   **`ProposalLifecycleManagerV1`:**
    *   **Responsibilities:** Handles the submission of governance proposals (e.g., for protocol upgrades, parameter changes, text proposals). Validates proposal structure and initial requirements (e.g., proposer eligibility, potential proposal deposit). Manages state transitions of proposals (e.g., Pending -> ActiveVoting -> Approved/Rejected -> Executed/Failed).
    *   **Interactions:** Consumes "SubmitProposal" DLI events. Emits `ProposalSubmitted`, `VotingPeriodStarted`, `ProposalOutcomeFinalized` events. Updates `ProposalState` DLI objects.
*   **`VotingManagerV1`:**
    *   **Responsibilities:** Manages the vote casting process for active proposals. Receives votes, verifies voter eligibility and voting power (see Section 4). Securely and verifiably tallies votes. Determines if quorum and approval thresholds are met.
    *   **Interactions:** Consumes "CastVote" DLI events. Emits `VoteCast` events. Updates `VoteRecord` and `ProposalState` DLI objects. Queries PoP/reputation data.
*   **`ParameterManagerV1`:**
    *   **Responsibilities:** Securely holds the current values of all governable DLI parameters. Provides an authenticated interface for updating these parameters if a governance proposal to do so is approved and executed.
    *   **Interactions:** Consumes "ExecuteProposal" DLI events (if the proposal targets parameter changes). Emits `ParameterUpdated` events.
*   **`TreasuryManagerV1` (Future Scope - Post MVP):**
    *   **Responsibilities:** Manages on-DLI community funds or treasury (if implemented). Handles proposals for spending treasury funds and executes approved disbursements.
    *   **Interactions:** Would consume specific proposal types and interact with token transfer logic.

These logic units are part of the core DLI protocol. Their execution is deterministic and validated by all EchoNet Witnesses as part of the PoW consensus process (see `tech_specs/pow_protocol.md`).

## 3. Integration with `did:echonet`

The `did:echonet` method (specified in `tech_specs/did_echonet_method.md`) is fundamental to the governance framework.

*   **Participant Identification:** All participants in governance processes (e.g., proposers, voters, council members if any) are identified by their `did:echonet` DIDs.
*   **Authentication & Authorization:**
    *   All governance actions submitted as DLI events (e.g., submitting a proposal, casting a vote) MUST be cryptographically signed using the private key associated with an `authentication` or `assertionMethod` verification method in the participant's `NexusDIDDocumentV1`.
    *   The relevant governance logic unit (`ProposalLifecycleManagerV1`, `VotingManagerV1`) is responsible for verifying these signatures against the public key retrieved from the DID Document.
*   **No Anonymous Participation (for binding actions):** While discussion may be open, formal governance actions like voting and proposing require authenticated DIDs.

## 4. Integration with PoP System (for Voting Power/Eligibility)

To ensure that those most invested and positively engaged in the EchoNet ecosystem have a commensurate voice, voting power and/or eligibility for certain governance roles will be linked to the Proof-of-Engagement (PoP) system (as detailed in `tech_specs/poe_protocol.md`). PoP generates reputation and, potentially, tokens.

### 4.1. Source of Voting Power/Eligibility
Several options exist for determining voting power. The chosen method should balance incentivizing participation with preventing plutocracy and ensuring fairness.

*   **Chosen Approach (Recommendation): Derived Governance Reputation (Option C from prompt)**
    *   **Mechanism:** A non-transferable "governance reputation score" is maintained as part of (or derived from) the `reputationScore` within each user's `NexusUserObjectV1`. This score is influenced by:
        *   Long-term, consistent positive PoE score accumulation.
        *   Potential bonuses for other beneficial activities (e.g., successful past governance participation, acting as a reliable Witness or SSN, community moderation if tracked).
        *   Subject to decay for prolonged inactivity in engagement or governance.
    *   **Rationale:** This approach mitigates pure plutocracy (vote-buying if based on transferable PoP tokens alone) and rewards sustained, positive contributions to the ecosystem beyond just holding tokens. It aligns with the "fair compensation" and "user empowerment" ethos. It requires the reputation system to be well-designed and resistant to gaming.
*   **Alternative for Initial MVP (Simpler): Direct PoP Token Balance (Option A from prompt)**
    *   If a PoP utility token is implemented early and is directly tied to engagement, its balance could be used as a simpler, initial measure of voting power. This is more plutocratic but easier to implement initially if a complex reputation derivation is deferred.
    *   *Decision for MVP:* The primary target is reputation-weighted voting. If this proves too complex for the very first governance iteration, a PoP token balance (if available in MVDLI context, which is unlikely given MVP scope) or even "one-DID-one-vote" for very basic initial proposals might be used as a placeholder, with a clear roadmap to reputation-weighting. For this specification, we assume the framework targets reputation-weighting.

### 4.2. Snapshotting Voting Power
*   To prevent manipulation of voting power during an active voting period (e.g., by suddenly acquiring tokens or performing actions to boost reputation), the voting power (or the underlying reputation score/token balance used to calculate it) for all eligible DIDs is **snapshotted** at a specific DLI state (e.g., a specific block height or timestamp).
*   This snapshot typically occurs at the **start of the voting period** for a given proposal, or at the end of the proposal submission/discussion phase.
*   The `VotingManagerV1` uses these snapshotted values for tallying votes for that specific proposal.

### 4.3. Accessing Voting Power Data
*   The `VotingManagerV1` logic unit must have a secure and verifiable way to access the snapshotted reputation scores (from `NexusUserObjectV1` state) or relevant PoP token balances associated with each voter's DID at the time of the snapshot. This is an internal DLI state query.

## 5. Core Governance State Objects (Conceptual)

The governance framework will manage several key DLI state objects. Their detailed Protobuf structures will be defined in a subsequent, more specific governance components specification. Conceptual examples:

*   **`GovernanceParametersV1`:** A singleton DLI state object holding all current governable parameters of the EchoNet protocol (e.g., `PoEScoreFactorsV1` values, PoW committee sizes, staking amounts, fee rates). Updated by `ParameterManagerV1`.
*   **`ProposalRecordV1`:** Represents a single governance proposal. Contains:
    *   `proposal_id` (unique ID).
    *   `proposer_did`.
    *   `submission_timestamp`.
    *   `title`, `description`, `category`.
    *   `proposed_changes` (e.g., parameter change details, link to code for protocol upgrade).
    *   `status` (Pending, ActiveVoting, Approved, Rejected, Executed, Failed, Expired).
    *   `voting_start_timestamp`, `voting_end_timestamp`.
    *   `final_tally_results` (yes, no, abstain counts/weights).
*   **`VoteCastRecordV1`:** Records an individual vote on a proposal. Contains:
    *   `proposal_id`.
    *   `voter_did`.
    *   `vote_option` (yes, no, abstain).
    *   `voting_power_at_snapshot` (the snapshotted power with which this vote was cast).
    *   `timestamp_cast`.
    *   `voter_signature`.

These records provide the auditable history of governance activities.

## 6. Event Emission for Governance Actions

To enable external observers, client applications, and other DLI modules to track governance activity, the governance logic units MUST emit DLI events upon significant actions. These events are part of the DLI's immutable log.

**Key Governance Events:**
*   `ProposalSubmittedEvent(proposal_id, proposer_did, type, submission_timestamp)`
*   `VotingPeriodStartedEvent(proposal_id, voting_start_timestamp, voting_end_timestamp, snapshot_height)`
*   `VoteCastEvent(proposal_id, voter_did, vote_option, voting_power)`
*   `ProposalOutcomeFinalizedEvent(proposal_id, outcome_status, final_tally_yes, final_tally_no, final_tally_abstain)`
*   `ProposalExecutedEvent(proposal_id, execution_details_hash)`
*   `ParameterUpdatedEvent(parameter_name, old_value_hash, new_value_hash, effective_from_timestamp)`
*   `CouncilMemberElectedEvent(council_id, member_did, election_epoch)` (if councils are implemented)

## 7. Security & Integrity

*   **Witness Validation:** All DLI events related to governance (proposal submissions, votes, parameter changes via approved proposals) are subject to validation by EchoNet Witnesses as part of the core DLI consensus (PoW). This ensures that only valid, authorized governance actions modify DLI state.
*   **Cryptographic Signatures:** All actions initiated by DIDs (proposing, voting) must be signed, ensuring authenticity and non-repudiation.
*   **Parameter Immutability (except via governance):** The `ParameterManagerV1` ensures that DLI parameters can only be changed through a successful governance proposal explicitly targeting them. Direct, unauthorized modification is prevented.
*   **Vote Secrecy (Optional/Future):** While transparency is a core principle, future iterations might explore privacy-preserving voting mechanisms (e.g., using ZKPs for vote casting) if deemed necessary by the community, though this adds significant complexity. For MVP, votes are likely public.
*   **Resistance to Censorship/Manipulation:** Storing proposals, votes, and outcomes on the decentralized DDS and DLI state, and using a robust P2P network for event gossip, provides resistance against censorship or manipulation by any single entity.

## 8. Extensibility

The governance framework is designed with modularity in mind:
*   New types of proposals or voting mechanisms can be introduced via protocol upgrades (which themselves would go through the existing governance process).
*   New DLI-native logic units for specialized governance functions (e.g., different types of councils, participatory budgeting) can be added.
*   The clear separation of concerns (`ProposalLifecycleManager`, `VotingManager`, etc.) facilitates targeted upgrades.

## 9. Protocol Versioning

All components of the governance framework, including DLI state object structures (e.g., `ProposalRecordV1`), DLI-native logic units, and emitted event structures, will adhere to the versioning strategy outlined in `echonet_v3_versioning_strategy.md`. This ensures that upgrades to the governance system can be managed in a clear and backward-compatible (or explicitly breaking) manner.

---
This specification lays the architectural foundation for on-DLI governance in EchoNet. Subsequent documents will detail the precise mechanics of proposal submission, voting procedures, specific parameter management interfaces, and the implementation of the core logic units.
