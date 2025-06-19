**EchoNet Logging & Metrics Strategy (v3.0)**

This document defines a strategy for transparent logging and the collection of key metrics for `EchoNet` core DLI operations. This data is primarily intended for system health monitoring, debugging, performance analysis, security incident response, understanding DLI dynamics, and providing quantitative insights for the `EchoNet` governance community. This aligns with the Expanded KISS Principle: "Stimulate Engagement, Sustain Impact" through "Transparent Logging & Metrics" and "Output Analytics."

**1. Logging & Metrics Philosophy for `EchoNet`**

*   **Observability as a Core Tenet:** `EchoNet` must be observable to its operators, developers, and (in aggregate/anonymized form) its community, to build trust and enable effective management.
*   **Actionable Insights:** Logs and metrics should provide actionable information, not just raw data. They should help answer questions about system behavior, performance, and health.
*   **Structured for Machines, Readable by Humans:** Logs should be structured for easy machine parsing and analysis, while also being human-readable for debugging. Metrics should be well-defined and consistently named.
*   **Privacy by Design:** Avoid logging sensitive user data (e.g., PII, specific content of private interactions beyond what's necessary for an event ID) in general system or DLI operational logs. DIDs are pseudonymous; care must be taken not to create log/metric patterns that inadvertently deanonymize users. Aggregate and anonymize metrics exposed publicly.
*   **Efficiency & Performance Impact:** Logging and metrics collection, while crucial, should be implemented efficiently to minimize performance overhead on nodes and the network. Employ sampling for very high-frequency events if necessary.
*   **Standardization:** Use common formats and conventions across all `EchoNet` components and node types to simplify aggregation and analysis.

**2. Structured Logging**

*   **A. Log Format:**
    *   **Recommendation:** JSON.
    *   **Rationale:** Widely supported by log management tools, easily machine-parsable, flexible for adding custom fields, and still human-readable. Logfmt is a simpler alternative if JSON is deemed too verbose for some high-frequency logs.
*   **B. Common Log Fields (for every log entry):**
    *   `timestamp`: `TimestampInt64` (ISO 8601 format in UTC, e.g., "2023-10-27T10:30:05.123Z")
    *   `nodeID`: `DIDString` (DID of the node generating the log)
    *   `nodeType`: `string` (enum: "Witness", "SSN", "ASN", "ECN", "Indexer", "ClientShim", "GeneralPurpose")
    *   `module`: `string` (Name of the core module, e.g., "DDS.Storage", "PoW.Validation", "Discovery.DHT", "Monetization.AdMarket")
    *   `subComponent`: `string` (Optional, specific sub-component or function name, e.g., "StoreChunk", "ValidateEvent")
    *   `severity`: `string` (enum: `DEBUG`, `INFO`, `WARN`, `ERROR`, `CRITICAL`)
    *   `requestID` or `traceID`: `string` (UUID, for correlating a sequence of log entries related to a single operation or event flow across modules/nodes)
    *   `message`: `string` (The main log message, human-readable)
    *   `errorCode`: `string` (Optional, if the log pertains to a defined error from `echonet_v3_error_handling_strategy.md`, e.g., "DDS-0001")
    *   `data`: `object` (Optional, key-value pairs for structured contextual data relevant to the log entry, e.g., `{"chunkID": "...", "peerDID": "...", "latency_ms": 120}`)
*   **C. Key Operations to Log (with specific contextual fields in `data`):**
    *   **User/Client Onboarding:**
        *   DID creation events (client-side, if observable by any helper service).
        *   Wallet setup events (client-side).
    *   **Content Creation & Management:**
        *   `ContentObject` submission attempt: `creatorDID`, `contentType`, `clientAssertedTimestamp`.
        *   `ContentObject` validation by Witness: `contentID`, `validatorDID`, `validationResult (success/fail)`, `validationLatency_ms`.
        *   `ContentObject` successfully persisted to DDS (from Replication & Redundancy): `contentID`, `numReplicasOrShards`.
    *   **DDS Operations:**
        *   `StoreChunk` call: `chunkID`, `targetSSNDID`, `success/failure`, `latency_ms`, `error_if_any`.
        *   `RetrieveChunk` call: `chunkID`, `requestorDID_or_NodeID`, `servingSSNDID`, `success/failure`, `latency_ms`, `bytes_transferred`.
        *   PoSR challenge issued: `challengerWitnessDID`, `targetSSNDID`, `challengedChunkID`.
        *   PoSR response received/validated: `responderSSNDID`, `chunkID`, `isValid`, `proofVerificationLatency_ms`.
        *   Self-healing event: `triggerReason (node_offline/proof_failed)`, `chunkID_repaired`, `newSSNDIDs_involved`.
    *   **Proof-of-Witness Validation:**
        *   Event received for validation: `eventID`, `eventType`, `sourceDID_or_NodeID`.
        *   Attestation created/sent: `eventID`, `attestingWitnessDID`, `isValid`.
        *   Attestation received: `eventID`, `sourceWitnessDID`, `isValid`.
        *   `PoWReceipt` generated/received: `receiptID`, `eventIDValidated`, `networkValidatedTimestamp`, `committeeSize`, `numAttestations`.
        *   Witness selection event: `committeeID` (if any), `selectedWitnessDIDs`.
    *   **Discovery Protocol (DHT/Gossip):**
        *   DHT query received/sent: `queryType (FIND_NODE/FIND_VALUE)`, `targetID`, `peerDID`.
        *   DHT store operation: `key`, `storingNodeDID`, `success/failure`.
        *   Routing table update: `addedPeerDID`, `removedPeerDID`, `bucket_index`.
        *   Gossip message received/sent: `messageType`, `peerDID`, `messageSizeBytes`.
    *   **Engagement & Interaction:**
        *   `InteractionEvent` submission: `actorDID`, `interactionType`, `targetContentID/targetDID`.
        *   `InteractionEvent` validation by Witness: `eventID`, `validatorDID`, `validationResult`, `poeScoreAssigned` (if applicable).
    *   **Monetization (Direct Payouts, Ad Marketplace, PoE Rewards):**
        *   DLI-Native "Contract" execution start/end: `contractName`, `functionName`, `inputParameters (anonymized/summarized if sensitive)`, `outcome (success/fail)`, `execution_ms`.
        *   Ad campaign creation: `campaignID`, `advertiserDID`, `budgetAmount`.
        *   Ad interaction (impression/click) validation: `adInteractionID`, `campaignID`, `consumerDID`, `validationResult`.
        *   Payout distribution event: `payoutPoolID`, `totalAmountDistributed`, `numRecipients`.
    *   **Reputation System:**
        *   Reputation score update event: `targetDID`, `changeAmount`, `reasonCode`, `newReputationScore`.
    *   **P2P Networking Layer:**
        *   Peer connection established/dropped: `peerDID`, `remoteAddress`.
        *   Significant network errors: `peerDID`, `errorDetails`.
    *   **Security Events:**
        *   Failed signature validations: `signerDID_attempted`, `dataTypeSigned`.
        *   Input validation failures (high severity): `sourceDID_or_IP`, `endpoint_or_function`, `validationErrorCode`.
        *   Suspected spam/DoS activity detected: `source_identifier`, `activity_pattern`.
        *   Critical errors/panics: Full error message and stack trace (if applicable).
*   **D. Log Levels & Configurability:**
    *   Nodes MUST allow configuration of log verbosity per module (e.g., `DEBUG` for Discovery.DHT during troubleshooting, `INFO` for PoW.Validation generally, `WARN` for most).
    *   Default log level should be `INFO` for production environments.
*   **E. Log Aggregation & Analysis (Conceptual):**
    *   While individual nodes generate logs, `EchoNet` itself does not mandate a specific centralized log aggregation system.
    *   Node operators are responsible for their own log management.
    *   However, the `EchoNet` community or supporting organizations might promote standards or provide tools for voluntary log sharing/aggregation to monitor overall network health (e.g., using ELK Stack, Grafana Loki, or emerging decentralized alternatives). This would require careful consideration of privacy and data sanitization.

**3. Key System Metrics (Quantitative Data)**

*   **A. Categories of Metrics:**
    *   **Network Health & Participation:**
        *   `echonet_nodes_active_total` (gauge, labels: `nodeType`): Total number of active nodes by type.
        *   `echonet_dht_size_estimated` (gauge): Estimated number of nodes participating in the DHT.
        *   `echonet_gossip_message_propagation_latency_seconds` (histogram): Latency for sample gossip messages to propagate.
        *   `echonet_protocol_versions_in_use` (gauge, labels: `protocolVersion`, `nodeType`): Count of nodes by protocol version.
    *   **DDS Performance & Health:**
        *   `echonet_dds_total_stored_bytes` (gauge, labels: `ssnDID` (optional), `storageTier`): Total data stored.
        *   `echonet_dds_chunk_retrieval_latency_seconds` (histogram, labels: `servingSSNDID` (optional)): Latency for chunk retrievals.
        *   `echonet_dds_chunk_storage_operations_total` (counter, labels: `ssnDID` (optional), `status (success/fail)`): Count of chunk storage attempts.
        *   `echonet_dds_posr_challenges_total` (counter, labels: `ssnDID`, `status (success/fail)`): PoSR challenge outcomes.
        *   `echonet_dds_data_repair_bytes_total` (counter): Bytes repaired by self-healing.
        *   `echonet_dds_active_ssns_total` (gauge): Number of SSNs currently passing PoSR.
    *   **Consensus (Proof-of-Witness) Performance:**
        *   `echonet_pow_event_validation_throughput_total` (counter, labels: `eventType`, `committeeID` (optional)): Number of events validated.
        *   `echonet_pow_validation_latency_seconds` (histogram, labels: `eventType`): Time taken from event submission to PoWReceipt generation.
        *   `echonet_pow_committee_participation_rate` (gauge, labels: `committeeID`): Percentage of selected Witnesses participating.
        *   `echonet_pow_slashing_events_total` (counter, labels: `witnessDID`, `reason`): Number of Witness slashing events.
        *   `echonet_pow_attestations_per_event_avg` (gauge): Average attestations collected for validated events.
    *   **Monetization Activity (Aggregated/Anonymized where appropriate):**
        *   `echonet_monetization_payouts_value_total` (counter, labels: `tokenID`, `sourceType`): Total value of payouts processed.
        *   `echonet_monetization_active_ad_campaigns_total` (gauge): Number of currently active ad campaigns.
        *   `echonet_monetization_ad_spend_total` (counter, labels: `tokenID`): Total value transacted in ad marketplace.
        *   `echonet_monetization_ad_impressions_validated_total` (counter).
        *   `echonet_monetization_ad_clicks_validated_total` (counter).
    *   **Proof-of-Engagement (PoE) Activity:**
        *   `echonet_poe_qualified_interactions_total` (counter, labels: `interactionType`).
        *   `echonet_poe_rewards_distributed_value_total` (counter, labels: `tokenID`).
        *   `echonet_poe_score_avg` (gauge, labels: `interactionType`): Average PoE score for qualified interactions.
    *   **Node Resource Utilization (Per Node - for node operators' own monitoring):**
        *   `echonet_node_cpu_usage_percent` (gauge)
        *   `echonet_node_memory_usage_bytes` (gauge)
        *   `echonet_node_disk_io_bytes_total` (counter, labels: `operation (read/write)`)
        *   `echonet_node_network_io_bytes_total` (counter, labels: `direction (sent/received)`)
        *   `echonet_node_active_p2p_connections` (gauge)
*   **B. Metrics Collection & Exposure:**
    *   **Internal Collection:** Nodes should collect these metrics internally using appropriate libraries (e.g., Prometheus client libraries).
    *   **Standard Endpoint:** Each node type should expose a Prometheus-compatible `/metrics` HTTP endpoint for authorized scraping. Access to this endpoint should be configurable and potentially restricted (e.g., localhost by default, or specific IPs).
    *   **Privacy for Public Metrics:** Any metrics intended for public dashboards (e.g., overall network health) must be aggregated and anonymized by design. Individual node metrics are primarily for the node operator or authorized monitoring systems. DIDs should not be used as labels in publicly exposed aggregate metrics unless the DID represents a public service entity.
*   **C. Metrics Visualization & Alerting (Conceptual):**
    *   Node operators and the community would use tools like Prometheus for scraping, Grafana for visualization, and Alertmanager for setting up alerts on abnormal metric conditions (e.g., low Witness participation, high error rates, low PoSR success).

**4. "Output Analytics" - Measuring Real-World Impact (Conceptual)**

These analytics are derived from the core DLI state and metrics but focus on understanding `EchoNet`'s success in achieving its broader mission. These are typically generated by dedicated community or foundation-run analytics services querying the DLI (respecting privacy).

*   **Creator Economy:**
    *   Distribution of creator earnings (e.g., Gini coefficient for payouts).
    *   Median/average earnings per active creator.
    *   Growth in number of creators earning above certain thresholds.
    *   Proportion of revenue from ads vs. tips vs. subscriptions.
*   **Content Ecosystem:**
    *   Content diversity (e.g., based on tags, content types).
    *   Growth rate of new, validated content.
    *   Ratio of high-reputation content vs. low-reputation or flagged content.
*   **Engagement Quality:**
    *   Trends in average PoE scores for interactions.
    *   Ratio of high-quality comments to total comments.
    *   Effectiveness of anti-spam and anti-gaming measures (e.g., reduction in flagged content over time).
*   **Decentralization & Network Health:**
    *   Distribution of Witness power/stake.
    *   Geographic distribution of SSNs and Witnesses.
    *   Growth in the number of independent node operators.

**5. Security & Privacy for Logs/Metrics**

*   **No Sensitive PII in Logs:** System-level logs should avoid direct PII. User content (e.g., comment text, article bodies) should NOT be logged by default in general operational logs of unrelated nodes (like Witnesses or DHT nodes). Client-side logs are the user's responsibility.
*   **DID is Pseudonymous:** DIDs are the primary identifiers in logs/metrics. While pseudonymous, large-scale correlation of DID activity across many public sources could be a deanonymization risk.
*   **Anonymization/Aggregation for Public Data:** Any metrics or analytics published for general public consumption must be aggregated and anonymized to a level that prevents re-identification of individual user behavior patterns.
*   **Access Control to Metrics Endpoints:** The `/metrics` endpoint on nodes should be protected and not publicly exposed by default. Node operators can configure access.
*   **Log Retention Policies:** Node operators should define their own log retention policies based on their needs and any applicable regulations. `EchoNet` protocol itself doesn't mandate specific log retention on individual nodes beyond what might be needed for short-term dispute resolution if applicable.
*   **Secure Log Transport (if aggregated):** If logs are shipped to a central (or federated) analysis platform, secure transport (TLS) must be used.

**6. Versioning of Log/Metric Formats**

*   **Log Fields:** Adding new common log fields is generally a non-breaking change. Changing the meaning or removing existing common fields in a widely used log analysis setup is a breaking change and should be managed with care (e.g., versioning the log schema if it becomes complex).
*   **Metric Names & Labels:** Changes to metric names or labels (especially removing or renaming) are breaking changes for dashboards and alerting rules. These should be versioned or managed with clear deprecation policies, similar to API versioning (`echonet_v3_versioning_strategy.md`). Consider a `echonet_metrics_version` info metric.

This logging and metrics strategy aims to provide robust observability into `EchoNet` for its diverse stakeholders, crucial for its operation, maintenance, security, and governed evolution.
