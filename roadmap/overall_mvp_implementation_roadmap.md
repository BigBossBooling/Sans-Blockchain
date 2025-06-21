# DigiSocialBlock (EchoNet Protocol) - Overall MVP Implementation Roadmap

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Introduction

This document presents the Overall Minimal Viable Product (MVP) Implementation Roadmap for the DigiSocialBlock platform, underpinned by the EchoNet DLI (Distributed Ledger Inspired) protocol. It synthesizes the detailed MVP plans for three foundational modules:

1.  **Module 1: Core DLI Functionality** (from `implementation_plans/dli_core_mvp_plan.md`)
2.  **Module 2: Identity & Privacy Features** (from `implementation_plans/identity_privacy_mvp_plan.md`)
3.  **Module 3: Content Validation & Incentives (Initial PoE & AI Stubs)** (from `implementation_plans/content_validation_mvp_plan.md`)
4.  **Module 4: Decentralized Governance (MVP Aspects)** (conceptually covered, drawing from `implementation_plans/governance_mvp_plan.md`)


This roadmap aims to provide a coherent, phased approach to developing and integrating these core components to achieve a demonstrable and valuable initial version of the EchoNet platform.

## 2. Purpose

The purpose of this Overall MVP Implementation Roadmap is to:

*   Outline the **sequencing** of development efforts across the different MVP modules.
*   Identify key **dependencies** between tasks and modules.
*   Define overarching **milestones** for the MVP development lifecycle.
*   Reiterate common **development practices and principles** (e.g., technology stack choices, testing strategies, CI/CD) that apply to the entire MVP effort.
*   Serve as the **master guide** for the initial implementation phase, ensuring all development streams align towards a unified MVP launch.
*   Provide a basis for future, more detailed sprint planning and resource allocation.

This document will be updated as development progresses and new insights are gained.
---

## 3. Core Principles & Assumptions for MVP Development

This section outlines the foundational principles and key assumptions that will guide the development of the EchoNet MVP.

### 3.1. Guiding Philosophy

The primary guiding philosophy for the EchoNet MVP development is the **Expanded KISS Principle**, as articulated in earlier project documentation:

*   **K - Know Your Core, Keep it Clear:**
    *   *Relevance to MVP:* Focus on the absolute essential functionalities of the DLI, identity, and consent mechanisms. Avoid feature creep. Ensure each component has a well-defined purpose and interface. This clarity is crucial for building a stable foundation.
*   **I - Iterate Intelligently, Integrate Intuitively:**
    *   *Relevance to MVP:* The MVP itself is the first major iteration. Subsequent features will build upon it. Integration between the MVDLI core, Identity/Privacy MVP, and Content Validation MVP must be planned for smooth, intuitive interactions between these modules from the outset.
*   **S - Systematize for Scalability, Synchronize for Synergy:**
    *   *Relevance to MVP:* While full scalability is a post-MVP concern, the initial system design (e.g., choice of Go, libp2p, Protobufs) should not preclude future scalability. Core components must synchronize effectively (e.g., DIDs used by PoE, consent records linked to DIDs).
*   **S - Sense the Landscape, Secure the Solution:**
    *   *Relevance to MVP:* Implement foundational security practices from the start (e.g., secure key management for DIDs, signature validation for DLI events). While comprehensive security hardening is post-MVP, basic vulnerabilities should be addressed. Understand the landscape of decentralized applications and potential attack vectors.
*   **S - Stimulate Engagement, Sustain Impact (The Authentic Connection):**
    *   *Relevance to MVP:* For the MVP, this means ensuring the core functionalities are usable and demonstrate tangible value (e.g., a user can create an identity, publish a piece of content, and see it validated and stored). This initial positive feedback loop is vital for sustaining development momentum and future community engagement.

Adherence to these principles will help ensure the MVP is robust, focused, and provides a solid groundwork for the full EchoNet vision.

### 3.2. Key Assumptions

The development of the EchoNet MVP operates under the following key assumptions:

*   **Technical Foundation:**
    *   The MVP will be built upon the EchoNet v3.0 conceptual architecture and the detailed technical specifications outlined in the `tech_specs/*.md` documents (covering DLI core data structures, DDS protocol, PoW protocol, content hashing/timestamping, mobile node considerations, DID method, data consent protocol, PoE protocol, AI/ML integration points, governance framework, dispute resolution, and constitutional framework).
*   **Technology Stack (Conceptual - for MVDLI and initial modules):**
    *   **Primary Language for DLI Core & P2P Networking:** Go (Golang), chosen for its concurrency features, performance, and strong ecosystem for P2P systems.
    *   **P2P Networking Library:** go-libp2p, for its modularity and comprehensive P2P capabilities.
    *   **Data Serialization:** Protocol Buffers v3 (Protobuf3), for efficient, typed, and évolvable data structures.
    *   **Node-Local State Database:** An embedded key-value store such as BadgerDB or LevelDB (via Go bindings), for managing local node state efficiently.
    *   **Performance-Critical Components (Future Option):** Rust may be considered for specific performance-critical cryptographic components or highly optimized consensus logic post-MVP, potentially integrated via FFI if Go performance proves insufficient in targeted areas.
*   **Resource Availability (Conceptual):**
    *   It is assumed that a development team with the necessary skills in Go, distributed systems, P2P networking, cryptography, and the specific libraries (libp2p, Protobufs) will be available. (Specific team size, roles, and funding are outside the scope of this technical roadmap but are critical project management considerations).
*   **MVP Scope Adherence:**
    *   Development efforts will strictly focus on the features and functionalities defined within the scope of the integrated MVP plans (covering Modules 1-4 as outlined in this document). More advanced features, optimizations, and full-scale implementations will be deferred to post-MVP iterations.
*   **Iterative Development:**
    *   The overall MVP is the first major deliverable in a series of iterative development cycles. Feedback gathered from testing and deploying the MVP will be crucial for refining existing features and prioritizing future development.
*   **CI/CD & Testing Practices:**
    *   Development will adhere to the Continuous Integration/Continuous Deployment (CI/CD) strategy outlined in `echonet_v3_cicd_strategy.md` from the beginning. This includes automated builds, linting, and unit testing.
    *   Test-Driven Development (TDD) or Behavior-Driven Development (BDD) practices will be strongly encouraged for all new code to ensure quality and maintainability.
    *   A simulated network environment will be used extensively for integration testing of P2P interactions and DLI functionalities.
---

## 4. Module Sequencing & Dependencies

This section outlines the proposed sequence for implementing the core modules of the EchoNet MVP, highlighting key dependencies between them. The goal is to build a stable foundation first, then layer additional functionalities.

### 4.1. Foundational DLI Layer Implementation Sequence (Module 1 MVP)

The implementation of the MVDLI core (as detailed in `implementation_plans/dli_core_mvp_plan.md`) must follow a strict sequence due to inherent dependencies. This ensures that foundational components are in place before dependent systems are built.

*   **1. Core Data Structure Implementation (`tech_specs/dli_core_data_structures.md`)**
    *   **Focus:** Implement the Protocol Buffer definitions (or chosen language equivalents, e.g., Go structs generated from `.proto` files) for `NexusContentObjectV1`, `CoreContentMetadataV1`, `NexusUserObjectV1` (basic form), `NexusInteractionRecordV1` (basic form), `WitnessProofV1` (basic form), and `DDSChunk`.
    *   **Rationale:** These data structures are the "nouns" of the EchoNet DLI. All other components will create, consume, or manipulate these structures. Their precise implementation is a prerequisite for any further development.
    *   **Key Deliverable:** Implemented and unit-tested data structure libraries, including serialization and deserialization logic.
*   **2. Content Hashing & Timestamping Logic (`tech_specs/content_hashing_timestamping_specs.md`)**
    *   **Focus:** Implement the canonicalization procedures and hashing algorithms (SHA-256) for generating `ContentID`s (for `NexusContentObjectV1`), `core_metadata_hash` (for `CoreContentMetadataV1`), and `chunk_id`s (for `DDSChunk`s). Implement logic for handling client-asserted timestamps and preparing for network-validated timestamps.
    *   **Rationale:** Essential for data integrity, uniqueness, and content-addressability before any storage, validation, or retrieval operations can be meaningfully performed. Depends on Step 1 (Core Data Structures).
    *   **Key Deliverable:** Verified hashing utilities and timestamp handling functions.
*   **3. Basic P2P Networking Layer Setup (from `implementation_plans/dli_core_mvp_plan.md`)**
    *   **Focus:** Initialize go-libp2p host, establish node identity (using basic key pairs), implement peer discovery (e.g., bootstrap lists, mDNS for local networks), and set up basic pub/sub topics for event gossip.
    *   **Rationale:** Forms the communication backbone for all inter-node interactions required by subsequent DDS, PoW, and Discovery components. Depends on basic understanding of data to be gossiped (conceptual from Step 1).
    *   **Key Deliverable:** Functional P2P layer where nodes can connect, discover each other (locally or via bootstrap), and exchange basic messages via pub/sub.
*   **4. Rudimentary Distributed Data Stores (DDS) Protocol Implementation (`tech_specs/dds_protocol.md` & MVDLI plan)**
    *   **Focus:** Implement basic `StoreChunk` and `RetrieveChunk` RPCs and logic for MVDLI SSN nodes (e.g., in-memory or simple file-per-chunk storage). Implement client-side logic to interact with these RPCs.
    *   **Rationale:** Content chunks need to be stored before they can be referenced by `NexusContentObjectV1` metadata, validated by PoW, or discovered. Depends on Step 1 (for `DDSChunk` structure), Step 2 (for `chunk_id`s), and Step 3 (for RPC communication).
    *   **Key Deliverable:** MVDLI SSN nodes capable of storing and serving chunks; client logic to put/get chunks.
*   **5. Rudimentary Proof-of-Witness (PoW) Protocol Implementation (`tech_specs/pow_protocol.md` & MVDLI plan)**
    *   **Focus:** Implement logic for a simplified committee (e.g., fixed initial Witnesses). Implement DLI event submission (e.g., for new `NexusContentObjectV1` metadata), basic validation (schema checks, signature verification - preliminary, as full DID logic is Module 2), attestation generation by Witnesses, and `WitnessProofV1` assembly (using a list of individual signatures for MVP).
    *   **Rationale:** This is the core consensus mechanism that validates DLI events. It depends on Step 1 (for event/proof structures), Step 2 (for validating `ContentID`s, timestamps), and Step 3 (for event and attestation gossip). It also conceptually depends on Step 4 (as Witnesses might need to verify that content chunks referenced in an object are available on DDS).
    *   **Key Deliverable:** Witnesses able to receive content objects, perform basic validation, and generate a basic `WitnessProofV1`.
*   **6. Basic Discovery Protocol (DHT) Implementation (from `discovery_protocol_refined.md` & MVDLI plan)**
    *   **Focus:** Implement basic Kademlia DHT `STORE_VALUE` (e.g., `ContentID` -> MVDLI SSN address; `ChunkID` -> MVDLI SSN address) and `FIND_VALUE` operations using go-libp2p's Kademlia DHT implementation.
    *   **Rationale:** Makes content and DDS storage locations discoverable across the network. Depends on Step 3 (P2P layer for DHT to operate), Step 4 (to know which SSN addresses host which chunks/content), and conceptually Step 5 (as proofs might also be made discoverable).
    *   **Key Deliverable:** Nodes can publish and resolve key-value pairs in the rudimentary DHT, specifically for content/chunk locations.
*   **7. Initial Mobile Node Roles (Host Functionality - `tech_specs/mobile_node_specifications.md` & MVDLI plan)**
    *   **Focus:** Implement client-side logic for a "Host" role (conceptualized for a CLI or test application initially, rather than a full mobile app for MVDLI). This includes creating a `NexusContentObjectV1` with its chunks, submitting chunks to MVDLI SSNs, submitting the content object for PoW validation, and then being able to retrieve it via the Discovery protocol and DDS.
    *   **Rationale:** Validates the end-to-end usability and integration of all preceding MVDLI components from the perspective of a primary user agent. This ensures the DLI core is not just internally functional but can serve its intended purpose. Depends on all preceding DLI components (Steps 1-6) being minimally functional.
    *   **Key Deliverable:** A test client or CLI tool capable of publishing a piece of content to the MVDLI and retrieving it, demonstrating the full lifecycle.

This sequence prioritizes building the system from the ground up, ensuring foundational layers are stable before more complex logic is introduced.

### 4.2. Identity & Privacy Integration Sequence (Module 2 MVP)

Once the Foundational DLI Layer (Module 1 MVP) is operational and stable, the Identity & Privacy features (as detailed in `implementation_plans/identity_privacy_mvp_plan.md`) can be integrated. This module critically depends on the MVDLI's ability to process and validate DLI events, store data, and make it discoverable.

*   **1. `did:echonet` Library/SDK Functions (Client-Side - Task 2.1 from Identity MVP Plan)**
    *   **Focus:** Implement client-side logic for Ed25519 key pair generation, derivation of `did:echonet` strings from public keys, construction of basic `NexusDIDDocumentV1` protobuf messages, and functions for signing data and verifying signatures.
    *   **Rationale:** These are foundational client-side tools necessary for any DID operations, preparing for interaction with the DLI. This step has minimal direct DLI dependency for the library functions themselves but is a prerequisite for DLI interactions.
    *   **Key Deliverable:** A functional client-side library (e.g., Go package) for `did:echonet` operations.
*   **2. DID Registration on MVDLI (Task 2.2 from Identity MVP Plan)**
    *   **Focus:** Develop the client logic to create a "DID Registration" DLI event (containing the `NexusDIDDocumentV1`, signed by the initial authentication key). Extend MVDLI Witness PoW validation to handle this new event type (basic schema, signature checks). Integrate with MVDLI DDS to store the `NexusDIDDocumentV1` and with MVDLI Discovery (DHT) to publish the `did -> ContentID_of_DID_Doc` mapping.
    *   **Rationale:** This activates DIDs on the EchoNet DLI, making them resolvable and usable. Directly depends on a functional MVDLI (P2P for event gossip, PoW for validation, DDS for storage, Discovery for resolution mapping).
    *   **Key Deliverable:** DID registration process operational on the MVDLI; DIDs can be created and their documents stored and mapped.
*   **3. DID Resolution via MVDLI (Task 2.3 from Identity MVP Plan)**
    *   **Focus:** Implement client-side resolver functions that query the MVDLI Discovery (DHT) for a DID Document's `ContentID` and then retrieve the document from the MVDLI DDS.
    *   **Rationale:** Enables any participant to retrieve DID Documents, which is essential for verifying signatures and discovering service endpoints. Depends on successful DID registration (Step 2 of this sequence) and the underlying MVDLI Discovery and DDS components.
    *   **Key Deliverable:** DID resolution functional; clients can fetch DID Documents from the MVDLI.
*   **4. Basic `NexusUserObjectV1` Creation & Linking (Task 2.4 from Identity MVP Plan)**
    *   **Focus:** As part of the DID Registration DLI event processing, create and store a minimal `NexusUserObjectV1` on the MVDLI DDS. This object will contain the `user_did`, `registration_timestamp`, and a link to/from the `NexusDIDDocumentV1` (e.g., `nexus_user_object_ref` in DID Doc, `did_document_ref` in User Object).
    *   **Rationale:** Establishes the core on-DLI anchor for user-specific attributes beyond the DID Document (like future reputation). Depends on DID registration (Step 2).
    *   **Key Deliverable:** `NexusUserObjectV1` instances created and linked with new DIDs, stored on DDS.
*   **5. Data Consent Grant/Revoke DLI Events (Client-Side & MVDLI Basic Validation - Task 2.5 from Identity MVP Plan)**
    *   **Focus:** Implement client-side logic to construct and sign `NexusConsentRecordV1` messages for granting and revoking consent. Develop "ConsentGrant" and "ConsentRevoke" DLI event types and extend MVDLI Witness PoW validation to perform basic checks on these events (valid signatures, recognized DIDs, basic structure).
    *   **Rationale:** Enables the core actions of the data consent protocol. Depends on registered DIDs (Step 2) for `grantor_did` and `grantee_did`.
    *   **Key Deliverable:** Client can create and submit consent grant/revoke DLI events; MVDLI Witnesses perform basic validation.
*   **6. Basic Consent Registry & Query (MVDLI - Task 2.6 from Identity MVP Plan)**
    *   **Focus:** Upon successful Witness validation of consent events, store the `NexusConsentRecordV1` on MVDLI DDS. Implement a rudimentary Consent Registry by using the MVDLI DHT to store mappings like `hash(grantor, grantee, category) -> status_and_ContentID_of_Record`. Develop a basic client-side query function for consent status.
    *   **Rationale:** Makes consent decisions verifiable and queryable by relevant parties. Depends on consent grant/revoke DLI event processing (Step 5) and MVDLI DDS/Discovery.
    *   **Key Deliverable:** Consent records stored on DDS; basic consent status queryable via the rudimentary Consent Registry.

This sequence ensures that DID functionalities are established before building consent mechanisms that rely on these DIDs.

### 4.3. Content & Monetization Integration Sequence (Module 3 MVP)

With the DLI core and Identity/Privacy layers in place, the initial features for content validation (PoE) and conceptual monetization can be implemented, as detailed in `implementation_plans/content_validation_mvp_plan.md`. This module relies on users (DIDs) creating interactions that are then scored and may affect reputation or (conceptually) rewards.

*   **1. PoE-Eligible Interaction Processing (MVDLI Witness Logic - Task 3.1 from Content MVP Plan):**
    *   **Focus:** Extend MVDLI Witness logic to identify specific `NexusInteractionRecordV1` types (e.g., comments, predefined quality reactions) as PoE-eligible and route them for PoE scoring after basic DLI validation (schema, signature).
    *   **Rationale:** This is the entry point for the PoE system, filtering relevant interactions. Depends on MVDLI PoW (Module 1) for basic event validation and `NexusInteractionRecordV1` structure (Module 1). Actor DIDs (Module 2) are crucial for identifying who performed the interaction.
    *   **Key Deliverable:** MVDLI Witnesses can categorize incoming interactions and flag PoE-eligible ones for further processing.
*   **2. Basic PoE Quality Scoring Algorithm Implementation (Task 3.2 from Content MVP Plan):**
    *   **Focus:** Implement the simplified, deterministic PoE Quality Scoring algorithm within the Witness logic. Use fixed initial weights (conceptual `PoEScoreFactorsV1` stored in config or as DLI constants for MVP). Include basic anti-gaming checks (e.g., velocity limits per DID for PoE-triggering interactions).
    *   **Rationale:** This calculates the core "quality" or "value" metric for an interaction, which is fundamental to PoE. Depends on Step 1 of this sequence and the actor's `NexusUserObjectV1.reputationScore` (Module 2) as an input to the scoring.
    *   **Key Deliverable:** Witnesses can calculate and assign a basic PoE Quality Score to eligible interactions and include it in the `WitnessProofV1`.
*   **3. `AIOracleService` Interface & Stub Implementation (Task 3.3 from Content MVP Plan):**
    *   **Focus:** Define the gRPC/RPC interface for the `AIOracleService` as specified in `tech_specs/ai_ml_integration_points.md`. Implement a stubbed version of this service that returns predefined/randomized scores for different types of inputs (content, interaction). No actual AI model implementation in this MVP step.
    *   **Rationale:** Establishes the contract for future AI/ML model integration without the immediate overhead of AI development. Allows PoE scoring logic to be built with AI signals in mind. Minimal direct DLI dependency for the stub itself, can be an independent mock service.
    *   **Key Deliverable:** A defined RPC interface for `AIOracleService` and a functional stub service that PoE logic can call.
*   **4. Integration of AIOracleService Stub into PoE Scoring (Task 3.4 from Content MVP Plan):**
    *   **Focus:** Modify the Witness PoE scoring logic (from Step 2 of this sequence) to make calls to the stubbed `AIOracleService` for relevant data (e.g., comment text). Incorporate the dummy AI signals returned by the stub as factors in the PoE Quality Score calculation.
    *   **Rationale:** Tests the complete pipeline for PoE scoring, including the hook for AI-derived signals, ensuring the data flow is correct. Depends on Step 2 (basic PoE scoring) and Step 3 (AIOracleService stub).
    *   **Key Deliverable:** PoE scoring algorithm within Witnesses now incorporates (dummy) AI signals from the stubbed oracle.
*   **5. Reputation Score Update (`NexusUserObjectV1` - Task 3.5 from Content MVP Plan):**
    *   **Focus:** Implement DLI-native logic (or extend Witness state modification capabilities post-consensus) to update the `NexusUserObjectV1.reputationScore` of the interacting user (actor DID) based on the validated PoE Quality Scores from their interactions. For MVP, this can be a direct update.
    *   **Rationale:** This closes the loop by connecting the quality of engagement back to the user's on-DLI reputation. Depends on PoE scoring (Step 2/4) and the existence and accessibility of `NexusUserObjectV1` (Module 2).
    *   **Key Deliverable:** `NexusUserObjectV1.reputationScore` is updated based on validated PoE activity.
*   **6. Rudimentary PoE Reward Pool & Distribution Logic (Logging - Task 3.6 from Content MVP Plan):**
    *   **Focus:** Implement DLI-native logic (e.g., as part of a periodic conceptual "epoch end" event) to iterate through recent `WitnessProofV1`s containing PoE scores. Calculate potential reward shares based on these scores and a conceptual, fixed reward pool amount for the period. **Log these calculated potential rewards to DLI state or node logs.** No actual token transfers occur in this MVP stage.
    *   **Rationale:** Demonstrates the economic feedback loop conceptually and allows for testing the reward calculation logic without requiring full tokenomics and wallet integration. Depends on PoE scoring (Step 2/4).
    *   **Key Deliverable:** Calculation of conceptual PoE rewards is performed and logged by the DLI.
*   **7. Client-Side Submission of PoE-Eligible Interactions (Task 3.7 from Content MVP Plan):**
    *   **Focus:** Extend the test client/CLI tool (from Module 1 MVP) to allow users to submit `NexusInteractionRecordV1` instances that are specifically intended for PoE processing (e.g., posting a comment, issuing a "quality reaction" to a piece of content).
    *   **Rationale:** Enables end-to-end testing of the PoE MVP flow, from user interaction submission through to Witness validation, scoring, reputation update, and conceptual reward logging. Depends on all preceding steps in this Module 3 sequence being testable.
    *   **Key Deliverable:** Test client/CLI can generate PoE-eligible interactions and submit them to the MVDLI.

This sequence builds the initial content validation and user incentive mechanisms on top of the established DLI and identity foundations.

### 4.4. Decentralized Governance Integration Sequence (Module 4 MVP)

Following the establishment of the DLI core, identity, and initial engagement (PoE/reputation) systems, the MVP for decentralized governance (as detailed in `implementation_plans/governance_mvp_plan.md`) can be integrated. This allows the community to begin participating in the evolution of the platform.

*   **1. Core Governance Data Structures (Task G1 from Governance MVP Plan):**
    *   **Focus:** Implement Protobuf definitions for MVP-scoped versions of `NexusProposalV1` (for text and simple parameter changes), `VoteV1`, and the governable subset of `ProtocolParametersV1`.
    *   **Rationale:** These are the foundational data types for governance operations. Depends on Module 1 (Core Data Structures - for base types like DID, Timestamp).
    *   **Key Deliverable:** Implemented and unit-tested governance data structure libraries.
*   **2. Basic `ProposalLifecycleManagerV1` Logic (Task G2 from Governance MVP Plan):**
    *   **Focus:** Implement DLI-native logic for submitting text and simple parameter change proposals, basic validation of proposal structure, and managing proposal state transitions (e.g., Submitted -> ActiveVoting -> VotingEnded).
    *   **Rationale:** Enables the creation and tracking of governance proposals. Depends on G1, and MVDLI PoW (Module 1) for DLI event validation.
    *   **Key Deliverable:** Logic for proposal submission and state management operational within DLI nodes.
*   **3. Basic `VotingManagerV1` Logic (Task G3 from Governance MVP Plan):**
    *   **Focus:** Implement DLI-native logic for vote casting (1 DID, 1 vote, weighted by `NexusUserObjectV1.reputationScore` snapshotted at vote start), vote tallying (sum of weighted votes), and outcome determination based on simple majority and fixed quorum.
    *   **Rationale:** Enables the core voting process. Depends on G1, G2, MVDLI PoW (Module 1), and critically on `NexusUserObjectV1.reputationScore` from Module 2 (Identity & Privacy) and Module 3 (PoE for reputation updates).
    *   **Key Deliverable:** DIDs can cast weighted votes on active proposals; votes are tallied, and outcomes determined.
*   **4. Basic `ParameterManagerV1` Logic (Task G4 from Governance MVP Plan):**
    *   **Focus:** Allow 1-2 predefined non-critical DLI parameters (from `ProtocolParametersV1`) to be updatable. Implement logic to apply the parameter change to the DLI state upon a passed governance proposal.
    *   **Rationale:** Demonstrates the ability of governance to modify protocol parameters. Depends on G3 (for successful proposal outcome).
    *   **Key Deliverable:** Successful parameter change proposal leads to an update in the DLI's active parameter set. `ParameterUpdatedEvent` emitted.
*   **5. DLI State Integration for Governance (Task G5 from Governance MVP Plan):**
    *   **Focus:** Ensure governance state (proposals, votes, current parameters) is stored and retrievable from the MVDLI's state mechanism (e.g., embedded KV store).
    *   **Rationale:** Makes governance information persistent and queryable. Depends on G1-G4.
    *   **Key Deliverable:** Governance state persisted and accessible.
*   **6. Witness Validation for Governance Events (Task G6 from Governance MVP Plan):**
    *   **Focus:** Extend MVDLI Witness logic to validate new DLI event types for proposal submission, vote casting, and parameter update execution triggers.
    *   **Rationale:** Ensures the integrity of governance operations. Depends on MVDLI PoW (Module 1) and the specific logic of G2, G3, G4.
    *   **Key Deliverable:** MVDLI Witnesses validate all governance-related DLI events.
*   **7. Client-Side Interaction (CLI / Test Application - Task G7 from Governance MVP Plan):**
    *   **Focus:** Extend the test client/CLI to allow submitting proposals, listing active proposals, casting votes, querying proposal outcomes, and viewing current values of governable parameters.
    *   **Rationale:** Provides the means to interact with and test the Governance MVP features. Depends on all preceding steps in this Module 4 sequence.
    *   **Key Deliverable:** Test client/CLI supports all basic governance MVP user stories.
*   **8. Unit Tests for Governance MVP Components (Task G8 from Governance MVP Plan):**
    *   **Focus:** Implement unit tests for the MVP-scoped features of `ProposalLifecycleManagerV1`, `VotingManagerV1`, and `ParameterManagerV1` as per `testing_strategies/governance_unit_tests_strategy.md`.
    *   **Rationale:** Ensures core governance logic is robust and correct. Depends on G2, G3, G4.
    *   **Key Deliverable:** Comprehensive unit tests for implemented governance logic.

This sequence introduces basic on-DLI governance capabilities after the core DLI, identity, and initial engagement/reputation systems are in place.

### 4.5. Conceptual Visual Dependency Chart Outline

To provide a clear visual overview of the inter-task and inter-module dependencies outlined in sections 4.1, 4.2, 4.3, and 4.4, a dependency chart should be created. This section outlines what such a chart would represent.

*   **Purpose of the Visual Chart:**
    *   To offer an at-a-glance understanding of the development flow for the entire EchoNet MVP.
    *   To clearly illustrate which tasks must be completed before others can begin (dependencies).
    *   To help identify the critical path for development, ensuring resources can be prioritized effectively.
    *   To show potential parallel work streams where tasks do not have direct sequential dependencies.

*   **Recommended Chart Type (Conceptual):**
    *   A **Directed Acyclic Graph (DAG)** or a **PERT chart** would be suitable.
    *   Alternatively, a simpler **flowchart** or **swimlane diagram** (if wanting to also show team/responsibility areas) could be used, focusing on clear dependency lines.

*   **Key Elements to Include in the Chart:**
    *   **Nodes/Boxes:** Each major task or deliverable from the sequences defined in sections 4.1, 4.2, 4.3, and 4.4 (e.g., "M1_T1: Core Data Structures," "M2_T2: DID Registration," "M3_T4: AI Stub Integration," "M4_T3: Basic Voting Logic"). Task numbering (Module_Task) should be used for clarity.
    *   **Arrows/Lines:** Clearly indicate dependencies between tasks. An arrow from Task A to Task B means Task A must be completed before Task B can start.
    *   **Critical Path:** Highlight the sequence of tasks that directly determines the minimum time required to complete the entire MVP.
    *   **Parallel Streams:** Visually group or arrange tasks that can be worked on concurrently. For example, client-side DID library development (M2_T1) can occur in parallel with some early MVDLI P2P setup (M1_T3).
    *   **Module Grouping:** Optionally, use color-coding or swimlanes to visually distinguish tasks belonging to Module 1 (DLI Core), Module 2 (Identity/Privacy), Module 3 (Content/Monetization), and Module 4 (Governance).

*   **Tooling for Actual Generation (Note):**
    *   The actual visual chart is not generated in this markdown document. It would be created using specialized diagramming software (e.g., Lucidchart, Microsoft Visio, diagrams.net / draw.io), project management tools with Gantt/PERT capabilities, or code-based diagramming tools like Mermaid.js (which can be embedded in markdown later). This section serves as the specification for *what* the chart should convey.

*   **Example Snippet of How a Dependency Might be Described for the Chart (Textual representation to inform chart creation):**
    *   **Task ID:** M2_T2
    *   **Task Name:** DID Registration on MVDLI
    *   **Depends On:** M1_T3 (Basic P2P), M1_T4 (Rudimentary DDS), M1_T5 (Rudimentary PoW), M1_T6 (Basic Discovery), M2_T1 (DID Lib/SDK)
    *   *(This indicates that M2_T2 cannot start until all listed M1 tasks and M2_T1 are substantially complete or their necessary interfaces are stable.)*

    *   **Task ID:** M3_T5
    *   **Task Name:** Reputation Score Update (`NexusUserObjectV1`)
    *   **Depends On:** M3_T2 (Basic PoE Scoring) OR M3_T4 (AI Stub Integration if AI signal directly impacts score), M2_T4 (Basic `NexusUserObjectV1` Creation)
    *   *(This shows dependencies within the same module and on a previous module.)*

    *   **Task ID:** M4_T3
    *   **Task Name:** Basic `VotingManagerV1` Logic
    *   **Depends On:** M4_T1 (Core Gov Data Structures), M4_T2 (Basic Proposal Lifecycle), M2_T4 (Basic `NexusUserObjectV1` Creation for reputation scores), M3_T5 (Reputation Score Update logic for PoP-derived reputation)
    *   *(This shows dependencies on previous modules and tasks within the governance module itself.)*


This visual chart will be a valuable companion to the textual roadmap, aiding project managers, developers, and stakeholders in understanding the overall MVP implementation flow.

## 5. Consolidated Technology Stack Summary

This section reiterates the core technology choices for the EchoNet MVP, ensuring a consistent understanding across the development team. These choices are derived from the "Key Assumptions" (Section 3.2) and the `implementation_plans/dli_core_mvp_plan.md`.

*   **Primary Programming Language (DLI Core, Node Logic, Services):**
    *   **Go (Golang):** Chosen for its strong concurrency features (goroutines, channels), excellent networking libraries, performance characteristics suitable for P2P systems, and a mature ecosystem in the DLT space.
*   **P2P Networking Library:**
    *   **go-libp2p:** The Go implementation of the libp2p framework will be used for its modularity in handling transports, stream multiplexing, peer discovery (DHT, mDNS), NAT traversal, and secure communication channels.
*   **Data Serialization:**
    *   **Protocol Buffers v3 (Protobuf3):** Used for defining all core data structures, DLI events, and RPC message payloads. This ensures efficient binary serialization, strong typing, and cross-language compatibility (though Go is primary for MVP). `protoc-gen-go` will be used for Go code generation.
*   **Node-Local State Database (for individual DLI nodes):**
    *   **Embedded Key-Value Store:**
        *   **Primary Candidate:** BadgerDB (pure Go, transactional, optimized for SSDs).
        *   **Alternative:** LevelDB (via Go bindings like `goleveldb`).
    *   **Rationale:** Provides persistent storage for node-specific data (e.g., DHT routing tables, local indexes, configuration, temporary Witness sets for MVDLI) without requiring external database dependencies for each node.
*   **Cryptography:**
    *   Standard Go cryptographic libraries.
    *   Ed25519 for digital signatures (DIDs, attestations).
    *   SHA-256 for hashing (`ContentID`, `chunk_id`, etc.).
    *   (X25519 for key agreement if encrypted messaging features were in MVP scope, currently deferred).
*   **Client-Side Logic (for Test Client/CLI):**
    *   **Go (Golang):** The initial test client and CLI tool will be developed in Go to leverage the same core data structure libraries and P2P components directly.
*   **Future Considerations (Post-MVP):**
    *   **Rust:** May be evaluated for highly performance-critical cryptographic routines or specific consensus algorithm components if Go performance becomes a bottleneck in those isolated areas. Integration would likely be via FFI.

This stack is chosen to promote development efficiency, leverage mature ecosystems, and meet the performance and concurrency demands of a decentralized DLI system.

---
## 6. Overall MVP Progression: Milestones & Conceptual Timeline

This section defines high-level milestones for the overall EchoNet MVP, drawing from the key deliverables of each module sequence. It also provides a conceptual phasing for the development effort.

### 6.1. Summary of Key Module Deliverables & Overarching MVP Milestones

The detailed task deliverables are outlined in Section 4. Here, we group them into overarching milestones that represent significant demonstrable progress.

**Foundational DLI Layer Milestones (Module 1 MVP)**
*   **Milestone M1.A: Core Network & Storage Primitives Operational**
    *   Description: Basic P2P networking allows nodes to connect and exchange messages. Core data structures are implemented and serializable. Rudimentary DDS allows client-side storage and retrieval of individual content chunks on test SSN nodes. Basic content hashing (`ContentID`, `chunk_id`) is functional.
    *   Key Deliverables: Bootstrappable P2P network, implemented data structure libraries (Protobufs in Go), functional `StoreChunk`/`RetrieveChunk` on MVDLI SSNs, `ContentID` generation utilities.
    *   *Corresponds to completing approximately M1 tasks 1-4.*
*   **Milestone M1.B: End-to-End MVDLI Content Lifecycle Operational**
    *   Description: Simplified PoW validation is functional for basic content submission events. A basic `WitnessProofV1` is generated. Basic Kademlia DHT allows `ContentID` registration (mapping to SSN addresses) and lookup. The initial CLI/test client can publish a content object (metadata + chunks) and retrieve it by `ContentID` through the full MVDLI stack.
    *   Key Deliverables: Test content objects successfully published, validated by fixed Witnesses, stored on MVDLI DDS, and discovered/retrieved by `ContentID` via DHT and DDS. CLI tool demonstrates this full loop.
    *   *Corresponds to completing M1 tasks 5-7, building on M1.A.*

**Identity & Privacy Layer Milestones (Module 2 MVP)**
*   **Milestone M2.A: Decentralized ID (DID) Lifecycle Operational on MVDLI**
    *   Description: Users (via the test client/CLI) can generate `did:echonet` key pairs and DIDs. They can construct and register a `NexusDIDDocumentV1` with the MVDLI. These DID Documents can be resolved by any client via the MVDLI's Discovery (DHT) and DDS. A basic `NexusUserObjectV1` is created and linked to the DID on the DLI.
    *   Key Deliverables: Functional client-side DID generation library; DID registration DLI events validated by MVDLI Witnesses; DID Documents stored on DDS and resolvable via DHT; `NexusUserObjectV1` created and associated with DIDs.
    *   *Depends on M1.B. Corresponds to completing approximately M2 tasks 1-4.*
*   **Milestone M2.B: Basic Data Consent Mechanism Functional on MVDLI**
    *   Description: Users (via the test client/CLI) can grant and revoke a basic consent type (e.g., for a predefined `data_category_ref`) using a `NexusConsentRecordV1`. The status of this consent can be queried via the rudimentary Consent Registry (DHT-based).
    *   Key Deliverables: Consent grant/revoke DLI events validated by MVDLI Witnesses; `NexusConsentRecordV1`s stored on DDS; basic consent status queryable.
    *   *Depends on M2.A. Corresponds to completing M2 tasks 5-6.*

**Content & Monetization Layer Milestones (Module 3 MVP)**
*   **Milestone M3.A: Basic PoE Scoring & Reputation Update Functional**
    *   Description: PoE-eligible interactions (e.g., comments submitted via test client/CLI) are identified and processed by MVDLI Witnesses. Basic PoE Quality Scores are calculated using fixed factors and incorporating dummy signals from the stubbed `AIOracleService`. The `NexusUserObjectV1.reputationScore` of the interacting DID is updated based on these scores.
    *   Key Deliverables: Submitted interactions receive a PoE Quality Score in their `WitnessProofV1`; `NexusUserObjectV1.reputationScore` changes based on validated PoE activity; `AIOracleService` interface defined and stub operational.
    *   *Depends on M1.B and M2.A. Corresponds to completing approximately M3 tasks 1-5.*
*   **Milestone M3.B: Conceptual End-to-End PoE Reward Logging Operational**
    *   Description: The rudimentary PoE Reward Pool logic (non-token-based) can calculate and log potential reward distributions based on accumulated PoE Quality Scores over a test period. The test client/CLI can be used to generate interactions that feed into this process.
    *   Key Deliverables: Logged output demonstrating the conceptual reward calculation based on test interactions and their PoE scores.
    *   *Depends on M3.A. Corresponds to completing M3 tasks 6-7.*

**Decentralized Governance Layer Milestones (Module 4 MVP)**
*   **Milestone M4.A: Basic Proposal & Parameter Change System Operational**
    *   Description: Users (via test client/CLI) can submit simple text proposals and proposals to change predefined, non-critical DLI parameters. A basic PoP/reputation-weighted voting mechanism is used, votes are tallied, and outcomes determined. Successful parameter change proposals result in updates to the DLI state.
    *   Key Deliverables: Functional proposal submission for text/parameter changes; DIDs can vote with reputation-derived weight; votes tallied and outcomes decided; DLI parameters updated via governance; CLI support for these actions.
    *   *Depends on M1.B (core DLI), M2.A (DIDs), M3.A (Reputation for voting weight). Corresponds to completing approximately G1-G8 from `implementation_plans/governance_mvp_plan.md`.*

**Overall EchoNet MVP v0.1 Completion:**
*   **Description:** All above milestones (M1.A, M1.B, M2.A, M2.B, M3.A, M3.B, **and M4.A**) are achieved. The system demonstrates a basic but complete end-to-end flow: user identity creation, content publication and interaction, PoE scoring and reputation updates, conceptual reward logging, and basic on-DLI governance for proposals and parameter changes.
*   **Key Deliverables:** A functional, integrated DLI network running on test nodes with a CLI client capable of exercising all core MVP features including basic governance. Comprehensive test suite and initial documentation.

### 6.2. Overall MVP Scope & Core Deliverables

This section clarifies the unified scope and key deliverables for the entire EchoNet MVP v0.1, synthesizing the goals of individual module MVPs.

**User-Facing Core Value & Capabilities:**
The EchoNet MVP v0.1 will allow users (interacting via a test client/CLI) to:
*   Create and manage a decentralized identity (`did:echonet`), including generating key pairs and a basic DID Document.
*   Publish basic text-based content (`NexusContentObjectV1`) to the decentralized network, have it validated by Witnesses, and stored on rudimentary Distributed Data Stores (DDS).
*   Discover and view content published by themselves or others using `ContentID`s.
*   Perform basic social interactions on content, such as posting comments or "quality reactions" (`NexusInteractionRecordV1`).
*   Accrue conceptual Proof-of-Engagement (PoE) scores based on these interactions, which will influence their on-DLI reputation score (`NexusUserObjectV1.reputationScore`). (Scores are logged; no real token movement for rewards in this MVP).
*   Grant and revoke a basic type of data consent for their information using `NexusConsentRecordV1`, with the status being queryable.
*   Participate in basic on-DLI governance by:
    *   Submitting simple text proposals or proposals to change predefined, non-critical DLI parameters.
    *   Viewing active proposals.
    *   Voting on active proposals using their PoP-derived reputation score as voting weight.
    *   Viewing the outcome of proposals.
*   Experience these interactions through a foundational DLI where data is stored decentrally and validated by a simplified consensus mechanism, including basic mobile client interactions as a 'Host' (conceptually tested via CLI).

**Key Technical Deliverables (Summarized):**
*   A functional, interconnected MVDLI network composed of nodes capable of P2P communication, event gossip, and basic DHT operations (simulated or small testnet).
*   Implemented and tested core data structures (e.g., `NexusContentObjectV1`, `NexusInteractionRecordV1`, `NexusUserObjectV1`, `NexusDIDDocumentV1`, `NexusConsentRecordV1`, `NexusProposalV1`, `VoteV1`, `WitnessProofV1`, `ProtocolParametersV1` subset).
*   Operational (though rudimentary for MVP) core DLI protocols:
    *   Distributed Data Stores (DDS) for basic chunk storage and retrieval.
    *   Proof-of-Witness (PoW) consensus for DLI event validation by a fixed set of Witnesses.
    *   Discovery Protocol (DHT) for basic content and DID Document resolution.
*   Functional DID lifecycle: client-side DID/key generation, DLI registration of DID Documents, and DLI resolution of DID Documents.
*   Functional basic Data Consent lifecycle: client-side consent grant/revoke operations, DLI validation, and a rudimentary, queryable Consent Registry.
*   Functional (rudimentary) Proof-of-Engagement (PoE) scoring for specific interactions, updates to on-DLI reputation scores, and conceptual logging of potential rewards.
*   Functional basic Decentralized Governance: submission of text/parameter change proposals, PoP/reputation-weighted voting, vote tallying, outcome determination, and DLI parameter updates.
*   A basic Command Line Interface (CLI) or test client application capable of demonstrating these core user stories and DLI functionalities.
*   Unit and initial integration tests for all core components, run as part of a CI pipeline.

**Explicit Non-Goals for this Overall MVP v0.1 (reiterated):**
*   Full token-based economy, actual token generation (TGE), or on-DLI value transfer for PoE rewards or any other purpose.
*   Advanced or production-ready AI/ML models for content/interaction analysis (only stubs for integration testing).
*   A complex advertising marketplace or direct creator monetization channels beyond conceptual PoE.
*   Full suite of mobile node roles (Super-Host, Decelerator, Mobile Witness) or a polished native mobile application.
*   Sophisticated or dynamic governance mechanisms (e.g., complex voting systems, treasury management, automated NIP execution, advanced dispute resolution).
*   Large-scale performance, stress testing, or comprehensive security hardening beyond foundational best practices.
*   User-friendly graphical interfaces (GUIs) for end-users or node operators.
*   Full implementation of all aspects of all `tech_specs` documents; only MVP-scoped items are included.

### 6.3. Conceptual Phasing & Timeline

While specific calendar dates are subject to team size, velocity, and unforeseen challenges, a conceptual phasing helps visualize the MVP progression. We can define main phases for the overall MVP development:

*   **Phase A: DLI Core Bootstrap (Focus: Milestones M1.A)**
    *   **Duration (Notional):** e.g., Sprints 0-3 / "Month 1-2"
    *   **Primary Activities:**
        *   Setup of development environment, CI/CD pipeline, code repositories.
        *   Implementation of core data structures (M1_T1) and hashing/timestamping logic (M1_T2).
        *   Establishment of basic P2P networking with go-libp2p (M1_T3).
        *   Implementation of rudimentary DDS `StoreChunk`/`RetrieveChunk` (M1_T4).
    *   **Goal:** Achieve a state where nodes can connect, exchange basic data, and store/retrieve content chunks reliably on individual MVDLI SSNs. Foundational libraries for data handling are stable.

*   **Phase B: MVDLI End-to-End Flow & Initial Identity (Focus: Milestones M1.B, M2.A)**
    *   **Duration (Notional):** e.g., Sprints 4-7 / "Month 2-4"
    *   **Primary Activities:**
        *   Implementation of rudimentary PoW protocol (M1_T5).
        *   Implementation of basic Discovery (DHT) for content (M1_T6).
        *   Development of the initial CLI/test client for content publishing/retrieval (M1_T7), achieving M1.B.
        *   Parallel development of client-side `did:echonet` library (M2_T1).
        *   Integration of DID Registration and Resolution with the now functional MVDLI (M2_T2, M2_T3).
        *   Basic `NexusUserObjectV1` creation (M2_T4), achieving M2.A.
    *   **Goal:** A fully operational MVDLI where content can be published, validated, stored, and discovered. Users can create and resolve DIDs on this MVDLI.

*   **Phase C: Integrating Privacy, Engagement & Initial Governance Loops (Focus: Milestones M2.B, M3.A, M3.B, M4.A)**
    *   **Duration (Notional):** e.g., Sprints 8-12 / "Month 4-6"
    *   **Primary Activities:**
        *   Implementation of Data Consent DLI events and basic Consent Registry (M2_T5, M2_T6), achieving M2.B.
        *   Development of PoE-eligible interaction processing and basic PoE Quality Scoring by Witnesses (M3_T1, M3_T2).
        *   Implementation of the `AIOracleService` interface and stub (M3_T3) and its integration into PoE scoring (M3_T4).
        *   Reputation score updates based on PoE (M3_T5), achieving M3.A.
        *   Rudimentary (logged) PoE reward distribution logic (M3_T6).
        *   Extension of CLI/test client for PoE interactions (M3_T7), achieving M3.B.
        *   Implementation of core Governance MVP tasks (G1-G8 from `implementation_plans/governance_mvp_plan.md`), achieving M4.A.
    *   **Goal:** Demonstrate the core mechanics of user consent, content engagement scoring, reputation updates, conceptual reward logging, and basic on-DLI governance. This completes the overall MVP v0.1.

**Note on Parallelism:**
*   Client-side library work (e.g., M2_T1 for DID functions) can often start earlier, in parallel with foundational DLI P2P/DDS work, as long as data structure contracts are stable.
*   The `AIOracleService` stub (M3_T3) can be developed independently once its interface is agreed upon.
*   Core governance data structures (G1) can be developed in parallel with later stages of Phase B.

This phased approach allows for iterative testing and integration, ensuring each layer is functional before building more complex features on top. The notional durations are for illustrative purposes only and would be refined during actual project planning.

---
## 7. Unified Testing Strategy for MVP

A comprehensive testing strategy is crucial for ensuring the quality, reliability, and correctness of the EchoNet MVP. This strategy will encompass multiple levels of testing, applied throughout the development lifecycle, and aligns with the principles in `echonet_v3_cicd_strategy.md`.

*   **Test-Driven Development (TDD) / Behavior-Driven Development (BDD):**
    *   Development will be guided by TDD/BDD principles where applicable. Unit tests will be written before or concurrently with feature code to define and verify component behavior.
*   **Unit Tests:**
    *   **Focus:** Verify the correctness of individual functions, methods, modules, and data structures in isolation.
    *   **Examples:** Testing canonicalization and hashing functions, data structure serialization/deserialization, signature generation/verification, individual RPC handlers (with mocks), PoE scoring formula logic, governance proposal validation.
    *   **Tools:** Standard Go testing package, `testify/assert` for assertions.
*   **Integration Tests:**
    *   **Focus:** Verify the interaction between different components within a single node or service, and between closely related DLI core modules.
    *   **Examples:**
        *   Testing the flow from an RPC call to a DDS node through its internal logic to storage.
        *   Verifying that Witness PoW logic correctly processes and validates various DLI event types (content, DID, consent, governance) using local data structures and cryptographic utilities.
        *   Ensuring the DID client library correctly interfaces with local key storage and DLI registration events.
        *   Testing PoE scoring integration with reputation updates.
    *   **Tools:** Go testing package, potentially using in-memory versions of dependencies or mocks.
*   **Simulated Network Tests (Multi-Node):**
    *   **Focus:** Verify interactions between multiple DLI nodes in a controlled, simulated P2P network environment.
    *   **Examples:**
        *   Testing P2P node discovery (mDNS, DHT bootstrap).
        *   Verifying event gossip and pub/sub mechanisms for all DLI event types.
        *   Testing DDS chunk storage and retrieval across different nodes.
        *   Validating the PoW consensus flow with multiple Witnesses for all DLI event types.
        *   Testing DHT PUT/GET operations for content, DID resolution, and consent registry.
        *   Simulating governance proposal lifecycle across multiple nodes.
    *   **Tools:** Libp2p's testing utilities (e.g., `p2p/tools/tester`), custom test harnesses using Docker to orchestrate multiple node instances.
*   **End-to-End (E2E) Tests:**
    *   **Focus:** Verify complete user stories and functional flows from the perspective of an end-user (simulated via the CLI/test client for the MVP).
    *   **Examples:**
        *   Full lifecycle: User generates DID -> Publishes content -> Content validated & stored -> Discoverable -> Another user resolves DID & retrieves content.
        *   User grants/revokes consent -> Service queries consent -> Status reflected.
        *   User posts comment -> PoE score generated -> Reputation updated -> Conceptual reward logged.
        *   User submits governance proposal -> Other users vote -> Proposal outcome determined -> Parameter (if applicable) updated.
    *   **Tools:** The CLI/test client application will be the primary driver for E2E tests, with scripts automating command sequences and verifying outputs/ DLI state changes.
*   **Continuous Integration (CI):**
    *   The CI pipeline (as per `echonet_v3_cicd_strategy.md` and Section 8.1) will automatically run linters, unit tests, and potentially some integration tests on every commit or pull request.
    *   Regular (e.g., nightly) automated runs of more comprehensive integration and simulated network tests.

This multi-layered testing approach aims to catch issues early, ensure components integrate correctly, and validate that the overall MVP meets its functional requirements.

## 8. Development Practices & Tooling Integration for MVP

This section details how overarching development practices and specific tooling strategies will be applied consistently across all MVP modules to ensure quality, collaboration, and efficiency.

### 8.1. CI/CD Pipeline Integration Across MVP Modules

The Continuous Integration/Continuous Deployment (CI/CD) pipeline strategy, as defined in `echonet_v3_cicd_strategy.md`, will be integral to the MVP development process for all modules (DLI Core, Identity & Privacy, Content Validation & Incentives, Governance).

*   **Reiteration of CI/CD Goals for MVP:**
    *   **Rapid Feedback:** Provide developers with quick feedback on their changes.
    *   **Code Quality:** Automatically enforce coding standards, linting, and formatting.
    *   **Early Bug Detection:** Catch issues early through automated unit, integration, and (eventually) E2E tests.
    *   **Consistent Build & Test Environment:** Ensure all code is built and tested in a standardized environment, reducing "works on my machine" issues.
    *   **Automated Artifact Generation:** Produce consistent build artifacts (binaries, Docker images for test nodes).

*   **Pre-Commit Hooks (MVP Context):**
    *   **Tooling:** Utilize tools like Husky or pre-commit.
    *   **Checks:**
        *   **Linters:** Ruff (if any Python utility scripts), `golangci-lint` for Go.
        *   **Code Formatters:** Black (Python), `gofmt`/`goimports` for Go.
        *   **Proto Linters:** `buf lint` for Protocol Buffer definitions.
    *   **Enforcement:** Strongly encouraged for all developers to ensure code quality before changes are even committed to local branches.

*   **On-Commit/Pull Request (PR) Pipeline for MVP Modules:**
    *   Triggered automatically on every push to a feature branch or when a PR is created/updated against the `develop` branch.
    *   **1. Build & Compilation:** Each module's components (Go binaries for DLI nodes, client CLI) must compile successfully. Protobuf code generation (`protoc`) will be a part of this step.
    *   **2. Static Analysis:**
        *   Go: `go vet`, `staticcheck`, and other tools included in `golangci-lint`.
        *   (If Python utils exist): MyPy for type checking, Bandit for security analysis.
    *   **3. Unit Testing:**
        *   All new code for MVP modules (DLI core logic, DID library functions, PoE scoring algorithms, governance logic etc.) must be accompanied by comprehensive unit tests.
        *   Enforce code coverage checks (e.g., using Go's built-in coverage tools). Aim for a minimum coverage threshold (e.g., >80%) for core, critical logic. Test failures will block PR merging.
    *   **4. Integration Testing (MVP Scope - Automated in CI):**
        *   Focus on automated tests for interactions *between components of the same module* initially (e.g., testing DDS RPC handlers with an in-memory storage backend within a single test process).
        *   As modules mature, introduce tests for interactions *between the core MVP modules* (e.g., DID registration event triggering PoW validation and DDS storage; PoE scoring using DID reputation).
        *   Mock external dependencies or modules that are not yet built or are outside the immediate test scope.
    *   **5. Security Scanning (Automated):**
        *   **SAST (Static Application Security Testing):** `gosec` for Go code.
        *   **Dependency Scanning:** Check for known vulnerabilities in third-party Go modules (e.g., using `govulncheck`) and other dependencies.
        *   Results will be reported in the PR; critical vulnerabilities must be addressed before merging.

*   **Post-Merge/Main Branch Pipeline for MVP (Simulated Testnet Environment):**
    *   Triggered automatically after a PR is merged into the `develop` branch (and eventually for releases tagged from `main`).
    *   **1. Automated Deployment to Simulated Testnet:**
        *   Build fresh Docker images for DLI nodes (incorporating MVDLI, Identity, Content Validation, and Governance components as they are completed).
        *   Automatically deploy these images to a consistent, containerized (Docker Compose or Kubernetes-lite like k3s/Kind) multi-node test environment. This environment will simulate a small EchoNet network.
    *   **2. Automated End-to-End (E2E) Scenario Tests:**
        *   Run a suite of automated E2E tests using the CLI/test client against the deployed simulated testnet.
        *   These tests will cover core MVP user stories as defined in Section 6.1 and 6.2 (e.g., DID creation -> content publish -> PoE interaction -> governance proposal -> vote -> parameter check).
        *   Test failures here would indicate issues with inter-module integration or deployment configurations.

*   **Artifact Management (Conceptual for MVP):**
    *   **Build Artifacts:** Versioned Go binaries for DLI nodes and the CLI/test client.
    *   **Container Images:** Versioned Docker images for DLI nodes, stored in a container registry (e.g., Docker Hub, GitHub Container Registry, or a private registry). Image tagging will align with code versions/tags.
    *   **Test Reports:** Store and make accessible unit test, integration test, and E2E test reports (including coverage) for each build.

*   **Branching Strategy for MVP:**
    *   **Main Branches:**
        *   `main`: Represents the most stable, "released" state of the MVP (even if internal releases). Protected branch.
        *   `develop`: Integration branch where features are merged. Represents the current state of active MVP development. Nightly or regular builds deployed to a staging/dev testnet should come from this branch.
    *   **Feature Branches:** Developers create `feature/<feature-name>` branches off `develop` for their individual tasks.
    *   **Pull Requests (PRs):** All code changes are submitted to `develop` via PRs. PRs must pass the "On-Commit/Pull Request Pipeline" checks before being eligible for review and merging.
    *   **Hotfix Branches (Post-MVP Release):** If critical bugs are found in `main`, `hotfix/<issue-id>` branches can be created from `main`, fixed, and merged back into both `main` and `develop`.

This CI/CD setup ensures that each module of the MVP is developed with consistent quality checks and that inter-module integrations are tested early and often, aligning with the "Iterate Intelligently, Integrate Intuitively" principle.

### 8.2. Test-Driven Development (TDD) Approach for MVP

To further enhance code quality, design clarity, and developer confidence during the MVP development, Test-Driven Development (TDD) will be the primary methodology for implementing new core logic.

*   **Reiteration of TDD Principle:** TDD follows a short, iterative cycle:
    1.  **Red:** Write a failing test that defines a desired improvement or new function.
    2.  **Green:** Write the minimal amount of code necessary to make the test pass.
    3.  **Refactor:** Clean up the code (both production and test code) to improve readability, maintainability, and performance while ensuring all tests still pass.
*   **Scope of TDD for MVP:**
    *   TDD is mandated for all new **core DLI logic** (e.g., PoW event validation rules, DDS chunk management, Discovery DHT operations, governance state transitions).
    *   It will be applied to **business logic within DLI-native "contracts"** (even if these are conceptual Go modules in the MVP, like the ReputationUpdateContract or PoEDistributionContract, ProposalLifecycleManager).
    *   Critical **library functions** (e.g., canonicalization, hashing utilities, `did:echonet` helper functions, vote tallying) will also be developed using TDD.
    *   User Interface (UI) development for the CLI/test client might use other testing approaches (like BDD or manual E2E testing for usability), but the underlying command handlers and logic interacting with the DLI should still be TDD-driven where possible.
*   **Unit Test Focus:**
    *   TDD naturally produces a comprehensive suite of unit tests that cover individual components thoroughly.
    *   These unit tests also serve as a form of **executable specification**, clearly demonstrating how each component is intended to behave based on its inputs.
*   **Integration with CI/CD:**
    *   The unit tests generated through TDD are a fundamental part of the "On-Commit/Pull Request Pipeline" (detailed in section 8.1 and `echonet_v3_cicd_strategy.md`).
    *   The CI server will execute all TDD-generated unit tests. Builds (and PR merges) **must fail** if any of these unit tests fail, ensuring that regressions are caught immediately.
*   **Test Coverage:**
    *   While TDD focuses on testing behavior, a high level of unit test code coverage is an expected outcome.
    *   Coverage will be monitored using tools integrated into the CI pipeline (e.g., Go's built-in coverage tools). The aim for TDD-developed core components will be to achieve and maintain high coverage (e.g., >80-90%).
*   **Developer Workflow:**
    1.  Select a small piece of functionality from an MVP task.
    2.  Write a unit test specifying the expected behavior for that functionality. Run it; it should fail (Red).
    3.  Write the simplest production code to make the unit test pass (Green).
    4.  Refactor the production code and test code for clarity and efficiency, ensuring tests continue to pass.
    5.  Repeat for the next piece of functionality.
*   **Benefits for MVP:**
    *   **Increased Confidence:** Provides high confidence in the correctness of individual components.
    *   **Reduced Bugs:** Catches bugs early in the development cycle, reducing the cost of fixing them.
    *   **Improved Design:** Writing tests first often leads to better-designed, more modular, and more testable code (loosely coupled components with clear interfaces).
    *   **Facilitates Refactoring:** A comprehensive test suite allows developers to refactor and improve code with confidence, knowing that regressions will be caught. This is vital as the MVP evolves.
    *   **Living Documentation:** Unit tests act as up-to-date documentation for how components are supposed to work.

By enforcing TDD for core logic, the EchoNet MVP aims to build a high-quality, robust, and maintainable foundation from the outset.

### 8.3. Documentation Standards & Collaboration Tools for MVP Development

Consistent documentation and effective collaboration are vital for the success of the MVP and for laying the groundwork for future development. This aligns with the `echonet_v3_documentation_strategy.md`.

*   **Living Technical Documentation:**
    *   **Principle:** Technical documentation is not an afterthought but an integral part of the development process. It should be a "living" resource, updated as the code evolves.
    *   **Focus for MVP:**
        *   **Key Architectural Decisions:** Any significant design choices or deviations made during MVP implementation from the initial `tech_specs` should be documented with rationale.
        *   **API Specifications:**
            *   Protobuf definitions (`.proto` files) for data structures and RPC services serve as the primary API specification. These will be version-controlled with the codebase.
            *   Generated documentation from Protobufs (e.g., using `protoc-gen-doc`) will be utilized.
        *   **Setup Guides for MVDLI Nodes:** Clear, concise instructions on how to compile, configure, and run an MVDLI node in a test environment.
        *   **Usage Instructions for Test Client/CLI:** Detailed examples of how to use the test client/CLI to perform all MVP user stories (content publishing, DID registration, consent actions, PoE interactions, basic governance actions).
    *   **Code Comments:**
        *   Developers are expected to write clear, concise comments in the Go code, especially for:
            *   Public functions and struct fields (GoDoc style).
            *   Complex or non-obvious logic.
            *   Assumptions or known limitations.
        *   Comments should explain *why* something is done, not just *what* is done (if the code itself is clear on the "what").

*   **Collaboration Tools & Processes:**
    *   **Version Control:**
        *   **Git:** The distributed version control system will be used.
        *   **Hosting Platform:** A shared Git hosting platform like GitHub or GitLab will be used for the central repositories.
    *   **Branching Strategy (Reiteration):**
        *   As detailed in Section 8.1 (CI/CD), a strategy like Gitflow (with `main`, `develop`, feature branches, and PRs) will be employed to manage code changes systematically.
    *   **Pull Request (PR) / Merge Request (MR) Workflow:**
        *   All code contributions, including documentation changes, must be submitted via PRs/MRs to the `develop` branch.
        *   **Peer Review:** Require at least one other developer to review and approve a PR/MR before merging. Reviews should focus on correctness, design, adherence to standards, and test coverage.
        *   **CI Checks:** All CI checks (build, lint, tests, security scans as defined in Section 8.1) must pass before a PR/MR can be merged.
    *   **Issue Tracking:**
        *   A dedicated issue tracker (e.g., GitHub Issues, GitLab Issues, Jira) will be used to manage:
            *   Bug reports.
            *   Feature requests for post-MVP.
            *   MVP development tasks (broken down from the MVP plans).
        *   Commits and PRs should be linked to the relevant issue(s) for traceability.
    *   **Communication Channels:**
        *   **Primary Chat:** A dedicated team chat platform (e.g., Slack, Discord, Mattermost) for quick discussions, questions, and coordination.
        *   **Meetings:** Regular (e.g., daily or thrice-weekly) short virtual stand-up meetings for developers to synchronize, discuss blockers, and plan immediate work. Less frequent (e.g., weekly or bi-weekly) meetings for broader MVP progress review and planning.
    *   **Knowledge Sharing:**
        *   **Internal Wiki / Shared Document Space:** (e.g., Confluence, Google Workspace, or the project's Git repository for markdown-based design notes). Used for:
            *   Design notes and ad-hoc technical discussions not captured in formal `tech_specs` or this roadmap.
            *   Meeting minutes and action items.
            *   Onboarding information for new team members.

*   **Documentation Updates with Code:**
    *   When a code change in a PR/MR affects existing functionality, design, or API contracts, the relevant technical documentation (e.g., `tech_specs/*.md`, setup guides, code comments) **must** be updated as part of the same PR/MR. This ensures documentation remains synchronized with the codebase.

Adherence to these documentation and collaboration standards will be crucial for maintaining clarity, consistency, and shared understanding as the EchoNet MVP is developed by the team.

## 9. High-Level Post-MVP Outlook

The successful delivery of the EchoNet MVP v0.1, as defined by the milestones in Section 6.1, marks the completion of the foundational software. It will provide a functional, albeit simplified, decentralized system demonstrating core user stories. The post-MVP phase will focus on iterative enhancements, hardening, and expansion of features towards the full EchoNet vision.

Key areas for development beyond the MVP include:

*   **Enhanced DLI Core Features:**
    *   **Full Proof-of-Witness (PoW):** Implementation of dynamic Witness registration, staking/bonding mechanisms (requires tokenomics integration), VRF-based committee selection, and robust slashing conditions for Witness misbehavior.
    *   **Advanced Distributed Data Stores (DDS):** Implementation of data replication across SSNs, erasure coding for enhanced durability, full Proof-of-Storage-Receipt (PoSR) challenges and validation, and dynamic node announcements/discovery.
    *   **Robust Discovery Protocol:** Scalable and secure DHT implementation, advanced peer routing, and potentially alternative discovery mechanisms.
    *   **Network Bootstrapping & Incentives:** Refined mechanisms for new nodes joining the network, and initial incentive structures for SSN and Witness participation (linked to tokenomics).
*   **Identity & Privacy Enhancements:**
    *   **Full DID Lifecycle Management:** Robust implementation of DID updates (key rotation, service endpoint changes) and deactivation.
    *   **Advanced Key Management:** Integration with hardware security modules (HSMs) for node operators, more sophisticated client-side key management solutions (including recovery options).
    *   **Verifiable Credentials (VCs):** Integration of VC issuance, holding, and presentation capabilities using `did:echonet`.
    *   **Selective Disclosure & Zero-Knowledge Proofs (ZKPs):** Research and potential integration of ZKPs for enhanced privacy in data sharing and attestations.
*   **Content Validation, Monetization & Social Features:**
    *   **Sophisticated Proof-of-Engagement (PoE):** Refined PoE Quality Scoring algorithms, outcome-based scoring (e.g., for effective shares, accurate flags), dynamic adjustment of `PoEScoreFactorsV1` via governance.
    *   **Real AI/ML Model Integration:** Transition from `AIOracleService` stubs to integration with actual, trained AI/ML models for content quality, interaction analysis, and anomaly detection. Development of the AI Oracle network.
    *   **Full Tokenomics Implementation:** Introduction of the ECHO token, integration with wallets, and enabling actual on-DLI value transfer for PoE rewards, tipping, and future content monetization features.
    *   **Advertising Module:** Design and implementation of the decentralized advertising marketplace.
    *   **Expanded Social Features:** Richer interaction types, content discovery algorithms, user-curated feeds/lists, notification systems.
*   **Mobile Client & Node Capabilities:**
    *   Development of full-featured native mobile applications (iOS, Android) for the "Host" role.
    *   Implementation and testing of more advanced mobile node roles (Super-Host, Decelerator, potentially limited Mobile Witness roles) as per `tech_specs/mobile_node_specifications.md`.
*   **Governance:**
    *   Implementation of the on-DLI governance mechanisms outlined in `echonet_v3_governance_model.md` for protocol upgrades, parameter changes, and treasury management (beyond MVP's basic parameter changes).
*   **Scalability, Performance & Security:**
    *   Continuous performance profiling and optimization of DLI nodes and protocols.
    *   Horizontal scaling strategies for DLI components.
    *   Comprehensive security audits, penetration testing, and ongoing security hardening.
    *   Advanced anti-gaming and anti-spam mechanisms.
*   **Developer Ecosystem & Documentation:**
    *   Creation of comprehensive SDKs for various programming languages.
    *   Detailed developer documentation, tutorials, and community support channels.

The post-MVP roadmap will be driven by community feedback, research breakthroughs, and the evolving needs of the EchoNet ecosystem, always adhering to the core principles of decentralization, user empowerment, and iterative development.

*(This concludes the main sections of the Overall MVP Implementation Roadmap.)*
