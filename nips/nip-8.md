---
nip: 8
title: Agent State Model (ASM) - A Unified State Management Framework
author: jolestar(@jolestar)
status: Draft
type: Standards Track
category: Core
created: 2025-05-15
updated: 2025-05-18
requires: NIP-1, NIP-2, NIP-4, NIP-7
---

# NIP-8: Agent State Model (ASM)

## Abstract

**Agent State Model (ASM)** defines a unified framework for describing, merging, querying, and managing the lifecycle of persistent states in AI agents. It serves as the standard for state definition within Agent Capability Packages (ACPs) as specified in NIP-7.
It extends **JSON-Schema 2020-12** with a minimal set of `x-asm` annotations:

| Area             | Annotation                       | Purpose                                  |
|------------------|----------------------------------|------------------------------------------|
| **Container kind** | `container`                      | `object`, `map`, `list`, `log`           |
| **CRDT policy**  | `crdt` or per-field `x-crdt`     | `lww_register`, `rga_text`, `or_map`, ... |
| **Lifecycle**    | `x-ttl`, `x-retention`, `x-compression` | automated ageing, summarisation, compression |
| **Visibility**   | `x-visibility`                   | `private`, `shared`, `public`            |

Together with a small JSON-AST query language (ASM-QL) and the standard `state.*` tool family, ASM allows routers and runtimes to treat **conversation history, long-term memory, settings, and arbitrary application data** in a unified, verifiable way.

## Motivation

| Need                 | Current pain                       | ASM solution                                       |
|----------------------|------------------------------------|----------------------------------------------------|
| Multiple data shapes | Only record-like objects in NIP-7  | Adds `map`, `list`, `log` containers               |
| Conflict-free merging| Each plugin hand-picks a CRDT      | Policy is declared in schema → runtime acts uniformly |
| Unified querying     | Ad-hoc filters per capability      | ASM-QL: portable, verifiable query AST             |
| Memory & chat history| Not standardised                   | Built-in **ConversationLog** & **MemoryStore** models |
| Data isolation       | Name clashes across ACPs           | **Namespace** rules under DID hierarchy            |

This NIP addresses the need for a standardized and comprehensive state management framework for AI agents. Existing mechanisms (like NIP-7 for record-like objects) are insufficient for handling diverse data shapes, ensuring conflict-free data merging across different agent capabilities (ACPs), providing a unified query language, standardizing common state types like conversation history and memory, and enforcing data isolation. ASM provides these capabilities, leading to more robust, interoperable, and manageable agent states.

## Specification

### Terminology

| Term           | Meaning                                                                 |
|----------------|-------------------------------------------------------------------------|
| **Container**  | Top-level state shape (`object`, `map`, `list`, `log`).                 |
| **CRDT policy**| Conflict-resolution algorithm bound to container / field.                 |
| **Memory Scope**| Named partition (e.g. `sc:note`, `sc:chat`) governed by ASM rules.      |
| **ASM-QL**     | JSON serialisable query language evaluated by runtime and optionally verified on-chain. |

### Built-in Core Models

Nuwa runtimes **MUST** create exactly one instance of each core model for every user on first run.

#### ConversationLog `did:nuwa:core:ConversationLog#v1`
```json
{
  "$id": "did:nuwa:core:ConversationLog#v1",
  "type": "object",
  "x-asm": {
    "container": "log",
    "crdt": "append_only",
    "x-ttl": "P14D",
    "x-compression": "br",
    "x-visibility": "private"
  },
  "properties": {
    "entries": {
      "type": "array",
      "x-crdt": "log_rga",
      "items": {
        "type": "object",
        "properties": {
          "id":        {"type":"string","format":"uuid"},
          "role":      {"type":"string","enum":["user","assistant"]},
          "content":   {"type":"string"},
          "timestamp": {"type":"string","format":"date-time"}
        },
        "required":["id","role","content","timestamp"]
      }
    }
  },
  "required": ["entries"]
}
```

**Note**:
- `"container": "log"`: Specifies that this is an append-only event stream.
- `"crdt": "append_only"`: Conflict-free replicated data type for appending entries.
- `"x-ttl": "P14D"`: Entries older than 14 days may be deleted. This value can be configured.
- `"x-compression": "br"`: Data may be stored using Brotli compression, chosen for its balance of compression ratio and speed.
- `"x-visibility": "private"`: Access is restricted to the owner.

#### MemoryStore `did:nuwa:core:MemoryStore#v1`
```json
{
  "$id": "did:nuwa:core:MemoryStore#v1",
  "type": "object",
  "x-asm": {
    "container": "map",
    "keyType": "string",
    "valueRef": "#/definitions/MemoryItem",
    "x-retention": {
      "window": "P90D",
      "strategy": "evict_low_importance"
    },
    "x-visibility": "private"
  },
  "properties": {
    "items": {
      "type": "object",
      "additionalProperties": { "$ref": "#/definitions/MemoryItem" }
    }
  },
  "definitions": {
    "MemoryItem": {
      "type": "object",
      "properties": {
        "value":     {"type":"string"},
        "importance":{"type":"number"},
        "createdAt": {"type":"string","format":"date-time"}
      },
      "required":["value","importance","createdAt"]
    }
  }
}
```

**Note**:
- `"container": "map"`: Specifies that this is a key-value map.
- `"x-retention": {"window": "P90D", "strategy": "evict_low_importance"}`: Retains data for 90 days and evicts low-importance items. "Low importance" is determined by the `importance` field.
- `"x-visibility": "private"`: Access is restricted to the owner.

#### AgentSettings `did:nuwa:core:AgentSettings#v1`
```json
{
  "$id": "did:nuwa:core:AgentSettings#v1",
  "type": "object",
  "x-asm": {
    "container": "object",
    "x-visibility": "private"
  },
  "properties": {
    "language": {"type":"string","enum":["zh","en","ja","es"]},
    "tone":     {"type":"string","enum":["casual","formal","kids"]},
    "theme":    {"type":"string","enum":["light","dark"]},
    "notifOpt": {"type":"boolean"}
  },
  "required": ["language"]
}
```

**Note**:
- `"container": "object"`: Specifies that this is a structured object.
- `"x-visibility": "private"`: Access is restricted to the owner.
- `"required": ["language"]`: Language is mandatory to ensure proper localization.

### Container Kinds & CRDT Policies

| Kind     | Typical usage             | Default CRDT   |
|----------|---------------------------|----------------|
| `object` | Fixed structured record   | per-field mix  |
| `map`    | Arbitrary key → value     | **OR-Map**     |
| `list`   | Ordered sequence          | **RGA / Y-list**|
| `log`    | Append-only event stream  | Monotonic vector|

The canonical `x-crdt` keywords are:
```
lww_register • mv_register • rga_text • grow_only_set • or_map • counter • flag • log_rga
```

### Lifecycle & Visibility

| Annotation      | Format                  | Semantics                                     |
|-----------------|-------------------------|-----------------------------------------------|
| `x-ttl`         | ISO-8601 duration       | Hard expiry; entries beyond TTL MAY be deleted. |
| `x-retention`   | `{window,strategy}`     | Sliding window & summarisation rules.         |
| `x-compression` | `"br"` / `"zstd"` / `"gzip"` | Runtime MAY store snapshots compressed.       |
| `x-visibility`  | `private` / `shared` / `public` | Router MUST enforce access control.           |

### ASM-QL — Query Language

Example AST:
```json
{
  "select": ["id","title"],
  "from":   "did:nuwa:state:note#v1",
  "where":  {"tags":{"$contains":"meeting"}},
  "order":  [{"field":"updatedAt","direction":"desc"}],
  "limit":  20,
  "cursor": "opaque-base64"
}
```
Runtimes MAY translate to SQL, RocksDB iterators, or CR-SQLite views.
Provers MAY embed the same AST in a Merkle proof for verifiable reads.

### `state.*` Tool Semantics

| Tool           | Required params             | Behaviour                                  |
|----------------|-----------------------------|--------------------------------------------|
| `state.create` | `schema_uri`, `object`      | Validate via ASM, emit **create-op**       |
| `state.update` | `schema_uri`, `id`, `patch` | JSON-Patch / `$inc`, `$push` diff          |
| `state.query`  | `query` (ASM-QL)            | Stream results or single shot              |
| `state.delete` | `schema_uri`, `id`, `mode`  | Tombstone or hard delete                   |

All ops are persisted into the NIP-4 CRDT Event Log; snapshots anchored on-chain by NIP-13.

### Namespace Rules

| Namespace | Pattern                                       | Purpose                                     |
|-----------|-----------------------------------------------|---------------------------------------------|
| **core**  | `did:nuwa:core:<Model>#<ver>`                 | Platform-built-in                           |
| **cap**   | `did:nuwa:cap:<cap_id>:state:<Name>#<ver>`    | Each ACP’s private data                     |
| **app**   | `did:nuwa:app:<app_name>:state:<Name>#<ver>`  | Multi-capability application                |
| **user**  | `did:nuwa:user:<did_suffix>:state:<Name>#<ver>`| Experimental / ad-hoc                     |

*   Storage **MUST** be table-isolated per namespace.
*   Registry **MUST NOT** allow duplicate DID + state path except for version bump.

### Runtime Rules

1.  **Schema resolution** — `$id` MUST resolve via local cache → Registry → DID resolver.
2.  **Container isolation** — Distinct `memory_scope` ↔ distinct CRDT log.
3.  **TTL enforcement** — Background task removes or summarises expired log ranges.
4.  **Compression** — If `x-compression` set, runtime MAY store chunks compressed and anchor CAR hash.

### Integration with NIP-7 (Agent Capability Protocol)

The Agent State Model (ASM) defined in this NIP is the designated framework for specifying the `schema` section within an Agent Capability Package (`.acp.yaml` file) as detailed in NIP-7. Capabilities should define their state objects using ASM-compliant JSON Schemas, leveraging the `x-asm` annotations for CRDT policies, lifecycle management, and visibility controls. The `schema_uri` used in `state.*` tool calls within an ACP context will refer to the `$id` of such an ASM-compliant schema.

## Rationale

The design choices for ASM prioritize extending familiar standards like JSON-Schema with minimal, targeted annotations (`x-asm`) to reduce the learning curve and promote adoption.
*   **Extending JSON-Schema**: Leverages a well-understood validation and description language. The `x-asm` annotations provide the necessary extensions for state-specific concerns like CRDT policies, lifecycle, and visibility without reinventing core schema definition.
*   **Container Kinds**: The selected container kinds (`object`, `map`, `list`, `log`) cover the most common data structures needed for agent state, offering a balance between flexibility and specialized handling (e.g., `log` for append-only streams).
*   **CRDT Policies**: Explicitly declaring CRDT policies in the schema allows runtimes to handle conflict resolution uniformly and predictably, which is crucial for distributed and collaborative agent environments. The chosen CRDTs are well-established options.
*   **ASM-QL**: A dedicated, serializable query language (ASM-QL) provides a standard way to query state, enabling portability and potential for on-chain verification. Its JSON-AST format is designed for ease of parsing and translation.
*   **Built-in Models**: Core models like `ConversationLog`, `MemoryStore`, and `AgentSettings` address common agent needs out-of-the-box, promoting consistency across the ecosystem.
*   **Alternative Designs**: Simpler key-value store approaches were considered but lacked the rich typing, validation, and CRDT integration offered by the JSON-Schema-based approach. A more complex, custom state definition language was avoided to favor simplicity and existing tooling.
*   **Related Work**: This NIP draws inspiration from CRDT research (see References), existing schema languages, and distributed data systems.

## Backwards Compatibility


## Test Cases

Test cases are mandatory for this NIP and will be provided in a separate document or repository. They will cover:
*   Validation of various ASM schemas, including all `x-asm` annotations.
*   Correct application of CRDT policies for each container type under concurrent updates.
*   Execution of ASM-QL queries, including all specified clauses and operators.
*   Lifecycle policy enforcement (TTL, retention).
*   Visibility rule enforcement.
*   `state.*` tool semantics and their interaction with the CRDT event log.
*   Namespace isolation and resolution.
A link to the test suite will be added here once available.

## Reference Implementation

A reference implementation of an ASM-compliant runtime module is planned. This will include:
*   A library for parsing and validating ASM schemas.
*   Implementations of the specified CRDTs for each container type.
*   An ASM-QL query engine (potentially translating to an embedded DB or in-memory operations).
*   Handlers for the `state.*` tool family.
*   Mechanisms for lifecycle management and visibility control.
A link to the reference implementation repository will be added here once available.

## Security Considerations

*   **Signature validation**: Runtime MUST reject unsigned or tampered schemas. Schemas, especially those resolved from external sources, should be verifiable (e.g., via digital signatures tied to the schema's DID or source).
*   **Size limit**: Objects >32 MiB require `storage.large` permission. Runtimes must enforce configured size limits for individual state entries and overall state storage per agent/user to prevent denial-of-service attacks.
*   **Query cost**: ASM-QL execution SHOULD be metered and may tie into NIP-3 payment flow. Complex or inefficient queries could lead to excessive resource consumption. Runtimes should implement query complexity analysis, timeouts, or resource quotas.
*   **Privacy**: `x-visibility` enforced via capability tokens (ZCAP-LD). Runtimes must rigorously enforce these visibility rules to prevent unauthorized data access or leakage between private, shared, and public scopes.
*   **Schema Trust**: Loading schemas from untrusted DIDs or URLs can introduce vulnerabilities if the schema itself contains malicious definitions (e.g., overly permissive validation rules or problematic default values). Schema sources should be configurable and subject to policies.
*   **CRDT Complexity**: While CRDTs handle conflicts, understanding their specific merge behaviors is crucial. Developers must choose appropriate CRDT policies for their data to avoid unexpected outcomes.
*   **Injection in ASM-QL**: If ASM-QL queries are constructed from user inputs, care must be taken to prevent injection attacks, similar to SQL injection. Using parameterized queries or strict input validation for query parameters is recommended.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

## References

1.  [**JSON-Schema 2020-12**](https://json-schema.org/specification-links.html#2020-12)
2.  [**DIDComm v2**, Decentralized Identity Foundation](https://identity.foundation/didcomm-messaging/spec/)
3.  [**ZCAP-LD**, W3C CCG](https://w3c-ccg.github.io/zcap-ld/)
4.  [**Automerge Binary File Format**](https://github.com/automerge/automerge)
5.  [**Conflict-Free Replicated Data Types** (Shapiro et al.)](https://hal.inria.fr/inria-00555588/document)
