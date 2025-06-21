# DigiSocialBlock (Nexus Protocol) - Phase 7: Core MVP Implementation & Testing Plan

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Phase Objective

The main objective of Phase 7: Core MVP Implementation & Testing is to implement and rigorously test the core Minimum Viable Product (MVP) features of the DigiSocialBlock platform (underpinned by the EchoNet DLI Protocol). This phase aims to translate the comprehensive technical specifications (from `tech_specs/`) and the detailed MVP implementation roadmaps (from `roadmap/overall_mvp_implementation_roadmap.md` and individual module MVP plans) into a functional, deployable, and well-tested initial version of the application.

## 2. Core Implementation Principles & Methodologies

This section outlines the foundational principles and methodologies that will guide all development efforts during Phase 7.

### 2.1. Overarching Philosophy
All development work in Phase 7 will be guided by the **Expanded KISS Principle**, previously detailed in `roadmap/overall_mvp_implementation_roadmap.md` (Section 3.1). This ensures that solutions are:
*   **K**nown (clear purpose, well-defined interfaces).
*   **I**terative and Integrated (MVP as first iteration, smooth module integration).
*   **S**ystematic and Synchronized (foundations for scalability, components work in synergy).
*   **S**ecure and Sensed (foundational security, awareness of landscape).
*   **S**timulating and Sustainable (demonstrable value, basis for future engagement).
This philosophy is critical for building a robust, focused, and impactful initial version of EchoNet.

### 2.2. Key Development Methodologies
*   **Agile Sprint-Based Development:**
    *   Implementation will be managed using agile sprints, proposed as 2-week cycles.
    *   Each sprint will aim to deliver a small, demonstrable increment of functionality from the MVP tasks outlined in `roadmap/overall_mvp_implementation_roadmap.md`.
    *   Standard agile ceremonies will be conceptually adopted:
        *   **Sprint Planning:** To select and plan tasks for the upcoming sprint.
        *   **Daily Stand-ups:** Brief daily check-ins for team synchronization and impediment identification.
        *   **Sprint Reviews:** Demonstrate completed functionality at the end of each sprint to stakeholders (internal for MVP).
        *   **Sprint Retrospectives:** Reflect on the sprint process to identify areas for improvement.
*   **Test-Driven Development (TDD):**
    *   As detailed in `roadmap/overall_mvp_implementation_roadmap.md` (Section 8.2), TDD will be the primary methodology for developing all new core DLI logic, DLI-native "contract" logic (conceptual Go modules for MVP), and critical library functions across all MVP modules.
    *   This ensures robust unit test coverage from the start and helps drive clear, testable designs.
*   **Continuous Integration/Continuous Delivery (CI/CD):**
    *   All code will be integrated into a CI/CD pipeline as defined in `echonet_v3_cicd_strategy.md` and reiterated in `roadmap/overall_mvp_implementation_roadmap.md` (Section 8.1).
    *   This pipeline will automate builds, linting, static analysis, unit testing, scoped integration testing, and security scans on every commit/pull request.
    *   Automated deployment to a simulated testnet environment will occur post-merge to the `develop` branch, triggering E2E tests.

### 2.3. Focus on Code Integrity & Quality
*   **Code Reviews:** Mandatory peer code reviews are required for all pull requests (PRs) before merging into the main development branch (`develop`). Reviews will focus on correctness, design adherence, coding standards, test coverage, and security.
*   **Coding Standards:** Strict adherence to defined coding standards for Go (e.g., Effective Go, standard Go project layout) and any other languages used (e.g., Python for utility scripts). Linters and formatters in the CI/CD pipeline will help enforce this.
*   **Documentation ("Living Documentation"):**
    *   Technical documentation and code comments must be updated alongside development, as part of the same PR that introduces code changes. This aligns with `echonet_v3_documentation_strategy.md` and `roadmap/overall_mvp_implementation_roadmap.md` (Section 8.3).
    *   Focus includes GoDoc for public APIs, comments for complex logic, and updates to relevant `tech_specs/*.md` or `implementation_plans/*.md` if design decisions evolve during implementation.

### 2.4. Collaboration & Communication
*   The collaboration tools and processes outlined in `roadmap/overall_mvp_implementation_roadmap.md` (Section 8.3) will be adopted. This includes:
    *   **Version Control:** Git, hosted on GitHub/GitLab.
    *   **Branching Strategy:** Gitflow-like model (`main`, `develop`, feature branches, PRs).
    *   **Issue Tracking:** Use of GitHub Issues, GitLab Issues, or Jira for managing tasks, bugs, and features.
    *   **Communication:** Dedicated chat channels (e.g., Slack/Discord) and regular team meetings.
    *   **Knowledge Sharing:** Use of an internal wiki or shared document space.

## 3. Overall Goals for Phase 7

The high-level goals for this implementation and testing phase are:

*   **Implement MVDLI Core (Module 1 MVP):** Successfully build and test the foundational Distributed Ledger Inspired (DLI) core, including P2P networking, basic Proof-of-Witness (PoW) consensus, rudimentary Distributed Data Stores (DDS), and basic Discovery (DHT) functionalities.
*   **Implement User Identity & Privacy MVP (Module 2 MVP):** Integrate `did:echonet` for decentralized identity management and the basic On-System Data Consent protocol.
*   **Implement Content Validation & Incentives MVP (Module 3 MVP):** Implement the initial Proof-of-Engagement (PoE) protocol for scoring interactions, updating reputation, and conceptually logging rewards, including stubbed interfaces for AI/ML signals.
*   **Implement Decentralized Governance MVP (Module 4 MVP):** Implement basic on-DLI governance mechanisms, allowing for simple proposal submission (text, parameter change), PoP/reputation-weighted voting, and DLI parameter updates.
*   **Module Integration:** Ensure all four MVP modules are cohesively integrated into a single, functional application that demonstrates the core end-to-end user stories.
*   **Thorough Testing:** Perform comprehensive unit, integration, simulated network, and end-to-end (E2E) testing as outlined in the `roadmap/overall_mvp_implementation_roadmap.md` and `testing_strategies/governance_unit_tests_strategy.md` (and other forthcoming testing strategy documents).
*   **Initial Performance Bottleneck Identification:** Identify and, where feasible within MVP scope, address any critical performance bottlenecks observed during testing.
*   **MVP Readiness:** Prepare a functional MVP suitable for internal User Acceptance Testing (UAT) and gathering initial feedback to inform subsequent development cycles.
*   **Documentation:** Maintain and update technical documentation (code comments, API docs from Protobufs, setup guides) throughout the implementation process.

## 4. High-Level Overview of Phase 7 Approach

Phase 7 will be characterized by an agile, iterative development process, focusing on delivering functional increments of the MVP. Key aspects of the approach include:

*   **Sprint-Based Development:** Work will likely be organized into sprints (or equivalent agile cycles), each focusing on a subset of tasks from the MVP implementation plans.
*   **Continuous Integration/Continuous Deployment (CI/CD):** The CI/CD pipeline established conceptually in `echonet_v3_cicd_strategy.md` and detailed for MVP in `roadmap/overall_mvp_implementation_roadmap.md` will be fully utilized to automate builds, testing, and deployments to simulated test environments.
*   **Test-Driven Development (TDD):** TDD will be the primary methodology for core logic development, ensuring robustness and comprehensive unit test coverage.
*   **Iterative Refinement:** Regular testing and internal reviews will provide feedback for iterative refinement of components as they are built and integrated.
*   **Focus on MVP Scope:** Strict adherence to the defined MVP scope for each module to ensure timely delivery of core functionalities.

## 5. Implementation Sprints: Foundational DLI EchoNet Core & Network (Module 1 MVP)

This section details the implementation sprints for the Foundational DLI EchoNet Core & Network (Module 1 MVP). These sprints will build the essential layer of EchoNet, upon which all other MVP modules (Identity, Content Validation, Governance) will depend. The tasks are derived from `roadmap/overall_mvp_implementation_roadmap.md` (Section 4.1) and `implementation_plans/dli_core_mvp_plan.md`.

### 5.1. Component: Core Data Structures & Serialization
*   **Sprint Goal(s):** Implement and unit test all core Protobuf data structures for MVDLI.
*   **Key Implementation Tasks:** Create `.proto` files; generate Go code; unit test serialization.
*   **Technology Integration Notes:** Protocol Buffers v3, `protoc-gen-go`.
*   **Key Deliverables:** Go library of data structures; passing unit tests.
*   **Unit/Integration Test Focus:** Correctness, binary/JSON serialization, default values.

### 5.2. Component: Content Hashing & Timestamping Logic
*   **Sprint Goal(s):** Implement functions for `ContentID`s, `core_metadata_hash`, `chunk_id`s; handle timestamps.
*   **Key Implementation Tasks:** Develop canonicalization for pre-images; SHA-256 hashing; timestamp utilities. Unit test with known vectors.
*   **Technology Integration Notes:** Go crypto libraries, Protobuf timestamps.
*   **Key Deliverables:** Hashing library; timestamp functions.
*   **Unit/Integration Test Focus:** Deterministic hashing; correct canonicalization; timestamp validation.

### 5.3. Component: Basic P2P Networking Layer
*   **Sprint Goal(s):** Establish P2P network where nodes discover and exchange messages via pub/sub. Node identity.
*   **Key Implementation Tasks:** Init `go-libp2p` Host (ED25519 keys); bootstrap list; mDNS; pub/sub topics. Test app for messages.
*   **Technology Integration Notes:** `go-libp2p`, `go-libp2p-pubsub`, mDNS.
*   **Key Deliverables:** Nodes join P2P, discover peers, use pub/sub.
*   **Unit/Integration Test Focus:** Connection, discovery, pub/sub propagation.

### 5.4. Component: Rudimentary DDS Protocol
*   **Sprint Goal(s):** Implement basic `StoreChunk`/`RetrieveChunk` RPCs on MVDLI SSN (in-memory/file storage). Client logic.
*   **Key Implementation Tasks:** Define DDS RPC Protobufs; implement SSN RPC handlers; implement client library.
*   **Technology Integration Notes:** Go gRPC, Protobufs, `go-libp2p` streams.
*   **Key Deliverables:** SSN stores/serves chunks via RPC; client library.
*   **Unit/Integration Test Focus:** RPC handlers (mock storage); client store/retrieve success; hash validation.

### 5.5. Component: Rudimentary PoW Protocol
*   **Sprint Goal(s):** Fixed Witnesses receive/validate `NexusContentObjectV1` metadata, generate/gossip attestations, aggregate into `WitnessProofV1`.
*   **Key Implementation Tasks:** Witness subscribes to content; basic validation (schema, `ContentID`); `WitnessAttestationV1` creation (no VRF); gossip attestations; aggregate attestations.
*   **Technology Integration Notes:** `go-libp2p-pubsub`, Go crypto (placeholder sigs if full DID not ready), Protobufs.
*   **Key Deliverables:** Witnesses validate, produce attestations; `WitnessProofV1` formed.
*   **Unit/Integration Test Focus:** Validation logic; content submission -> gossip -> validation -> attestation -> proof aggregation.

### 5.6. Component: Basic Discovery Protocol (DHT)
*   **Sprint Goal(s):** Nodes join Kademlia DHT; `STORE_VALUE` (`ContentID` -> SSN multiaddress) & `FIND_VALUE` work.
*   **Key Implementation Tasks:** Integrate `go-libp2p-kad-dht`; DHT bootstrap; wrapper functions for PUT/GET.
*   **Technology Integration Notes:** `go-libp2p-kad-dht`.
*   **Key Deliverables:** Nodes join DHT; content/chunk locations publishable/resolvable.
*   **Unit/Integration Test Focus:** DHT bootstrap; PUT/GET success across nodes.

### 5.7. Component: Initial Mobile Node Host Functionality (CLI/Test App)
*   **Sprint Goal(s):** CLI app performs full content lifecycle: create, chunk, store on SSN, submit for PoW, retrieve by `ContentID`.
*   **Key Implementation Tasks:** Develop CLI commands; integrate client libraries (hashing, DDS, PoW, Discovery); orchestrate operations.
*   **Technology Integration Notes:** Go CLI framework (e.g., Cobra), all DLI client libs.
*   **Key Deliverables:** CLI tool demonstrates E2E MVDLI core. Achieves Milestone M1.B.
*   **Unit/Integration Test Focus:** E2E CLI tests for content lifecycle; data integrity.

---
## 6. Implementation Sprints: User Identity & Privacy (Module 2 MVP)

This section details the implementation sprints for the User Identity & Privacy features (Module 2 MVP). These sprints build upon a functional MVDLI Core (Module 1 MVP), leveraging its P2P networking, event validation, data storage, and discovery capabilities. The tasks are derived from `roadmap/overall_mvp_implementation_roadmap.md` (Section 4.2) and `implementation_plans/identity_privacy_mvp_plan.md`.

### 6.1. Component: `did:echonet` Library/SDK Functions (Client-Side)
*   **Sprint Goal(s):** Develop and test a client-side Go library for `did:echonet` and `NexusDIDDocumentV1` management, including signing/verification.
*   **Key Implementation Tasks:** Ed25519 keygen; `did:echonet` identifier generation; `NexusDIDDocumentV1` construction (Protobufs, `VerificationMethod`, `publicKeyMultibase`); sign/verify functions.
*   **Technology Integration Notes:** Go `crypto/ed25519`, SHA256, Multibase libraries. `NexusDIDDocumentV1` Protobuf.
*   **Key Deliverables:** Tested Go library for client-side DID operations.
*   **Unit/Integration Test Focus:** Keygen uniqueness; DID derivation (vs known vectors); `NexusDIDDocumentV1` construction (valid/invalid); signature creation/verification. Test `publicKeyMultibase`.

### 6.2. Component: DID Registration & Resolution on MVDLI
*   **Sprint Goal(s):** Clients can register new `did:echonet` DIDs and `NexusDIDDocumentV1`s on MVDLI; DIDs are resolvable.
*   **Key Implementation Tasks:**
    *   Client: CLI command for DID/Doc gen; "DIDRegistration" DLI event (signed `NexusDIDDocumentV1`); submit event.
    *   MVDLI Witness: Extend PoW for "DIDRegistration" (schema, signature, DID format validation). Gen `WitnessProofV1`.
    *   MVDLI DDS: Store `NexusDIDDocumentV1` (get `ContentID`).
    *   MVDLI Discovery (DHT): Publish `did -> ContentID_of_DID_Doc` mapping.
    *   Client Resolver: Function to query DHT then fetch Doc from DDS.
*   **Technology Integration Notes:** MVDLI P2P, PoW, DDS, Discovery. Protobuf for DLI event.
*   **Key Deliverables:** CLI DID registration & resolution; Witnesses validate DID events; Docs on DDS, mapping in DHT.
*   **Unit/Integration Test Focus:** Witness validation of DID events. E2E registration: create DID -> submit -> validate -> store -> map. E2E resolution: resolve DID -> query DHT -> fetch Doc. Errors: invalid sig, malformed Doc.

### 6.3. Component: Basic `NexusUserObjectV1` Creation & Linking
*   **Sprint Goal(s):** Minimal `NexusUserObjectV1` created on DID registration, stored on DDS, linked to `NexusDIDDocumentV1`.
*   **Key Implementation Tasks:** Extend "DIDRegistration" Witness logic: post-DID Doc validation/storage, construct minimal `NexusUserObjectV1` (`user_did`, `registration_timestamp`, initial reputation); store on DDS (get `ContentID`); update `NexusDIDDocumentV1.nexus_user_object_ref` (or `NexusUserObjectV1.did_document_ref`).
*   **Technology Integration Notes:** DDS, PoW validation framework.
*   **Key Deliverables:** `NexusUserObjectV1` created per DID, stored on DDS, link established.
*   **Unit/Integration Test Focus:** `NexusUserObjectV1` creation during DID reg; link correctness & retrievability; (optional) DHT mapping `user_did -> ContentID_of_NexusUserObjectV1`.

### 6.4. Component: Data Consent Grant/Revoke DLI Events & Basic Consent Registry
*   **Sprint Goal(s):** Users (via CLI) grant/revoke basic data consent; status queryable via rudimentary Consent Registry.
*   **Key Implementation Tasks:**
    *   Client: Construct/sign `NexusConsentRecordV1` (GRANT/REVOKED); CLI commands for "ConsentGrant"/"ConsentRevoke" DLI events.
    *   MVDLI Witness: Extend PoW for consent events (sig, DIDs, `data_category_ref` validation). Gen `WitnessProofV1`.
    *   MVDLI DDS: Store `NexusConsentRecordV1`.
    *   MVDLI Consent Registry (DHT-based): Key `hash(grantor,grantee,category)` -> Value `{status, expiry, ContentID_of_Record}`. Update on events.
    *   Client: CLI/lib function to query Consent Registry.
*   **Technology Integration Notes:** MVDLI PoW, DDS, Discovery/DHT. DIDs from this module. Protobuf for DLI events.
*   **Key Deliverables:** CLI for consent grant/revoke/query; Witnesses validate consent events; Records on DDS; status queryable.
*   **Unit/Integration Test Focus:** Client consent record creation/signing. Witness validation. E2E: Grant -> Query (GRANTED) -> Revoke -> Query (REVOKED). Errors: unauthorized revoke. Expiry check in query logic.

---
## 7. Implementation Sprints: Content Validation & Incentives (Module 3 MVP)

This section details the implementation sprints for the Content Validation & Incentives features (Module 3 MVP). These sprints build upon a functional MVDLI Core (Module 1 MVP) and the User Identity & Privacy layer (Module 2 MVP), particularly leveraging DIDs and `NexusUserObjectV1` for reputation. Tasks are from `roadmap/overall_mvp_implementation_roadmap.md` (Section 4.3) and `implementation_plans/content_validation_mvp_plan.md`.

### 7.1. Component: PoE-Eligible Interaction Processing (MVDLI Witness Logic)
*   **Sprint Goal(s):** MVDLI Witnesses can identify specific `NexusInteractionRecordV1` types as PoE-eligible and route them for basic PoE scoring after initial DLI validation.
*   **Key Implementation Tasks:**
    *   Define a list of PoE-eligible `interactionType` enum values in Witness configuration/code (e.g., `COMMENT`, `QUALITY_REACTION`).
    *   Extend MVDLI Witness PoW validation logic: after standard validation of a `NexusInteractionRecordV1` (schema, signature by `actor_did`), check if its type is PoE-eligible.
    *   If eligible, pass the validated interaction (or relevant data) to the PoE Quality Scoring component.
*   **Technology Integration Notes:** Modifies existing MVDLI PoW logic (Module 1). Depends on `NexusInteractionRecordV1` structure (Module 1) and resolved `actor_did` (Module 2).
*   **Key Deliverables:** MVDLI Witnesses can correctly identify and forward PoE-eligible interactions for scoring.
*   **Unit/Integration Test Focus:**
    *   Unit tests: Witness logic correctly identifies eligible/ineligible interaction types.
    *   Integration tests: Submit various interaction types; verify only eligible ones are passed to the (next-step) scoring module.

### 7.2. Component: Basic PoE Quality Scoring Algorithm Implementation
*   **Sprint Goal(s):** A simplified, deterministic PoE Quality Scoring algorithm is functional within Witness logic, using fixed initial weights and basic anti-gaming checks.
*   **Key Implementation Tasks:**
    *   Implement the core scoring logic based on `tech_specs/poe_protocol.md`. For MVP, use fixed/hardcoded `PoEScoreFactorsV1` (e.g., base score for type, bonus for comment length up to a cap, multiplier for actor's reputation).
    *   Input: `NexusInteractionRecordV1`, `actor_did`'s `NexusUserObjectV1.reputationScore`.
    *   Implement basic anti-gaming checks: e.g., rate limit for PoE-eligible interactions per DID per time window; min/max length for comment text.
    *   Ensure the final PoE Quality Score is recorded in the `specific_proof_data` field of the `WitnessProofV1` for the interaction.
*   **Technology Integration Notes:** Go language. Access DLI state to get `actor_did`'s reputation (from Module 2's `NexusUserObjectV1`).
*   **Key Deliverables:** PoE Quality Scores are calculated for eligible interactions and included in their `WitnessProofV1`. Basic anti-gaming checks are applied.
*   **Unit/Integration Test Focus:**
    *   Unit tests: Verify score calculation against various inputs and fixed factors. Test individual anti-gaming rule triggers.
    *   Integration tests: Submit interaction, verify correct PoE score in the resulting `WitnessProofV1`. Test that interactions violating basic anti-gaming rules receive zero or penalty scores.

### 7.3. Component: `AIOracleService` Interface & Stub Implementation
*   **Sprint Goal(s):** A defined gRPC/RPC interface for the `AIOracleService` is implemented, along with a stubbed version that returns predefined/randomized scores for testing integration.
*   **Key Implementation Tasks:**
    *   Define Protobuf messages (`GetAISignalRequest`, `GetAISignalResponse`, `AIContentQualitySignalV1`, etc.) and gRPC service for `AIOracleService` as per `tech_specs/ai_ml_integration_points.md`.
    *   Implement a stub `AIOracleService` (e.g., as a separate Go process or an in-process mock):
        *   It receives `GetAISignalRequest`.
        *   Returns a dummy `GetAISignalResponse` with predefined or randomized scores in `ai_signal_payload` based on `data_type_hint` or `model_id`. No actual AI model execution.
        *   Includes a valid (dummy) `oracle_signature`.
*   **Technology Integration Notes:** Protobuf, gRPC (Go).
*   **Key Deliverables:** Compilable `.proto` files for AI Oracle interface. A functional stub `AIOracleService` that can be called over RPC.
*   **Unit/Integration Test Focus:** Unit tests for request/response serialization. Test that the stub service responds correctly to different types of requests.

### 7.4. Component: Integration of AIOracleService Stub into PoE Scoring
*   **Sprint Goal(s):** Dummy AI signals from the stubbed `AIOracleService` are successfully retrieved and incorporated as factors into the PoE Quality Score by Witnesses.
*   **Key Implementation Tasks:**
    *   Modify the Witness PoE scoring logic (from Task 7.2) to:
        *   Identify interactions/content requiring AI signals.
        *   Make an RPC call to the (stubbed) `AIOracleService` with appropriate data or references.
        *   Receive the dummy `AISignal` and incorporate its scores as additional factors into the PoE Quality Score calculation.
*   **Technology Integration Notes:** Requires gRPC client logic within Witness nodes.
*   **Key Deliverables:** PoE Quality Scores generated by Witnesses now reflect (dummy) input from the `AIOracleService` stub.
*   **Unit/Integration Test Focus:**
    *   Unit tests: Mock the AIOracleService client call within Witness logic to test PoE score changes with different AI signals.
    *   Integration tests: Witness node successfully calls the external stubbed `AIOracleService` and uses its response in scoring.

### 7.5. Component: Reputation Score Update (`NexusUserObjectV1`)
*   **Sprint Goal(s):** The `reputationScore` in an actor's `NexusUserObjectV1` is updated on the DLI based on the PoE Quality Scores from their validated interactions.
*   **Key Implementation Tasks:**
    *   Design and implement DLI-native logic (e.g., a conceptual "ReputationUpdateProcess" or by extending Witness responsibilities post-consensus on an interaction's `WitnessProofV1`).
    *   This logic reads the `PoE_Quality_Score` from `WitnessProofV1.specific_proof_data`.
    *   It retrieves the actor's `NexusUserObjectV1` from DLI state/DDS.
    *   It applies a defined formula to update the `reputationScore` (e.g., `new_rep = old_rep + score * factor - decay`). For MVP, a simple additive update based on score is sufficient.
    *   The updated `NexusUserObjectV1` is saved back to DLI state/DDS (this state change itself needs validation if not done by Witnesses directly).
*   **Technology Integration Notes:** Access/update DLI state for `NexusUserObjectV1` (Module 2). Process `WitnessProofV1` (Module 1).
*   **Key Deliverables:** `NexusUserObjectV1.reputationScore` is demonstrably updated on the DLI following PoE-scored interactions.
*   **Unit/Integration Test Focus:**
    *   Unit tests: Test the reputation update formula with various PoE scores.
    *   Integration tests: Submit an interaction -> get PoE score in proof -> verify `NexusUserObjectV1` reputation change for the actor.

### 7.6. Component: Rudimentary PoE Reward Pool & Distribution Logic (Logging for MVP)
*   **Sprint Goal(s):** Potential PoE rewards based on accumulated PoE scores over a test period are calculated and logged by a DLI-native process.
*   **Key Implementation Tasks:**
    *   Define a conceptual DLI global variable or log for the "PoE Reward Pool" (e.g., just a counter or a configuration setting for "tokens_per_period").
    *   Implement DLI-native logic (e.g., triggered by a periodic "epoch end" event, or manually for MVP):
        *   Fetch relevant `WitnessProofV1`s containing PoE scores for the defined period.
        *   Sum all eligible `PoE_Quality_Score`s to get `Total_Period_PoE_Points`.
        *   Calculate potential reward share for each interaction/actor: `(score / Total_Period_PoE_Points) * Conceptual_PoolAllocation`.
        *   **Log** these calculated shares (e.g., "DID X earned Y conceptual_reward_units for period Z") to DLI state logs or node logs. **No actual token transfers.**
*   **Technology Integration Notes:** DLI state/event processing.
*   **Key Deliverables:** Logs demonstrating the correct calculation of conceptual PoE reward shares based on interaction scores.
*   **Unit/Integration Test Focus:** Unit tests for the reward calculation algorithm. Integration tests to ensure that a series of interactions with varying PoE scores result in correctly proportioned conceptual rewards being logged.

### 7.7. Component: Client-Side Submission of PoE-Eligible Interactions
*   **Sprint Goal(s):** The test client/CLI tool can submit `NexusInteractionRecordV1` instances (e.g., comments, quality reactions) that are intended for PoE processing.
*   **Key Implementation Tasks:**
    *   Add new commands to the CLI tool (developed in Module 1) to:
        *   Create a `NexusInteractionRecordV1` of type `COMMENT` with text payload, targeting a `ContentID`.
        *   Create a `NexusInteractionRecordV1` of type `QUALITY_REACTION` with a predefined reaction value, targeting a `ContentID`.
    *   Ensure these interactions are signed by the actor's DID (from Module 2) and submitted to the MVDLI for processing.
*   **Technology Integration Notes:** Utilizes DID library (Module 2), P2P submission (Module 1).
*   **Key Deliverables:** CLI tool can be used to generate test data (PoE-eligible interactions) for the Content Validation MVP.
*   **Unit/Integration Test Focus:** E2E tests: Use CLI to submit comment -> verify it's processed, scored, reputation updated, and conceptual reward logged. Same for quality reactions.

---
## 8. Implementation Sprints: Decentralized Governance (Module 4 MVP)

This section details the implementation sprints for the Decentralized Governance features (Module 4 MVP). These sprints build upon the MVDLI Core (Module 1), Identity & Privacy (Module 2), and Content Validation & Incentives (Module 3 for reputation scores used in voting). Tasks are derived from `roadmap/overall_mvp_implementation_roadmap.md` (Section 4.4) and `implementation_plans/governance_mvp_plan.md`.

### 8.1. Component: Core Governance Data Structures Implementation
*   **Sprint Goal(s):** Implement and unit test core Protobuf data structures for Governance MVP (MVP-scoped `NexusProposalV1`, `VoteV1`, `ProtocolParametersV1` subset).
*   **Key Implementation Tasks:**
    *   Create/update `.proto` files for governance data structures as specified in `tech_specs/governance_framework_spec.md` and `tech_specs/constitutional_framework_spec.md`.
    *   Generate Go code.
    *   Unit test serialization/deserialization and basic validation rules.
*   **Technology Integration Notes:** Protocol Buffers v3, `protoc-gen-go`. Relies on base types from Module 1.
*   **Key Deliverables:** Usable Go data structure libraries for governance.
*   **Unit/Integration Test Focus:** Correctness of Protobuf definitions, serialization, default values, validation of enums or required fields.

### 8.2. Component: Basic `ProposalLifecycleManagerV1` Logic
*   **Sprint Goal(s):** Implement DLI-native logic for submitting text proposals and simple parameter change proposals, and manage basic state transitions (Submitted -> Voting -> Ended).
*   **Key Implementation Tasks:**
    *   Develop DLI-native logic (e.g., in Go, as part of node software) to handle "SubmitProposal" DLI events.
    *   Validate proposal structure, proposer DID's authority (basic check for MVP).
    *   Implement state transition functions for proposals based on time or other triggers (e.g., moving to `ActiveVoting` after a set period or specific DLI event).
*   **Technology Integration Notes:** MVDLI DLI state management for storing proposal objects. Interaction with PoW for event validation.
*   **Key Deliverables:** Proposals can be submitted to the DLI, validated (basic), and their state transitions managed.
*   **Unit/Integration Test Focus:**
    *   Unit tests for proposal submission logic (valid and invalid proposals).
    *   Unit tests for state transition functions (e.g., can a proposal in `VotingEnded` state revert to `ActiveVoting`? - should fail).
    *   Integration test: Submit proposal via CLI, verify its state on DLI.

### 8.3. Component: Basic `VotingManagerV1` Logic
*   **Sprint Goal(s):** DIDs can cast votes on active proposals, with votes weighted by `NexusUserObjectV1.reputationScore`. Votes are tallied, and outcomes determined by simple majority and a fixed quorum.
*   **Key Implementation Tasks:**
    *   Implement DLI-native logic to handle "CastVote" DLI events.
    *   Verify voter eligibility (is DID valid? Is proposal in `ActiveVoting` state? Has voter already voted, if only one vote allowed?).
    *   Retrieve `NexusUserObjectV1.reputationScore` for the voter DID from DLI state (snapshotted at vote start for the proposal).
    *   Implement vote tallying logic (sum of reputation-weighted votes for Aye/Nay options).
    *   Implement outcome determination logic based on a simple majority of weighted votes and a fixed quorum (e.g., X% of total network reputation or X number of unique voters for MVP).
*   **Technology Integration Notes:** Access to `NexusUserObjectV1` (Module 2) for reputation scores. DLI state for vote storage/tallying.
*   **Key Deliverables:** Votes can be cast and are correctly weighted. Proposal outcomes (Pass/Fail) are determined based on tallies, quorum, and threshold.
*   **Unit/Integration Test Focus:**
    *   Unit tests for vote validation, vote weight calculation, tallying various vote distributions.
    *   Unit tests for quorum check and majority threshold logic.
    *   Integration test: Multiple DIDs vote on a proposal via CLI, verify final tally and outcome.

### 8.4. Component: Basic `ParameterManagerV1` Logic
*   **Sprint Goal(s):** One or two predefined, non-critical DLI parameters can be updated in the DLI state via a successfully passed governance proposal.
*   **Key Implementation Tasks:**
    *   Identify 1-2 simple parameters in `ProtocolParametersV1` for MVP modification (e.g., a text string, a numerical logging level).
    *   Implement DLI-native logic within `ParameterManagerV1` that is triggered by a `ProposalLifecycleManagerV1` signal indicating a "ParameterChangeProposal" has passed.
    *   This logic should validate the proposed new parameter value (e.g., type check) and update the DLI state representation of `ProtocolParametersV1`.
    *   Emit `ParameterUpdatedEvent`.
*   **Technology Integration Notes:** DLI state management for `ProtocolParametersV1`. Interaction with `ProposalLifecycleManagerV1`.
*   **Key Deliverables:** A designated DLI parameter can be successfully modified through a governance vote. The change is reflected in the DLI state and an event is emitted.
*   **Unit/Integration Test Focus:**
    *   Unit tests for parameter update logic (valid/invalid parameter names, type mismatches, out-of-range values if applicable for MVP).
    *   Integration test: Full flow - submit parameter change proposal -> vote to pass -> verify parameter in DLI state is updated -> verify event.

### 8.5. Component: DLI State Integration for Governance
*   **Sprint Goal(s):** All MVP-relevant governance state (proposals, votes/tallies, current parameters) is correctly and persistently stored and retrievable from the MVDLI's state mechanism.
*   **Key Implementation Tasks:**
    *   Define clear keying schemes or storage paths for governance objects within the chosen embedded KV store (e.g., BadgerDB).
    *   Ensure all governance logic units (`ProposalLifecycleManagerV1`, `VotingManagerV1`, `ParameterManagerV1`) use these schemes to read and write their respective states.
    *   Implement query functions (callable internally by DLI logic or via read-only RPCs if needed for CLI) to retrieve proposal lists, specific proposal details, vote tallies, and current parameters.
*   **Technology Integration Notes:** Embedded KV store (BadgerDB/LevelDB) used by MVDLI.
*   **Key Deliverables:** Persistent storage and queryable access for all MVP governance state.
*   **Unit/Integration Test Focus:** Test persistence of proposals and parameters across conceptual node restarts (if feasible in simulation). Test query functions for accuracy.

### 8.6. Component: Witness Validation for Governance Events
*   **Sprint Goal(s):** MVDLI Witnesses correctly validate all DLI events related to the Governance MVP.
*   **Key Implementation Tasks:**
    *   Extend the MVDLI Witness PoW validation logic to include specific validation rules for new DLI event types:
        *   `SubmitProposal`: Check structure, signature, proposer eligibility (basic for MVP).
        *   `CastVote`: Check structure, signature, voter eligibility, proposal status/timing.
        *   (Conceptual) `ExecuteProposal` (trigger for parameter change): Verify it corresponds to a legitimately passed proposal.
*   **Technology Integration Notes:** Modifies existing MVDLI PoW event validation framework.
*   **Key Deliverables:** Governance-related DLI events are validated by MVDLI Witnesses, ensuring integrity of governance operations. Invalid governance events are rejected.
*   **Unit/Integration Test Focus:** Unit tests for each new validation rule in Witness logic. Integration tests submitting valid and various invalid governance events to ensure correct acceptance/rejection.

### 8.7. Component: Client-Side Interaction (CLI / Test Application) for Governance
*   **Sprint Goal(s):** The test client/CLI tool is extended to allow users to perform all governance actions defined for the MVP.
*   **Key Implementation Tasks:**
    *   Add CLI commands to:
        *   Submit a text proposal.
        *   Submit a simple parameter change proposal (for the 1-2 designated parameters).
        *   List currently active (and possibly recently ended) proposals.
        *   Cast a vote (Aye/Nay/Abstain) on an active proposal, using the user's DID.
        *   Query the status, current tally, and final outcome of a specific proposal.
        *   Query the current value of a governable DLI parameter.
*   **Technology Integration Notes:** Go CLI framework, client-side libraries for DLI interaction (P2P, event submission, DID management).
*   **Key Deliverables:** A functional CLI tool that enables end-to-end testing and demonstration of the Governance MVP features. This achieves Milestone M4.A.
*   **Unit/Integration Test Focus:** E2E test scripts using the CLI for all governance user stories: create proposal -> list -> vote -> check outcome -> check parameter change (if applicable).

### 8.8. Component: Unit Tests for Governance MVP Components
*   **Sprint Goal(s):** Core DLI-native governance logic (managers, state transitions, validation) is thoroughly unit-tested.
*   **Key Implementation Tasks:**
    *   Write unit tests for all new functions and methods in `ProposalLifecycleManagerV1`, `VotingManagerV1`, and `ParameterManagerV1` logic.
    *   Follow the detailed test case categories outlined in `testing_strategies/governance_unit_tests_strategy.md`, focusing on MVP-scoped features.
*   **Technology Integration Notes:** Go testing package, mocking libraries.
*   **Key Deliverables:** High unit test coverage for all new governance code, integrated into CI pipeline.
*   **Unit/Integration Test Focus:** As per `testing_strategies/governance_unit_tests_strategy.md`.

---
## 9. Overall MVP Testing & Quality Assurance

A comprehensive testing strategy is crucial for ensuring the quality, reliability, and correctness of the integrated EchoNet MVP. This strategy builds upon the unit and component-level testing defined for each module and focuses on inter-module integration and end-to-end system behavior. It aligns with the principles in `roadmap/overall_mvp_implementation_roadmap.md` (Section 7) and `echonet_v3_cicd_strategy.md`.

*   **Reiteration of Core Testing Principles:**
    *   **Test-Driven Development (TDD):** Applied for core logic within each module.
    *   **Continuous Integration (CI):** Automated builds, linting, static analysis, unit tests, and security scans run on every commit/PR.
*   **Levels of Testing for Integrated MVP:**
    *   **Unit Tests:** Continue to be the foundation, ensuring individual components within each module function correctly.
    *   **Intra-Node Integration Tests:** Verify that components within a single MVDLI node (e.g., PoW validator, DDS service, DID event handler, PoE scoring logic, Governance proposal processor) interact correctly. Mocks will be used for P2P interactions at this stage.
    *   **Inter-Module Integration Tests (Simulated Network):**
        *   Focus on validating the interactions and data flows between the four core MVP modules (DLI Core, Identity, Content Validation/PoE, Governance) running on multiple nodes in a simulated P2P network.
        *   **Examples:**
            *   DID (Module 2) registered via MVDLI PoW (Module 1), stored on DDS (Module 1), resolved via Discovery (Module 1).
            *   Content (Module 1) published by a DID (Module 2), interaction (Module 1) by another DID (Module 2) triggers PoE scoring (Module 3) which reads reputation from `NexusUserObjectV1` (Module 2) and updates it.
            *   Governance proposal (Module 4) submitted by a DID (Module 2), voted on by DIDs (Module 2) using reputation from `NexusUserObjectV1` (Module 2, updated by Module 3), outcome changes a DLI parameter (Module 1/4).
    *   **Simulated Network Environment:**
        *   Utilize Docker and Docker Compose (or a lightweight Kubernetes like k3s/Kind) to create reproducible multi-node test environments.
        *   Libp2p's testing tools can also be leveraged for more direct P2P network simulation.
    *   **End-to-End (E2E) Scenario Tests:**
        *   Driven by the extended CLI/test client application.
        *   Cover key user stories that span all integrated MVP modules, as defined in `roadmap/overall_mvp_implementation_roadmap.md` (Section 6.1, "Overall EchoNet MVP v0.1 Completion").
        *   These tests will be automated and run regularly on the deployed simulated testnet (via CI post-merge to `develop`).
*   **Role of CI Pipeline (Extended):**
    *   The CI pipeline will be extended to include automated execution of intra-node integration tests.
    *   The post-merge pipeline will deploy all integrated MVP modules to the simulated testnet and run the automated E2E test suite.
    *   Test reports and coverage (for unit and potentially integration tests) will be generated and monitored.

## 10. User Acceptance Testing (UAT) Plan for MVP (Internal)

Once the integrated MVP is stable and has passed automated E2E tests, internal User Acceptance Testing (UAT) will be conducted.

*   **Objectives:**
    *   Validate that the core functionalities of the integrated MVP meet the defined requirements and user stories from the perspective of the internal team (acting as initial users/testers).
    *   Gather feedback on usability (of the CLI/test client), identify any misunderstandings of features, and uncover bugs not caught by automated tests.
    *   Build confidence in the MVP before considering any broader internal or community demonstration.
*   **Key Scenarios for UAT:**
    *   **Scenario 1 (Full Content Lifecycle):**
        1.  User A generates a new DID.
        2.  User A publishes a new piece of text content.
        3.  User B generates a new DID.
        4.  User B discovers and retrieves User A's content.
    *   **Scenario 2 (Engagement & Reputation):**
        1.  User B posts a quality comment on User A's content.
        2.  Verify PoE score is generated (via logs/DLI state query).
        3.  Verify User B's reputation score in `NexusUserObjectV1` is updated.
        4.  Verify conceptual PoE reward for User B is logged.
    *   **Scenario 3 (Identity & Consent):**
        1.  User A grants User B consent to access a (hypothetical for MVP) "basic_profile_data" category.
        2.  User B (or a test service acting as User B) queries and verifies this consent.
        3.  User A revokes the consent.
        4.  User B's query now shows consent as revoked.
    *   **Scenario 4 (Basic Governance):**
        1.  User A (with some initial reputation) submits a text proposal.
        2.  User B (and A) vote on the proposal (weighted by reputation).
        3.  Verify the proposal outcome (pass/fail based on MVP rules for quorum/majority).
        4.  User A submits a parameter change proposal for a designated test parameter.
        5.  Users A and B vote; proposal passes.
        6.  Verify the DLI parameter is updated by querying its value.
*   **Process for Feedback & Bug Reporting:**
    *   Use the established issue tracker (GitHub Issues, Jira, etc.).
    *   Dedicated UAT period (e.g., 1 week) where team members actively try to break the system using the CLI and follow test scenarios.
    *   Regular UAT feedback sessions to discuss findings.

## 11. MVP v0.1 Release Readiness Criteria (Conceptual)

The integrated EchoNet MVP v0.1 will be considered "ready" for internal demonstration and to serve as the foundation for the next phase of development when the following criteria are met:

1.  **All Defined MVP Milestones Achieved:** All milestones M1.A, M1.B, M2.A, M2.B, M3.A, M3.B, and M4.A (as defined in `roadmap/overall_mvp_implementation_roadmap.md`, Section 6.1) are completed and their key deliverables demonstrated.
2.  **Successful E2E Test Suite Execution:** The automated E2E test suite covering all core MVP user stories (including those spanning multiple modules) passes consistently on the simulated testnet environment.
3.  **Acceptable Unit & Integration Test Coverage:** Unit test coverage for core logic in all modules meets the predefined targets (e.g., >80-90%). Key integration points are covered by automated integration tests.
4.  **Successful Internal UAT:** Key UAT scenarios pass, and any critical or major bugs identified during UAT have been addressed and verified.
5.  **Core Documentation Complete for MVP:**
    *   `.proto` files for all data structures and RPC services are finalized and versioned for the MVP.
    *   Basic setup guides for MVDLI nodes are available.
    *   Usage instructions for the CLI/test client covering all MVP features are documented.
    *   GoDoc for public APIs of core libraries is generated.
6.  **Stable Build & Deployment:** The CI/CD pipeline can reliably build, test, and deploy the integrated MVP to the simulated testnet.
7.  **Known Issues Logged:** All known bugs or limitations for the MVP are documented in the issue tracker with appropriate severity and plans for addressing them in future iterations.

Achieving these criteria will signify that Phase 7 has successfully delivered a foundational, testable, and demonstrable version of the EchoNet platform, ready for further evolution.

---
*(This concludes the main sections of the Phase 7 Core MVP Implementation & Testing Plan.)*
