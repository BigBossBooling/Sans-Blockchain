# DigiSocialBlock - Mobile Node Specifications

**Document Version:** 1.0
**Date:** 2023-10-28 (Placeholder - will be updated by actual generation date)

## 1. Mobile Node Philosophy in `EchoNet`

### 1.1. Goals
The integration of mobile devices as active participants in the EchoNet DLI (also referred to as DigiSocialBlock) is driven by several core goals:
*   **Maximize Decentralization:** Enable a vast number of users to contribute to network functions, reducing reliance on centralized infrastructure or always-on desktop nodes.
*   **Enable Mass Participation:** Lower the barrier to entry for interacting with and contributing to EchoNet, leveraging the ubiquity of smartphones.
*   **User Empowerment:** Allow users to control their data, participate in content distribution, and potentially earn rewards directly on their primary devices.

### 1.2. Balancing Participation with Constraints
EchoNet recognizes the inherent resource constraints of mobile devices, including:
*   **Battery Life:** Limited and precious.
*   **Data Caps:** Mobile data can be expensive or limited.
*   **Storage:** Finite and often shared with many other apps.
*   **Processing Power:** Less than desktop or server hardware.
*   **Intermittent Connectivity:** Frequent network changes (Wi-Fi to cellular, loss of signal).
*   **Background Restrictions:** Mobile OSes (iOS, Android) heavily restrict background processing to save battery.

Therefore, mobile participation is designed to be **opportunistic and resource-aware**, with different roles catering to varying device capabilities and user preferences.

### 1.3. "Always-On" Not Assumed
Unlike traditional server nodes, mobile nodes are not expected to be "always-on." Their participation is often event-driven (e.g., when the app is in the foreground) or periodic (e.g., limited background syncs).

## 2. Defined Mobile Node Roles & Capabilities (Technical Details)

Mobile nodes can operate in several distinct roles, often dynamically based on user settings, device capabilities, and current context (e.g., battery level, network type).

### 2.A. Host (Standard Mobile User/Creator/Consumer)

This is the default role for most mobile users of an EchoNet client application.

*   **P2P Communication:**
    *   **Primary Mode:** Acts as a client to more stable "Super-Host" nodes (see below) or dedicated desktop/server nodes for reliable interaction with the DLI (submitting events, fetching data).
    *   **Protocols:** Utilizes libp2p (see Section 3) with a focus on transports suitable for mobile, including WebSockets Secure (WSS) for client-server like connections, and potentially QUIC.
    *   **NAT Traversal:** Relies on libp2p's NAT traversal mechanisms (e.g., Hole Punching via relays, AutoRelay) when attempting direct P2P connections.
    *   **Opportunistic Direct P2P:** May engage in direct P2P with other Hosts if network conditions are favorable (e.g., both on the same Wi-Fi) and discovery is successful.
*   **DDS Integration (as per `dds_refined_architecture.md`):**
    *   **Origin Node:** When a user creates content, their mobile device acts as the initial Origin Node. It holds the content chunks temporarily and uses `StoreChunk` (from `tech_specs/dds_protocol.md`) to upload them to SSNs for persistent storage and replication. The mobile Host is not expected to guarantee long-term availability of these originated chunks itself.
    *   **Client-Side Caching:** Maintains a local cache of frequently accessed content chunks or content from followed creators/topics. This cache is managed according to storage limits (see Section 5).
    *   **Not a Full Storage Node:** Hosts are *not* expected to operate as SSNs or ASNs. They do not generally offer their storage for arbitrary network chunks beyond their own originated content or limited caching.
*   **Resource Management:**
    *   **Background Activity:** Minimal background activity, primarily for receiving critical notifications or very limited data sync if user-enabled. Adheres strictly to OS background execution limits.
    *   **Data Usage:** User-configurable settings for mobile data usage (allow/disallow, caps). Prioritizes Wi-Fi. Provides warnings for high data consumption.
    *   **Battery Awareness:** App monitors battery level and significantly reduces or pauses P2P activity, background sync, and other intensive operations when battery is low (e.g., < 20%).
*   **Witness Interaction:**
    *   Submits their created `NexusContentObjectV1` or `NexusInteractionRecordV1` to the network for validation by full Witness nodes.
    *   Queries the network (via connected Super-Hosts or directly if capable) for `WitnessProofV1`s related to events they are interested in.
    *   **Does not act as a DLI Witness in this standard Host role.**

### 2.B. Super-Host (Opt-in, more capable/connected mobile or stationary plugged-in device)

A user can opt-in to this role if their device is often charging, has a stable internet connection (e.g., Wi-Fi), and they are willing to contribute more resources.

*   **P2P Communication:**
    *   More active and persistent P2P connections compared to a standard Host.
    *   May act as a **libp2p relay node** for a limited number of other Host nodes, particularly if it has a publicly reachable IP or successfully established NAT traversals.
    *   Maintains a larger active connection pool.
*   **DDS Integration:**
    *   **Enhanced Client-Side Caching:** Can allocate more storage for caching a wider range of content (e.g., all content from followed creators, popular content in their network vicinity).
    *   **Limited ECN-like Behavior:** May participate in storing and serving a small, limited set of *replicated* popular or subscribed content chunks. This is strictly opt-in and resource-limited. It would announce these chunks to the DHT. This helps in local content availability.
    *   Still *not* a full SSN/ASN. Storage commitments are best-effort and for a limited set of data.
*   **Resource Management:**
    *   User explicitly consents to higher resource usage (CPU, battery, data, storage).
    *   Employs battery/data saving strategies but with higher thresholds or more lenient rules (e.g., might perform more P2P when on cellular if user allows).
    *   Ideal for devices that are frequently plugged in (e.g., a tablet used at home).
*   **Discovery (as per `discovery_protocol_refined.md`):**
    *   More active participation in the Discovery System (e.g., Kademlia DHT). Can perform full iterative lookups and may contribute more actively to the DHT's health if its connection is stable.

### 2.C. Decelerator (Mobile node assisting in local content distribution)

This role focuses on hyper-local content sharing, often for popular content within a very limited geographic or network vicinity.

*   **P2P Communication:**
    *   Primarily uses short-range P2P technologies if supported by the OS and consented by the user:
        *   **Wi-Fi Direct (Android):** For direct device-to-device transfer without an AP.
        *   **Bluetooth LE/Classic (iOS/Android):** For discovery and potentially small data transfers or initiating connections over Wi-Fi.
        *   **Nearby Connection APIs (e.g., Android's Nearby Connections API, Apple's MultipeerConnectivity):** Abstract these platform specifics.
    *   Also uses standard libp2p for local network (WLAN) peer discovery and chunk exchange.
*   **DDS Integration:**
    *   Serves `DDSChunk`s from its local cache (content previously downloaded and consumed) to other nearby mobile nodes that request them.
    *   Helps reduce redundant downloads of popular content from distant SSNs, saving mobile data for users in the vicinity.
    *   Does not store data it hasn't already acquired for its own use.
*   **Resource Management:**
    *   Typically activates when the device is on Wi-Fi and ideally charging.
    *   User must explicitly consent to local sharing features.
    *   Limited duration of active "deceleration" to save battery unless plugged in.
*   **Discovery:** Uses local peer discovery mechanisms (mDNS, platform-specific nearby services) in addition to potentially querying the DHT for local peers.

### 2.D. Witness (Mobile Witness - specific, limited capacity)

Full DLI Witnessing (as defined in `tech_specs/pow_protocol.md`) is resource-intensive and requires high uptime, making it generally unsuitable for typical mobile devices. However, limited, specialized forms of participation are conceivable:

*   **Participation Model:**
    *   **Strictly Opt-In:** Requires explicit user consent due to resource implications.
    *   **Not General-Purpose Witnesses:** Mobile Witnesses would *not* participate in all Witness committees or validate all event types.
    *   **Potential Limited Roles:**
        1.  **Low-Resource Event Committees:** Participate in validating extremely lightweight DLI events that require minimal computation and data transfer (e.g., "presence pings" if EchoNet had such a feature, simple reaction tallies if consensus on them was needed separately). Event types must be carefully selected.
        2.  **Geo-Specific Attestations (Highly Experimental & Complex):** If EchoNet implements features requiring coarse-grained, privacy-preserving location attestation (e.g., "this content is popular in X region"), mobile devices *could* contribute such attestations. This is very complex due to GPS spoofing, privacy concerns, and requiring robust proof-of-location mechanisms. *This is a very speculative role.*
        3.  **Client-Side Pre-Validation Assistance (Helper Role, Not DLI Consensus):**
            *   A mobile app could perform extensive pre-validation checks on content or interactions *before* they are submitted to the main DLI Witness network. This helps reduce the load on full Witnesses by filtering out malformed or clearly invalid events early.
            *   The output of this pre-validation is *not* a formal DLI attestation but a client-side check.
*   **P2P Communication:**
    *   Must be able to reliably receive event data for the specific, limited event types it's eligible to witness.
    *   Must be able to broadcast its `WitnessAttestationV1` (for its limited scope) to the network or designated aggregators.
    *   Likely relies on a stable connection to a Super-Host or desktop node for relaying these communications.
*   **Resource Management:**
    *   **Strict Time Limits:** Participation in any active witnessing task (if selected for a rare, suitable committee) must be very short-lived.
    *   **Minimal Data Processing:** Only for very small, predefined event types.
    *   **Battery/Data Critical:** Witnessing functions would immediately cease if battery is low or on metered connection unless explicitly overridden by the user for a short emergency.
*   **Security:**
    *   **Key Management:** Witness signing keys *must* be protected by hardware-backed keystores (Android Keystore, iOS Secure Enclave). See Section 6.
    *   **Risk Assessment:** Mobile devices are generally more susceptible to malware than dedicated servers. The risk of compromised Mobile Witnesses must be factored into the trust model for any committees they participate in. Their attestations might carry less weight or require more corroboration.
    *   **Eligibility:** Stricter eligibility (e.g., higher reputation, specific device security attestations if available) might be needed.

## 3. P2P Networking for Mobile

*   **Protocol Stack:** **Libp2p** is the recommended foundation due to its:
    *   Transport agnosticism (TCP, QUIC, WebSockets, WebRTC).
    *   NAT traversal solutions (STUN, TURN, AutoRelay, Hole Punching).
    *   Stream multiplexing.
    *   Peer discovery mechanisms (mDNS, DHT, rendezvous points).
    *   Encryption and security protocols (TLS 1.3, Noise).
*   **Connection Management:**
    *   **Graceful Handling of Intermittency:** Robust reconnect logic, session resumption where possible.
    *   **Connection Pooling:** Maintain a small pool of active connections to Super-Hosts or important peers.
    *   **Opportunistic Connections:** Attempt direct P2P but quickly fall back to relays if unsuccessful.
*   **Discovery for Mobile:**
    *   **DHT Client:** Mobile nodes typically act as DHT clients, not full routing nodes.
    *   **Proxying/Delegation:** May perform DHT lookups via a connected Super-Host or gateway node to save resources.
    *   **Local Discovery:** mDNS/DNS-SD for discovering peers on the same local network.
    *   **Rendezvous Points:** Predefined or governance-updated rendezvous servers can help bootstrap connections.

## 4. DDS Interaction for Mobile

*   **Light Client Protocol:** Mobile nodes do not download the entire DLI state or large DDS indices. They fetch specific data as needed.
*   **Chunk Management:**
    *   **On-Demand Fetching:** Retrieve `DDSChunk`s using `RetrieveChunk` RPC when content is accessed.
    *   **Smart Caching:** Implement LRU or other caching strategies for DDS chunks. Cache size is user-configurable and/or dynamically adjusted based on available storage.
    *   **Partial Retrieval:** Utilize byte range requests in `RetrieveChunk` if only parts of a chunk are needed (e.g., for streaming media previews).
*   **Background Synchronization:**
    *   **Very Limited:** Only for critical updates or user-subscribed content (opt-in).
    *   **Constraints:** Adheres to OS rules (e.g., Android's WorkManager, iOS's Background App Refresh).
    *   **Triggers:** Typically when on Wi-Fi and charging.

## 5. Resource Management Strategies (Platform Agnostic)

These apply across all mobile roles, with varying intensity.

*   **Battery Awareness:**
    *   Mobile app regularly queries OS APIs for battery level and charging status.
    *   Defines operational tiers: e.g., Full Functionality (charging, >50%), Reduced P2P (on battery, >20%), Minimal/Paused (on battery, <20%).
*   **Data Usage Control:**
    *   User settings: "Wi-Fi only," "Allow mobile data (with warning/cap)," "No background data on mobile."
    *   App tracks its data usage and alerts user if approaching configured limits.
    *   Prioritize data-efficient P2P protocols and payload compression.
*   **Storage Management:**
    *   Clear display of storage used by EchoNet cache and any originated content.
    *   User options to: clear cache, set cache size limits, offload originated content once securely replicated to SSNs.
    *   Graceful handling of low storage conditions (e.g., stop caching, prompt user).
*   **Background Processing Rules:**
    *   Strict adherence to Android Doze/App Standby and iOS background execution limits.
    *   Use of foreground services (with persistent user notification) only when essential for an active, user-initiated task (e.g., Super-Host relaying, active Decelerator sharing).

## 6. Security & Privacy for Mobile Nodes

*   **Key Management:**
    *   **Hardware-Backed Keystores:** All private keys (user DIDs, any Mobile Witness signing keys) MUST be generated and stored within the platform's hardware security module (e.g., Android Keystore using StrongBox, iOS Secure Enclave). Keys should be marked as non-exportable.
    *   Biometric authentication (fingerprint, face ID) should be used to authorize sensitive operations like signing transactions or attestations.
*   **Data Encryption on Device:**
    *   Cached content chunks, private user data, and application settings stored on the device MUST be encrypted using platform-provided file encryption or database encryption mechanisms (e.g., SQLCipher, Android's file-based encryption). Encryption keys should be protected by the hardware keystore.
*   **Permissions:**
    *   Request only essential permissions. Clearly explain why each permission is needed (e.g., Network for P2P, Storage for cache, Location *only if* geo-specific features are implemented and user opts-in).
    *   Use runtime permissions model.
*   **Anti-Malware & Device Integrity:**
    *   While users are primarily responsible for their device's overall security, the EchoNet app should:
        *   Follow secure coding practices (OWASP Mobile Top 10).
        *   Consider using SafetyNet Attestation (Android) or DeviceCheck (iOS) to assess device integrity before enabling highly sensitive roles like Mobile Witness (though these are not foolproof).
*   **Privacy with P2P:**
    *   Minimize IP address exposure in P2P interactions by preferring communication through relays (Super-Hosts, dedicated relays) when direct connection is not essential or for privacy-sensitive users.
    *   Libp2p features like AutoNAT can help in discovering public IPs, which users should be aware of if direct connections are made.
    *   Avoid logging sensitive personal data.

## 7. Upgradeability of Mobile Node Software

*   **Distribution:** Primarily via official app stores (Google Play Store, Apple App Store).
*   **Protocol Compatibility:** Mobile apps must be able to handle DLI protocol changes gracefully:
    *   Check for protocol version compatibility with connected peers/nodes.
    *   Support forced updates for critical security or protocol changes, as per app store mechanisms.
    *   Adherence to `echonet_v3_versioning_strategy.md` is crucial. App should signal its supported protocol versions.

---
These specifications aim to provide a robust framework for integrating mobile devices as valuable and diverse participants in the EchoNet ecosystem, always respecting their resource limitations and security posture.
