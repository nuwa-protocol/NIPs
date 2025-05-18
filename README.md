# NIPs - Nuwa Improvement Proposals

Welcome to the Nuwa Improvement Proposal (NIP) repository. NIPs are the primary way to propose new features, collect community input on issues, and document design decisions for the Nuwa Protocol and its ecosystem.

## Process

The NIP process is intended to provide a consistent and controlled path for new features, standards, and protocols to be proposed, debated, and adopted. For details on the NIP process, including how to submit a NIP, please refer to [NIP-0: NIP Process](./NIPs/nip-0.md) and use the [nip-template.md](./nip-template.md) for submissions.

## Current NIPs

Below is a list of current Nuwa Improvement Proposals. Each NIP has a status indicating its current stage in the process (e.g., Draft, Proposed, Accepted, Final, Rejected).

| NIP Number | Title                                                      | Summary                                                                                                                                                              | Status |
|------------|------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|
| [0](./NIPs/nip-0.md)  | NIP Process                                                | Defines the Nuwa Improvement Proposal (NIP) process, outlining stages, roles, and criteria for proposing, discussing, and integrating changes.                 | Living |
| [1](./NIPs/nip-1.md)  | Agent Single DID Multi-Key Model                           | Proposes a decentralized identity model for Agents using a single master DID to manage multiple operational keys and standardizes service discovery.             | Draft  |
| [2](./NIPs/nip-2.md)  | DID-Based Authentication Protocol                          | Defines a general-purpose authentication protocol based on NIP-1 DIDs for proving identity and message integrity.                                                | Draft  |
| [3](./NIPs/nip-3.md)  | Custodian Delegated Control Protocol                       | Allows Web2 users to create Agent DIDs via a Custodian service without directly managing crypto wallets.                                                             | Draft  |
| [4](./NIPs/nip-4.md)  | A2A Payment Channel Protocol                               | Defines a protocol for A2A payment channels for efficient, off-chain micropayments, especially for protocol-level fees.                                          | Draft  |
| [5](./NIPs/nip-5.md)  | Fiat Proxy Service for AI Agents                           | Enables AI Agents to interact with fiat payment systems through a standardized, discoverable Fiat Proxy service, authenticated via NIP-2.                        | Draft  |
| [6](./NIPs/nip-6.md)  | Unified Agent State Synchronization                        | Defines a P2P protocol using CRDTs to synchronize an agent's state across multiple devices/applications, ensuring consistency and offline support.                 | Draft  |
| [7](./NIPs/nip-7.md)  | Agent Capability Protocol — Capability Package Specification | Specifies the ".acp.yaml" format for modular Agent Capability Packages, bundling schema, prompts, tools, and metadata.                                          | Draft  |
| [8](./NIPs/nip-8.md)  | Agent State Model (ASM): A Unified State Management Framework | Extends JSON-Schema with `x-asm` annotations for describing, merging (CRDTs), querying (ASM-QL), and managing persistent agent states.                        | Draft  |
| [9](./NIPs/nip-9.md)  | Agent LLM Gateway Protocol                                 | Defines a protocol for AI Agents to access LLM services through verifiable, identity-bound gateways with standardized request/response and payment integration. | Draft  |
| [10](./NIPs/nip-10.md) | MCP Identity Authentication and Payment Extension Protocol   | Extends the Model Context Protocol (MCP) with standard mechanisms for identity authentication (NIP-1/2) and payment (NIP-4).                                  | Draft  |
| [11](./NIPs/nip-11.md) | A2A Agent Service Payment Protocol                         | Defines a protocol for on-demand payment for application-level services provided by AI Agents to User Agents, using direct pre-payment or NIP-4 channels.      | Draft  |

## How to Contribute

Contributions to NIPs are welcome! Please read [NIP-0: NIP Process](./NIPs/nip-0.md) for details on how to contribute. New proposals should use the [nip-template.md](./nip-template.md).

Discussions related to NIPs typically occur in the GitHub issues or discussion forums linked within each NIP document.

## Copyright

All NIPs are in the public domain. Copyright and related rights are waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
