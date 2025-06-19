**EchoNet Communication Protocols (v3.0)**

This document defines standardized communication protocols for direct inter-module and inter-node interactions within `EchoNet`. These protocols primarily focus on Remote Procedure Call (RPC) mechanisms for synchronous or asynchronous request/response operations, distinct from the broader, event-driven notifications handled by the conceptual `EchoNet` Event Bus. This aligns with the Expanded KISS Principle of "Systematize for Scalability, Synchronize for Synergy."

**1. Communication Protocol Philosophy**

*   **Clarity & Explicitness:** Protocol messages (requests and responses) and service method signatures must be clearly defined, unambiguous, and use standardized data types.
*   **Efficiency:** Protocols should favor efficient serialization formats and minimize unnecessary data transmission, especially for high-frequency operations.
*   **Strong Typing & Validation:** Leverage strongly-typed data structures (from `echonet_v3_core_data_structures.md`) for message payloads to reduce errors and improve developer experience. Rigorous validation of incoming requests against these schemas is essential.
*   **Versioning:** All RPC services, methods, and message structures must be versioned according to `echonet_v3_versioning_strategy.md` to allow for evolution and manage compatibility.
*   **Consistent Error Handling:** RPC responses must use the standardized error reporting mechanisms defined in `echonet_v3_error_handling_strategy.md`.
*   **Security by Design:** Authentication, authorization, and transport layer security are integral to the protocol design.
*   **Interoperability:** While a primary implementation language might be chosen for core nodes, the RPC framework should ideally support language neutrality to allow diverse client implementations or specialized nodes in other languages.

**2. Choice of RPC Framework/Style (Conceptual)**

*   **Primary Conceptual Choice: gRPC with Protocol Buffers (Protobuf)**
    *   **Rationale:**
        *   **Efficiency:** Protocol Buffers provide a compact binary serialization format, efficient for network transmission and storage.
        *   **Strong Typing & Schema Definition:** `.proto` files offer a language-neutral way to define service interfaces and message structures, enforcing data contracts.
        *   **Language Neutrality:** gRPC supports code generation for many popular programming languages (Go, Rust, Python, Java, C++, JavaScript, etc.), facilitating diverse implementations of nodes or clients.
        *   **Performance:** gRPC is built on HTTP/2, offering benefits like multiplexing and streaming.
        *   **Streaming Support:** Useful for scenarios like large data transfers from DDS or continuous event streams (though the latter might also use the Event Bus).
        *   **Ecosystem & Tooling:** Well-established with good community support and tooling.
        *   **Precedent:** Used in many large-scale distributed systems and some DLT/P2P projects.
*   **Alternatives Considered (and why gRPC/Protobuf is preferred for this conceptual stage):**
    *   **JSON-RPC 2.0:** Simpler, human-readable (uses JSON), but less efficient for network traffic and lacks the strong typing and schema enforcement of Protobuf out-of-the-box.
    *   **Custom Binary Protocols (e.g., over libp2p streams):** Can be highly optimized but require more manual effort for definition, parsing, and ensuring cross-language compatibility. gRPC provides a framework on top of such lower-level capabilities. `EchoNet`'s core P2P discovery (DHT, gossip) might use more custom libp2p stream protocols, but application-layer RPCs benefit from gRPC's structure.

**3. Standard Request/Response Structure (Conceptual Envelope for gRPC Messages)**

While gRPC and Protobuf define message structures per RPC call, a common set of metadata fields should be included in requests and responses, potentially via a common header structure or by convention.

*   **Conceptual Common Request Header (`CommonRequestHeader`):**
    *   `requestID`: `string` (UUID, for tracing and correlating async responses)
    *   `sourceNodeID`: `DIDString` (Authenticated DID of the calling node/user)
    *   `protocolVersion`: `SemanticVersionString` (Version of the `EchoNet-ProtocolVersion` the caller is using)
    *   `apiVersion`: `SemanticVersionString` (Version of the specific RPC service/method being called)
    *   `timestamp`: `TimestampInt64` (Client-asserted timestamp of request generation)
    *   `signature`: `SignatureBytes` (Signature of the request payload + key header fields by `sourceNodeID`)
*   **Conceptual Common Response Header (`CommonResponseHeader`):**
    *   `correspondingRequestID`: `string` (Copied from the request)
    *   `serverNodeID`: `DIDString` (DID of the responding node)
    *   `timestamp`: `TimestampInt64` (Timestamp of response generation)
    *   `protocolVersion`: `SemanticVersionString` (Protocol version of the responder)
    *   `apiVersion`: `SemanticVersionString` (API version of the responder for this service)

*   **RPC Response Structure (General):**
    ```protobuf
    message GenericRpcResponse {
      CommonResponseHeader header = 1;
      // One of the following will be set:
      oneof result {
        bytes success_payload = 2; // Serialized specific success response message
        RpcError error_details = 3;  // Standardized error structure
      }
    }

    message RpcError {
      string error_code = 1;       // From echonet_v3_error_handling_strategy.md (e.g., "DDS-0001")
      string error_message = 2;    // Human-readable message template
      string severity = 3;         // "Warning", "RecoverableError", "FatalUserError", "FatalSystemError"
      map<string, string> contextual_data = 4; // Key-value string pairs for context
    }
    ```

**4. Definition of Key RPC Services & Methods**

These definitions translate methods from `echonet_v3_component_interfaces.md` into conceptual RPC service definitions using Protobuf/gRPC style. Specific request/response message types would be defined using data structures from `echonet_v3_core_data_structures.md`.

**Service: `DDSNodeRPCService`**
*   (Implements parts of `ChunkStorageService` interface for direct node-to-node interaction)
*   **Methods:**
    *   `rpc StoreChunk(StoreChunkRequest) returns (StoreChunkResponse)`
        *   `StoreChunkRequest`: `{ commonHeader: CommonRequestHeader, chunkID: ContentIDString, chunkData: bytes, storageTierHint: string }`
        *   `StoreChunkResponse`: `{ commonHeader: CommonResponseHeader, success: bool, error: RpcError }`
    *   `rpc RetrieveChunk(RetrieveChunkRequest) returns (RetrieveChunkResponse)`
        *   `RetrieveChunkRequest`: `{ commonHeader: CommonRequestHeader, chunkID: ContentIDString }`
        *   `RetrieveChunkResponse`: `{ commonHeader: CommonResponseHeader, chunkData: bytes, error: RpcError }`
    *   `rpc GeneratePoSRProof(GeneratePoSRProofRequest) returns (GeneratePoSRProofResponse)`
        *   `GeneratePoSRProofRequest`: `{ commonHeader: CommonRequestHeader, challenge: PoSRChallenge }` (PoSRChallenge structure TBD)
        *   `GeneratePoSRProofResponse`: `{ commonHeader: CommonResponseHeader, proof: PoSRProof, error: RpcError }` (PoSRProof structure TBD)
    *   `rpc GetNodeAdvertisement(GetNodeAdvertisementRequest) returns (GetNodeAdvertisementResponse)`
        *   `GetNodeAdvertisementRequest`: `{ commonHeader: CommonRequestHeader }`
        *   `GetNodeAdvertisementResponse`: `{ commonHeader: CommonResponseHeader, advertisement: NodeAdvertisement, error: RpcError }`

**Service: `EventSubmissionRPCService`**
*   (Implements `ContentSubmissionService` and `InteractionHandlerService` for submitting events to the network, likely to a Witness or gateway node)
*   **Methods:**
    *   `rpc SubmitValidatedEvent(SubmitValidatedEventRequest) returns (SubmitValidatedEventResponse)`
        *   Description: For submitting events that have already been packaged and signed by the user/client, now needing network validation.
        *   `SubmitValidatedEventRequest`: `{ commonHeader: CommonRequestHeader, eventID: HashString, eventType: string, eventPayload: bytes, clientAssertedTimestamp: TimestampInt64 }` (Matches parameters of `NetworkEventValidator.SubmitEventForValidation`)
        *   `SubmitValidatedEventResponse`: `{ commonHeader: CommonResponseHeader, submissionTrackingID: string, acceptedStatus: bool, error: RpcError }` (AcceptedStatus indicates if syntax is okay and passed to validation queue)

**Service: `DiscoveryRPCService` (Kademlia-like RPCs for DHT)**
*   (Implements parts of `ResourceDiscoveryProvider` for direct DHT node interactions)
*   **Methods:**
    *   `rpc Ping(PingRequest) returns (PongResponse)`
        *   `PingRequest`: `{ commonHeader: CommonRequestHeader }`
        *   `PongResponse`: `{ commonHeader: CommonResponseHeader }`
    *   `rpc FindNode(FindNodeRequest) returns (FindNodeResponse)`
        *   `FindNodeRequest`: `{ commonHeader: CommonRequestHeader, targetNodeID: DIDString }`
        *   `FindNodeResponse`: `{ commonHeader: CommonResponseHeader, closestNodes: array<NodeAdvertisement>, error: RpcError }`
    *   `rpc StoreValue(StoreValueRequest) returns (StoreValueResponse)`
        *   `StoreValueRequest`: `{ commonHeader: CommonRequestHeader, key: HashString, value: DHTRecordValue_ContentLocation, signature: SignatureBytes }` (Value includes its own publisher signature)
        *   `StoreValueResponse`: `{ commonHeader: CommonResponseHeader, success: bool, error: RpcError }`
    *   `rpc FindValue(FindValueRequest) returns (FindValueResponse)`
        *   `FindValueRequest`: `{ commonHeader: CommonRequestHeader, key: HashString }`
        *   `FindValueResponse`: `{ commonHeader: CommonResponseHeader, value: DHTRecordValue_ContentLocation, closerNodes: array<NodeAdvertisement>, error: RpcError }` (Returns value if found, or closer nodes)

**Service: `AdMarketplaceRPCService` (Conceptual)**
*   **Methods:**
    *   `rpc SubmitAdCampaign(SubmitAdCampaignRequest) returns (SubmitAdCampaignResponse)`
        *   `SubmitAdCampaignRequest`: `{ commonHeader: CommonRequestHeader, campaignDefinition: AdCampaignDefinition }`
        *   `SubmitAdCampaignResponse`: `{ commonHeader: CommonResponseHeader, campaignID: HashString, error: RpcError }`
    *   `rpc RequestAdsForSlot(RequestAdsForSlotRequest) returns (RequestAdsForSlotResponse)`
        *   `RequestAdsForSlotRequest`: `{ commonHeader: CommonRequestHeader, slotContext: AdSlotContext }` (AdSlotContext TBD, includes tags, ContentID of page, etc.)
        *   `RequestAdsForSlotResponse`: `{ commonHeader: CommonResponseHeader, eligibleAds: array<AdCreativePointer>, error: RpcError }` (AdCreativePointer TBD, contains AdCreativeContentID and bid info)

**Service: `ReputationQueryService`**
*   (Implements `ReputationReader`)
*   **Methods:**
    *   `rpc GetUserReputation(GetUserReputationRequest) returns (GetUserReputationResponse)`
        *   `GetUserReputationRequest`: `{ commonHeader: CommonRequestHeader, userDID: DIDString, reputationType: string }`
        *   `GetUserReputationResponse`: `{ commonHeader: CommonResponseHeader, score: int64, error: RpcError }`

**5. Message Serialization Format**

*   **Protocol Buffers (Protobuf):** As identified in holistic optimizations and fitting with gRPC, Protobuf is the chosen serialization format for all RPC message payloads.
    *   **Benefits:** Efficient binary format, strong typing via `.proto` definitions, code generation, good for performance-critical P2P communication.
    *   All data structures in `echonet_v3_core_data_structures.md` would be defined as Protobuf messages.

**6. Error Handling in RPCs**

*   RPC responses will use the standardized `RpcError` structure (defined above).
*   The `error_code` field will contain a specific code from `echonet_v3_error_handling_strategy.md` (e.g., `DDS-0001`, `VAL-0002`).
*   `error_message` provides a template message, and `contextual_data` offers specific details related to that instance of the error.
*   This allows clients to programmatically handle errors based on codes and severity while having human-readable information for logging.

**7. Security Considerations for RPCs**

*   **Authentication:**
    *   All inter-node RPC calls must be authenticated. The `CommonRequestHeader.sourceNodeID` and `signature` (signing the request content) achieve this. The receiving node verifies the signature using the public key associated with the `sourceNodeID` (retrieved via Discovery Service if needed).
*   **Authorization:**
    *   After authentication, the receiving node must authorize the requested operation based on the `sourceNodeID`'s role, reputation, or specific permissions related to the target resource.
    *   Example: Only a Witness node can issue a `GeneratePoSRProof` request to an SSN for a chunk it's supposed to be tracking. An SSN can only `StoreValue` in the DHT for chunks it actually hosts.
*   **Encryption (Transport Layer Security - TLS):**
    *   All RPC communication channels between nodes should be encrypted using mutually authenticated TLS (mTLS) or an equivalent secure channel protocol provided by the underlying P2P networking layer (e.g., libp2p's SECIO, TLS, Noise). This protects against eavesdropping and man-in-the-middle attacks. Node DIDs can be linked to their TLS certificates.
*   **Input Validation:** Rigorous validation of all incoming RPC request parameters against defined schemas (`echonet_v3_core_data_structures.md`) and business rules is critical to prevent exploits.
*   **Rate Limiting & Resource Management:** Nodes should implement rate limiting on incoming RPC requests to prevent DoS/spam attacks and manage resource consumption.

**8. Versioning of RPC Services & Methods**

*   RPC services and their methods will be versioned according to the strategy in `echonet_v3_versioning_strategy.md`.
*   **API Endpoint Versioning:** The `CommonRequestHeader.apiVersion` field specifies the version of the service/method the client intends to call. Nodes can support multiple API versions simultaneously during a transition period.
*   **Service Name Versioning (Alternative):** For gRPC, services can be versioned by including a version number in their package name (e.g., `service DDSNodeRPCServiceV1`, `service DDSNodeRPCServiceV2`).
*   **Breaking Changes:** Incompatible changes to an RPC method (parameter changes, return type changes, significant behavioral modifications) require a MAJOR version increment for that API/service.
*   **Backward Compatibility:** Adding new optional parameters to a request or new optional fields to a response is generally a MINOR, backward-compatible change.

This communication protocol strategy provides a robust foundation for direct inter-node and inter-module interactions within `EchoNet`, promoting clarity, efficiency, security, and manageable evolution.
