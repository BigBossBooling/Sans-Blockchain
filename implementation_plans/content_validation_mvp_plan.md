# EchoNet - Content Validation & Incentives MVP Implementation Plan

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Content Validation MVP - Goals & Scope

### 1.1. Goals
The primary goal of the Content Validation & Incentives Minimal Viable Product (MVP) is to implement the foundational elements of the Proof-of-Engagement (PoE) protocol and the basic integration points (interfaces and stubs) for future AI/ML-driven content/interaction quality signals. This MVP will demonstrate the core feedback loop of interaction -> scoring -> reputation update -> conceptual reward logging.

The Content Validation MVP must demonstrate:
*   Processing of specific `NexusInteractionRecordV1` types (e.g., comments, predefined "quality" reactions) by MVDLI Witnesses.
*   Calculation of a basic PoE Quality Score for these interactions using a simplified, deterministic algorithm and fixed scoring factors.
*   Integration of stubbed `AIOracleService` calls within the PoE scoring logic, using predefined/dummy AI signals.
*   Updating the `actor_did`'s `NexusUserObjectV1.reputationScore` based on the calculated PoE Quality Score.
*   Logging of potential PoE rewards to a conceptual reward pool (no actual token movement).
*   Client-side capability to submit PoE-eligible interactions.

### 1.2. Scope
**In Scope for Content Validation MVP:**
*   Witness logic to identify and process PoE-eligible interaction types (e.g., comments, a specific "quality_reaction" type).
*   Implementation of a simplified, deterministic PoE Quality Scoring algorithm (as per `tech_specs/poe_protocol.md`) using hardcoded or configuration-file-based `PoEScoreFactorsV1` for MVP.
*   Basic anti-gaming checks integrated into PoE scoring (e.g., velocity limits per DID, min/max length for comments).
*   Definition of the gRPC/RPC interface for the `AIOracleService` (as per `tech_specs/ai_ml_integration_points.md`).
*   Implementation of a **stubbed** `AIOracleService` that returns predefined or randomized `AIContentQualitySignalV1` / `AIInteractionQualitySignalV1` for testing purposes. No actual AI model training or execution.
*   Integration of these dummy AI signals as factors into the PoE Quality Scoring algorithm.
*   DLI-native logic to update the `reputationScore` in `NexusUserObjectV1` based on validated PoE Quality Scores (direct state update for MVP, batching later).
*   A conceptual DLI-level variable or log for a "PoE Reward Pool" and logic to calculate and *log* potential reward distributions based on accumulated PoE scores over a defined period. **No actual token transfers or wallet interactions for rewards in this MVP.**
*   Extension of the test client/CLI to submit PoE-eligible interactions.

**Non-Goals for Content Validation MVP:**
*   Development or integration of actual AI/ML models for content quality, interaction quality, or anomaly detection.
*   Full-scale, on-DLI PoE reward distribution involving real token value or complex treasury management.
*   Advanced anti-gaming measures requiring complex pattern analysis or machine learning.
*   Governance mechanisms for updating `PoEScoreFactorsV1` or registering AIOracleService models (factors are fixed for MVP).
*   Sophisticated PoE score adjustments based on downstream impact (e.g., for shares) or delayed outcomes (e.g., for flags).
*   User interface for viewing detailed PoE scores, reputation history, or managing AI signal preferences.
*   Slashing conditions related to PoE abuse by Witnesses (beyond basic PoW integrity).

## 2. Key Implementation Tasks & Prioritization for Content Validation MVP

These tasks build upon the MVDLI Core and the Identity & Privacy MVP.

*   **Task 3.1: PoE-Eligible Interaction Processing (MVDLI Witness Logic)**
    *   **Description:** Extend the MVDLI PoW Witness validation logic to identify specific `NexusInteractionRecordV1.interactionType` values (e.g., `COMMENT`, a predefined `QUALITY_REACTION` type) as PoE-eligible. Route these for further PoE-specific processing after basic DLI validation.
    *   **Inputs:** `tech_specs/poe_protocol.md` (Section 2), `tech_specs/dli_core_data_structures.md` (for `NexusInteractionRecordV1`), MVDLI PoW framework.
    *   **Priority:** **High**.
*   **Task 3.2: Basic PoE Quality Scoring Algorithm Implementation**
    *   **Description:** Implement a simplified, deterministic version of the PoE Quality Scoring algorithm within the Witness logic. This will use fixed (hardcoded or config-based) weights from a conceptual `PoEScoreFactorsV1` for the MVP. The algorithm should take basic interaction properties (e.g., comment length, reaction type) and actor's current reputation as inputs. Include rudimentary anti-gaming checks like per-DID velocity limits for PoE interactions or min/max comment length. The calculated score should be included in the `WitnessProofV1.specificProofData`.
    *   **Inputs:** `tech_specs/poe_protocol.md` (Sections 4, 5), `tech_specs/dli_core_data_structures.md`.
    *   **Priority:** **High**.
*   **Task 3.3: `AIOracleService` Interface & Stub Implementation**
    *   **Description:** Define the gRPC (or equivalent RPC) service interface for `AIOracleService` including `GetAISignalRequest` and `GetAISignalResponse` messages (as per `tech_specs/ai_ml_integration_points.md`). Implement a **stub version** of this service. This stub will not run any AI models but will return predefined or randomized `AIContentQualitySignalV1` or `AIInteractionQualitySignalV1` based on the `data_type_hint` or `model_id` in the request. This allows testing the integration flow.
    *   **Inputs:** `tech_specs/ai_ml_integration_points.md` (Sections 3, 4).
    *   **Priority:** **Medium** (Interface definition is high, stub allows parallel development).
*   **Task 3.4: Integration of AIOracleService Stub into PoE Scoring**
    *   **Description:** Modify the PoE Quality Scoring algorithm (Task 3.2) in the Witness logic to make a call to the (stubbed) `AIOracleService` for relevant interactions. Incorporate the dummy AI signals returned by the stub as additional factors in the PoE Quality Score calculation.
    *   **Inputs:** Task 3.2 output, Task 3.3 output.
    *   **Priority:** **Medium**.
*   **Task 3.5: Reputation Score Update (`NexusUserObjectV1`)**
    *   **Description:** Implement DLI-native logic (this could be a conceptual "ReputationUpdateContract" triggered by Witnesses, or a direct state modification function callable after PoW consensus on an interaction) to update the `reputationScore` field in the actor's `NexusUserObjectV1`. The update amount will be derived from the `PoE_Quality_Score` in the `WitnessProofV1`. For MVP, this can be a direct, immediate update rather than complex batching.
    *   **Inputs:** `tech_specs/poe_protocol.md` (Section 5), `tech_specs/dli_core_data_structures.md` (for `NexusUserObjectV1`), MVDLI state management.
    *   **Priority:** **Medium**.
*   **Task 3.6: Rudimentary PoE Reward Pool & Distribution Logic (Conceptual for MVP)**
    *   **Description:**
        *   Conceptually define a DLI global variable representing the "PoE Reward Pool" (e.g., just a counter).
        *   Implement basic DLI-native logic (e.g., as part of a periodic dummy "end_of_epoch" event processed by Witnesses) to:
            *   Iterate through `WitnessProofV1`s for PoE-eligible interactions within a defined (short for MVP) period.
            *   Calculate potential reward shares based on their `PoE_Quality_Score` (as per `tech_specs/poe_protocol.md`, Section 6.3).
            *   **Log** these calculated shares to the DLI state or node logs (e.g., "DID X earned Y conceptual_reward_units"). **No actual token transfers.**
    *   **Inputs:** `tech_specs/poe_protocol.md` (Section 6).
    *   **Priority:** **Low** (Logging/calculation is sufficient for MVP to prove the concept).
*   **Task 3.7: Client-Side Submission of PoE-Eligible Interactions**
    *   **Description:** Extend the MVDLI test client/CLI tool to allow submission of `NexusInteractionRecordV1` instances that are designated as PoE-eligible (e.g., a command to post a comment on a known `ContentID`, a command to issue a "quality_reaction").
    *   **Inputs:** `tech_specs/dli_core_data_structures.md`, MVDLI client/CLI.
    *   **Priority:** **Medium** (Essential for testing the end-to-end PoE flow).

## 3. Dependencies on MVDLI Core (Module 1) & Identity/Privacy MVP (Module 2)

This MVP relies heavily on previously established components:
*   **MVDLI Core (Module 1 - from `implementation_plans/dli_core_mvp_plan.md`):**
    *   Functioning P2P network for event gossip (`NexusInteractionRecordV1`, `WitnessAttestationV1`, `WitnessProofV1`).
    *   MVDLI Witness validation framework to process these new interaction events and their PoE scores.
    *   Basic DDS for storing `NexusInteractionRecordV1` and their `WitnessProofV1`s.
    *   Basic Discovery (DHT) may be needed if PoE scores depend on resolving context about content or users.
    *   Core data structures and canonicalization/hashing utilities.
*   **Identity & Privacy MVP (Module 2 - from `implementation_plans/identity_privacy_mvp_plan.md`):**
    *   Actors (`actor_did`) must be resolvable `did:echonet` DIDs.
    *   `NexusUserObjectV1` must exist and be updatable to store and reflect changes in `reputationScore`.

## 4. Development & Testing Approach for Content Validation MVP

*   **Unit Tests:**
    *   For the PoE Quality Scoring algorithm (given various inputs and fixed factors, ensure deterministic output).
    *   For the `AIOracleService` stub (ensure it returns expected dummy data for given requests).
    *   For reputation update logic.
*   **Integration Tests (within the MVDLI simulated network environment):**
    *   Submit various PoE-eligible interactions using the extended CLI tool.
    *   Verify that MVDLI Witnesses correctly identify these, calculate PoE Quality Scores (including dummy AI signals), and generate `WitnessProofV1`s containing these scores.
    *   Query/verify that `NexusUserObjectV1.reputationScore` for the actor DID is updated as expected.
    *   Inspect DLI logs to confirm that conceptual reward calculations are being performed and logged correctly.
    *   Test basic anti-gaming checks (e.g., submitting interactions above velocity limits and seeing them scored low or rejected).

## 5. Next Steps After Content Validation MVP

Successful completion of this MVP will pave the way for more advanced features:
1.  **Implement More Sophisticated Anti-Gaming Measures:** Based on data and patterns observed from MVP usage.
2.  **Develop/Integrate Real AI/ML Models:** Progress the `AIOracleService` from a stub to an actual service that calls one or more registered AI/ML models for quality/anomaly detection. This involves model training, deployment, and Oracle node implementation.
3.  **Full Implementation of PoE Reward Distribution:** Integrate with the EchoNet tokenomics to distribute actual token rewards from the PoE Reward Pool. This requires robust wallet interactions and secure DLI-native contract logic.
4.  **Governance Mechanisms:** Implement DLI governance for managing `PoEScoreFactorsV1`, registering AI models for the `AIOracleService`, and adjusting other PoE protocol parameters.
5.  **Refine Reputation System:** Potentially add decay, more input factors beyond PoE, and more nuanced reputation effects.
6.  **Expand PoE to More Interaction Types:** Include outcome-based scoring for shares, flags, etc.

This Content Validation & Incentives MVP plan focuses on establishing the core mechanics of PoE and AI signal integration, providing a foundation for EchoNet's unique approach to fostering a quality user experience and fair compensation.
