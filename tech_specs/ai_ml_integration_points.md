# DigiSocialBlock - AI/ML Integration Points Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. AI/ML Integration Philosophy for `EchoNet`

The integration of Artificial Intelligence / Machine Learning (AI/ML) models within the EchoNet (also referred to as DigiSocialBlock) DLI ecosystem is intended to augment network intelligence, improve user experience, and enhance platform integrity. However, it is approached with caution and clear principles:

*   **Role of AI/ML: Decision Support, Not Unilateral Control:**
    *   AI/ML outputs primarily serve as **signals or decision support** for human actors (e.g., Witnesses, community moderators, users) or for DLI-native scoring mechanisms (e.g., PoE).
    *   AI/ML models will **not** make final, unilateral censorship decisions or directly control core DLI consensus mechanisms without an explicit human-in-the-loop or governable DLI process interpreting their outputs.
*   **Transparency & Auditability:**
    *   Preference for open-source AI/ML models or models whose behavior, biases, and training data characteristics are well-understood and publicly documented, especially if their outputs can influence rewards, penalties, or content visibility.
    *   Decisions or scores influenced by AI/ML signals should be auditable (e.g., "EmPower1 Blockchain's AI/ML audit log principles" – an AIOracleService providing attestations for model version and input hash helps achieve this).
*   **Determinism for Consensus-Critical Operations:**
    *   If AI/ML model outputs are directly used by all participants in a consensus-critical calculation (e.g., all Witnesses must arrive at the exact same PoE score component based on an AI signal), the model execution **must be deterministic**. This often implies using specific model versions, quantized models, or simpler model architectures where input X always produces output Y on any compliant infrastructure.
    *   Non-deterministic AI/ML outputs are treated as heuristics or one of many inputs.
*   **Updatability & Governance:**
    *   The selection, registration, versioning, and updating of AI/ML models whose outputs are used by the core EchoNet protocol (e.g., by an `AIOracleService`) MUST be subject to EchoNet governance (as per `echonet_v3_governance_model.md`).
*   **Focus on Augmentation:** AI/ML should augment human capabilities and DLI mechanisms, not replace them entirely.

## 2. Potential Integration Points & Data Flows

### 2.A. Content Quality Scoring (Input to PoE for Original Content)

*   **Trigger:** A new `NexusContentObjectV1` (from `tech_specs/dli_core_data_structures.md`) is submitted for DLI validation.
*   **Data Input to AI Model (via `AIOracleService`):**
    *   Content body (or features extracted from it, e.g., text embeddings, image feature vectors).
    *   `CoreContentMetadataV1` fields (title, tags, description).
    *   Creator's DID (`creator_did`).
*   **AI Model Output (Conceptual `AIContentQualitySignalV1` - see Section 4):**
    *   Scores for readability, coherence, estimated originality (heuristic), topic relevance to tags, potential policy violation flags (e.g., hate speech likelihood, NSFW likelihood).
*   **Integration:**
    *   Witnesses, during their validation process (as per `tech_specs/pow_protocol.md`), could query a registered `AIOracleService` with the content data (or its hash if oracles can fetch it).
    *   The `AIOracleService` returns the `AIContentQualitySignalV1` along with an attestation.
    *   This signal becomes one of several inputs for Witnesses when calculating an initial PoE Quality Score for the content itself (if EchoNet implements direct PoE for content items, or for the creator's initial "publication interaction"). This is distinct from PoE on subsequent user interactions with the content.

### 2.B. Interaction Quality Scoring (Input to PoE for Comments, etc.)

*   **Trigger:** A new `NexusInteractionRecordV1` (e.g., a comment, as per `tech_specs/dli_core_data_structures.md`) is submitted for PoE validation.
*   **Data Input to AI Model (via `AIOracleService`):**
    *   Interaction payload (e.g., comment text from `CommentPayloadV1`).
    *   `actor_did`'s current reputation (from `NexusUserObjectV1`).
    *   Context of the parent `NexusContentObjectV1` (e.g., its topic, existing interactions).
*   **AI Model Output (Conceptual `AIInteractionQualitySignalV1` - see Section 4):**
    *   Scores for sentiment (positive, negative, neutral, toxic), constructiveness, spam likelihood, relevance to parent content/interaction.
*   **Integration:**
    *   Witnesses query the `AIOracleService` during validation of the `NexusInteractionRecordV1`.
    *   The returned `AIInteractionQualitySignalV1` is used as a factor in the PoE Quality Score calculation detailed in `tech_specs/poe_protocol.md`.

### 2.C. Anomaly/Spam Detection (General DLI Activity)

*   **Trigger:** Various DLI events: new `NexusUserObjectV1` registration, high frequency of `NexusInteractionRecordV1` from a single DID, unusual transaction patterns (if EchoNet had more complex DLI transactions), rapid follow/unfollow patterns.
*   **Data Input to AI Model (via `AIOracleService` or specialized DLI analytics nodes):**
    *   Event metadata (type, frequency, source DID).
    *   User interaction graphs (connections between DIDs based on interactions).
    *   Transaction velocity and patterns associated with a DID.
*   **AI Model Output (Conceptual `AIAnomalySignalV1` - see Section 4):**
    *   Scores for Sybil likelihood (detecting fake/multiple accounts controlled by one entity).
    *   Bot activity score (differentiating human-like from automated interaction patterns).
    *   Potential coordinated inauthentic behavior score.
*   **Integration:**
    *   **Witnesses:** Can use these signals during event validation to apply stricter scrutiny or flag events from DIDs with high anomaly scores. This might influence the `is_valid` decision or PoE scoring.
    *   **Reputation System (`tech_specs/poe_protocol.md`):** DIDs consistently flagged for anomalous behavior might have their `reputationScore` negatively impacted by the DLI's reputation update logic.
    *   **Community Moderation Tools:** These signals can be fed into dashboards for human moderators to investigate potentially problematic DIDs or activities.
    *   **Rate Limiting/Resource Allocation:** DIDs with high bot scores might face stricter rate limits for DLI interactions.

### 2.D. Content Categorization/Tagging Assistance (Optional Application Layer)

*   **Trigger:** New `NexusContentObjectV1` submitted.
*   **Data Input to AI Model:** Content body, title.
*   **AI Model Output:** Suggested tags, content categories, topics.
*   **Integration:** This is typically an application-layer feature. The EchoNet client application could query an AI service (which could be an `AIOracleService` or any third-party API) to get tag suggestions for the user during content creation. These tags, once confirmed by the user, become part of `CoreContentMetadataV1`. This is less critical for DLI core functions but enhances usability.

## 3. `AIOracleService` (Conceptual RPC Service)

To standardize access to AI/ML insights without requiring every Witness or DLI node to run complex models, a network of "AI Oracle" nodes is proposed.

*   **Purpose:** Provide on-demand AI/ML signal generation for DLI components.
*   **Operation:**
    1.  **Oracle Nodes:** Specialized EchoNet participants (potentially incentivized) that register to run specific, governance-approved AI/ML models (identified by `model_id` and `model_version`).
    2.  **Model Execution:** Oracles fetch the necessary data (e.g., content from DDS if given a `ContentID` or hash) and execute the requested model.
    3.  **RPC Definition (Conceptual - using Protobuf service syntax):**
        ```protobuf
        syntax = "proto3";

        package digisocialblock.aioracle.v1;

        import "google/protobuf/any.proto"; // For flexible signal payloads

        // Assuming AI signal structures like AIContentQualitySignalV1 are defined elsewhere
        // and can be packed into google.protobuf.Any.

        service AIOracleService {
          // Requests an AI-derived signal for given data using a specific model.
          rpc GetAISignal(GetAISignalRequest) returns (GetAISignalResponse);
        }

        message GetAISignalRequest {
          // Identifier for the data to be analyzed.
          // Could be a ContentID, InteractionID, DID, or hash of direct data.
          string data_reference_id = 1;

          // Optional: Direct data payload if small enough and not referenced by ID.
          bytes direct_data_payload = 2;

          // Type of data being provided (e.g., "content_object", "interaction_record_comment").
          // Helps oracle select appropriate pre-processing.
          string data_type_hint = 3;

          // Identifier of the AI/ML model to be used.
          // Registered via EchoNet governance.
          string model_id = 4;

          // Specific version of the model.
          string model_version = 5;

          // Optional: Parameters specific to this model invocation.
          google.protobuf.Any model_params = 6;
        }

        message GetAISignalResponse {
          // The requested data_reference_id or a hash of direct_data_payload.
          string input_data_echo_id = 1;

          string model_id_echo = 2;
          string model_version_echo = 3;

          // The AI-derived signal payload, packed into Any.
          // e.g., could contain AIContentQualitySignalV1, AIInteractionQualitySignalV1, etc.
          google.protobuf.Any ai_signal_payload = 4;

          // Timestamp of when the signal was generated by the oracle.
          google.protobuf.Timestamp oracle_timestamp = 5;

          // DID of the oracle node that generated this signal.
          string oracle_did = 6;

          // Signature by the oracle_did over (input_data_echo_id, model_id_echo, model_version_echo, hash(ai_signal_payload), oracle_timestamp).
          // This is the oracle's attestation.
          bytes oracle_signature = 7;

          // Optional: Error if signal generation failed.
          AIOracleError error = 8;
        }

        message AIOracleError {
          int32 code = 1;
          string message = 2;
        }
        ```
*   **Model Registration & Governance:**
    *   A process managed by EchoNet governance will define how AI/ML models are proposed, reviewed (for performance, potential biases, resource requirements, determinism if needed), tested, and officially registered with a unique `model_id` and `model_version`.
    *   Only registered models can be requested via the `AIOracleService` for DLI-relevant tasks.
*   **Determinism/Consistency:**
    *   For AI signals directly influencing DLI consensus (e.g., a score component used by all Witnesses), if multiple oracles are queried for the same input, model, and version, they **MUST** produce bitwise identical `ai_signal_payload`s. This may require specific model architectures, quantization, and standardized execution environments for Oracle nodes.
    *   If determinism cannot be guaranteed for a particular model, its output is treated as a non-binding heuristic.

## 4. Data Structures for AI Signals (Protobuf3 Examples)

These structures are examples and would be packed into `google.protobuf.Any` in the `GetAISignalResponse.ai_signal_payload`.

```protobuf
syntax = "proto3";

package digisocialblock.ai.signals.v1;

import "google/protobuf/struct.proto"; // For flexible map-like structures

// Signal for content quality assessment.
message AIContentQualitySignalV1 {
  // e.g., Flesch reading ease, Gunning fog index. Normalized to 0.0 - 1.0.
  float readability_score = 1;
  // Semantic coherence of the text. Normalized to 0.0 - 1.0.
  float coherence_score = 2;
  // Heuristic score for originality (e.g., based on n-gram rarity, not true plagiarism detection). 0.0 - 1.0.
  float originality_heuristic_score = 3;
  // Map of policy violation likelihoods. Key: policy category (e.g., "hate_speech", "spam", "nsfw"). Value: score (0.0 - 1.0).
  map<string, float> policy_violation_scores = 4;
  // Relevance of content to its declared tags/category. 0.0 - 1.0.
  float topic_relevance_score = 5;
  // Overall quality score, potentially a weighted average of other scores. 0.0 - 1.0.
  float overall_quality_score = 6;
}

// Signal for interaction quality assessment (e.g., for comments).
message AIInteractionQualitySignalV1 {
  // Sentiment score: e.g., -1.0 (very negative) to 1.0 (very positive).
  float sentiment_score = 1;
  // Is the interaction constructive or destructive? 0.0 - 1.0.
  float constructiveness_score = 2;
  // Likelihood of being spam. 0.0 - 1.0.
  float spam_likelihood_score = 3;
  // Relevance to parent content or interaction. 0.0 - 1.0.
  float relevance_score = 4;
  // Toxicity score. 0.0 - 1.0 (higher is more toxic).
  float toxicity_score = 5;
}

// Signal for DLI-level anomaly detection.
message AIAnomalySignalV1 {
  // Likelihood that the DID is part of a Sybil network. 0.0 - 1.0.
  float sybil_likelihood_score = 1;
  // Likelihood that activity from this DID is automated/bot-like. 0.0 - 1.0.
  float bot_activity_score = 2;
  // Score indicating potential participation in coordinated inauthentic behavior. 0.0 - 1.0.
  float coordinated_behavior_score = 3;
}
```

## 5. Security & Trust for AI Integration

*   **Oracle Accountability:** `AIOracleService` responses are signed by `oracle_did`. Oracles build reputation. Consistently faulty or malicious oracles (e.g., signing incorrect model outputs, or failing to meet SLA for registered models) can be penalized by EchoNet governance (stake slashing if oracles are staked, reputation loss).
*   **Model Transparency & Explainability:** While full explainability of complex models is hard, efforts should be made to:
    *   Use models whose general behaviors and failure modes are understood.
    *   Clearly document the purpose, training data概览, and known limitations of registered models.
*   **Bias Mitigation:** AI models can inherit biases from their training data. A continuous effort, guided by governance and community feedback, is needed to:
    *   Audit registered models for significant biases.
    *   Promote research and use of bias mitigation techniques.
    *   Ensure diverse datasets for training where possible.
*   **Human Oversight & Appeal:**
    *   AI-derived signals are inputs, not final judgments for critical actions like account suspension or large reward denials.
    *   Clear mechanisms for users to appeal decisions they believe were unfairly influenced by an AI signal.
    *   Community moderation tools will incorporate these AI signals but allow human moderators to override or interpret them.
*   **Input Integrity:** The `AIOracleService` must ensure it's operating on the correct, untampered input data. Using hashes (`data_reference_id`) that DLI components can independently verify is crucial.

## 6. Versioning

*   **AI Models:** Each registered AI model MUST have a unique `model_id` and be strictly versioned (`model_version`). Updates to models require registration of a new version.
*   **AI Signal Structures:** The Protobuf definitions for signal payloads (e.g., `AIContentQualitySignalV1`) are versioned as per standard Protobuf evolution practices and the overall `echonet_v3_versioning_strategy.md`. Changes to these structures may require corresponding updates in consuming components (Witnesses, PoE logic).
*   **AIOracleService API:** The RPC service itself is versioned.

---
This specification provides the foundational hooks and conceptual framework for integrating AI/ML capabilities into EchoNet. The emphasis is on leveraging AI as a supportive tool within a human-governed, transparent, and auditable decentralized system. Actual AI model development and training are separate, ongoing research and engineering efforts.
