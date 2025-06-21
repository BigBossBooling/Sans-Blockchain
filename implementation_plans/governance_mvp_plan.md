# EchoNet - Decentralized Governance MVP Implementation Plan

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Governance MVP - Goals & Scope

### 1.1. Goals
The primary goal of the Decentralized Governance Minimal Viable Product (MVP) is to implement the foundational on-DLI mechanisms that allow the EchoNet protocol and its parameters to be managed and evolved by its community of participants in a transparent and verifiable manner. This MVP will demonstrate the core lifecycle of a governance proposal, from submission through voting to outcome determination, focusing initially on simple proposal types.

The Governance MVP must demonstrate:
*   The ability for a DID holder to submit simple proposal types: a text proposal (for general signaling) and a proposal to change a predefined, non-critical DLI parameter.
*   A basic voting mechanism where DIDs can cast votes, with vote weight derived from their `NexusUserObjectV1.reputationScore` (influenced by PoP).
*   Vote tallying based on these weighted votes.
*   Determination of proposal outcomes based on predefined rules (e.g., simple majority, fixed quorum).
*   If a parameter change proposal passes, the corresponding DLI parameter is updated in the DLI state.
*   All governance actions (proposals, votes) are recorded as DLI events and validated by MVDLI Witnesses.

### 1.2. Scope
**In Scope for Governance MVP:**
*   Core governance data structures (simplified `NexusProposalV1`, `VoteV1`, relevant subset of `ProtocolParametersV1`).
*   DLI-native logic for `ProposalLifecycleManagerV1` (handling submission and state transitions for text and simple parameter change proposals).
*   DLI-native logic for `VotingManagerV1` (handling vote casting, PoP/reputation-weighted tallying based on `NexusUserObjectV1.reputationScore`, outcome determination with simple majority and fixed quorum).
*   DLI-native logic for `ParameterManagerV1` (allowing updates to 1-2 predefined non-critical parameters from `ProtocolParametersV1` via successful governance proposals).
*   Storage and retrieval of governance state (proposals, votes, current parameters) using the MVDLI's DLI state mechanism.
*   Extension of MVDLI Witness logic to validate governance DLI events (proposal submission, vote casting, parameter updates).
*   Client-side interaction (via CLI/test application) for submitting proposals, listing proposals, casting votes, querying proposal status/outcomes, and querying current values of governable parameters.
*   Unit tests for all implemented governance logic, guided by `testing_strategies/governance_unit_tests_strategy.md`.

**Non-Goals for Governance MVP:**
*   Complex multi-stage voting processes (e.g., multiple rounds, ranked-choice voting).
*   Advanced dispute resolution framework implementation (beyond conceptual logging or handling if a dispute itself were a proposal type, which is not the focus here).
*   Full treasury management (creation of a treasury, proposals for spending from it).
*   Actual DLI protocol code upgrades triggered and managed via governance (NIPs). This MVP focuses on parameter changes.
*   Sophisticated UIs for governance participation (CLI is sufficient).
*   Dynamic adjustment of quorum or voting thresholds via governance (these will be fixed for MVP).
*   Vote delegation.
*   Gas fees for governance transactions (MVDLI assumes simplified or no gas fees initially).
*   Advanced security measures against complex governance attacks (focus on functional correctness and basic integrity).

## 2. Key Implementation Tasks & Prioritization for Governance MVP

These tasks build upon the MVDLI Core (Module 1) and Identity & Privacy MVP (Module 2).

*   **Task G1: Implement Core Governance Data Structures (in Go)**
    *   **Description:** Implement the Protocol Buffer definitions for MVP-scoped versions of `NexusProposalV1` (supporting text and simple parameter change types), `VoteV1` (simple vote options), and the relevant subset of `ProtocolParametersV1` that will be governable in the MVP. Ensure these are integrated with the existing Protobuf setup.
    *   **Inputs:** `tech_specs/governance_framework_spec.md`, `tech_specs/constitutional_framework_spec.md` (for `ProtocolParametersV1`), `tech_specs/dli_core_data_structures.md` (for base types).
    *   **Priority:** **High**.
*   **Task G2: Basic `ProposalLifecycleManagerV1` Logic**
    *   **Description:** Implement the DLI-native logic to handle:
        *   Submission of text proposals and simple parameter change proposals (validating structure, proposer DID).
        *   Transitioning proposals through states: `Submitted` -> `ActiveVoting` (after a fixed delay or minimal validation) -> `VotingEnded`.
        *   Basic validation of proposal content (e.g., parameter name exists, proposed value is of correct type for parameter change).
    *   **Inputs:** Task G1, `tech_specs/governance_framework_spec.md`.
    *   **Priority:** **High**.
*   **Task G3: Basic `VotingManagerV1` Logic**
    *   **Description:** Implement the DLI-native logic to:
        *   Accept `VoteV1` submissions for proposals in `ActiveVoting` state.
        *   Verify voter eligibility (valid DID).
        *   Retrieve `NexusUserObjectV1.reputationScore` for the voter DID (from DLI state, snapshotted at the start of the vote).
        *   Apply vote weight based on this reputation score (e.g., score = weight).
        *   Tally Aye/Nay votes (sum of weights).
        *   After `VotingEnded` state, determine outcome based on simple majority (>50% of total weighted Aye+Nay votes) and a fixed quorum (e.g., X% of total potential reputation must have voted).
    *   **Inputs:** Task G1, `tech_specs/governance_framework_spec.md`, `tech_specs/pop_voting_mechanics_spec.md` (conceptual), Identity & Privacy MVP (for `NexusUserObjectV1` access).
    *   **Priority:** **High**.
*   **Task G4: Basic `ParameterManagerV1` Logic**
    *   **Description:**
        *   Identify 1-2 non-critical DLI parameters from `ProtocolParametersV1` (e.g., a logging verbosity level, a test string parameter) to be updatable in the MVP.
        *   Implement DLI-native logic that, upon notification of a successful "ParameterChangeProposal" (from `ProposalLifecycleManagerV1` via `VotingManagerV1`), updates the specified parameter in the DLI's global state representation of `ProtocolParametersV1`.
        *   Emit a `ParameterUpdatedEvent`.
    *   **Inputs:** Task G1, `tech_specs/constitutional_framework_spec.md`, `tech_specs/governance_framework_spec.md`.
    *   **Priority:** **Medium**.
*   **Task G5: DLI State Integration for Governance**
    *   **Description:** Ensure that all governance objects (current `ProtocolParametersV1`, individual `NexusProposalV1` states, `VoteV1` records or aggregated tallies) are persistently stored and retrievable using the MVDLI's state mechanism (e.g., the embedded KV store). Define clear keys/paths for accessing this state.
    *   **Inputs:** Task G1-G4, MVDLI state management component.
    *   **Priority:** **Medium**.
*   **Task G6: Witness Validation for Governance Events**
    *   **Description:** Extend the MVDLI Witness PoW validation logic to recognize and validate new DLI event types related to governance:
        *   Proposal submission (structure, signature, valid proposer).
        *   Vote casting (structure, signature, valid voter, active proposal, within voting period).
        *   (Conceptual) Parameter update execution trigger (validating it came from a passed proposal).
    *   **Inputs:** MVDLI PoW framework, `tech_specs/pow_protocol.md`.
    *   **Priority:** **Medium**.
*   **Task G7: Client-Side Interaction (CLI / Test Application)**
    *   **Description:** Extend the MVDLI test client/CLI tool to include commands for:
        *   Submitting a text proposal.
        *   Submitting a parameter change proposal for the designated MVP parameters.
        *   Listing active proposals.
        *   Casting a vote (Aye/Nay) on an active proposal.
        *   Querying the status and outcome of a specific proposal.
        *   Querying the current value of a governable parameter.
    *   **Inputs:** All preceding tasks in this MVP, MVDLI client/CLI.
    *   **Priority:** **Medium** (Essential for testing and demonstrating governance).
*   **Task G8: Unit Tests for Governance MVP Components**
    *   **Description:** Implement comprehensive unit tests for all new DLI-native logic in `ProposalLifecycleManagerV1`, `VotingManagerV1`, and `ParameterManagerV1`, covering valid cases, invalid inputs, state transitions, and outcome calculations, as guided by `testing_strategies/governance_unit_tests_strategy.md`.
    *   **Inputs:** `testing_strategies/governance_unit_tests_strategy.md`, Tasks G2, G3, G4.
    *   **Priority:** **High**.

## 3. Dependencies on Other MVP Modules

The Governance MVP relies significantly on functionalities developed in prior MVP modules:
*   **MVDLI Core (Module 1 - from `implementation_plans/dli_core_mvp_plan.md`):**
    *   **P2P Network & Event Gossip:** Essential for propagating governance DLI events (proposals, votes).
    *   **Witness Validation Framework:** Required to validate and reach consensus on governance events.
    *   **DLI State Storage:** Needed to store `ProtocolParametersV1`, `NexusProposalV1` states, vote records, etc.
    *   **Core Data Structures:** Base types used in governance structures.
    *   **DDS (indirectly):** If proposal descriptions or evidence become very large, they might be stored on DDS with references in the proposal object. For MVP, assume descriptions are small enough for DLI state.
*   **Identity & Privacy MVP (Module 2 - from `implementation_plans/identity_privacy_mvp_plan.md`):**
    *   **`did:echonet` DIDs:** Proposers and voters are identified by their DIDs. Signatures are verified against keys in `NexusDIDDocumentV1`.
    *   **`NexusUserObjectV1`:** The `reputationScore` field within this object is crucial for PoP-weighted voting as envisioned for the MVP. The `VotingManagerV1` will need to query this.
*   **Content Validation MVP (Module 3 - from `implementation_plans/content_validation_mvp_plan.md`):**
    *   The mechanisms for updating `NexusUserObjectV1.reputationScore` based on PoE activities provide the actual data that the Governance MVP's voting system will use for vote weighting.

## 4. Development & Testing Approach for Governance MVP

*   **Test-Driven Development (TDD):** Strongly enforce TDD for all core DLI-native logic units (`ProposalLifecycleManagerV1`, `VotingManagerV1`, `ParameterManagerV1`) to ensure correctness of state transitions and validation rules.
*   **Unit Testing:** Comprehensive unit tests for each function and module as outlined in `testing_strategies/governance_unit_tests_strategy.md`.
*   **Integration Testing (within MVDLI simulated network):**
    *   Test the full lifecycle of a text proposal: submission -> active voting -> voting by multiple DIDs -> tallying -> outcome determination.
    *   Test the full lifecycle of a parameter change proposal: submission -> voting -> outcome -> verification that the DLI parameter state is updated correctly (queried via CLI).
*   **Scenario Testing:**
    *   Votes not meeting quorum.
    *   Votes meeting quorum but failing to reach majority.
    *   Votes succeeding.
    *   Attempts to vote outside voting period or with invalid DIDs.
    *   Attempts to propose invalid parameter changes.
*   **CLI-Driven E2E Tests:** Use the extended CLI/test client to automate these scenarios.

## 5. Next Steps After Governance MVP

Successful completion of the Governance MVP will enable basic community-driven evolution of the platform. Subsequent steps will focus on:
1.  **Implementing More Proposal Types:** Such as proposals for community fund spending (once a treasury exists), protocol feature activations (NIPs), or constitutional amendments (with higher thresholds).
2.  **Advanced Voting Mechanisms:** Exploring and implementing features like vote delegation, quadratic voting (if deemed suitable), or privacy-preserving voting components.
3.  **Full Dispute Resolution Framework:** Integrating the more advanced aspects of the dispute resolution protocol, potentially with dedicated arbitrator roles and staking.
4.  **Treasury Management:** Implementing the `TreasuryManagerV1` for on-DLI management of community funds.
5.  **Sophisticated Parameter Management:** Including more complex parameter types, validation ranges, and potentially interdependencies between parameters.
6.  **Enhanced UI/UX:** Developing user-friendly interfaces for governance participation beyond the CLI.
7.  **Robust Security & Auditing:** Further hardening of all governance components against attacks.

This Governance MVP plan focuses on establishing the essential machinery for on-DLI decision-making, which is a cornerstone of EchoNet's decentralized ethos.
