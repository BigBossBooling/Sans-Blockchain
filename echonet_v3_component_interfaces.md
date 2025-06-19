**EchoNet Component Interfaces (v3.0)**

This document defines explicit interfaces between `EchoNet` core components, conceptualized as Go-like interfaces or API contracts. This aims to improve independent development, testability, and the potential for swappable implementations where appropriate, adhering to the "Modular Interfaces" aspect of the Expanded KISS Principle.

**1. Interface Design Philosophy**

*   **Explicitness:** Interfaces should clearly define the methods, parameters, return types, and potential errors.
*   **Minimalism (Role-Based):** Interfaces should be role-specific and contain only the methods necessary for that role. Components may implement multiple small interfaces rather than one large one.
*   **Dependency Inversion:** Components should depend on abstractions (interfaces) rather than concrete implementations of other components.
*   **Clear Data Contracts:** Parameter and return types should use the standardized data structures defined in `echonet_v3_core_data_structures.md`.
*   **Error Reporting:** Methods that can fail must explicitly return specific error types defined in `echonet_v3_error_handling_strategy.md`.
*   **Synchronicity Specification:** Clearly indicate if a method call is expected to be synchronous or asynchronous in its primary effect.

---
**General Type Usage in Interfaces:**
*   Types like `ContentIDString`, `DIDString`, `HashString`, `TimestampInt64`, `SignatureBytes`, `CurrencyAmountString`, `TokenIDString` refer to definitions in `echonet_v3_core_data_structures.md`.
*   Specific `ErrorType`s (e.g., `ErrDDSChunkNotFound`) refer to definitions in `echonet_v3_error_handling_strategy.md`.
*   Data structures (e.g., `ContentObject`, `InteractionEvent`) refer to definitions in `echonet_v3_core_data_structures.md`.

---
**Interface: `ContentSubmissionService`**
*   **Purpose:** Defines the contract for submitting new or updated content to `EchoNet` for validation and storage.
*   **Implemented By:** A component within or fronting the Proof-of-Witness (PoW) Validation module.
*   **Consumed By:** Content Creation & Management Module (client-side or user's node).
*   **Methods:**
    *   `SubmitContent(content: ContentObject) -> (receiptID: HashString, error: ErrorType)`
        *   Description: Submits a `ContentObject` for validation and eventual storage. The call is likely asynchronous in its full lifecycle (PoW consensus, DDS storage). The `receiptID` might initially be a submission acknowledgement ID, later updated or correlated with a `PoWReceipt.receiptID`.
        *   Parameters:
            *   `content`: `ContentObject`
        *   Returns: An identifier for the submission process or a `PoWReceipt.receiptID` if validation is rapid; or an error (e.g., `ErrCCMSerializationFormatUnsupported`, `ErrPOWInvalidEventFormatForValidation`, `ErrNetworkUnreachable`).
        *   Synchronicity: Asynchronous for full processing; may return synchronous acknowledgement.

---
**Interface: `DDSStorageServiceProvider`**
*   **Purpose:** Defines the contract for nodes providing Distributed Data Store (DDS) services, allowing other components to store, retrieve, and manage content chunks. (Ref: `echonet_v3_core_definitions.md #DistributedDataStoresModule`)
*   **Implemented By:** Standard Storage Nodes (SSNs), Archival Storage Nodes (ASNs).
*   **Consumed By:** Content Creation & Management Module (indirectly via `ContentPersistenceService`), Replication & Redundancy Module, Content Discovery Module (client-side for retrieval), Proof-of-Witness Module (for PoSR).
*   **Methods:**
    *   `StoreChunk(chunkID: ContentIDString, chunkData: bytes, storageTierHint: string) -> (success: bool, error: ErrorType)`
        *   Description: Stores a single data chunk. `storageTierHint` (e.g., "hot", "archive") suggests desired storage class.
        *   Parameters:
            *   `chunkID`: `ContentIDString`
            *   `chunkData`: `bytes`
            *   `storageTierHint`: `string` (enum: "standard", "archive")
        *   Returns: Success status or error (e.g., `ErrDDSInsufficientStorageCapacity`, `ErrDDSStorageNodeUnresponsive`, `VAL-XXXX` for invalid chunkID).
        *   Synchronicity: Asynchronous (acknowledges receipt; actual replication managed by Replication & Redundancy).
    *   `RetrieveChunk(chunkID: ContentIDString) -> (chunkData: bytes, error: ErrorType)`
        *   Description: Retrieves the data for a given `chunkID`.
        *   Parameters:
            *   `chunkID`: `ContentIDString`
        *   Returns: The chunk data or error (e.g., `ErrDDSChunkNotFound`, `NET-XXXX` for network issues).
        *   Synchronicity: Synchronous.
    *   `RespondToPoSRChallenge(challenge: PoSRChallenge) -> (proof: PoSRProof, error: ErrorType)`
        *   Description: Responds to a Proof-of-Storage/Retrievability challenge. (PoSRChallenge & PoSRProof structures TBD).
        *   Parameters:
            *   `challenge`: `PoSRChallenge`
        *   Returns: The `PoSRProof` or error (e.g., `ErrDDSInvalidPoSRChallenge`, `ErrDDSPoSRVerificationFailed`).
        *   Synchronicity: Synchronous.
    *   `GetNodeMetrics() -> (metrics: NodeAdvertisement, error: ErrorType)`
        *   Description: Returns current metrics/advertisement for the storage node.
        *   Returns: `NodeAdvertisement` data or error.
        *   Synchronicity: Synchronous.

---
**Interface: `ContentPersistenceCoordinator`**
*   **Purpose:** Manages the overall persistence (storage, replication, redundancy) of content after it has been validated.
*   **Implemented By:** Replication & Redundancy Module.
*   **Consumed By:** Proof-of-Witness Module (to trigger persistence after validation).
*   **Methods:**
    *   `PersistValidatedContent(contentObject: ContentObject, powReceipt: PoWReceipt) -> (success: bool, error: ErrorType)`
        *   Description: Initiates the storage of content chunks on DDS according to replication/erasure coding schemes, using available `DDSStorageServiceProvider` nodes.
        *   Parameters:
            *   `contentObject`: `ContentObject` (contains `bodyChunkIDs` if applicable, or `bodyHash` for single chunk)
            *   `powReceipt`: `PoWReceipt` (proof of validation)
        *   Returns: Success status or error (e.g., `ErrDDSReplicationFactorNotMet`, `ErrDDSErasureCodingFailed`, `ErrDPDHTStoreRejected` if DHT update fails).
        *   Synchronicity: Asynchronous.

---
**Interface: `EventValidationService`**
*   **Purpose:** Defines the contract for the Proof-of-Witness (PoW) module to validate various types of network events.
*   **Implemented By:** Proof-of-Witness (PoW) Validation Module.
*   **Consumed By:** Any module submitting an event for network consensus (Content Creation, Engagement & Interaction, Monetization modules, DDS for PoSR responses).
*   **Methods:**
    *   `ValidateEvent(eventData: bytes, eventType: string) -> (receipt: PoWReceipt, error: ErrorType)`
        *   Description: Submits an event for validation by a Witness Committee. The eventData is the canonicalized, signed event structure.
        *   Parameters:
            *   `eventData`: `bytes` (e.g., serialized `ContentObject`, `InteractionEvent`, `AdInteractionEvent`, `PoSRProof`)
            *   `eventType`: `string` (e.g., "ContentObject", "InteractionEvent_Comment", "PoSRProof")
        *   Returns: A `PoWReceipt` upon successful validation by consensus, or an error (e.g., `ErrPOWAttestationThresholdNotMet`, `ErrPOWInvalidEventFormatForValidation`, `VAL-XXXX` for data issues).
        *   Synchronicity: Asynchronous (consensus takes time). The returned `PoWReceipt` is the eventual result. Initial call might return a submission ID.

---
**Interface: `DiscoveryService`**
*   **Purpose:** Defines the contract for discovering resources (content locations, node services) on `EchoNet`. (Ref: `echonet_v3_core_definitions.md #DiscoveryProtocolModule`)
*   **Implemented By:** Discovery Protocol Module (e.g., Kademlia DHT nodes).
*   **Consumed By:** All modules needing to find content or other nodes (Content Discovery, DDS, Replication & Redundancy, Client SDKs).
*   **Methods:**
    *   `FindContentLocations(contentID: ContentIDString) -> (locations: array<DDSChunkLocation>, error: ErrorType)`
        *   Description: Finds SSNs that store chunks for the given `ContentID`.
        *   Returns: List of locations or error (e.g., `ErrDPDHTLookupNotFound`).
        *   Synchronicity: Synchronous (though may involve multiple network hops).
    *   `AdvertiseService(advertisement: NodeAdvertisement) -> (success: bool, error: ErrorType)`
        *   Description: A node advertises its services (e.g., SSN, Witness) to the network.
        *   Returns: Success status or error (e.g., `ErrDPDHTStoreRejected`).
        *   Synchronicity: Asynchronous.
    *   `FindServiceNodes(serviceType: string, regionHint: string, maxResults: int) -> (nodes: array<NodeAdvertisement>, error: ErrorType)`
        *   Description: Finds nodes offering a specific service.
        *   Returns: List of node advertisements or error.
        *   Synchronicity: Synchronous.

---
**Interface: `InteractionHandlerService`**
*   **Purpose:** Defines how user interactions (comments, reactions, etc.) are submitted to `EchoNet`.
*   **Implemented By:** A component within or fronting the Engagement & Interaction Module, which then uses `EventValidationService`.
*   **Consumed By:** Client SDKs / User-facing applications.
*   **Methods:**
    *   `SubmitInteraction(interaction: InteractionEvent) -> (receiptID: HashString, error: ErrorType)`
        *   Description: Submits a user interaction for validation and processing.
        *   Parameters:
            *   `interaction`: `InteractionEvent`
        *   Returns: An identifier for the submission (potentially `InteractionEvent.eventID` or a `PoWReceipt.receiptID`) or an error (e.g., `VAL-XXXX`, `ErrPOWInvalidEventFormatForValidation`).
        *   Synchronicity: Asynchronous for full processing.

---
**Interface: `PoERewardOracle`**
*   **Purpose:** Provides an interface for calculating and triggering Proof-of-Engagement rewards.
*   **Implemented By:** Proof-of-Engagement (PoE) Rewards Module.
*   **Consumed By:** `EchoNet` DLI-native "PoE Contract" (conceptually, triggered by a scheduler or governance action).
*   **Methods:**
    *   `CalculatePoEScoresForEvent(interactionEvent: InteractionEvent, witnessAttestations: array<WitnessAttestation>) -> (poeScore: int32, error: ErrorType)`
        *   Description: Calculates the PoE score for a specific validated interaction, based on witness input and PoE rules. This might be an internal method called during extended validation by Witnesses.
        *   Synchronicity: Synchronous.
    *   `TriggerPoERewardDistribution(periodID: string) -> (payoutSummaryID: HashString, error: ErrorType)`
        *   Description: Initiates the process of calculating and distributing PoE rewards for a given period.
        *   Returns: An ID for the payout batch or an error (e.g., `ErrPOERewardPoolEmpty`).
        *   Synchronicity: Asynchronous.

---
**Interface: `MonetizationTransactionService` (for DLI-native "Contracts")**
*   **Purpose:** Defines how DLI-native contracts (Payout, Ad Campaign Escrow) interact with the underlying ledger for value transfer.
*   **Implemented By:** A core `EchoNet` ledger/DLI transaction processing layer.
*   **Consumed By:** DLI-native "contracts" within Direct Monetization, Ad Marketplace, PoE Rewards modules.
*   **Methods:**
    *   `TransferTokens(fromAddress: DIDString, toAddress: DIDString, amount: CurrencyAmountString, tokenID: TokenIDString, memo: string) -> (transactionID: HashString, error: ErrorType)`
        *   Description: Executes a token transfer. The "fromAddress" for a contract would be its own DLI-managed address. Requires authorization (e.g., only callable by the contract's own logic triggered by validated events).
        *   Returns: Transaction ID or error (e.g., `ErrDMWalletInsufficientBalance`, `ErrDMPayoutInvalidRecipientDID`).
        *   Synchronicity: Asynchronous (settlement depends on `EchoNet` event validation).
    *   `GetBalance(address: DIDString, tokenID: TokenIDString) -> (balance: CurrencyAmountString, error: ErrorType)`
        *   Description: Retrieves the balance of a given token for a specific address.
        *   Synchronicity: Synchronous (reads current validated state).

---
**Interface: `AdCampaignManager`**
*   **Purpose:** Defines the contract for managing ad campaigns within the Decentralized Ad Marketplace.
*   **Implemented By:** Decentralized Ad Marketplace Module (likely its DLI-native "Ad Contract" component).
*   **Consumed By:** Advertisers (via Client SDKs), Ad Serving Logic.
*   **Methods:**
    *   `CreateCampaign(campaignDef: AdCampaignDefinition) -> (campaignID: HashString, error: ErrorType)`
        *   Description: Registers a new ad campaign and escrows the budget.
        *   Returns: Campaign ID or error (e.g., `VAL-XXXX` for definition, `ErrDMWalletInsufficientBalance`).
        *   Synchronicity: Asynchronous.
    *   `RecordAdInteraction(interaction: AdInteractionEvent) -> (success: bool, error: ErrorType)`
        *   Description: Records a validated ad impression or click against a campaign, debiting its budget.
        *   Returns: Success or error (e.g., `ErrADMCampaignBudgetExhausted`).
        *   Synchronicity: Asynchronous (subject to `AdInteractionEvent` validation).
    *   `GetCampaignStatus(campaignID: HashString) -> (status: AdCampaignStatus, error: ErrorType)` (AdCampaignStatus TBD data structure)
        *   Description: Retrieves current status and performance metrics for a campaign.
        *   Synchronicity: Synchronous.
    *   `FinalizeCampaign(campaignID: HashString) -> (refundAmount: CurrencyAmountString, error: ErrorType)`
        *   Description: Ends a campaign and refunds any unused budget.
        *   Synchronicity: Asynchronous.

---
**Interface: `ReputationOracle`**
*   **Purpose:** Provides access to user reputation scores.
*   **Implemented By:** Unified User Identity & Reputation System.
*   **Consumed By:** PoE Rewards Module, Proof-of-Witness Module (for selection/weighting), Ad Marketplace (for targeting/fraud), Engagement & Interaction (for weighted flagging).
*   **Methods:**
    *   `GetUserReputation(userDID: DIDString, reputationType: string) -> (score: int64, error: ErrorType)`
        *   Description: Retrieves a specific type of reputation score for a user.
        *   Parameters:
            *   `userDID`: `DIDString`
            *   `reputationType`: `string` (e.g., "overall", "content_quality", "poe_contributor", "witness_reliability")
        *   Returns: Reputation score or error if user/type not found.
        *   Synchronicity: Synchronous.
    *   `UpdateUserReputation(userDID: DIDString, eventType: string, eventMagnitude: float64, eventContentID: ContentIDString) -> (newScore: int64, error: ErrorType)`
        *   Description: Called by other modules to report events that should influence reputation. The Reputation system internally processes this.
        *   Synchronicity: Asynchronous.

---

This initial set of interfaces provides a basis for clearer separation of concerns and responsibilities. Further refinement will occur as each module's internal logic is detailed. Each `error: ErrorType` implies a specific error from `echonet_v3_error_handling_strategy.md` relevant to the operation.
