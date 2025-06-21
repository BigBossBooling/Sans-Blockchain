# EchoNet DDS Node RPC API Specification

**Document Version:** 1.0
**Date:** 2023-10-29 (Placeholder for actual generation date)

## 1. Introduction

This document outlines the Remote Procedure Call (RPC) service and methods that govern interactions with Distributed Data Store (DDS) Nodes within the EchoNet protocol. These RPCs enable clients and other network participants to store, retrieve, and manage data chunks and fragments across the decentralized storage layer.

## 2. Core Service Definition Location

The primary Protocol Buffer (protobuf3) definitions for the `DDSNodeService` and all its associated request and response messages are formally specified in the following file:

*   **`proto/dds_specific.proto`**

This file contains the precise message structures for:
*   `StoreChunkRequest`, `StoreChunkResponse`
*   `RetrieveChunkRequest`, `RetrieveChunkResponse`
*   `AnnounceAvailabilityRequest`, `AnnounceAvailabilityResponse`
*   `RequestChunkForReplicationRequest`, `RequestChunkForReplicationResponse`
*   `RequestErasureFragmentRequest`, `RequestErasureFragmentResponse`

It also includes the service definition for `DDSNodeService`, which utilizes `echonet_core.v3.StorageChallengeRequest` and `echonet_core.v3.StorageChallengeResponse` for the `HandlePoSRChallenge` method.

For the detailed protobuf definitions, please refer directly to: [`proto/dds_specific.proto`](../proto/dds_specific.proto). *(Link is conceptual and depends on final document structure; the filename is the key reference.)*

## 3. Detailed Descriptions and Purpose

Comprehensive explanations of each RPC method available through the `DDSNodeService`, including their specific purpose, expected behavior, detailed breakdown of request/response parameters, and any method-specific error handling considerations, are documented within the DDS Protocol Structures document:

*   **[`tech_specs/dds_protocol_structures.md`](./dds_protocol_structures.md)**

This document should be consulted for a full understanding of how to interact with the DDS Node RPC API.

## 4. Reference to General Communication Protocols

All DDS Node RPCs defined and referenced herein are expected to adhere to the overall communication patterns, security considerations (e.g., secure channels, message signing where appropriate), and error handling strategies established for the EchoNet DLI ecosystem.

For these general guidelines, please refer to:
*   `echonet_v3_communication_protocols.md` (Conceptual, assuming this document will define underlying transport, e.g., gRPC over libp2p streams, and security).
*   `echonet_v3_error_handling_strategy.md` (For understanding common error types and reporting).

This `dds_rpc_api.md` document serves as a central reference point for locating the DDS RPC specifications, which are detailed primarily within `proto/dds_specific.proto` for the definitions and `tech_specs/dds_protocol_structures.md` for their explanations.
