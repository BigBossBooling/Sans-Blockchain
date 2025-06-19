**EchoNet Core Data Structures (v3.0)**

This document details core data structures used within the `EchoNet` system, specifying explicit field types, clear naming, conceptual validation logic, and considerations for canonicalization. These definitions align with the core purposes identified in `echonet_v3_core_definitions.md`.

---
**General Type Definitions:**
*   `DIDString`: `string` (Decentralized Identifier string, e.g., "did:ethr:0x...")
*   `ContentIDString`: `string` (SHA256 hex string representing a unique content identifier)
*   `TimestampInt64`: `int64` (Unix epoch milliseconds)
*   `SignatureBytes`: `bytes` (Cryptographic signature)
*   `HashString`: `string` (SHA256 hex string)
*   `URLString`: `string` (A valid URL)
*   `SemanticVersionString`: `string` (e.g., "1.0.2")
*   `CurrencyAmountString`: `string` (Represents a numeric currency amount, supporting large numbers and precision, e.g., using a string to avoid floating point issues)
*   `TokenIDString`: `string` (Identifier for a specific type of token within EchoNet, e.g., "EchoCredit", "StableCoinUSD")

---

**Data Structure: `CoreMetadataObject`**
*   **Purpose:** Stores essential, largely immutable metadata associated with a `ContentObject`. (Ref: `echonet_v3_core_definitions.md #ContentCreationModule`)
*   **Fields:**
    *   `title`: `string`
        *   Description: The title of the content.
        *   Validation: Max length (e.g., 256 chars); must not be empty for article types.
    *   `tags`: `array<string>`
        *   Description: List of descriptive tags or keywords.
        *   Validation: Max array length (e.g., 20); each tag max length (e.g., 50 chars), no special chars other than hyphen.
    *   `summary`: `string` (optional)
        *   Description: A short summary or abstract of the content.
        *   Validation: Max length (e.g., 1024 chars).
    *   `language`: `string` (e.g., "en-US", "es-MX")
        *   Description: Primary language of the content, using ISO 639-1 codes.
        *   Validation: Must be a valid ISO 639-1 code.
    *   `customAttributes`: `map<string, string>` (optional)
        *   Description: Key-value pairs for additional, type-specific metadata.
        *   Validation: Max key/value length; max number of attributes.
*   **Canonicalization for Hashing (Conceptual):**
    *   Fields ordered alphabetically. Values (including array elements and map entries sorted by key) concatenated with defined separators.

---

**Data Structure: `ContentObject`**
*   **Purpose:** Represents a single piece of publishable, identifiable content within EchoNet. (Ref: `echonet_v3_core_definitions.md #ContentCreationModule`, `#ContentHash&Timestamping`)
*   **Fields:**
    *   `contentID`: `ContentIDString`
        *   Description: Unique, content-addressable identifier (hash of: `version`, `creatorDID`, `clientAssertedTimestamp`, `contentType`, `bodyHash`, `coreMetadataHash`).
        *   Validation: Must be a valid SHA256 hex string; must match on-the-fly recalculation if constituent parts are available for validation.
    *   `version`: `SemanticVersionString` (e.g., "1.0.0")
        *   Description: Version of the ContentObject structure itself.
        *   Validation: Must follow semantic versioning rules.
    *   `creatorDID`: `DIDString`
        *   Description: The Decentralized Identifier of the content creator.
        *   Validation: Must be a valid DID string format.
    *   `clientAssertedTimestamp`: `TimestampInt64`
        *   Description: Timestamp (Unix epoch ms) asserted by the client at the moment of creation/submission.
        *   Validation: Must be a positive integer; should not be unreasonably in the future or past.
    *   `networkValidatedTimestamp`: `TimestampInt64` (optional, populated after PoW validation)
        *   Description: Timestamp (Unix epoch ms) validated by EchoNet Witnesses (median from PoW Receipt).
        *   Validation: If present, must be a positive integer.
    *   `contentType`: `string` (enum or namespaced string, e.g., "text/markdown;version=1.2", "image/jpeg", "application/echo-shortpost.v1+json")
        *   Description: Defines the MIME type and version of the content body's schema/format.
        *   Validation: Must be a valid MIME type format or a registered `EchoNet` content type.
    *   `bodyHash`: `HashString`
        *   Description: Cryptographic hash (e.g., SHA256) of the raw content body.
        *   Validation: Must be a valid hash string; ensures integrity of the body.
    *   `bodyChunkIDs`: `array<ContentIDString>` (optional)
        *   Description: Ordered list of `ContentIDString`s for the DDS chunks that constitute the full content body. Used if the body is chunked.
        *   Validation: If present, array elements must be valid `ContentIDString`s. Max array length depends on max content size.
    *   `coreMetadataHash`: `HashString`
        *   Description: Hash of the canonically serialized `CoreMetadataObject`.
        *   Validation: Must be a valid hash string.
    *   `coreMetadataRef`: `ContentIDString` (optional)
        *   Description: A pointer (`ContentIDString`) to a separate `ContentObject` that holds the `CoreMetadataObject`, allowing metadata to be shared or independently updated if design allows (though `coreMetadataHash` implies immutability for `ContentID` calculation). *Further design needed if metadata is to be independently mutable while preserving `ContentID` for body.* For now, assume `coreMetadataHash` is primary.
        *   Validation: If present, must be a valid `ContentIDString`.
    *   `previousVersionID`: `ContentIDString` (optional)
        *   Description: `ContentID` of the previous version of this content, if this is an update.
        *   Validation: If present, must be a valid `ContentIDString`.
    *   `signature`: `SignatureBytes`
        *   Description: Cryptographic signature of the `ContentID` (or its constituent parts before final hashing) by the `creatorDID`.
        *   Validation: Must be a valid signature verifiable with `creatorDID`'s public key.
*   **Canonicalization for Hashing/Signing (Conceptual):**
    *   A strict, deterministic serialization of all fields that contribute to the `ContentID` (e.g., `version`, `creatorDID`, `clientAssertedTimestamp`, `contentType`, `bodyHash`, `coreMetadataHash`) is required. Fields are typically ordered alphabetically by name, values converted to UTF-8, and concatenated with defined separators. The `signature` is applied to this canonical representation.

---

**Data Structure: `InteractionEvent`**
*   **Purpose:** Represents a generic user interaction with a piece of content or another user. (Ref: `echonet_v3_core_definitions.md #Engagement&InteractionModule`)
*   **Fields:**
    *   `eventID`: `HashString` (SHA256 hex of canonicalized core event data)
        *   Description: Unique identifier for this interaction event.
        *   Validation: Must be a valid SHA256 hex string.
    *   `interactionType`: `string` (enum: "comment", "reaction", "share", "flag", "follow")
        *   Description: Type of interaction.
        *   Validation: Must be one of the allowed enum values.
    *   `actorDID`: `DIDString`
        *   Description: The DID of the user performing the interaction.
        *   Validation: Must be a valid DID string format.
    *   `targetContentID`: `ContentIDString` (optional)
        *   Description: The `ContentID` of the content being interacted with.
        *   Validation: If present, must be a valid `ContentIDString`. Required for comment, reaction, share, flag.
    *   `targetDID`: `DIDString` (optional)
        *   Description: The DID being interacted with (e.g., for a follow).
        *   Validation: If present, must be a valid DID string format. Required for follow.
    *   `clientAssertedTimestamp`: `TimestampInt64`
        *   Description: Timestamp from the client when the interaction occurred.
        *   Validation: Must be a positive integer.
    *   `networkValidatedTimestamp`: `TimestampInt64` (optional, from PoW Receipt)
        *   Description: Timestamp validated by EchoNet Witnesses.
        *   Validation: If present, must be a positive integer.
    *   `payload`: `bytes` (serialized specific interaction data, e.g., `CommentPayload`, `ReactionPayload`)
        *   Description: Contains data specific to the `interactionType`.
        *   Validation: Must deserialize correctly based on `interactionType`; internal payload fields must be valid. Max byte size.
    *   `poeScore`: `int32` (optional, assigned by Witnesses)
        *   Description: Proof-of-Engagement score if applicable.
        *   Validation: Non-negative.
    *   `signature`: `SignatureBytes`
        *   Description: Signature of the `eventID` (or its constituent parts) by the `actorDID`.
        *   Validation: Must be a valid signature.
*   **Canonicalization for Hashing/Signing (Conceptual):**
    *   Order: `interactionType`, `actorDID`, `targetContentID` (if present), `targetDID` (if present), `clientAssertedTimestamp`, hash of `payload`. Concatenate with separators.

---

**Data Structure: `CommentPayload`**
*   **Purpose:** Specific payload for an `InteractionEvent` of type "comment".
*   **Fields:**
    *   `parentCommentID`: `HashString` (optional, SHA256 hex of another `InteractionEvent` of type "comment")
        *   Description: If this is a reply, the `eventID` of the parent comment.
        *   Validation: If present, must be a valid SHA256 hex string.
    *   `commentBody`: `string`
        *   Description: The text content of the comment.
        *   Validation: Not empty; max length (e.g., 5000 chars); no forbidden characters/patterns (anti-spam).
    *   `mentions`: `array<DIDString>` (optional)
        *   Description: List of DIDs mentioned in the comment.
        *   Validation: Max array length; elements must be valid DIDs.

---

**Data Structure: `ReactionPayload`**
*   **Purpose:** Specific payload for an `InteractionEvent` of type "reaction".
*   **Fields:**
    *   `reactionType`: `string` (e.g., "like", "love", "insightful", "dislike")
        *   Description: The specific type of reaction.
        *   Validation: Must be from a defined set of reaction types.

---

**Data Structure: `FlagPayload`**
*   **Purpose:** Specific payload for an `InteractionEvent` of type "flag".
*   **Fields:**
    *   `flagReason`: `string` (enum or standardized code)
        *   Description: Reason for flagging the content.
        *   Validation: Must be from a defined set of flag reasons.
    *   `flagNotes`: `string` (optional)
        *   Description: Additional notes from the flagger.
        *   Validation: Max length.

---

**Data Structure: `WitnessAttestation`**
*   **Purpose:** A Witness's signed confirmation of validating a specific event on `EchoNet`. (Ref: `echonet_v3_core_definitions.md #Proof-of-WitnessValidation`)
*   **Fields:**
    *   `eventIDToAttest`: `HashString` (e.g., a `ContentIDString` for new content, or an `InteractionEvent.eventID`)
        *   Description: The unique ID of the event being attested to.
        *   Validation: Must be a valid hash string.
    *   `eventType`: `string` (enum: "ContentObject", "InteractionEvent", "PoSRChallengeResponse", etc.)
        *   Description: Type of event being attested.
        *   Validation: Must be a valid event type.
    *   `witnessDID`: `DIDString`
        *   Description: The DID of the Witness making the attestation.
        *   Validation: Must be a valid DID; must be an eligible Witness.
    *   `witnessAssertedTimestamp`: `TimestampInt64`
        *   Description: Timestamp asserted by the Witness for this attestation. Used to calculate median network timestamp.
        *   Validation: Must be a positive integer.
    *   `isValid`: `boolean`
        *   Description: True if the Witness deems the event valid, false otherwise (for negative attestations if supported, or simply don't attest if invalid).
        *   Validation: Boolean.
    *   `attestationSpecificData`: `bytes` (optional)
        *   Description: Additional data related to the attestation (e.g., PoE score assigned, PoSR proof result).
        *   Validation: Depends on `eventType`.
    *   `signature`: `SignatureBytes`
        *   Description: Signature by `witnessDID` over the canonicalized representation of (`eventIDToAttest`, `eventType`, `witnessAssertedTimestamp`, `isValid`, hash of `attestationSpecificData`).
        *   Validation: Must be a valid signature.
*   **Canonicalization for Signing (Conceptual):**
    *   Order: `eventIDToAttest`, `eventType`, `witnessAssertedTimestamp`, `isValid`, hash of `attestationSpecificData`. Concatenate with separators.

---

**Data Structure: `PoWReceipt` (Proof-of-Witness Receipt)**
*   **Purpose:** Verifiable proof that a specific event was validated by a quorum of the `EchoNet` Witness Committee. (Ref: `echonet_v3_core_definitions.md #ContentHash&Timestamping`, `#Proof-of-WitnessValidation`)
*   **Fields:**
    *   `receiptID`: `HashString` (SHA256 hex of canonicalized receipt data)
        *   Description: Unique ID for this receipt.
    *   `eventIDValidated`: `HashString`
        *   Description: The unique ID of the event that this receipt validates.
        *   Validation: Must be a valid hash string.
    *   `networkValidatedTimestamp`: `TimestampInt64`
        *   Description: The median timestamp agreed upon by the Witness committee for the validated event.
        *   Validation: Must be a positive integer.
    *   `witnessCommitteeID`: `HashString` (optional)
        *   Description: An identifier for the specific Witness committee that performed this validation (e.g., hash of committee member DIDs sorted, or VRF-derived ID).
        *   Validation: If present, must be a valid hash string.
    *   `attestingWitnessDIDs`: `array<DIDString>`
        *   Description: List of DIDs of the M Witnesses whose attestations form this receipt.
        *   Validation: Array length must meet M_of_N threshold; DIDs must be valid and part of the committee.
    *   `aggregatedSignature`: `SignatureBytes` (optional, if using BLS or similar multi-signatures)
        *   Description: A single cryptographic signature aggregating the signatures from the M attesting Witnesses.
        *   Validation: Must be a valid multi-signature verifiable against the public keys of `attestingWitnessDIDs`.
    *   `individualAttestationRefs`: `array<HashString>` (optional, if not using `aggregatedSignature`; stores `eventID`s of `WitnessAttestation` objects)
        *   Description: List of identifiers for the individual `WitnessAttestation` objects.
        *   Validation: Each must be a valid reference to an existing attestation.
    *   `aggregatorDID`: `DIDString` (optional)
        *   Description: DID of the node that compiled this receipt.
        *   Validation: If present, must be a valid DID.
*   **Canonicalization for Hashing (Conceptual):**
    *   Order: `eventIDValidated`, `networkValidatedTimestamp`, `witnessCommitteeID`, sorted `attestingWitnessDIDs`, `aggregatedSignature` (if present) or sorted `individualAttestationRefs` (if present). Concatenate.

---

**Data Structure: `DDSChunkLocation`**
*   **Purpose:** Describes a specific storage node (SSN) that holds a replica or fragment of a data chunk. (Ref: `echonet_v3_core_definitions.md #DistributedDataStoresModule`)
*   **Fields:**
    *   `ssnDID`: `DIDString`
        *   Description: DID of the Standard Storage Node.
        *   Validation: Must be a valid DID of a registered SSN.
    *   `ssnNodeAddress`: `string` (e.g., "ip4/123.45.67.89/tcp/4001/p2p/Qm...")
        *   Description: Network address of the SSN (e.g., libp2p multiaddr format).
        *   Validation: Must be a valid network address format.
    *   `chunkID`: `ContentIDString`
        *   Description: The `ContentID` of the specific chunk stored at this location.
        *   Validation: Must be a valid `ContentIDString`.
    *   `fragmentInfo`: `string` (optional, e.g., "reed_solomon_10_16_frag_3")
        *   Description: Information about erasure coding fragment, if applicable. Index of the fragment.
        *   Validation: If present, must follow defined format.
    *   `lastVerifiedTimestamp`: `TimestampInt64`
        *   Description: Timestamp when this SSN last successfully passed a PoSR challenge for this chunk/fragment.
        *   Validation: Must be a positive integer.
    *   `region`: `string` (optional, e.g., "us-east-1")
        *   Description: Self-reported or network-inferred region of the SSN.
        *   Validation: If present, must be from a list of known regions.
*   **Canonicalization (Not typically signed directly, but part of DHT value):**
    *   Fields usually ordered for consistent storage/transmission in DHT records.

---

**Data Structure: `DHTRecordValue_ContentLocation`**
*   **Purpose:** The value stored in the DHT for a `ContentID` key, pointing to its locations. (Ref: `echonet_v3_core_definitions.md #DiscoveryProtocolModule`)
*   **Fields:**
    *   `locations`: `array<DDSChunkLocation>`
        *   Description: List of locations where chunks/fragments of the content can be found.
        *   Validation: Max array length; each element must be a valid `DDSChunkLocation`.
    *   `publisherDID`: `DIDString`
        *   Description: DID of the node that published/republished this DHT record.
        *   Validation: Must be a valid DID.
    *   `expiryTimestamp`: `TimestampInt64`
        *   Description: When this DHT record should be considered stale if not refreshed.
        *   Validation: Must be a positive integer.
    *   `signature`: `SignatureBytes`
        *   Description: Signature by `publisherDID` over the canonicalized list of locations and expiry.
        *   Validation: Must be a valid signature.

---
**Data Structure: `AdCampaignDefinition`**
*   **Purpose:** Defines parameters for an advertising campaign. (Ref: `echonet_v3_core_definitions.md #DecentralizedAdvertisingMarketplaceModule`)
*   **Fields:**
    *   `campaignID`: `HashString` (SHA256 hex of canonicalized campaign data)
        *   Description: Unique ID for this campaign.
    *   `advertiserDID`: `DIDString`
        *   Description: DID of the advertiser.
        *   Validation: Must be a valid DID.
    *   `adCreativeContentID`: `ContentIDString`
        *   Description: `ContentID` of the actual ad content to be displayed.
        *   Validation: Must be a valid `ContentIDString`.
    *   `budget`: `CurrencyAmountString`
        *   Description: Total budget for the campaign in a specified token.
        *   Validation: Must be a positive numerical string.
    *   `tokenID`: `TokenIDString`
        *   Description: The token used for the budget and bidding.
        *   Validation: Must be a registered/supported token ID.
    *   `bidPriceCPM`: `CurrencyAmountString` (optional, Cost Per Mille/Thousand Impressions)
        *   Description: Max amount willing to pay per 1000 impressions.
        *   Validation: If present, must be a positive numerical string.
    *   `bidPriceCPC`: `CurrencyAmountString` (optional, Cost Per Click)
        *   Description: Max amount willing to pay per click.
        *   Validation: If present, must be a positive numerical string. (At least one bid type required).
    *   `targetingCriteria`: `bytes` (Serialized `AdTargetingCriteria` object)
        *   Description: Specifies audience and content targeting.
        *   Validation: Must deserialize and validate according to `AdTargetingCriteria`.
    *   `startDate`: `TimestampInt64`
        *   Description: Campaign start date/time.
    *   `endDate`: `TimestampInt64` (optional)
        *   Description: Campaign end date/time.
        *   Validation: If present, must be after `startDate`.
    *   `signature`: `SignatureBytes`
        *   Description: Signature by `advertiserDID` over the canonicalized campaign data.
*   **Canonicalization for Hashing/Signing (Conceptual):**
    *   Order fields alphabetically, serialize, concatenate.

---

**Data Structure: `AdTargetingCriteria`**
*   **Purpose:** Nested structure within `AdCampaignDefinition` specifying targeting.
*   **Fields:**
    *   `targetTags`: `array<string>` (optional)
        *   Description: Content tags to target.
    *   `targetCreatorDIDs`: `array<DIDString>` (optional)
        *   Description: Specific creator DIDs whose content/audience to target.
    *   `targetUserReputationMin`: `int32` (optional)
        *   Description: Minimum user reputation score to target.
    *   `targetGeoRegions`: `array<string>` (optional)
        *   Description: Geographic regions to target.

---
**Data Structure: `AdInteractionEvent`**
*   **Purpose:** Records a validated interaction (impression or click) with an ad. (Ref: `echonet_v3_core_definitions.md #DecentralizedAdvertisingMarketplaceModule`)
*   **Fields:**
    *   `adInteractionID`: `HashString` (SHA256 hex of canonicalized data)
    *   `campaignID`: `HashString` (references `AdCampaignDefinition.campaignID`)
        *   Validation: Must be a valid campaign ID.
    *   `adCreativeContentID`: `ContentIDString`
        *   Validation: Must match the one in the campaign.
    *   `consumerDID`: `DIDString`
        *   Description: DID of the user who interacted with the ad.
        *   Validation: Must be a valid DID.
    *   `interactionType`: `string` (enum: "impression", "click")
    *   `contextualContentID`: `ContentIDString` (optional)
        *   Description: `ContentID` of the content where the ad was displayed.
    *   `networkValidatedTimestamp`: `TimestampInt64` (from PoW Receipt of this ad interaction event)
    *   `cost`: `CurrencyAmountString`
        *   Description: Cost incurred for this specific interaction, based on bid.
        *   Validation: Must be positive.
    *   `witnessConfirmation`: `PoWReceipt` (or its `receiptID`)
        *   Description: Proof that this ad interaction was validated by Witnesses.
*   **Canonicalization for Hashing/Signing (Conceptual):**
    *   Order fields, serialize, concatenate for `adInteractionID`. This event is then validated by Witnesses.

---

**Data Structure: `PayoutRecord`**
*   **Purpose:** Represents a record of a payout made to a creator or PoE earner. (Ref: `echonet_v3_core_definitions.md #DirectCreatorPayoutsModule`, `#Proof-of-EngagementRewardsModule`)
*   **Fields:**
    *   `payoutID`: `HashString` (SHA256 hex of canonicalized data)
    *   `recipientDID`: `DIDString`
    *   `amount`: `CurrencyAmountString`
    *   `tokenID`: `TokenIDString`
    *   `sourceType`: `string` (enum: "direct_ad_revenue", "general_ad_pool", "poe_reward_pool", "tip", "subscription")
    *   `sourceReferenceID`: `HashString` (optional, e.g., `ContentID` for ad revenue, `PoERewardBatchID`)
    *   `payoutTimestamp`: `TimestampInt64` (Network time of payout execution)
    *   `transactionBundleID`: `HashString` (ID of the `EchoNet` transaction bundle that executed this payout)
*   **Canonicalization for Hashing (Conceptual):**
    *   Order fields, serialize, concatenate.

---

**Data Structure: `UserProfileCore`**
*   **Purpose:** Minimal, publicly accessible user profile data linked to a DID. (Ref: `echonet_v3_core_definitions.md #UnifiedUserIdentity&ReputationSystem`)
*   **Fields:**
    *   `profileID`: `HashString` (SHA256 hex of `ownerDID` and `profileVersion`)
    *   `ownerDID`: `DIDString`
        *   Validation: Must be a valid DID.
    *   `profileVersion`: `uint32`
        *   Description: Version number, incremented on change.
    *   `displayName`: `string` (optional)
        *   Validation: Max length, character set restrictions.
    *   `avatarContentID`: `ContentIDString` (optional, points to an image ContentObject)
        *   Validation: If present, must be a valid `ContentIDString`.
    *   `bio`: `string` (optional)
        *   Validation: Max length.
    *   `socialLinks`: `map<string, URLString>` (optional, e.g., "twitter" -> "url")
        *   Validation: Keys from defined set, values must be valid URLs.
    *   `lastUpdatedTimestamp`: `TimestampInt64`
    *   `signature`: `SignatureBytes` (signature by `ownerDID` over `profileID` and `lastUpdatedTimestamp`)
*   **Canonicalization for Hashing/Signing (Conceptual):**
    *   Order fields, serialize, concatenate for `profileID`. Signature covers key fields to prove ownership and integrity.

---

**Data Structure: `NodeAdvertisement`**
*   **Purpose:** Record published by nodes advertising their services (SSN, Witness, Indexer, etc.). (Ref: `echonet_v3_core_definitions.md #DiscoveryProtocolModule`)
*   **Fields:**
    *   `nodeDID`: `DIDString`
        *   Description: DID of the node.
    *   `nodeAddress`: `string` (e.g., libp2p multiaddr)
        *   Description: Network contact address.
    *   `serviceTypes`: `array<string>` (enum: "SSN", "ASN", "ECN", "Witness", "Indexer_TopicX", "Bootstrapper")
        *   Description: List of services offered by this node.
        *   Validation: Must be from defined service enums.
    *   `serviceSpecificData`: `bytes` (optional, serialized data relevant to the service, e.g., `SSNProperties`)
    *   `region`: `string` (optional)
    *   `stakeAmount`: `CurrencyAmountString` (optional, if staking is required for the service)
    *   `tokenIDForStake`: `TokenIDString` (optional)
    *   `registrationTimestamp`: `TimestampInt64`
    *   `expiryTimestamp`: `TimestampInt64`
        *   Description: When this advertisement needs to be refreshed.
    *   `signature`: `SignatureBytes` (by `nodeDID` over canonicalized advertisement data)
*   **Canonicalization for Signing (Conceptual):**
    *   Order fields, serialize, concatenate.

---
**Data Structure: `SSNProperties` (Example for `NodeAdvertisement.serviceSpecificData`)**
*   **Purpose:** Specific properties for an SSN.
*   **Fields:**
    *   `totalCapacityBytes`: `uint64`
    *   `availableCapacityBytes`: `uint64`
    *   `supportedErasureSchemes`: `array<string>` (e.g., ["reed_solomon_10_16"])
    *   `retrievalFeePerMB`: `CurrencyAmountString`
    *   `storageFeePerGBMonth`: `CurrencyAmountString` (for direct contracts)
    *   `tokenIDForFees`: `TokenIDString`

---

This is a foundational set. More structures (e.g., for DLI-native contracts, governance proposals, detailed PoE scoring events) would be needed as implementation details are further fleshed out. The key is to maintain consistency in typing, naming, validation, and canonicalization.The initial set of core data structures has been defined in the `echonet_v3_core_data_structures.md` file. This includes structures like `ContentObject`, `InteractionEvent`, `WitnessAttestation`, `PoWReceipt`, `DDSChunkLocation`, `AdCampaignDefinition`, `PayoutRecord`, `UserProfileCore`, and `NodeAdvertisement`, along with their fields, types, descriptions, conceptual validation rules, and notes on canonicalization.

This fulfills the main objective of the current subtask by providing a detailed specification for these key data structures, aligned with the previously defined core purposes of the modules.

The next logical step is to submit a report for this subtask.
