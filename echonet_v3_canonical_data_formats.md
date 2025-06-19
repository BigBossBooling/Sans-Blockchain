**EchoNet Canonical Data Formats & Serialization Rules (v3.0)**

This document specifies canonical data formats and serialization rules for `EchoNet` data structures that require deterministic hashing (e.g., for `ContentID` generation), verifiable digital signatures, or consistent input to DLI-native "contract" logic. This aligns with the Expanded KISS Principle "Systematize for Scalability, Synchronize for Synergy" by ensuring data integrity and unambiguous representation across a distributed system.

**1. Canonicalization Philosophy**

*   **Criticality:** Canonicalization is vital for:
    *   **Verifiable Signatures:** Ensuring that a signature generated on one node can be verified by any other node, proving data integrity and authenticity. The signed data must produce the exact same byte stream every time.
    *   **Consistent `ContentID`s:** The content-addressable nature of `EchoNet` relies on `ContentID`s being deterministic hashes of specific content and metadata. This requires a canonical representation of that input data.
    *   **Deterministic DLI-Native "Contract" Execution:** If DLI-native logic (e.g., for Payouts, Ad interactions, PoE) makes decisions based on hashed inputs or verifies signatures over inputs, these inputs must be canonicalized to ensure all Witnesses/nodes reach the same conclusions.
    *   **Preventing Replay Attacks (when combined with nonces/timestamps):** Consistent serialization is a prerequisite for effective nonce and timestamp validation in signed messages.
*   **Goals:**
    *   **Unambiguous Byte Representation:** For any given instance of a data structure that needs to be canonicalized, the serialization process must produce a single, uniquely defined sequence of bytes.
    *   **Cross-Platform Consistency:** The canonical form must be achievable across different programming languages and operating systems that `EchoNet` nodes or clients might use.
    *   **Simplicity and Clarity:** While rigorous, the rules should be as simple as possible to implement correctly and audit.

**2. Choice of Canonical Serialization Format**

*   **Primary Serialization (for RPC & General Storage): Protocol Buffers (Protobuf)**
    *   As stated in `echonet_v3_communication_protocols.md`, Protobuf is the chosen format for general message serialization due to its efficiency and strong typing.
    *   Protobuf has a defined binary encoding which is generally consistent. However, for cryptographic hashing, "generally consistent" is not enough; it must be strictly canonical.
*   **Challenges with Default Protobuf Binary Encoding for Canonicalization:**
    *   **Map Key Order:** The order of key-value pairs in a Protobuf `map` field is not guaranteed in its binary encoding. This is a major issue for canonical hashing.
    *   **Default Values:** The presence or absence of fields set to their default value can sometimes vary in encoding.
    *   **Unknown Fields:** While Protobuf parsers can ignore unknown fields, their presence or absence in the binary stream would alter a hash.
*   **`EchoNet` Canonical Form - Recommendation: Strict Subset of Protobuf or JCS-like approach for Hashing Payloads**

    To achieve true canonicalization for data that will be hashed or signed, `EchoNet` will adopt the following:

    **Option A (Preferred if feasible with Protobuf tooling): Strict Proto3 Serialization Rules for Hashing**
    1.  **Field Order:** Fields *must* be serialized in ascending order of their field numbers as defined in the `.proto` schema.
    2.  **Map Fields:** `map<key_type, value_type>` fields must be transformed into a `repeated` field of a nested message type `message MapEntry { key_type key = 1; value_type value = 2; }`. These `MapEntry` messages must then be sorted by their `key` field (lexicographically for strings, numerically for integers) before serialization. This ensures a deterministic order for map contents.
    3.  **Default Values:** Fields explicitly set to their default value (e.g., 0 for int32, empty string for string) *must* be included in the serialization (i.e., not omitted as is common in Proto3 for default values). This ensures that absence vs. presence of a default value doesn't change the hash. *This might require custom serialization options or a post-serialization step if standard Proto3 tools omit defaults.*
    4.  **Optional Fields (Proto3 `optional` keyword):** If an `optional` field is not set, it must be omitted. If set (even to its default value), it must be included.
    5.  **Unknown Fields:** Data to be hashed must *not* contain unknown fields. Schemas must be strictly adhered to.
    6.  **Standard Data Type Encoding:** Rely on Protobuf's well-defined binary encoding for primitive types (varints for integers, UTF-8 for strings, fixed-width for some numbers, IEEE 754 for floats/doubles – though floats are discouraged in hashed data).

    **Option B (Fallback if Strict Proto3 is problematic): JSON Canonicalization Scheme (JCS - RFC 8785)**
    1.  Convert the data structure to a JSON representation.
    2.  Apply JCS rules:
        *   UTF-8 encoding without BOM.
        *   No insignificant whitespace.
        *   Object keys sorted lexicographically (Unicode codepoint order).
        *   Specific number and string representations.
    3.  The resulting canonical JSON byte stream is then hashed.
    *   **Consideration:** This adds a JSON conversion step and might be less performant than a direct binary canonicalization if achievable.

    **Decision:** For `EchoNet`, **Option A (Strict Proto3 Serialization Rules)** is the target, as it aligns with the primary choice of Protobuf for general serialization, aiming for efficiency. Tooling or specific library configurations will be necessary to enforce the strictness regarding map ordering and default value inclusion. If this proves overly complex to implement reliably across all target languages, JCS will be the fallback for signed/hashed payloads.

**3. Canonicalization Rules for Core Data Structures**

The following rules apply to the structures defined in `echonet_v3_core_data_structures.md` when they need to be canonicalized for hashing or signing, assuming the "Strict Proto3 Serialization" approach.

*   **General Rules:**
    *   **Field Order:** Fields are processed (for hashing/signing payload construction) strictly in the order of their field numbers defined in the `.proto` schema.
    *   **Data Type Representation:**
        *   **Integers (`int32`, `int64`, `uint32`, `uint64`):** Protobuf varint encoding. `sint32`, `sint64` use ZigZag encoding. `fixed32`, `fixed64`, `sfixed32`, `sfixed64` use fixed-width, little-endian representation. (Standard Protobuf rules apply, ensure consistency).
        *   **Strings (`string`):** UTF-8 encoded byte sequence. Length-prefixed as per Protobuf spec.
        *   **Booleans (`bool`):** Encoded as a varint (0 or 1).
        *   **Bytes (`bytes`):** Length-prefixed raw byte sequence.
        *   **Arrays/Lists (`repeated` fields):** Elements are serialized in their existing order within the list. The entire list is length-prefixed or uses packed encoding if specified in `.proto`.
        *   **Maps (`map` fields):**
            *   MUST be converted to a `repeated` list of a new `message <FieldName>Entry { key_type key = 1; value_type value = 2; }`.
            *   This list of `Entry` messages MUST be sorted based on the `key` field before serialization. Sorting rules:
                *   Numeric keys: Sort numerically.
                *   String keys: Sort lexicographically based on Unicode codepoints.
        *   **Optional/Null Fields (Proto3 `optional` keyword or oneof):**
            *   If an `optional` field is not explicitly set, it is omitted from the serialization.
            *   If an `optional` field *is* set (even to its language-specific default like 0 or empty string), it MUST be included in the serialization.
            *   For `oneof` fields, only the chosen field is serialized. If no field in the `oneof` is set, nothing is serialized for it.
        *   **Default Values for non-optional fields (standard proto3 behavior):** Fields set to their default value (0 for numeric types, false for bool, empty for string/bytes/repeated/map) ARE serialized. *This rule is a clarification of standard Proto3 behavior which sometimes omits default scalar values on the wire but should be consistently included for hashing.*
        *   **Nested Structures (Messages as fields):** Canonicalization rules apply recursively. The nested message is serialized according to these rules, and its resulting byte stream (length-prefixed) is included.
        *   **Floating-Point Numbers (`float`, `double`):** IEEE 754 binary32 (for float) or binary64 (for double) representation, typically little-endian. **Strongly discouraged in any data structure whose hash contributes to a ContentID or is part of a DLI-native contract's critical logic due to potential precision issues across platforms.** `CurrencyAmountString` is preferred for monetary values.
*   **Schema Versions:** The `version` field within data structures like `ContentObject` (referring to the schema version of that structure) IS included in the canonical serialization.

**4. Specific Use Cases & Examples**

*   **A. `ContentID` Generation (Ref: `ContentObject` in `echonet_v3_core_data_structures.md` and `content_hash_timestamping_refined.md`)**
    *   The pre-image for the `ContentID` hash is the canonicalized binary serialization of a conceptual `ContentIDInput` structure containing:
        1.  `version`: `SemanticVersionString` (of the `ContentObject` structure itself)
        2.  `creatorDID`: `DIDString`
        3.  `clientAssertedTimestamp`: `TimestampInt64`
        4.  `contentType`: `string`
        5.  `bodyHash`: `HashString` (hash of the raw content body bytes)
        6.  `coreMetadataHash`: `HashString` (hash of the canonicalized `CoreMetadataObject`)
    *   **Canonicalization of `CoreMetadataObject` for `coreMetadataHash`:**
        *   All fields of `CoreMetadataObject` (title, tags, summary, language, customAttributes) are serialized according to the general rules (field number order, sorted map keys for `customAttributes`, existing order for `tags` array). The resulting byte stream is then hashed (e.g., SHA256) to produce `coreMetadataHash`.
    *   The `ContentIDInput` fields are then serialized in their defined order (e.g., numerical field order from their `.proto` definition), and this final byte stream is hashed (e.g., SHA256) to produce the `ContentIDString`.

*   **B. Digital Signatures (e.g., `ContentObject.signature`, `InteractionEvent.signature`, `WitnessAttestation.signature`)**
    *   The data to be signed (the "message") is always the canonicalized binary representation of the structure or a defined subset of its fields relevant for the signature's scope.
    *   For example, for `InteractionEvent`:
        1.  A conceptual "signing payload" is constructed containing `interactionType`, `actorDID`, `targetContentID` (if present, else a defined nil/empty representation), `targetDID` (if present, else nil/empty), `clientAssertedTimestamp`, and a hash of the canonicalized `payload` field of the `InteractionEvent`.
        2.  This signing payload structure is then serialized using the canonicalization rules (fields in their .proto defined order).
        3.  The hash (e.g., SHA256) of this resulting byte stream is what gets signed. The `eventID` field of `InteractionEvent` would store this hash. The `signature` field stores the digital signature of this `eventID`.
    *   This ensures that what is signed is precisely defined and reproducible.

*   **C. DLI-Native "Contract" Inputs & State Hashing**
    *   Any data passed as input to DLI-native contract logic that affects state transitions or requires signature verification must be canonicalized before processing or hashing.
    *   If the DLI state itself (or parts of it) needs to be hashed (e.g., for creating a state root or for an audit trail), the relevant state data structures must be canonicalized before hashing, following these defined rules.

**5. Impact on Implementation**

*   **Strict Adherence:** All `EchoNet` node implementations and client SDKs *must* strictly adhere to these canonicalization rules when preparing data for hashing or signing.
*   **Serialization Libraries:** While Protobuf libraries handle much of the binary encoding, specific care and potentially custom wrapper functions will be needed for:
    *   Ensuring map keys are sorted (by transforming maps to sorted lists of key-value entries before serialization).
    *   Consistently including fields set to their default values if this deviates from the standard library behavior for a given language but is required by the canonical form.
*   **Testing:** Rigorous cross-implementation testing is essential:
    *   Generate canonical byte streams for various data structure instances in a reference implementation.
    *   Verify that other implementations (potentially in different languages) produce the exact same byte stream for the same logical input data.
    *   Test signature generation and verification across different implementations using data canonicalized according to these rules.

**6. Versioning of Canonicalization Rules**

*   The canonicalization rules for a given data structure are implicitly tied to the `version` of that data structure (as defined in `echonet_v3_versioning_strategy.md` and typically present in a `version` or `schemaVersion` field within the structure like `ContentObject.version`).
*   If a change to a data structure is breaking *and* it also requires a change to its canonicalization rules (e.g., changing field order logic, though this should be avoided by relying on field numbers from `.proto` definitions), the MAJOR version of the data structure must be incremented.
*   The code responsible for canonicalizing a specific version of a data structure must apply the rules defined for that version. Changes to canonicalization rules for an existing data structure version are not allowed, as this would break determinism.

By defining and adhering to these canonicalization rules, `EchoNet` can ensure the data integrity, verifiability, and deterministic behavior essential for a robust and trustworthy decentralized system.
