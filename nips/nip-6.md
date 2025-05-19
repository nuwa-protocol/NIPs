---
nip: 6
title: Unified Agent State Synchronization
author: jolestar(@jolestar)
status: Draft
type: Standards Track
category: Core
created: 2025-05-13
updated: 2025-05-18
requires: NIP-1, NIP-2, NIP-3
--- 

## Abstract

This NIP defines a peer-to-peer (P2P) protocol for synchronizing an agent's state across multiple devices and application contexts. This allows an agent to maintain a consistent state and operate seamlessly regardless of which device or application is currently in use. The protocol aims to support nested JSON schema-like data structures, multi-platform compatibility, offline operation with eventual consistency, and optional end-to-end privacy.

## Motivation

Users often interact with agents from various devices (desktop, mobile, web) and through multiple applications on those devices. A robust synchronization mechanism is crucial to ensure that the agent's knowledge, capabilities, and context are consistent across all these devices and applications. This NIP builds upon existing NIPs for identity (NIP-1), authentication (NIP-2), and payments (NIP-3) to provide a comprehensive solution for unified agent state.

## Specification

### Core Concepts

1.  **Agent Identity**: Each agent is uniquely identified by its DID (as defined in NIP-1). All devices and applications associated with an agent share this common identity. The DID document lists authorized operational keys (for devices, applications, etc.), which are crucial for verifying the authenticity of discovery messages and synchronization updates.
2.  **Device/Application Identity & Authorization**:
    *   Each device or application instance will have its own unique cryptographic key pair. Its public key is registered in the Agent's DID document as a `verificationMethod` entry, as per NIP-1.
    *   **OperationalContextID**: Throughout this NIP, `OperationalContextID` (e.g., `DeviceID`, `ApplicationID`) refers to the unique fragment identifier of an operational key's `verificationMethod` entry within the Agent's DID document (e.g., if a `verificationMethod.id` is `did:agent:123#op-key-xyz`, then the `OperationalContextID` is `op-key-xyz`). This `OperationalContextID` is used in discovery and synchronization messages to uniquely identify a device or application instance within the context of its AgentDID.
    *   A device or application instance must be authorized by the agent's primary key (or a designated master device/application) to participate in state synchronization. This authorization can leverage mechanisms similar to those in NIP-2 for A2A authentication.
    *   Consideration for secure key sharing: New devices or applications, once authorized via NIP-1, need to securely obtain a shared secret or a specific decryption key (e.g., an encrypted `epriv` using a master password or temporary authorization token via QR code/OTP) to access and decrypt the agent's synchronized state. This process is detailed in the "Device Authorization Flow".
3.  **State Representation**:
    *   The agent's state will be represented using Conflict-free Replicated Data Types (CRDTs). CRDTs are well-suited for P2P environments as they allow for concurrent updates from multiple devices and guarantee eventual consistency without complex conflict resolution logic.
    *   The state should support nested JSON schema-like data structures.
    *   **Recommended CRDT Libraries**: Libraries like **Automerge** (good for JSON-like nested states) or **Yjs** (performant for fine-grained collaborative editing and rich data types) are strong candidates due to their robust CRDT implementations and support for complex data structures.
4.  **Peer Discovery**:
    *   Devices and application instances belonging to the same agent need to discover each other's current connection information (e.g., IP address, port, transport protocol, or inter-process communication channels). The Agent's DID document (NIP-1) serves as the authoritative source for verifying that a discovered peer (device or application) is legitimate, but not for storing its dynamic connection information.
    *   Possible mechanisms:
        *   **Decentralized Hash Table (DHT)**: See "Detailed Protocol".
        *   **Rendezvous Server**: See "Detailed Protocol".
        *   **Local Network/Inter-Process Discovery (mDNS/DNS-SD, IPC mechanisms)**: See "Detailed Protocol".
        *   **Discovery via Synchronization Layer**: See "Detailed Protocol".
5.  **Communication Protocol & Synchronization Layer**:
    *   Secure P2P communication channels will be established between devices and applications.
    *   Underlying P2P transport: WebRTC (for browser-based agents) or libp2p (for more general applications).
    *   All communication must be end-to-end encrypted using keys derived during the device/application authorization/authentication phase.
    *   **Synchronization Layer Options**:
        *   **GunDB**: Lightweight, easy to deploy, with built-in SEA for identity and encryption. Suitable for simpler state and rapid deployment.
        *   **OrbitDB**: Builds on IPFS and Merkle-CRDTs, offering decentralized and persistent state. Requires IPFS.
        *   **Matrix**: Provides strong E2EE, user identity, and can act as a secure message/state synchronization bus using its event DAG as a form of CRDT.
        *   Custom sync protocol built directly over Yjs or Automerge providers.
6.  **Synchronization Process**:
    *   **Initial Sync**: When a new device or application is authorized or an existing one comes online, it will attempt to connect to known peers and fetch the current state.
    *   **State Exchange**: Once connected, devices and applications will exchange their CRDT state. CRDTs inherently handle the merging of different versions of the state.
    *   **Delta Updates**: After the initial sync, devices and applications will only transmit changes (deltas) to the state to minimize bandwidth usage. Libraries like Yjs and Automerge support this.
    *   **Gossip Protocol**: Updates can be propagated through the network of connected peers using a gossip-like mechanism to ensure all devices and applications eventually receive all updates.
7.  **Offline Support and Resilience**: (Moved to its own sub-section under Specification for clarity, as it's a key feature)
    *   Devices and applications should be able to operate offline and accumulate state changes.
    *   When a device or application reconnects, it will sync its changes with its peers.
    *   The system should be resilient to devices and applications frequently joining and leaving the network.

### Detailed Protocol

*   **Device Authorization Flow**: This flow enables a new device or application instance to participate in state synchronization. It leverages the operational key registration process defined in NIP-1 and adds steps for securely sharing synchronization-specific secrets.
    *   **1. Operational Key Generation & Initial Registration (Leveraging NIP-1)**:
        *   The new device or application instance generates a new cryptographic key pair (public and private key).
        *   The new device or application instance undergoes the "Device/Operational Key Registration / Update Protocol" as outlined in NIP-1. This results in the new operational public key being added as a `verificationMethod` to the agent's DID document, associated with appropriate verification relationships. This step confirms the device/application as a recognized operational context under the agent's DID.
    *   **2. Request for State Synchronization Access**:
        *   Once recognized by the DID document (per NIP-1), the new device/application requests access to the synchronized state from an existing, already authorized device/application or through a mechanism controlled by the agent's master key.
    *   **3. User Authorization for State Access**:
        *   The user authorizes this request on an existing trusted device/application or via the master key. Authorization methods can include QR code scanning, OTP, secure link, or manual approval on an existing device, similar to the methods described in NIP-1 for key registration but specifically for state access.
    *   **4. Secure Transfer of Synchronization Secrets**:
        *   Upon successful authorization for state access, the necessary shared secrets or decryption keys (e.g., an agent-specific `epriv` for CRDT data, potentially encrypted with a temporary key derived from the authorization process or a master password) are securely transferred to the new device/application. This allows the new device/application to decrypt and participate in the shared state.
    *   **5. Confirmation and Sync Initiation**:
        *   The new device/application confirms receipt and successful decryption of the secrets and initiates the state synchronization process with peers.
*   **Peer Discovery Mechanisms (Primary and Fallback)**:
    *   **1. Local Network/Inter-Process Discovery (e.g., mDNS/DNS-SD, IPC mechanisms)**
        *   **Function**: For devices on the same local network or applications on the same device to discover each other directly.
        *   **Mechanism**: A specific service type (e.g., `_nuwa-agent-sync._tcp` for network, or a named pipe/socket for IPC) is advertised/used. TXT records (for network) or equivalent metadata would contain the Agent DID (for filtering), the specific `OperationalContextID` (i.e., the fragment of its `verificationMethod.id` from the Agent's DID document), connection port/path, and supported protocols.
        *   **Verification**: Discovered `OperationalContextID` and its claims should be verifiable against the Agent's DID document using the public key associated with that `OperationalContextID`.
        *   **Advantages**: Low latency, direct P2P/IPC, no internet traversal for local network or on-device communication.
    *   **2. Decentralized Hash Table (DHT)**
        *   **Function**: Global discovery when local/inter-process discovery fails or is not applicable.
        *   **DHT Key**: A stable key (`DHT_Agent_Key`) is derived from the Agent's DID (e.g., `hash(AgentDID)`).
        *   **DHT Value**: The value stored at `DHT_Agent_Key` should point to a dynamic "meeting place".
            *   **Recommended Approach: Pointer to Dynamic Resource**: The DHT value contains a pointer (e.g., a specific IPFS PubSub topic ID like `hash("nuwa-sync-" + AgentDID)`, a well-known Matrix room alias, or a URL to a micro-rendezvous service).
                *   *Device/Application Announcement*: When a device or application comes online, it retrieves this pointer from the DHT. It then joins this resource (e.g., subscribes to the PubSub topic) and broadcasts its connection information: `(OperationalContextID, ConnectionInfo, Timestamp, Signature)`. The `OperationalContextID` is the fragment of its `verificationMethod.id`. The `Signature` is created by the device/application signing its announcement with its private key.
                *   *Peer Discovery*: Devices and applications listen on this resource for announcements from other peers.
        *   **Verification**: All announcements must be signed by the announcing device/application. Discovering peers verify this signature using the public key for the announced `OperationalContextID` (obtained by resolving the Agent DID document and finding the corresponding `verificationMethod`). This ensures authenticity and authorization.
        *   **Bootstrapping DHT Entry**: The initial creation of the `DHT_Agent_Key` and its value (e.g., the PubSub topic) might require a rendezvous server if no device/application has established it yet.
    *   **3. Rendezvous Server(s)**
        *   **Function**: Acts as a fallback, a bootstrapping aid for DHT, or a primary mechanism if DHT is not preferred, especially for inter-device communication over the internet.
        *   **Mechanism**: Devices/applications register their `(AgentDID, OperationalContextID, ConnectionInfo, Timestamp, Signature)` with one or more pre-configured or well-known rendezvous servers. The `OperationalContextID` is the fragment of its `verificationMethod.id`. Devices/applications query the server for peers belonging to the same `AgentDID`.
        *   **Verification**: Signatures on registered information are crucial. The server itself doesn't need to be highly trusted if all exchanged information is self-certified by device/application signatures, verified against the Agent's DID document.
        *   **API**: Simple registration and lookup (by `AgentDID`, returning a list of signed device/application records).
    *   **4. Discovery via Synchronization Layer (If Applicable)**
        *   **Function**: Some chosen synchronization layers (e.g., Matrix, libp2p with specific discovery modules, OrbitDB/IPFS, or local IPC libraries) provide their own peer discovery mechanisms.
        *   **Verification**: Still relies on NIP-1 DIDs to verify that discovered peers are legitimate devices/applications of the agent.
    *   **Order of Operations & Bootstrapping Strategy**:
        *   A device/application typically attempts discovery in the following order:
            1.  **Known Peers**: Try connecting to peers from previous successful sessions (if connection info is cached and still valid).
            2.  **Local Network/Inter-Process Discovery (mDNS/DNS-SD, IPC)**.
            3.  **Synchronization Layer Native Discovery** (if the chosen sync layer has a robust mechanism).
            4.  **Decentralized Hash Table (DHT)**.
            5.  **Rendezvous Server** (especially if DHT fails or for bootstrapping the DHT entry).
        *   The DID document for the AgentDID is resolved early to obtain the list of authorized `OperationalContextID`s and their public keys for verifying any discovered peers.
*   **CRDT Data Structures for Common Agent State**:
    *   Define standard schemas for agent profile, conversation history, learned preferences, available tools/skills, etc.
    *   Utilize Automerge or Yjs to model these as nested JSON-like structures, allowing for rich data types and conflict-free merging.
*   **Message Formats for Synchronization**:
    *   Dependent on the chosen CRDT library (e.g., Yjs update messages, Automerge changesets) and synchronization layer (e.g., Matrix events, GunDB messages).
    *   Messages should include sender operational context ID, signature, and CRDT payload.
*   **Handling of Large State and Efficient Sync**:
    *   Delta updates are crucial.
    *   Lazy loading or partial synchronization for very large states (see Open Questions).

## Rationale

*(Placeholder: To be filled in with design choices, alternatives considered, and community consensus.)*

## Backwards Compatibility

*(Placeholder: To be filled in. This NIP defines a new protocol, so initial implementations would not have backwards compatibility issues with older versions of this specific protocol. Compatibility with other NIPs like NIP-1, NIP-2, NIP-3 is by design.)*

## Test Cases

*(Placeholder: To be filled in with specific test cases for device/application authorization, peer discovery, state synchronization, conflict resolution with CRDTs, offline operation, and security checks.)*

## Reference Implementation

*(Placeholder: To be provided once a reference implementation is available.)*

## Security Considerations

The security of this NIP relies heavily on the underlying NIPs for identity (NIP-1) and authentication (NIP-2).

*   **Data Integrity and Authenticity**:
    *   All state updates should be signed by the originating device or application instance's key (as registered in the Agent's DID document per NIP-1) to ensure authenticity and integrity. This prevents unauthorized modifications and allows peers to verify the source of updates.
    *   CRDTs themselves do not inherently provide authentication, so signatures are a crucial layer on top.
*   **End-to-End Encryption**:
    *   All P2P communication for state synchronization must be end-to-end encrypted using keys derived during the device/application authorization/authentication phase (potentially leveraging NIP-2 mechanisms or a shared secret established during device onboarding).
    *   State data should be encrypted at rest on devices/applications, using device/application-specific encryption or encryption derived from the shared agent secrets.
*   **Device/Application Authorization and Key Management**:
    *   The "Device Authorization Flow" (or more generally, "Operational Context Authorization Flow") is critical. Secure transfer of synchronization secrets (e.g., an agent-specific `epriv` for CRDT data) is paramount. This process must be protected against man-in-the-middle attacks.
    *   Revocation of operational keys (as per NIP-1) must effectively prevent a compromised or decommissioned device/application from participating in synchronization or accessing future state.
*   **Peer Discovery Security**:
    *   In all peer discovery mechanisms (Local, IPC, DHT, Rendezvous Server), announcements and registrations must be signed by the device/application's key.
    *   Peers must verify these signatures against the public keys listed in the Agent's DID document (NIP-1) before establishing a connection or accepting state from them. This prevents impersonation of legitimate operational contexts.
    *   Rendezvous servers, if used, should be treated as untrusted introducers; the trust is established end-to-end between devices/applications via signature verification.
*   **CRDT-Specific Considerations**:
    *   While CRDTs resolve conflicts automatically, malicious or malformed CRDT operations could potentially disrupt state or lead to excessive data growth. Input validation and potentially sandboxing CRDT operations might be necessary depending on the chosen CRDT library and its properties.
*   **Denial of Service**:
    *   Peer discovery mechanisms (especially public ones like DHTs or rendezvous servers) could be susceptible to DoS attacks (e.g., flooding with fake announcements). Rate limiting, verifiable claims, and potentially small proofs-of-work could be considered for mitigation if this becomes an issue.
    *   Synchronization itself could be a vector if a malicious peer attempts to send excessive or malformed data.
*   **Privacy**:
    *   While E2EE protects data in transit, metadata (e.g., which devices/applications are online, frequency of sync) might still be observable by entities facilitating peer discovery if not carefully designed (e.g., a centralized rendezvous server).
    *   The content of the synchronized state itself should be considered sensitive.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
