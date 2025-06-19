# EchoNet - Minimal Viable DLI (MVDLI) Core Implementation Plan

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. MVDLI Core - Goals & Scope

### 1.1. Goals
The primary goal of the Minimal Viable DLI (MVDLI) Core is to implement the absolute essential functionalities of the EchoNet Distributed Ledger Inspired (DLI) system. This initial version will prove the foundational concepts and provide a platform for iterative development and testing. It embodies the "Iterate Intelligently, Integrate Intuitively" principle by focusing on a working skeleton first.

The MVDLI must demonstrate:
*   Formation of a small, functional peer-to-peer (P2P) network.
*   Basic Decentralized Identifier (`did:echonet`) generation and management.
*   Publication of a simplified `NexusContentObjectV1` to the network.
*   Validation of this content object by a rudimentary Proof-of-Witness (PoW) committee.
*   Generation of a basic `WitnessProofV1`.
*   Storage of the content object's chunks on a simplified Distributed Data Store (DDS).
*   Retrieval of the content object by its `ContentID` via a basic Discovery mechanism.

### 1.2. Scope
**In Scope for MVDLI:**
*   Core data structures (`NexusUserObjectV1` basic, `NexusContentObjectV1` simplified, `NexusInteractionRecordV1` very basic/conceptual, `WitnessProofV1` basic, `DDSChunk`).
*   Canonicalization for hashing and signing as per `tech_specs/content_hashing_timestamping_specs.md`.
*   Basic P2P networking: node identity, peer discovery (bootstrap list, mDNS), event gossip (pub/sub).
*   Simplified `did:echonet` generation (local key pair) and resolution to a basic DID document.
*   `NexusContentObjectV1` creation (client-side) and submission.
*   Rudimentary PoW:
    *   Fixed initial set of Witness nodes (no on-DLI registration or dynamic staking yet).
    *   Simplified committee selection (e.g., all initial Witnesses participate or round-robin).
    *   Basic event validation (schema, signature).
    *   Attestation and `WitnessProofV1` generation (list of individual signatures, no complex aggregation).
*   Basic DDS:
    *   Single SSN-like node functionality.
    *   `StoreChunk` (in-memory or simple file-per-chunk storage).
    *   `RetrieveChunk`.
    *   No replication, erasure coding, or PoSR for MVDLI. Clients/Origin Nodes store on 1-2 known MVDLI SSNs.
*   Basic Discovery (conceptual Kademlia DHT):
    *   `STORE_VALUE` (`ContentID` -> SSN MVDLI address).
    *   `FIND_VALUE` (`ContentID`).
    *   Rudimentary node joining/bootstrapping for the DHT.
*   A simple Command Line Interface (CLI) tool or test application to perform publish and retrieve operations.

**Non-Goals for MVDLI (to be addressed in subsequent iterations):**
*   Full Proof-of-Engagement (PoE) rewards system.
*   Complex advertising marketplace.
*   Advanced mobile node roles (Super-Host, Decelerator, Mobile Witness beyond conceptual client-side validation). Standard mobile "Host" interaction via the CLI or test app is the target.
*   Full Witness registration, dynamic staking, VRF-based committee selection, and slashing.
*   Advanced DDS features: replication, erasure coding, full PoSR challenges, dynamic node announcements.
*   Sophisticated Discovery mechanisms (full DHT security, advanced peer routing).
*   User reputation scores.
*   Governance mechanisms.
*   Smart contract layer for DLI-native logic beyond basic PoW.
*   Scalability and performance optimizations beyond functional correctness.
*   Comprehensive security hardening (focus on functional security first).

## 2. Conceptual Technology Stack Choices (with Rationale)

The technology stack is chosen to align with EchoNet's decentralized nature, performance requirements, and developer ecosystem, following KISS principles.

*   **Core DLI Logic & P2P Networking:**
    *   **Recommendation:** **Go (Golang)**
    *   **Rationale:**
        *   Excellent concurrency support (goroutines, channels) essential for handling many P2P connections and asynchronous events.
        *   Strong standard library for networking.
        *   High performance, compiled language.
        *   Mature ecosystem for DLT and P2P systems (e.g., IPFS/libp2p, Ethereum clients, Tendermint are largely Go-based).
        *   Relatively easy to learn and productive for developers.
        *   Good tooling and cross-compilation capabilities.
*   **Performance-Critical Cryptography / Advanced Consensus Components (Optional - if Go proves insufficient later):**
    *   **Recommendation:** **Rust** (callable from Go via Foreign Function Interface - FFI, e.g., cgo or gRPC for microservices).
    *   **Rationale:**
        *   Memory safety without garbage collection, offering predictable performance.
        *   Exceptional performance, suitable for CPU-intensive tasks like complex cryptographic operations or highly optimized consensus algorithms.
        *   Growing and robust ecosystem for cryptography and systems programming.
        *   *For MVDLI, Go is expected to be sufficient. Rust would be considered for specific bottlenecks identified post-MVDLI.*
*   **Data Serialization:**
    *   **Choice:** **Protocol Buffers v3 (Protobuf3)**
    *   **Rationale:** As defined in `tech_specs/dli_core_data_structures.md`. Efficient binary format, strong typing, schema evolution capabilities, cross-language code generation. Deterministic serialization available for canonical hashing needs.
*   **P2P Library:**
    *   **Choice:** **go-libp2p** (Go implementation of the libp2p framework).
    *   **Rationale:** Modular, extensible P2P networking stack. Provides transports, stream multiplexing, peer discovery, NAT traversal, pub/sub, etc. Aligns with the Go choice for core logic.
*   **Database for Node-Local State (e.g., DHT persistence, Witness set info, local content index):**
    *   **Recommendation:** Embedded key-value store.
        *   **Primary candidate:** **BadgerDB** (pure Go, transactional, good performance).
        *   **Alternative:** **LevelDB** (via Go bindings like `goleveldb`).
    *   **Rationale:**
        *   Embedded: Simplifies deployment (no separate database server to manage for each node).
        *   Performance: Fast reads/writes for local state management.
        *   Persistence: Ensures node state (like DHT routing tables or known content) survives restarts.

## 3. Key Implementation Tasks & Prioritization for MVDLI

Tasks are prioritized based on dependencies and their criticality for achieving a demonstrable end-to-end flow.

*   **Task 1: Basic DID Management (MVDLI Scope)**
    *   **Description:** Implement `did:echonet` key pair generation (e.g., ED25519). Create a minimal client-side function to generate and store a key pair. Define a very basic DID Document structure (e.g., including the public key) that can be conceptually associated with the DID. No complex DID resolution or on-DLI registration for MVDLI.
    *   **Inputs:** `tech_specs/dli_core_data_structures.md` (for `NexusUserObjectV1` conceptual basis).
    *   **Priority:** **High** (Fundamental for identifying actors).
*   **Task 2: Core Data Structure Implementation & Canonicalization**
    *   **Description:** Translate Protobuf definitions from `tech_specs/dli_core_data_structures.md` (focus on `NexusContentObjectV1`, `CoreContentMetadataV1`, `DDSChunk`, basic `WitnessProofV1`) into Go structs using `protoc-gen-go`. Implement the precise canonicalization and hashing logic defined in `tech_specs/content_hashing_timestamping_specs.md` for `ContentID` and `core_metadata_hash`.
    *   **Inputs:** `tech_specs/dli_core_data_structures.md`, `tech_specs/content_hashing_timestamping_specs.md`.
    *   **Priority:** **High** (Core data handling).
*   **Task 3: P2P Networking Layer Setup (using go-libp2p)**
    *   **Description:** Initialize libp2p host. Implement node identity using key pairs from Task 1. Setup basic peer discovery (e.g., mDNS for local networks, configurable bootstrap peer list). Implement basic pub/sub topics for event gossip (e.g., "new_content_submitted", "new_attestation").
    *   **Inputs:** `echonet_v3_communication_protocols.md` (for general P2P philosophy).
    *   **Priority:** **High** (Foundation for network interaction).
*   **Task 4: Simplified `NexusContentObjectV1` Publishing Workflow**
    *   **Description:**
        *   Client-side: Allow creation of a `NexusContentObjectV1` (with `CoreContentMetadataV1`), compute its `core_metadata_hash` and `ContentID`, and sign a wrapper message containing the object (or the `ContentID` as proof of ownership).
        *   Network: Gossip this signed wrapper/object over the "new_content_submitted" pub/sub topic.
    *   **Inputs:** `tech_specs/dli_core_data_structures.md`, `tech_specs/content_hashing_timestamping_specs.md`.
    *   **Priority:** **Medium** (First major use case).
*   **Task 5: Rudimentary PoW Protocol Implementation (MVDLI Scope)**
    *   **Description:**
        *   Identify a fixed, small set of initial Witness nodes (their DIDs/public keys hardcoded or in a config file for MVDLI).
        *   Upon receiving a new content object via gossip, these Witnesses perform basic validation (e.g., is the `ContentID` correct? Is the creator signature valid if present? Are required fields there?).
        *   Witnesses generate a `WitnessAttestationV1` (from `tech_specs/pow_protocol.md`, simplified: no VRF proof needed for MVDLI as committee is fixed).
        *   Gossip attestations on a "new_attestation" pub/sub topic.
        *   Any node (or a designated MVDLI Witness) can collect attestations for a `ContentID`. If enough (e.g., >2/3 of the fixed set) are collected, assemble a basic `WitnessProofV1` (list of individual signatures).
    *   **Inputs:** `tech_specs/pow_protocol.md`, `tech_specs/dli_core_data_structures.md`.
    *   **Priority:** **Medium** (Core consensus for MVDLI).
*   **Task 6: Basic DDS Implementation (MVDLI Scope)**
    *   **Description:**
        *   Implement a basic SSN node that can handle `StoreChunkRequest` and `RetrieveChunkRequest` RPCs (from `tech_specs/dds_protocol.md`).
        *   `StoreChunk`: For MVDLI, store chunk data in memory (map of `chunk_id` to bytes) or as simple files in a directory, named by `chunk_id`.
        *   `RetrieveChunk`: Read from memory/file.
        *   No replication, erasure coding, or PoSR. Origin node (client) will be responsible for "storing" the chunks of its published content on one or two known MVDLI SSN addresses.
    *   **Inputs:** `tech_specs/dds_protocol.md`, `tech_specs/dli_core_data_structures.md`.
    *   **Priority:** **Medium** (Content persistence for MVDLI).
*   **Task 7: Basic Discovery (Conceptual DHT - MVDLI Scope)**
    *   **Description:**
        *   Use libp2p's Kademlia DHT implementation.
        *   After a `WitnessProofV1` is generated for a `NexusContentObjectV1` and its chunks are stored on an MVDLI SSN, the SSN (or the publisher) announces this by `PUT`ting key-value pairs like (`ContentID` -> `MVDLI_SSN_multiaddress`) and (`ChunkID` -> `MVDLI_SSN_multiaddress`) into the DHT.
        *   Clients use DHT `GET` operations to find which MVDLI SSN holds the content/chunks.
        *   Rudimentary DHT node joining (connecting to bootstrap DHT peers).
    *   **Inputs:** `discovery_protocol_refined.md`, `tech_specs/dds_protocol.md`.
    *   **Priority:** **Medium** (Content retrievability).
*   **Task 8: Simple Client Application / CLI Tool**
    *   **Description:** A command-line tool that can:
        *   Generate a new `did:echonet` identity.
        *   Create and publish a simple text `NexusContentObjectV1` (e.g., from a text file). This involves creating chunks, calculating `body_hash`, `core_metadata_hash`, `ContentID`, signing, submitting to DDS, and gossiping to PoW.
        *   Retrieve and display a `NexusContentObjectV1` by its `ContentID` (using Discovery and DDS).
    *   **Inputs:** All other MVDLI components.
    *   **Priority:** **Medium** (Essential for end-to-end testing and demonstration of MVDLI).

## 4. Development & Testing Approach for MVDLI

*   **Test-Driven Development (TDD):** Write unit tests for all core logic, especially:
    *   Canonicalization and hashing functions.
    *   Data structure serialization/deserialization.
    *   Signature generation and verification.
    *   Individual RPC handlers.
*   **Simulated Network Environment:**
    *   Use libp2p's testing tools or create simple scripts to launch multiple MVDLI nodes locally (or in containers) to simulate a small network.
    *   Test P2P interactions, event gossip, DHT operations, and the simplified PoW/DDS flows in this environment.
*   **Continuous Integration (CI):**
    *   Set up a basic CI pipeline (e.g., GitHub Actions, Jenkins) as outlined in `echonet_v3_cicd_strategy.md`.
    *   Automate builds, linting, and execution of unit tests on every commit/push.
*   **Incremental Integration:** Integrate components task by task, ensuring each part works before building the next layer.

## 5. Team & Skillset Considerations (Conceptual)

*   **Programming Languages:** Proficiency in Go is essential. Familiarity with Rust is a bonus for potential future optimizations.
*   **P2P Networking:** Strong understanding of P2P concepts and ideally experience with libp2p.
*   **Distributed Systems:** Knowledge of consensus mechanisms (even simplified), distributed storage, and DHTs.
*   **Cryptography:** Understanding of public key cryptography, hash functions, digital signatures.
*   **Software Engineering:** Commitment to good practices like TDD, CI, version control (Git).

## 6. Next Steps After MVDLI

Once the MVDLI is functional, stable, and demonstrates the core end-to-end flow, development will proceed iteratively:
1.  **Security Hardening:** Thorough security review and hardening of all MVDLI components.
2.  **Full PoW Implementation:** Integrate dynamic Witness registration, staking mechanisms, VRF-based committee selection, robust `WitnessProofV1` aggregation (e.g., BLS signatures), and initial slashing conditions.
3.  **Robust DDS Implementation:** Implement chunk replication, erasure coding, full PoSR challenges and proofs, and dynamic node announcements.
4.  **Full Discovery Protocol:** Secure and scalable DHT implementation, advanced peer routing.
5.  **Implement `NexusInteractionRecordV1` workflow** with PoW validation.
6.  **Develop Mobile "Host" Client:** Start implementing the mobile client based on `tech_specs/mobile_node_specifications.md` that can interact with the evolving DLI.
7.  **Begin implementation of further modules** as per the overall EchoNet architecture (e.g., Identity & Privacy enhancements, Content Validation & Anti-Spam tools, PoE).
8.  **Performance Optimization & Scalability Testing.**

This MVDLI plan provides a focused roadmap to build the foundational layer of EchoNet, enabling progressive enhancement towards the full vision.
