**EchoNet Input Validation Strategy (v3.0)**

This document outlines a comprehensive strategy for validating all external inputs to `EchoNet` functions, module interfaces (as defined in `echonet_v3_component_interfaces.md`), DLI-native "contract" interactions, and peer-to-peer messages. This strategy is crucial for system security, robustness, and data integrity, directly implementing the "Validate external inputs mercilessly" aspect of the Expanded KISS Principle.

**1. Input Validation Philosophy for `EchoNet`**

*   **Assume Zero Trust:** All external inputs are considered untrusted and potentially malicious or malformed until explicitly validated. This applies to data from client applications, other nodes in the P2P network, and even different internal modules if they cross trust boundaries.
*   **Fail-Fast & Early:** Validate inputs as early as possible upon receipt, at the entry point of a module or function. If invalid, reject the input immediately to prevent further processing with tainted data.
*   **Principle of Least Privilege (for data):** Functions and modules should only accept data that is strictly necessary for their operation. Superfluous data in requests should be viewed with suspicion or ignored.
*   **Specificity in Validation:** Use precise validation rules for each field based on its expected type, format, range, and contextual constraints. Leverage definitions in `echonet_v3_core_data_structures.md`.
*   **Clear Error Reporting:** When validation fails, return specific, actionable error types (as defined in `echonet_v3_error_handling_strategy.md`) that clearly indicate which field failed validation and why.
*   **Centralized Definitions, Decentralized Enforcement:** Define validation rules centrally (e.g., within data structure definitions or a shared validation library), but enforce them at the point of input in each component.

**2. Categorization of Inputs Requiring Validation**

*   **User-Generated Data (via Client Applications):**
    *   Content submissions (`ContentObject` and its `CoreMetadataObject`).
    *   Interaction events (`InteractionEvent` and its specific payloads like `CommentPayload`, `ReactionPayload`, `FlagPayload`).
    *   User profile updates (`UserProfileCore`).
    *   Ad campaign definitions (`AdCampaignDefinition`).
*   **Peer-to-Peer (P2P) Network Messages:**
    *   DHT messages (PING, FIND_NODE, STORE_VALUE, FIND_VALUE requests/responses as per `DiscoveryRPCService`).
    *   Gossip protocol messages.
    *   Witness attestations (`WitnessAttestation`) and PoW Receipts (`PoWReceipt`) propagated through the network.
    *   DDS chunk requests/responses between nodes.
*   **RPC Requests (Inter-Node/Client-to-Node):**
    *   All parameters within RPC requests defined in `echonet_v3_communication_protocols.md` (e.g., requests to `DDSNodeRPCService`, `EventSubmissionRPCService`, `AdMarketplaceRPCService`).
*   **Inputs to DLI-Native "Contract" Logic:**
    *   Parameters passed to functions within DLI-native contracts (e.g., `initCampaign` for AdContract, `distributeEarnings` for PayoutContract). These are often derived from validated network events.

**3. Standard Validation Checks to Apply (General List)**

This list leverages the validation rules already conceptually defined in `echonet_v3_core_data_structures.md` and expands on them for operational input validation.

*   **Type Checking:** Verify that each input field's data type matches its definition (e.g., `string`, `int64`, `bool`, `array`, `object/message`).
*   **Presence Checking (Mandatory Fields):** Ensure all required fields are present in the input.
*   **Format Validation:**
    *   `DIDString`: Must conform to the valid DID specification (e.g., "did:method:identifier").
    *   `ContentIDString`, `HashString`: Must be valid SHA256 hex strings (or other chosen hash format).
    *   `TimestampInt64`: Must be a valid integer representing Unix epoch milliseconds, often within a reasonable range (e.g., not too far in the past or future relative to network time).
    *   `URLString`: Must be a syntactically valid URL.
    *   `SemanticVersionString`: Must parse according to SemVer rules.
    *   `CurrencyAmountString`: Must be a string representing a positive numerical value, parsable into a high-precision decimal type.
    *   `TokenIDString`: Must be a recognized/registered token identifier.
*   **Range Checking (Numerical Values):** Ensure numerical values fall within acceptable minimum/maximum bounds (e.g., `poeScore` non-negative, `budget` positive).
*   **Length Checking:**
    *   Strings: Min/max character length (e.g., `CoreMetadataObject.title`, `CommentPayload.commentBody`).
    *   Arrays/Lists: Min/max number of elements (e.g., `CoreMetadataObject.tags`).
    *   Byte Sequences: Min/max byte length (e.g., `ContentObject.bodyChunkIDs` array size, `InteractionEvent.payload` max size).
*   **Enum Value Checking:** Ensure that fields expecting an enumerated value contain one of the predefined valid options (e.g., `InteractionEvent.interactionType`, `FlagPayload.flagReason`).
*   **Signature Verification:** For all data that is claimed to be signed (e.g., `ContentObject.signature`, `InteractionEvent.signature`, `WitnessAttestation.signature`, `NodeAdvertisement.signature`). This involves:
    *   Verifying the signature against the public key associated with the declared signer's DID.
    *   Ensuring the signature is computed over the correctly canonicalized representation of the data (as per `echonet_v3_canonical_data_formats.md`).
*   **Canonicalization Conformance (Pre-Hashing/Signature Verification):** Before hashing an input to compare with a provided hash (e.g., `ContentObject.contentID` vs. its constituents) or before verifying a signature, ensure the input data *can be* canonicalized according to rules. Reject if it contains elements that would prevent deterministic canonicalization (e.g., unsorted map keys if the canonical form requires sorted keys).
*   **Resource Limit Checks (Anti-DoS):**
    *   Maximum size of content body or individual chunks.
    *   Maximum number of elements in request arrays (e.g., max tags per content, max `bodyChunkIDs`).
    *   Maximum depth of nested structures.
*   **Duplicate/Replay Prevention:**
    *   For state-changing operations or unique events, check for duplicate `eventID`s, nonces, or client-asserted timestamps (within a defined window and context) to prevent replay attacks. `PoWReceipt` helps make validated events unique.
*   **Contextual & Business Logic Validation:**
    *   Cross-field validation (e.g., `AdCampaignDefinition.endDate` must be after `startDate`).
    *   State-dependent validation (e.g., an `AdInteractionEvent` must reference an existing, active `AdCampaignDefinition.campaignID`).
    *   Reputation checks (e.g., user has sufficient reputation to perform an action).

**4. Input Validation Points for Key `EchoNet` Interfaces/Functions/Messages**

Validation must occur at the entry point of any component that receives external data. "External" can mean from a user, another peer node, or even another internal module if it's a distinct trust/operational boundary.

*   **Interface: `ContentPublishingService` / `EventSubmissionRPCService`**
    *   **`SubmitContent(content: ContentObject)` / `SubmitValidatedEvent(eventPayload: bytes where eventType="ContentObjectPublication")`**
        *   `ContentObject.contentID`: Validate format; later verify it matches hash of canonicalized content.
        *   `ContentObject.version`: Must be a supported `SemanticVersionString`.
        *   `ContentObject.creatorDID`: Must be valid `DIDString`.
        *   `ContentObject.clientAssertedTimestamp`: Validate range.
        *   `ContentObject.contentType`: Must be a recognized/supported MIME type or `EchoNet` type.
        *   `ContentObject.bodyHash`: Validate format.
        *   `ContentObject.bodyChunkIDs` (if present): Validate array length limits; each element valid `ContentIDString`.
        *   `ContentObject.coreMetadataHash`: Validate format.
        *   `ContentObject.coreMetadataObject` (if passed directly before hashing into `coreMetadataHash`): Validate all its fields (title length, tags count/format, etc.) as per `CoreMetadataObject` definition.
        *   `ContentObject.signature`: Verify against `creatorDID` and canonicalized content pre-image.
        *   Return `VAL-XXXX` errors for field issues, `CCM-XXXX` for content-specific logic, `CHT-XXXX` for hash/timestamp issues.

*   **Interface: `ChunkStorageService` / `DDSNodeRPCService`**
    *   **`StoreChunk(chunkID: ContentIDString, chunkData: bytes, ...)`**
        *   `chunkID`: Must be valid `ContentIDString` format. Optionally, verify it matches `hash(chunkData)`.
        *   `chunkData`: Check against max chunk size limit; ensure not empty.
        *   Return `VAL-XXXX` or `DDS-XXXX` errors.
    *   **`RetrieveChunk(chunkID: ContentIDString)`**
        *   `chunkID`: Must be valid `ContentIDString` format.
        *   Return `VAL-XXXX` or `DDS-XXXX` errors.
    *   **`GeneratePoSRProof(challenge: PoSRChallenge)`**
        *   `PoSRChallenge` (structure TBD): Validate all fields (e.g., nonce format, chunk range validity).
        *   Return `VAL-XXXX` or `ErrDDSInvalidPoSRChallenge`.

*   **Interface: `NetworkEventValidator` (within PoW Module)**
    *   **`SubmitEventForValidation(eventID: HashString, eventType: string, eventPayload: bytes, ...)`**
        *   `eventID`: Validate `HashString` format.
        *   `eventType`: Must be a recognized event type string.
        *   `eventPayload`: Attempt to deserialize based on `eventType`. If fails, `ErrPOWInvalidEventFormatForValidation`. Then, apply all data structure specific validations (from `echonet_v3_core_data_structures.md`) to the deserialized payload.
        *   Verify signature on `eventPayload` (or `eventID` if that's the signed digest) matches the actor/creator DID within the payload.
        *   Apply any event-specific business rules (e.g., for `InteractionEvent`, does `targetContentID` exist? Is `actorDID` allowed to interact?).
        *   Return `VAL-XXXX`, `POW-XXXX`, or event-specific errors.

*   **Interface: `ResourceDiscoveryProvider` / `DiscoveryRPCService` (DHT Operations)**
    *   **`FindContentChunkLocations(chunkID: ContentIDString)` / `FindValue(key: HashString)`**
        *   `chunkID`/`key`: Validate `ContentIDString`/`HashString` format.
    *   **`RegisterNodeAdvertisement(advertisement: NodeAdvertisement)` / `StoreValue(key: HashString, value: DHTRecordValue_ContentLocation, ...)`**
        *   `advertisement`: Validate all fields of `NodeAdvertisement` (DIDs, addresses, service types, timestamps, signature).
        *   `key`: Validate `HashString` format.
        *   `value`: Validate all fields of `DHTRecordValue_ContentLocation` (especially signatures, expiry).
        *   Return `VAL-XXXX` or `DP-XXXX` errors.

*   **Interface: `InteractionSubmissionService`**
    *   **`SubmitSignedInteraction(signedInteraction: InteractionEvent)`**
        *   `InteractionEvent`: Validate all fields as per its definition in `echonet_v3_core_data_structures.md` (eventID format, interactionType enum, DIDs, timestamps, payload deserialization and validation, signature).
        *   Return `VAL-XXXX` or `EI-XXXX` errors.

*   **Interface: `AdCampaignOracle` / `AdMarketplaceRPCService`**
    *   **`CreateNewCampaign(signedCampaignDef: AdCampaignDefinition)`**
        *   `AdCampaignDefinition`: Validate all fields (IDs, DIDs, budget format/range, token type, bid prices, targeting criteria, dates, signature).
        *   Return `VAL-XXXX` or `ADM-XXXX` errors.
    *   **`LogAdInteraction(signedAdInteraction: AdInteractionEvent)`**
        *   `AdInteractionEvent`: Validate all fields (IDs, DIDs, type, cost, PoWReceipt presence/validity, signature).
        *   Return `VAL-XXXX` or `ADM-XXXX` errors.

*   **DLI-Native "Contract" Interactions (Conceptual)**
    *   **Example: `PayoutContract.distributeEarnings(periodID: string)`**
        *   `periodID`: Validate format/existence.
        *   Internal logic would fetch `VerifiedViewEvent`s; these events are already validated by PoW.
        *   Ensure payout calculations do not result in negative amounts or exceed pool balances.
    *   **Example: `AdContract.initCampaign(advertiserDID: DIDString, budget: CurrencyAmountString, ...)`**
        *   `advertiserDID`: Must be valid `DIDString`.
        *   `budget`: Must be positive `CurrencyAmountString`, check against minimums/maximums.
        *   All parameters passed to DLI-native contract functions must be validated for type, format, and logical consistency before altering state.

**5. Validation Order (General Guideline)**

A typical order can improve efficiency and security:
1.  **Authentication & Authorization:** Verify identity (e.g., signature) and basic permissions first.
2.  **Presence & Type Checks:** Ensure all mandatory fields are present and have the correct basic data types.
3.  **Format & Syntactic Validation:** Check specific formats (DIDs, Hashes, URLs, Timestamps).
4.  **Length & Range Checks:** Validate string lengths, array sizes, numerical ranges.
5.  **Resource Limit Checks:** E.g., overall request size, number of items.
6.  **Value & Enum Checks:** Validate against predefined sets of allowed values.
7.  **Canonicalization & Cryptographic Checks:** If data was signed over a canonical form, ensure the input *can* be canonicalized correctly before attempting signature verification or hash comparison.
8.  **Contextual & Business Logic Validation:** Cross-field consistency, state-dependent rules.

**6. Centralized vs. Decentralized Validation Logic**

*   **Centralized Definitions:** Validation rules for core data structures should be defined alongside those structures (as in `echonet_v3_core_data_structures.md`).
*   **Shared Validation Libraries/Functions:** Develop and use shared libraries (SDKs, common node libraries) that implement these validation rules. This ensures consistency across different modules and node types that process the same data.
*   **Decentralized Enforcement:** Each module or service endpoint is responsible for calling these shared validation functions upon receiving any external input. Do not trust that input received from another internal module has already been perfectly validated if it crossed a significant system boundary or if the modules have different trust levels/update cycles.

By rigorously applying these input validation strategies at every external interface and interaction point, `EchoNet` can significantly enhance its security, prevent data corruption, and ensure more predictable system behavior.
