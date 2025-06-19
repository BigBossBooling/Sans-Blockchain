**EchoNet Core Definitions (v3.0)**

This document provides clear definitions for each core module and major sub-component of the `EchoNet` system, focusing on their unambiguous purpose, the specific problem they address, and their key responsibilities. This is based on the "K - Know Your Core, Keep it Clear" principle and derived from the refined architectural documents.

---

**Module: Content Creation & Management**
*   **Core Purpose:** To provide a seamless, robust, and decentralized experience for users to create, manage, and prepare content for publication on `EchoNet`.
*   **Problem Solved:** Enables users to produce and manage their digital content (articles, posts, multimedia) without relying on centralized platforms, giving them control over their work and preparing it for interaction within the decentralized `EchoNet` ecosystem.
*   **Key Responsibilities:**
    *   Providing rich text editing and multimedia embedding capabilities.
    *   Serializing content into a standardized, compact format suitable for `EchoNet`.
    *   Managing content metadata (titles, tags, author DID, client-side timestamps).
    *   Facilitating local draft management with optional encrypted synchronization to user-controlled DDS.
    *   Handling content versioning by linking updated content to previous `ContentID`s.
    *   Performing client-side pre-publication validation and previews.
    *   Packaging, signing (with author's DID), and submitting finalized content to `EchoNet` for Witness validation and DDS distribution.

---

**Module: Content Discovery & Feed**
*   **Core Purpose:** To enable users to efficiently discover relevant, high-quality content and access personalized content feeds from the decentralized `EchoNet` data stores.
*   **Problem Solved:** Addresses the challenge of finding desired information and engaging content within a large, distributed data environment, moving beyond centralized recommendation algorithms and providing users with more control over their discovery experience.
*   **Key Responsibilities:**
    *   Processing user-initiated search queries (keyword, tag, author-based).
    *   Interfacing with the `EchoNet` Discovery Protocol (DHT, distributed indexers) to locate content.
    *   Aggregating and filtering search results from decentralized sources.
    *   Building and maintaining distributed, searchable indices of content metadata (via specialized, incentivized Index Nodes).
    *   Ordering search results and feed content using configurable ranking and relevance algorithms (considering quality signals, personalization, and social signals).
    *   Generating various types of content feeds (home, trending, topic-specific) based on user preferences, follows, and `EchoNet` signals.
    *   Optionally, suggesting content via a decentralized recommendation engine.

---

**Module: Engagement & Interaction**
*   **Core Purpose:** To facilitate meaningful, verifiable, and attributable interactions between users and content within the `EchoNet` ecosystem.
*   **Problem Solved:** Enables a rich social experience around content that is transparent and resistant to manipulation, by ensuring interactions are user-signed, validated, and can contribute to a user's reputation and potential rewards.
*   **Key Responsibilities:**
    *   Processing user comments, including signing, `EchoNet` validation, and storage on DDS.
    *   Handling user reactions (e.g., likes) as signed events, potentially batched for efficiency.
    *   Managing content shares/reverbs as new, linked content types on `EchoNet`.
    *   Facilitating follow/subscription relationships between users, topics, and publications as signed declarations.
    *   Processing user-submitted content flags as signed attestations for input into moderation systems.
    *   Enabling user mentions and managing a decentralized notification system based on `EchoNet` events.
    *   Ensuring all interactions are digitally signed by the user's DID for authenticity and non-repudiation.

---

**Module: Distributed Data Stores (DDS)**
*   **Core Purpose:** To provide persistent, decentralized, resilient, and cost-effective storage for all `EchoNet` content chunks and critical metadata, addressable via unique `ContentID`s.
*   **Problem Solved:** Addresses the need for content hosting that is not reliant on centralized servers, is resistant to censorship, and ensures data availability and durability even if individual storage providers fail or leave the network.
*   **Key Responsibilities:**
    *   Securely storing data chunks (pieces of content) addressed by their cryptographic hashes.
    *   Distinguishing roles for different node types: Origin Nodes, Standard Storage Nodes (SSN), Archival Storage Nodes (ASN), and Ephemeral Cache Nodes (ECN).
    *   Implementing data sharding (chunking) and erasure coding (e.g., Reed-Solomon) to ensure durability and storage efficiency.
    *   Managing data replication and redundancy according to defined parameters and placement strategies.
    *   Responding to Proof-of-Storage/Retrievability (PoSR) challenges from Witnesses to prove data integrity and availability.
    *   Serving requested data chunks to authorized clients efficiently.
    *   Participating in automated self-healing processes to repair and re-replicate lost or corrupted data.
    *   Supporting incentive models for storage providers.

---

**Module: Proof-of-Witness (PoW) Validation**
*   **Core Purpose:** To provide a lightweight, fast, secure, and decentralized consensus mechanism for validating the authenticity and integrity of events occurring on `EchoNet`.
*   **Problem Solved:** Establishes a trustworthy system for event validation (e.g., content publication, key interactions, storage proofs) without the computational overhead of traditional Proof-of-Work mining, enabling efficient and scalable DLI operations.
*   **Key Responsibilities:**
    *   Defining eligibility criteria (stake, reputation, performance) for nodes to act as Witnesses.
    *   Facilitating the dynamic and random selection of Witness Committees for specific events or batches of events.
    *   Performing defined validation checks specific to each event type (e.g., signature verification, schema compliance, policy adherence, PoSR proof validation).
    *   Generating signed attestations upon successful validation of an event.
    *   Participating in a consensus messaging flow to ensure a threshold (M of N) of committee members agree on an event's validity.
    *   Contributing to the generation of a "Proof-of-Witness Receipt" as verifiable proof of an event's validation, including a network timestamp.
    *   Handling disputes and penalizing malicious Witness behavior (e.g., via slashing).

---

**Module: Content Hash & Timestamping**
*   **Core Purpose:** To provide verifiable integrity for all `EchoNet` content and assign it a trustworthy, decentralized, and temporally ordered network timestamp.
*   **Problem Solved:** Ensures that content cannot be tampered with after publication (immutability via hashing) and that there is a reliable, network-agreed-upon time for when content or events were recognized, which is crucial for versioning, sequencing, and dispute resolution.
*   **Key Responsibilities:**
    *   Defining and implementing the cryptographic hashing algorithm (e.g., SHA-256) to generate unique `ContentID`s from content data and key immutable metadata.
    *   Facilitating the assignment of an initial, client-asserted timestamp by the content creator.
    *   Determining the final, authoritative "Network Timestamp" for an event as the median of timestamps provided by the attesting Witness Committee members.
    *   Structuring the "Proof-of-Witness Receipt" to securely include the `EventID` (e.g., `ContentID`), the `NetworkTimestamp`, and the collective signatures of the validating Witnesses.
    *   Exploring and implementing batching techniques for timestamping frequent, small events to improve efficiency.

---

**Module: Replication & Redundancy**
*   **Core Purpose:** To ensure high data durability, availability, and fault tolerance for all content stored within `EchoNet`'s Distributed Data Stores (DDS).
*   **Problem Solved:** Mitigates the risk of data loss or unavailability due to individual storage node failures, network partitions, or malicious actions, thereby maintaining the integrity and accessibility of `EchoNet` content over time.
*   **Key Responsibilities:**
    *   Defining and managing core redundancy parameters, including target replication factors (for full copies) and erasure coding schemes (e.g., Reed-Solomon `k`/`(k+m)`).
    *   Implementing intelligent data placement strategies to distribute replicas and erasure-coded fragments across geographically and administratively diverse Storage Nodes.
    *   Overseeing the automated self-healing process, including monitoring data health (via PoSR, heartbeats), detecting data loss or insufficient redundancy, and triggering data reconstruction and re-replication.
    *   Optimizing for data durability through proactive redundancy management and tiered storage (hot/cold).
    *   Optimizing for retrieval speed through geographically aware placement, support for Ephemeral Cache Nodes, and enabling parallel chunk downloads.

---

**Module: Discovery Protocol**
*   **Core Purpose:** To enable efficient, decentralized, and robust lookup of content locations (`ContentID` to DDS nodes), network node addresses/services (Witnesses, Indexers, etc.), and other resources within the `EchoNet` system.
*   **Problem Solved:** Allows participants in a large, dynamic, and decentralized network to find necessary data and services without relying on centralized directories or registries, ensuring censorship resistance and operational resilience.
*   **Key Responsibilities:**
    *   Implementing and maintaining a Distributed Hash Table (DHT, e.g., Kademlia-based) for structured resource lookup based on XOR distance.
    *   Managing Node IDs and routing tables (k-buckets) within the DHT.
    *   Facilitating the storage of key-value pairs in the DHT (e.g., `ContentID` -> list of SSN contact details).
    *   Augmenting DHT lookups with gossip protocols for broader information dissemination (e.g., node availability, service advertisements, network statistics).
    *   Supporting specific discovery tasks: content location, node service discovery, and general peer discovery.
    *   Optimizing for lookup speed (e.g., caching, concurrent queries) and network traffic reduction (e.g., efficient message design, controlled republishing).

---

**Module: Direct Creator Payouts**
*   **Core Purpose:** To enable creators to receive direct financial compensation for their content consumption and direct support from consumers, facilitated by `EchoNet` DLI logic.
*   **Problem Solved:** Addresses the issue of creator remuneration by removing intermediaries and providing transparent, automated mechanisms for value transfer based on verified content interaction or direct contributions.
*   **Key Responsibilities:**
    *   Managing "Creator Payout Pools" funded by general ad revenue or other platform sources.
    *   Processing "Verified View/Read Events" validated by `EchoNet` Witnesses.
    *   Executing DLI-native "Payout Contract" logic to calculate pro-rata shares for creators based on verified views/reads.
    *   Distributing earnings periodically from Payout Pools to creator wallets in batches.
    *   Facilitating direct tips/donations from consumer wallets to creator wallets.
    *   Supporting creator-defined subscriptions with recurring payment logic managed via `EchoNet`.
    *   Ensuring all payout transactions are recorded on `EchoNet` for transparency and verifiability.

---

**Module: Pay-to-Advertise**
*   **Core Purpose:** To allow advertisers to directly fund the promotion of their content or services within the `EchoNet` ecosystem, with payments managed transparently by DLI logic.
*   **Problem Solved:** Provides a mechanism for advertisers to reach their target audience within `EchoNet` by paying for verified interactions, with clear accountability for ad spend and direct pathways for compensating creators or the network.
*   **Key Responsibilities:**
    *   Enabling advertisers to create campaigns, define budgets, set bid prices, and specify targeting criteria.
    *   Managing ad budget escrows via DLI-native "Ad Contracts" on `EchoNet`.
    *   Processing "Verified Ad Interaction Events" (e.g., views, clicks on promoted content) validated by `EchoNet` Witnesses.
    *   Settling payments from escrow to relevant recipients (Creator Payout Pool, specific creators for direct placements) based on verified interactions and campaign terms.
    *   Refunding unused budget to advertisers upon campaign conclusion.
    *   Providing advertisers with verifiable reports of ad interactions and costs derived from `EchoNet` data.

---

**Module: Proof-of-Engagement (PoE) Rewards**
*   **Core Purpose:** To incentivize and reward users (both creators and consumers) for meaningful interactions and valuable contributions that enhance the quality and health of the `EchoNet` social ecosystem.
*   **Problem Solved:** Addresses the challenge of fostering high-quality engagement over low-effort or spammy interactions by economically rewarding actions that the community or protocol deems valuable, thereby improving the overall user experience and content discovery.
*   **Key Responsibilities:**
    *   Defining and quantifying metrics for various types of quality engagement (e.g., insightful comments, effective shares, accurate content flagging, valuable curation).
    *   Facilitating Witness validation of engagement events against these quality metrics, potentially using reputation scores and lightweight AI assistance, to generate a "PoE Quality Score."
    *   Managing a dedicated "PoE Reward Pool" funded from network sources (inflation, fees, ad revenue share).
    *   Executing DLI-native "PoE Contract" logic to periodically distribute rewards from the pool to users based on their accumulated PoE Quality Scores.
    *   Implementing and continuously refining robust anti-gaming mechanisms (e.g., reputation integration, Sybil resistance, collusion detection, rate limiting) to protect the integrity of the reward system.

---

**Module: Decentralized Advertising Marketplace**
*   **Core Purpose:** To provide a fair, transparent, and competitive environment where advertisers can bid for ad inventory and creators can directly monetize their content space, facilitated by `EchoNet`.
*   **Problem Solved:** Replaces opaque, centralized ad networks with an open marketplace, aiming to reduce intermediary fees, increase advertiser ROI through verifiable metrics, and give creators more control over ad monetization.
*   **Key Responsibilities:**
    *   Supporting different ad inventory types (creator-specific, general feed, keyword-based).
    *   Implementing auction/bidding mechanics (e.g., second-price auction) for ad slots, with bids submitted and managed on `EchoNet`.
    *   Defining and executing ad serving logic, considering bids, relevance, user/creator preferences, and other factors to select ads for display.
    *   Facilitating the delivery of ad creatives (which are `ContentID`-addressed objects on DDS).
    *   Ensuring transparent and verifiable performance metrics (impressions, clicks, CTR, spend) based on `EchoNet`-validated ad interaction events.
    *   Implementing mechanisms to prevent click/impression fraud and ensure fair auction practices.
    *   Managing the flow of funds from advertisers to creators or network pools via `EchoNet` DLI-native "Ad Contracts."

---

**System-Wide Component: Unified User Identity (DID) & Reputation System**
*   **Core Purpose:** To provide every participant with a secure, self-sovereign digital identity (DID) and to maintain a dynamic, multifaceted reputation score reflecting their behavior and contributions across `EchoNet`.
*   **Problem Solved:** Establishes a foundational layer of trust and accountability necessary for a decentralized social and economic system, enabling Sybil resistance, nuanced incentive structures, and community-driven moderation.
*   **Key Responsibilities:**
    *   Facilitating the creation and management of user DIDs.
    *   Associating all on-DLI actions (content creation, interactions, financial transactions, node operations) with DIDs.
    *   Calculating and updating user reputation scores based on a transparent, `EchoNet`-native algorithm that considers positive contributions (quality content, PoE, reliable node operation) and negative actions (spam, malicious behavior, failed PoSR).
    *   Making reputation scores (or relevant aspects) securely accessible to other `EchoNet` modules for decision-making (e.g., Witness selection, PoE weighting, fraud detection).
    *   Ensuring the reputation system itself is resistant to manipulation and evolves with community governance.

---

**System-Wide Component: `EchoNet` Wallet Module**
*   **Core Purpose:** To provide users with a secure, non-custodial interface for managing their DIDs, cryptographic keys, tokens, and for authorizing transactions on `EchoNet`.
*   **Problem Solved:** Empowers users with direct control over their digital assets and identity within the `EchoNet` ecosystem, eliminating reliance on third-party custodians for core interactions.
*   **Key Responsibilities:**
    *   Secure generation and storage of user private/public key pairs linked to their DIDs.
    *   Facilitating the signing of all `EchoNet` transactions (content publication, interactions, value transfers, contract calls) using user's private key.
    *   Displaying token balances (native `EchoNet` tokens, stablecoins, etc.) and transaction history.
    *   Interacting with various `EchoNet` modules to initiate actions (e.g., submitting content, bidding in ad marketplace, claiming PoE rewards).
    *   Providing a clear and understandable interface for users to manage their `EchoNet` participation.
