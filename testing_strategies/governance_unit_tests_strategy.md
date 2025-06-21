# DigiSocialBlock - Decentralized Governance Unit Testing Strategy

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Unit Testing Philosophy for Governance

### 1.1. Importance
The decentralized governance mechanisms are the ultimate authority for protocol evolution, parameter adjustments, and dispute resolution within the EchoNet (also referred to as DigiSocialBlock) DLI. Their correctness, reliability, integrity, and predictability are paramount for maintaining DLI stability, fostering community trust, and ensuring the long-term viability of the platform. Rigorous unit testing is the first line of defense in achieving these qualities.

### 1.2. Goals
The primary goals of unit testing the governance components are to:
*   **Verify Correctness:** Ensure that each isolated piece of governance logic behaves exactly as defined in its technical specifications (e.g., `tech_specs/governance_framework_spec.md`, `tech_specs/dispute_resolution_protocol_spec.md`, `tech_specs/constitutional_framework_spec.md`, and future proposal/voting specs).
*   **Cover Edge Cases:** Identify and test scenarios that are unusual, boundary conditions, or potential failure points.
*   **Ensure Adherence to Rules:** Validate that all defined governance rules (e.g., proposal validity, voter eligibility, quorum, thresholds) are correctly implemented and enforced.
*   **Test Security Assumptions:** Probe for potential vulnerabilities at the unit level (e.g., unauthorized actions, replay of votes if applicable, bypass of state transition logic).
*   **Facilitate Refactoring:** Provide a safety net that allows developers to improve and refactor governance code with confidence that existing functionality remains intact.

### 1.3. Approach
*   **Test-Driven Development (TDD):** As stated in the `roadmap/overall_mvp_implementation_roadmap.md` and aligning with the Expanded KISS Principle ("Iterate Intelligently" - "Automated Testing," and "Sense the Landscape" - "Test-Driven Development"), TDD will be heavily favored for implementing all core governance logic. Tests will be written before or concurrently with the implementation code.
*   **Isolation:** Unit tests must focus on testing individual components (functions, methods, DLI-native "contract" logic units) in isolation, using mocks and stubs for external dependencies.

## 2. Scope of Unit Tests for Governance

Unit tests will cover all critical, testable components of the decentralized governance framework, including but not limited to:

*   **DLI-Native "Contract" Logic Units:**
    *   `ProposalLifecycleManagerV1` (and any future versions)
    *   `VotingManagerV1` (and any future versions)
    *   `ParameterManagerV1` (and any future versions)
    *   Logic units related to the Dispute Resolution Protocol (e.g., `DisputeRegistryManagerV1` if such a distinct unit emerges, or logic within `ProposalLifecycleManagerV1` if disputes trigger proposals).
    *   Logic units for Protocol Upgrade Activation.
*   **Helper Functions & Libraries:** Any specific utility functions or libraries developed exclusively for supporting governance mechanisms (e.g., complex vote weight calculation libraries, canonicalization routines for signing governance objects, state transition validators).
*   **Data Structure Validation:** While Protobuf provides some level of validation, additional business rule validations on governance-specific data structures (e.g., `ProposalRecordV1`, `VoteCastRecordV1`, `DisputeCaseV1`) should be unit tested.

## 3. Test Case Categories & Examples for Each Governance Component

This section outlines categories of test cases. Specific test cases will be derived from the detailed technical specifications of each component. All error conditions should align with `echonet_v3_error_handling_strategy.md`.

### 3.A. Proposal Lifecycle Management (`ProposalLifecycleManagerV1`)
*(Based on `tech_specs/governance_framework_spec.md` and future detailed proposal spec)*

*   **Valid Proposal Submission:**
    *   Test with correct proposal format, valid proposer DID, sufficient deposit (if applicable), valid signature.
    *   Expected: Proposal created in "Pending" or "Submitted" state; `ProposalSubmittedEvent` emitted.
*   **Invalid Proposal Submissions:**
    *   Missing required fields (title, description, type).
    *   Invalid proposer DID (not registered, insufficient reputation/stake if required for proposing).
    *   Insufficient proposal deposit.
    *   Invalid signature on submission.
    *   Proposal for parameters that don't exist or are not governable.
    *   Expected: Submission rejected with appropriate error code.
*   **Proposal State Transitions:**
    *   Draft -> Submitted (if applicable).
    *   Submitted -> ActiveVoting (e.g., after a review period or deposit confirmation).
    *   ActiveVoting -> Succeeded (after successful vote).
    *   ActiveVoting -> Failed (after failed vote or quorum not met).
    *   ActiveVoting -> Expired (if voting period ends without conclusion).
    *   Succeeded -> Executed (for proposals that trigger actions).
    *   Test invalid state transitions (e.g., trying to move from Failed to ActiveVoting).
    *   Expected: Correct state changes; relevant events emitted.
*   **Time-Based Transitions:**
    *   Test proposal expiry if not acted upon within a defined period.
    *   Test automatic transition from ActiveVoting to Failed/Expired if voting period ends.
    *   Expected: Correct state change based on mocked DLI time/block height.

### 3.B. Voting Mechanics (`VotingManagerV1`)
*(Based on `tech_specs/governance_framework_spec.md` and future detailed voting spec)*

*   **Valid Vote Casting:**
    *   Vote on an active proposal by an eligible voter DID.
    *   Test all valid vote options (Aye, Nay, Abstain).
    *   Correct calculation and recording of vote weight based on PoP/reputation at snapshot.
    *   Expected: `VoteCastEvent` emitted; vote recorded correctly.
*   **Invalid Vote Casting:**
    *   Voting on a non-existent proposal.
    *   Voting on a proposal not in "ActiveVoting" state.
    *   Voter DID not eligible (e.g., insufficient reputation/stake/PoP at snapshot).
    *   Invalid vote option.
    *   Attempting to vote twice by the same DID on a proposal (if not allowed by rules for weighted voting updates).
    *   Invalid signature on the vote.
    *   Vote cast after voting period ended.
    *   Expected: Vote rejected with appropriate error code.
*   **Vote Delegation (if implemented in a future version):**
    *   Valid delegation of voting power to another DID.
    *   Voting with delegated power (ensure original voter cannot also vote).
    *   Revocation of delegation.
    *   Cascading delegations (if allowed/disallowed).
    *   Expected: Correct accounting of delegated voting power.
*   **Vote Weight Calculation:**
    *   Test with various PoP/reputation scores to ensure vote weight is calculated correctly as per the defined formula.
    *   Test with zero or negative reputation (if possible) and its effect on voting power.
    *   Test snapshotting logic: ensure votes use power from the correct snapshot, not current power.

### 3.C. Vote Tallying & Outcome Determination (`VotingManagerV1`)
*   **Correct Tallying:**
    *   Scenarios with various distributions of Aye, Nay, Abstain votes and weights.
    *   Ensure correct sum of weights for each option.
*   **Quorum Rules:**
    *   Scenario where quorum (minimum total voting power participated) is met.
    *   Scenario where quorum is not met.
    *   Expected: Proposal fails if quorum not met, regardless of Aye/Nay ratio.
*   **Passage Thresholds:**
    *   Scenario where Aye votes meet simple majority (e.g., >50% of non-abstain votes).
    *   Scenario where Aye votes meet a supermajority (e.g., >66% or >75% of non-abstain votes) if required for specific proposal types.
    *   Scenario where Aye votes do not meet the required threshold.
    *   Expected: Correct determination of proposal success or failure based on threshold.
*   **Handling of Tie Votes:**
    *   Test how tie votes are handled (e.g., proposal fails, status quo maintained, or other defined rule).
*   **Outcome Determination:**
    *   Verify correct setting of `ProposalState.status` to `Succeeded` or `Failed`.
    *   Verify correct population of `ProposalRecordV1.final_tally_results`.
    *   Verify emission of `ProposalOutcomeFinalizedEvent`.

### 3.D. Governance Roles & Committees (if specific DLI logic exists)
*(Based on `tech_specs/governance_roles_committees_spec.md` or framework)*
*   If there's DLI-native logic for electing council members or assigning DIDs to committees:
    *   Test election/selection logic with various candidate sets and voting patterns/inputs.
    *   Test term limits, rotation, and removal logic.
*   If committees use on-DLI multi-signature accounts for actions:
    *   Test threshold signature logic (e.g., M-of-N signatures required to authorize an action).

### 3.E. Dispute Resolution Protocol Logic
*(Based on `tech_specs/dispute_resolution_protocol_spec.md`)*
*   **Dispute Submission:**
    *   Valid submission with all required fields, valid DIDs, valid `subject_ref`, fee paid (if mocked).
    *   Invalid submissions (missing fields, invalid DID, non-existent `subject_ref`, insufficient fee).
    *   Expected: `DisputeCaseV1` created in `OPEN` state; `DisputeOpenEvent` emitted.
*   **Evidence Submission:**
    *   Valid evidence submission by authorized party to an open case.
    *   Invalid: submitting to a closed case, unauthorized submitter, invalid evidence format.
    *   Expected: `EvidenceV1` created and linked; `EvidenceSubmittedEvent` emitted.
*   **State Transitions:**
    *   Test all valid transitions for `DisputeCaseV1.status` (e.g., OPEN -> EVIDENCE_GATHERING -> AWAITING_ADJUDICATION -> ADJUDICATION_IN_PROGRESS -> RESOLVED_* / WITHDRAWN).
    *   Test invalid transitions.
*   **Resolution via Community Vote (MVP focus):**
    *   Test that a dispute reaching "AWAITING_ADJUDICATION" correctly triggers the input to the governance proposal system (`ProposalLifecycleManagerV1`).
    *   Test that the outcome of the governance vote (from `VotingManagerV1`) correctly translates into the `ResolutionV1` data and updates the `DisputeCaseV1` status.

### 3.F. Parameter Management (`ParameterManagerV1`)
*(Based on `tech_specs/constitutional_framework_spec.md`)*
*   **Valid Parameter Update:**
    *   Proposal to update an existing, governable parameter with a value within its defined sane range.
    *   Test update of various data types (int, string, bool, custom structures if parameters can be complex).
    *   Expected: Parameter updated in DLI state; `ParameterUpdatedEvent` emitted.
*   **Invalid Parameter Updates:**
    *   Attempt to update a non-existent parameter.
    *   Attempt to update a parameter that is not governable (e.g., a constitutional constant).
    *   Proposed value is outside the defined sane range for the parameter (unless an "unsafe" override is explicitly tested).
    *   Proposal does not come from a successfully executed governance decision.
    *   Expected: Update rejected with appropriate error.
*   **Access Control:**
    *   Test that only authorized logic (i.e., a finalized governance proposal execution path) can trigger a parameter update.

### 3.G. Protocol Upgrade Activation Logic
*(Based on `tech_specs/protocol_upgrade_process_spec.md` or framework)*
*   **Activation Trigger:**
    *   Test activation based on specific DLI block height or network timestamp after a successful governance vote approving the upgrade.
*   **State Transition:**
    *   Verify that DLI nodes correctly switch behavior (e.g., enable new transaction types, use new validation rules, recognize new data structures) precisely at the activation point.
    *   Test behavior just before and just after activation.
*   **Version Flag Checks:**
    *   Ensure components correctly check and respect protocol version flags in DLI state or objects.

## 4. Testing Edge Cases & Security Vulnerabilities

For all governance components, tests should specifically target:
*   **Empty/Null Inputs:** How does the logic handle missing DIDs, empty proposal descriptions, zero-value parameters where not sensible?
*   **Extremely Large Inputs:** Very long strings, large numbers of voters or proposals (within reasonable simulation limits for unit tests – performance is more for integration/stress tests).
*   **Boundary Conditions:** Test values at the exact edge of valid ranges (e.g., minimum/maximum deposit, quorum/threshold values).
*   **Replay Attacks (Conceptual):** Design tests to ensure that submitting the exact same DLI event (e.g., a vote, a proposal) multiple times is handled correctly (e.g., rejected if it would lead to double counting or invalid state change, potentially through use of nonces or sequence numbers if applicable).
*   **Unauthorized Access/Bypass:** Attempts to call internal state-changing functions without proper authorization (e.g., directly trying to change a parameter without a governance vote). Ensure strong encapsulation.
*   **Integer Overflow/Underflow:** For vote tallies, parameter calculations, etc.
*   **Gas/Fee Logic (if applicable in future):** Test scenarios with insufficient gas/fees for governance transactions.

## 5. Mocking & Dependencies

Effective unit testing requires isolating the component under test.
*   **DLI State Access:**
    *   Provide mock implementations for DLI state readers. For example, when testing `VotingManagerV1`, mock the functions that would retrieve a voter's PoP balance or reputation score for a given DID at a specific (snapshotted) block height.
    *   Mock access to the current block height/timestamp.
    *   Mock access to the `ProtocolParametersV1` object.
*   **External Module Interactions:**
    *   Governance logic should ideally interact with other modules via DLI state changes and events rather than direct RPC calls.
    *   If direct calls are unavoidable in some utility functions, those external services must be mocked.
    *   For example, the `ProposalLifecycleManagerV1` might read the `ProtocolParametersV1` for proposal deposit amounts; this read should go via a mocked state accessor.
*   **Cryptography:** Use actual cryptographic libraries for signing and verification tests, but potentially use fixed keys/seeds for deterministic test outcomes where randomness is not the focus.

## 6. Test Environment & Tooling

*   **Language-Specific Frameworks:**
    *   **Go:** Standard `testing` package, `testify/suite` for structuring tests, `testify/assert` and `testify/require` for assertions, Go's built-in mocking capabilities or libraries like `gomock`.
    *   **Rust (if used):** Standard `testing` framework (`#[test]`), `assert!` macros, mocking libraries like `mockall`.
*   **CI/CD Pipeline Integration:**
    *   All governance unit tests MUST be executed as part of the CI pipeline on every commit/PR (as detailed in `roadmap/overall_mvp_implementation_roadmap.md`, section 8.1).
    *   Builds/merges MUST fail if any governance unit tests fail.
*   **Code Coverage:**
    *   Track code coverage for governance modules using integrated tooling (e.g., Go's `go test -cover`).
    *   Aim for very high coverage targets (e.g., >90-95%) for these critical components due to their sensitivity and impact on DLI stability.

---
This unit testing strategy, particularly when combined with TDD, forms a critical foundation for developing secure, reliable, and correct decentralized governance mechanisms for EchoNet. It will be a living document, updated as new governance features are specified and implemented.
