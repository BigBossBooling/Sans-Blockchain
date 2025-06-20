# DigiSocialBlock - Proof-of-Engagement (PoE) Protocol Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. PoE Protocol Overview

### 1.1. Purpose
The Proof-of-Engagement (PoE) Protocol within DigiSocialBlock (also referred to as EchoNet) is designed to:
*   **Incentivize Quality Engagement:** Reward users for meaningful interactions that add value to content and the platform.
*   **Enhance Content Discovery:** Surface high-quality content and interactions through reputation and scoring mechanisms.
*   **Provide Anti-Spam Signals:** Penalize low-quality or manipulative interactions, contributing to a healthier ecosystem.
*   **Distribute Rewards Fairly:** Compensate users proportionally to the value of their engagement, as assessed by the network.
*   **Build User Reputation:** Allow users to build a transferable reputation based on their positive contributions.

### 1.2. Core Loop
1.  **User Interaction:** A user performs an action (e.g., comments, reacts) represented by a `NexusInteractionRecordV1`.
2.  **PoE Quality Scoring & Validation:** Witnesses evaluate the interaction based on defined criteria and an algorithm, assigning it a "PoE Quality Score." This includes anti-gaming checks. The score (or its constituent parts) is recorded in the `WitnessProofV1` for the interaction.
3.  **Reputation Update:** The validated PoE Quality Score influences the interacting user's `reputationScore` (in their `NexusUserObjectV1`).
4.  **Reward Calculation & Distribution:** Periodically, PoE Quality Scores are aggregated, and rewards from a dedicated pool are distributed to users based on their contributions.

## 2. PoE-Eligible Interactions

Not all interaction types defined in `NexusInteractionRecordV1.interactionType` (from `tech_specs/dli_core_data_structures.md`) are necessarily eligible for generating PoE scores or rewards. Eligibility is determined by governance and protocol specification.

**Initially Eligible Interaction Types for PoE Scoring:**
*   **`COMMENT` (`INTERACTION_TYPE_V1_COMMENT`):** Textual comments on content. Quality is key.
*   **`QUALITY_REACTION` (A specific subset or weighted version of `INTERACTION_TYPE_V1_REACTION`):** Certain reactions deemed more valuable than others (e.g., "Insightful," "Helpful" vs. a simple "Like"). Standard "Likes" might have zero or very low PoE weight.
*   **`EFFECTIVE_SHARE` (A specific subset or outcome-based version of `INTERACTION_TYPE_V1_SHARE`):** Shares that lead to significant downstream engagement by other users. This might require a multi-stage scoring process.
*   **`ACCURATE_FLAG` (A specific subset or outcome-based version of `INTERACTION_TYPE_V1_FLAG`):** Flags on content that are subsequently confirmed as valid by community moderation or other validation processes.
*   **`VALUABLE_VOTE` (A specific subset of `INTERACTION_TYPE_V1_VOTE`):** Votes in specific contexts, like on content quality polls or certain governance proposals, that are deemed to contribute to ecosystem health.

**Interactions generally NOT eligible for direct PoE rewards (though they affect the ecosystem):**
*   Simple `LIKE` reactions (unless weighted as part of `QUALITY_REACTION`).
*   Basic `USER_RELATION` events like "Follow" (though following high-reputation users might indirectly benefit discovery).
*   `TIP` events (as they are direct monetary transfers).

## 3. `PoEScoreFactorsV1` Data Structure

This conceptual structure (or a set of governable on-DLI parameters) defines the weights, thresholds, and factors used in the PoE Quality Scoring algorithm. Its values are updatable via EchoNet governance.

```protobuf
syntax = "proto3";

package digisocialblock.poe.protocol;

// PoEScoreFactorsV1 defines governable parameters for PoE scoring.
// These values are illustrative and subject to change/tuning via governance.
message PoEScoreFactorsV1 {
  // --- General Factors ---
  // Weight of the actor's current reputation in modulating the PoE score.
  // e.g., 0.1 means a 10% influence. Higher reputation can boost score.
  double actor_reputation_weight = 1;
  // Minimum actor reputation to be eligible for PoE rewards on certain interactions.
  int64 min_reputation_for_reward = 2;

  // --- Comment Specific Factors ---
  // Base score for submitting a non-empty comment.
  double comment_base_score = 10;
  // Weight for comment length (e.g., per character or word, up to a cap).
  double comment_length_char_weight = 11;
  uint32 comment_max_chars_for_length_score = 12; // e.g., 1000
  // Bonus for comments receiving upvotes from high-reputation users (meta-interaction).
  double comment_high_rep_upvote_bonus = 13;
  // Conceptual: NLP/semantic analysis score weight (if a deterministic oracle existed).
  // double comment_semantic_quality_weight = 14; // Highly complex, likely post-MVP

  // --- Quality Reaction Specific Factors ---
  // Base scores for different types of "quality" reactions. Map: reaction_type_id -> score.
  // e.g., "insightful_reaction" -> 5.0, "helpful_reaction" -> 4.0
  map<string, double> quality_reaction_base_scores = 20;
  // Minimum consensus (e.g., number of similar quality reactions) to amplify score.
  uint32 reaction_consensus_threshold = 21;
  double reaction_consensus_bonus_multiplier = 22;

  // --- Effective Share Specific Factors ---
  // Base score for any share.
  double share_base_score = 30;
  // Multiplier for each new unique view generated by the share on the target content.
  double share_downstream_view_multiplier = 31;
  // Multiplier for each new unique quality interaction generated by the share.
  double share_downstream_interaction_multiplier = 32;
  // Time window for attributing downstream engagement (e.g., 72 hours).
  uint64 share_attribution_window_seconds = 33;

  // --- Accurate Flag Specific Factors ---
  // Base score for a flag that is later confirmed as valid by moderators/consensus.
  double flag_accuracy_confirmed_bonus = 40;
  // Penalty multiplier if a flag is deemed malicious or consistently wrong (applied to flagger's reputation).
  double flag_malicious_penalty_factor = 41;

  // --- Global Thresholds ---
  // Minimum PoE Quality Score an interaction must achieve to be considered for rewards or reputation updates.
  double min_poe_score_for_reward = 50;
  // Global decay factor for older interactions' influence (if applicable directly in PoE score vs. reputation system).
  // double global_poe_decay_factor_per_day = 51;

  // Version of this factors structure.
  string factors_schema_version = 99; // e.g., "1.0.0"
}
```

## 4. PoE Quality Score Calculation & Validation by Witnesses

When a PoE-eligible `NexusInteractionRecordV1` is submitted to the DLI, selected Witnesses perform scoring and validation.

### 4.1. Input Data for Scoring
*   The `NexusInteractionRecordV1` itself.
*   `actor_did`'s current `reputationScore` (from their `NexusUserObjectV1`).
*   Context of the `targetContentID` (e.g., its current popularity, creator's reputation - though this can be complex to make deterministic quickly).
*   Recent interaction history of `actor_did` (for velocity checks).
*   For comments: Potentially other comments on the same content (for originality checks).
*   For shares/flags: Downstream outcomes (may require delayed or multi-stage scoring).
*   The current `PoEScoreFactorsV1` retrieved from DLI state.

### 4.2. Scoring Algorithm (Conceptual Steps - must be deterministic)
The exact algorithm will be complex and require iteration. Witnesses must execute the *same* deterministic algorithm.
1.  **Initialization:** `poe_quality_score = 0.0`.
2.  **Anti-Gaming Pre-Checks (see 4.3):** If flagged for gaming, assign very low or zero score, or mark as invalid.
3.  **Base Score:** Look up `interaction_type` in `PoEScoreFactorsV1`.
    *   If `COMMENT`: `poe_quality_score += factors.comment_base_score`.
        *   Add score based on `comment_length_char_weight` up to `comment_max_chars_for_length_score`.
        *   (Conceptual) If meta-interactions (upvotes on comment) are available *at scoring time* and from high-reputation users, add `comment_high_rep_upvote_bonus`.
    *   If `QUALITY_REACTION`: `poe_quality_score += factors.quality_reaction_base_scores[reaction.type_id]`.
        *   (Conceptual) If consensus met, apply `reaction_consensus_bonus_multiplier`.
    *   If `EFFECTIVE_SHARE`: `poe_quality_score += factors.share_base_score`.
        *   *Note:* Downstream effects are hard to score instantly. This might be a base score, with future "PoE Update" events crediting the share if downstream activity occurs within `share_attribution_window_seconds`.
    *   If `ACCURATE_FLAG`: Base score might be low or zero initially. Bonus applied *later* if flag confirmed.
4.  **Actor Reputation Modulation:** `poe_quality_score *= (1 + actor_reputation_score * factors.actor_reputation_weight)`. (Ensure `actor_reputation_weight` is scaled appropriately).
5.  **Normalization/Caps:** Ensure score stays within a defined range.

*This algorithm MUST be precisely defined and implemented identically by all Witnesses.*

### 4.3. Anti-Gaming Checks during Validation
(Referencing a future `echonet_v3_anti_gaming_mechanisms.md` or detailing key rules here)
*   **Velocity Checks:**
    *   Limit on number of PoE-eligible interactions per `actor_did` per unit of time (e.g., X comments/hour).
    *   Limit on interactions per `actor_did` on a single `targetContentID`.
*   **Content Analysis (for comments/text payloads):**
    *   Basic spam keyword filtering (using a governable list).
    *   Minimum/maximum length constraints beyond scoring benefits.
    *   (Conceptual) Similarity checks against other comments on the same content to discourage copy-pasting. This is computationally intensive.
*   **Collusion Detection (Conceptual & Difficult):**
    *   Monitor for patterns of disproportionate mutual upvoting or interaction between small, closed groups of DIDs.
    *   Statistical analysis of interaction graphs (long-term effort).
*   **Reputation-Based Throttling:** Very low reputation users might have their interactions subject to stricter velocity limits or require more meta-validation before accruing significant PoE.
*   **Pufferfish Accounts:** Identify and reduce weight for new accounts with little history trying to game PoE.

If an interaction fails these checks, Witnesses may assign it a zero or significantly reduced PoE Quality Score, or even deem the `NexusInteractionRecordV1` itself invalid for DLI recording.

### 4.4. Recording in `WitnessProofV1`
*   The final, deterministically calculated `PoE_Quality_Score` (or its constituent parts, if the score is finalized later, e.g., for shares) is placed into the `specific_proof_data` field of the `WitnessProofV1` generated for the `NexusInteractionRecordV1`.
*   This makes the score attributable and auditable.

## 5. Reputation Updates (`NexusUserObjectV1.reputationScore`)

*   **Contribution:** Positive `PoE_Quality_Score`s from validated interactions contribute to the `actor_did`'s `reputationScore` in their `NexusUserObjectV1`.
*   **Formula (Conceptual):** `new_reputation = old_reputation + (sum_of_recent_poe_scores * reputation_gain_factor) - decay_factor`.
    *   `reputation_gain_factor` and `decay_factor` are governable parameters.
    *   Reputation might decay slowly over time if the user is inactive.
*   **Frequency:** Reputation updates are likely processed in batches (e.g., hourly or daily) by a DLI-native "ReputationUpdate" process or contract. This process would:
    1.  Scan `WitnessProofV1`s for validated interactions with PoE scores from the last period.
    2.  Aggregate scores for each `actor_did`.
    3.  Apply the update formula.
    4.  Generate DLI events to update the `reputationScore` in the respective `NexusUserObjectV1`s. These updates themselves are validated by Witnesses.

## 6. PoE Reward Calculation & Distribution Protocol

### 6.1. PoE Reward Pool
*   A dedicated DLI address/account, funded by the EchoNet token supply (as per `echonet_v3_tokenomics.md`) or other revenue streams, designated for PoE rewards.
*   Governance controls the rate at which funds are allocated to this pool for distribution.

### 6.2. Distribution Period
*   Rewards are calculated and distributed in discrete periods (e.g., daily or weekly). This is a governable parameter.

### 6.3. Reward Algorithm
1.  **Collect Eligible Interactions:** At the end of a `DistributionPeriod`, a DLI-native "PoEDistributionContract" (or equivalent automated process) gathers all `WitnessProofV1`s for PoE-eligible `NexusInteractionRecordV1`s that:
    *   Occurred within that `DistributionPeriod`.
    *   Have a `PoE_Quality_Score` (from `WitnessProofV1.specificProof_data`) greater than `PoEScoreFactorsV1.min_poe_score_for_reward`.
    *   Meet other criteria (e.g., `actor_did` has `min_reputation_for_reward`).
2.  **Calculate Total Points:** Sum all eligible `PoE_Quality_Score`s for the period to get `Total_Period_PoE_Points`.
3.  **Determine Period Allocation:** The `PoEDistributionContract` determines the total amount of ECHO tokens to be distributed from the `PoERewardPool` for this period (e.g., `PoolAllocation_For_Period`).
4.  **Calculate Individual Rewards:** For each eligible interaction `i` with `score_i`:
    *   `Reward_i = (score_i / Total_Period_PoE_Points) * PoolAllocation_For_Period`.
5.  **Aggregate Rewards per User:** Sum all `Reward_i` for each unique `actor_did`.
6.  **Initiate Payouts:** The `PoEDistributionContract` generates a batch DLI transaction (or multiple transactions) instructing the DLI to transfer the aggregated rewards to each `actor_did`'s associated wallet. This payout transaction itself is validated by Witnesses.

## 7. Data Structures for PoE Events

*   Primarily relies on `NexusInteractionRecordV1` and the PoE score being embedded in its `WitnessProofV1.specificProofData`.
*   `PoEScoreFactorsV1` (as defined above) is a key data structure, managed by governance.
*   For reward distribution, the "PoEDistributionContract" would internally manage lists of `(actor_did, aggregated_reward_amount)`.
*   A `PoERewardClaimV1` structure is **not proposed for MVP** to keep direct push payouts simpler.

## 8. Error Codes Specific to PoE Protocol

(Aligned with `echonet_v3_error_handling_strategy.md`)
*   `POE_ERROR_UNSPECIFIED (0)`
*   `POE_LOW_QUALITY_SCORE (1)`: Interaction quality score below threshold for reputation/reward.
*   `POE_SUSPECTED_GAMING (2)`: Interaction flagged by anti-gaming mechanisms.
*   `POE_INELIGIBLE_INTERACTION_TYPE (3)`: The type of interaction is not eligible for PoE scoring.
*   `POE_ACTOR_REPUTATION_TOO_LOW (4)`: Actor's reputation below minimum for PoE rewards.
*   `POE_REWARD_CALCULATION_ERROR (5)`: Internal error in reward aggregation/distribution.
*   `POE_INSUFFICIENT_REWARD_POOL (6)`: Reward pool empty or insufficient for current period's planned allocation.
*   `POE_INVALID_SCORE_FACTORS (7)`: Current PoE scoring factors are invalid or unavailable.
*   `POE_STALE_INTERACTION_FOR_SCORING (8)`: Interaction is too old to be considered for current PoE period.

## 9. Security & Fairness Considerations

*   **Scoring Algorithm Robustness:** The PoE Quality Scoring algorithm must be carefully designed to be as objective as possible and resistant to manipulation. This is an ongoing challenge requiring iteration and community governance over the `PoEScoreFactorsV1`.
*   **Transparency:** `PoEScoreFactorsV1` and the general scoring logic (though complex) should be transparent to the community.
*   **Witness Integrity in Scoring:** Witnesses must deterministically apply the scoring algorithm. Collusion among Witnesses to unfairly inflate scores for certain users would be a serious attack. Random committee selection and robust slashing for Witness misbehavior are partial mitigations.
*   **Preventing Reward Farming:** Anti-gaming mechanisms are crucial.
*   **Distribution Fairness:** The proportional distribution based on validated quality scores aims for fairness, but the definition of "quality" itself is subject to community consensus and evolution.
*   **Delayed Scoring Impact:** For interactions like shares or flags where true value is known later, the protocol must accommodate delayed or updated PoE scoring without creating excessive complexity or state management issues.

## 10. Protocol Versioning

The PoE Protocol, including `PoEScoreFactorsV1`, scoring algorithms, and reward distribution logic, will adhere to the versioning strategy in `echonet_v3_versioning_strategy.md`. Upgrades will be managed via EchoNet governance.

---
This specification provides the foundation for EchoNet's Proof-of-Engagement system. Its success will depend on careful tuning of scoring factors, robust anti-gaming measures, and active community governance.
