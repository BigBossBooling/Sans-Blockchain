**EchoNet Versioning Strategy (v3.0)**

This document defines a comprehensive versioning strategy for core `EchoNet` Distributed Ledger Inspired (DLI) protocol elements, DLI-native "contract" logic, critical data schemas (as defined in `echonet_v3_core_data_structures.md`), and related software components. This strategy aims to ensure system integrity, facilitate manageable upgrades, enable auditability, and align with KISS principles like "Iterate Intelligently" and "Systematize for Scalability." User feedback referencing Prometheus Protocol's versioning principles (emphasizing clear, compatible evolution) has also been incorporated.

**1. Versioning Philosophy for `EchoNet`**

*   **Goals:**
    *   **Clarity & Predictability:** Version numbers must clearly and unambiguously communicate the nature and impact of changes (e.g., bug fixes, new features, breaking changes).
    *   **Backward Compatibility Priority:** Prioritize backward compatibility for MINOR and PATCH releases to minimize disruption to the network, its users, and node operators.
    *   **Explicit Breaking Changes:** MAJOR version changes will clearly signify incompatible updates. These require careful planning, communication, and coordinated network upgrades managed via governance.
    *   **Support for Robust Network Upgrades:** The strategy must accommodate how a decentralized network of independent nodes adopts new versions, especially for DLI protocol-level changes.
    *   **Auditability & Traceability:** All version changes should be traceable to specific alterations in specifications, code, or governance decisions.
    *   **Interoperability:** Enable nodes running slightly different (but compatible) versions to interoperate where feasible.
*   **Scope of Versioning:** The following elements within `EchoNet` require explicit versioning:
    *   **DLI Protocol:** The core set of rules governing peer-to-peer communication (message formats, handshake procedures), consensus (Proof-of-Witness logic), transaction structures, DLI state validation rules, and overall network operation.
    *   **DLI-Native "Contract" Logic:** The deterministic logic executed by Witnesses/nodes for specific platform functions (e.g., PayoutContracts, AdContracts, PoERulesEngine).
    *   **Data Schemas:** Critical data structures exchanged between nodes, stored on the DLI, or used in APIs (as defined in `echonet_v3_core_data_structures.md`).
    *   **Node Software:** The executable software run by various node types (Witnesses, Standard Storage Nodes, Archival Storage Nodes, Indexer Nodes, etc.).
    *   **Client SDKs:** Libraries provided to third-party developers for building applications that interact with `EchoNet`.
    *   **API Endpoints:** Any RPC or HTTP APIs exposed by nodes for client-to-node or peer-to-peer communication (beyond the core DLI gossip/DHT protocols).

**2. Versioning Scheme**

*   **Primary Scheme: Semantic Versioning (SemVer v2.0.0 - MAJOR.MINOR.PATCH)**
    *   **`MAJOR` (e.g., 1.x.x -> 2.0.0):** Incremented for incompatible changes. For the DLI Protocol, this means nodes running different MAJOR versions are not expected to be compatible. For DLI-native contracts or data schemas, a MAJOR change implies that older logic/clients cannot process the new format/rules without updates or adapters.
    *   **`MINOR` (e.g., x.1.x -> x.2.0):** Incremented when new functionality is added in a backward-compatible manner. Older clients/nodes should continue to operate with newer MINOR versions (e.g., by ignoring new fields or new, optional protocol features).
    *   **`PATCH` (e.g., x.x.1 -> x.x.2):** Incremented for backward-compatible bug fixes or performance improvements that do not alter public interfaces, data schemas significantly, or DLI consensus rules.
*   **Application to Different Elements:**
    *   **Global DLI Protocol Version (`EchoNet-ProtocolVersion`):**
        *   A single SemVer (e.g., `echonetprotocol/2.1.0`) for the entire core `EchoNet` DLI protocol. This version dictates fundamental consensus rules, transaction processing, and core P2P messaging.
        *   Nodes must advertise their supported protocol MAJOR.MINOR versions during handshake. Connection may be refused if MAJOR versions mismatch. MINOR version differences should be handled gracefully (newer nodes supporting older MINOR versions of peers).
    *   **DLI-Native "Contract" Logic Version:**
        *   Each piece of DLI-native logic (e.g., `PayoutContractLogic`, `AdContractLogic`, `PoERulesLogic`) should be versioned independently using SemVer (e.g., `payoutlogic/1.2.0`).
        *   The `EchoNet-ProtocolVersion` will specify which versions of these DLI-native contract logics are considered canonical and active for a given network state. This allows for targeted updates to specific economic or social rules.
    *   **Data Schema Version (as per `echonet_v3_core_data_structures.md`):**
        *   Critical data structures (e.g., `ContentObject`, `InteractionEvent`, `PoWReceipt`) will include an explicit `version: SemanticVersionString` field (e.g., `ContentObject.version` is `1.0.0`).
        *   This allows data instances to self-describe their schema version.
        *   Code processing these structures must be capable of handling different (compatible) versions or explicitly rejecting/transforming incompatible ones.
    *   **Node Software Version:**
        *   Each node implementation (e.g., `EchoNetWitnessNode-Go/1.2.3`, `EchoNetSSN-Rust/1.2.5`) will have its own SemVer.
        *   This software version must clearly document which `EchoNet-ProtocolVersion` it implements and is compatible with.
    *   **Client SDK Version:**
        *   SDKs (e.g., `EchoNetSDK-JS/2.1.0`) will follow SemVer. Their versioning should indicate compatibility with specific `EchoNet-ProtocolVersion` ranges.
    *   **API Endpoint Versioning (for Node RPC/HTTP APIs):**
        *   APIs exposed by nodes (beyond core P2P) should be versioned, typically via a path prefix (e.g., `/rpc/v1/DDSNodeService/RetrieveChunk`, `/api/v2/metrics`).

**3. Managing Breaking Changes (MAJOR Version Upgrades)**

*   **DLI Protocol & Critical DLI-Native Contracts:**
    *   **Clear Communication:** Announce upcoming breaking changes, rationale, and migration paths to the community and node operators well in advance.
    *   **Deprecation Policy:** Define a clear deprecation window for old MAJOR versions (e.g., N months of support after a new MAJOR version is activated).
    *   **Governance Approval:** MAJOR protocol upgrades and changes to critical DLI-native contract logic must be approved via `EchoNet`'s decentralized governance mechanism.
    *   **Activation Strategy (Coordinated by Governance):**
        *   **Flag Day/Epoch Activation:** The network switches to the new rules at a predefined block height or timestamp. Requires widespread node operator readiness.
        *   **Signaling & Phased Rollout:** Node operators signal readiness for the new version. The upgrade activates once a predefined supermajority (e.g., >75% of Witness stake or count) signals. This allows for a more gradual transition.
    *   **Network Partitioning:** Nodes running incompatible MAJOR protocol versions must not fully peer or participate in consensus with each other to protect the integrity of each versioned network segment. They might connect for basic version discovery only.
*   **Data Schemas & APIs:**
    *   When a breaking change is made (e.g., `ContentObject` structure changes incompatibly), the `version` field in the data structure is incremented to a new MAJOR version.
    *   APIs serving or expecting these structures would also increment their MAJOR version.
    *   Old API versions should be deprecated with a clear timeline.

**4. Data Schema Evolution & Migration**

*   **Backward Compatibility (Preferred):**
    *   Adding new *optional* fields to data structures is generally a MINOR change and backward compatible. Older parsers can ignore new fields.
    *   Avoid changing the meaning or type of existing fields.
*   **Forward Compatibility (Graceful Degradation):**
    *   Older software should be robust to encountering data with new, unknown fields (from newer software) and ignore them if possible, rather than crashing.
*   **Handling Breaking Schema Changes (MAJOR Schema Version Update):**
    *   **New Structure Version:** Introduce a new, explicitly versioned data structure (e.g., `ContentObject_v2` alongside `ContentObject_v1`) or update the internal `version` field to the new MAJOR.
    *   **Transformation Logic/Adapters:** Where feasible, provide client-side or server-side logic to transform data between an old and new schema version during a transition period.
    *   **DLI State Migration (Highly Sensitive):** If a breaking schema change impacts data persisted directly in the global DLI state that affects consensus:
        *   This requires a coordinated network upgrade (MAJOR protocol version change).
        *   The upgrade logic must include a deterministic state migration function executed by all nodes at the activation point. This is complex and demands rigorous testing.
        *   A "write new, read old/new, then switch to new only" strategy over multiple phases might be employed for complex state migrations.

**5. DLI-Native "Contract" Logic Upgradability**

*   Upgrades to DLI-native "contract" logic (e.g., the rules for PoE rewards or ad auctions) are managed through `EchoNet` DLI Protocol version upgrades and governance.
*   A new version of a "contract" logic (e.g., `PoERulesLogic/2.0.0`) would be bundled within a new `EchoNet-ProtocolVersion`.
*   The protocol defines which version of each DLI-native contract logic is active. Witnesses use the DLI-ProtocolVersion to determine the correct set of rules for validating and executing these "contract" interactions.
*   This ensures all validating nodes are using the same deterministic logic for a given epoch or protocol state.

**6. Discoverability of Versions**

*   **DLI Protocol Version:** Nodes exchange their highest mutually supported `EchoNet-ProtocolVersion` (MAJOR.MINOR) during the P2P handshake. Patch versions are generally compatible if MAJOR.MINOR match.
*   **DLI-Native "Contract" Logic Versions:** The active versions are implicitly defined by the current `EchoNet-ProtocolVersion`. This can be made explicit by querying a special state object or governance module on `EchoNet`.
*   **Data Schema Versions:** The `version` field embedded within data structures (e.g., `ContentObject.version`) makes individual data instances self-describing.
*   **Node Software & API Versions:** Nodes can expose their software version and supported API versions via a standardized status/info RPC endpoint (e.g., `/node/status`). API URLs will reflect their MAJOR version.

**7. Tooling & Automation (Links to CI/CD - `echonet_v3_cicd_strategy.md`)**

*   **SemVer Adherence Checks:** CI pipelines should include steps to check if PRs modifying versioned components correctly increment SemVer based on the nature of changes (e.g., using commit message conventions like Conventional Commits to infer breaking changes).
*   **Schema Management & Validation Tools:**
    *   Employ schema definition languages (e.g., Protocol Buffers, JSON Schema) to formally define data structures.
    *   Use linters and compatibility checkers for these schema languages in CI to prevent accidental breaking changes in MINOR/PATCH versions.
*   **Automated Compatibility Testing:**
    *   Develop test suites that explicitly verify backward compatibility for MINOR/PATCH updates to protocols, schemas, and APIs.
    *   Simulate mixed-version node environments in testnets to ensure interoperability during upgrade periods.
*   **Release Automation:** CI/CD pipelines should automate the process of tagging releases, building versioned artifacts, and generating changelogs.

**8. Governance**

*   The `EchoNet` decentralized governance mechanism is paramount for versioning:
    *   **Approval of Protocol Upgrades:** MAJOR `EchoNet-ProtocolVersion` upgrades, and significant DLI-native contract logic changes, must be proposed to and approved by `EchoNet` governance.
    *   **Coordination of Activation:** Governance decides on the activation timeline and method (flag day, signaling) for breaking changes.
    *   **Management of Deprecation:** Oversees the official deprecation and sunsetting schedules for older protocol versions.
    *   **Dispute Resolution:** Can play a role in resolving issues arising from versioning conflicts or ambiguities, though clear specifications should minimize this.

This versioning strategy aims to provide a clear, robust, and adaptable framework for the evolution of `EchoNet`, balancing the need for innovation with the stability and predictability required by a decentralized network.
