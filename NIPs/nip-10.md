---
nip: 10
title: MCP Identity Authentication and Payment Extension Protocol
author: jolestar(@jolestar)
discussions-to: <URL of the discussion thread, e.g., a GitHub issue or forum post>
status: Draft
type: Standards Track
category: Core
created: 2025-05-13
updated: 2025-05-18
requires:
  - NIP-1
  - NIP-2
  - NIP-4
---

## Abstract

This NIP proposes extensions to the Model Context Protocol (MCP) to include standard mechanisms for identity authentication and payment. This will enable MCP services to be commercialized securely, protect resources, and allow existing MCP clients and servers to more easily integrate these features.

## Motivation

*   The current Model Context Protocol (MCP) primarily focuses on the interaction between models and tools, lacking standard mechanisms for identity authentication and payment.
*   To enable MCP services to be commercialized securely and to protect resources, it is necessary to introduce these capabilities at the protocol level.
*   By directly extending MCP, rather than relying entirely on A2A encapsulation, existing MCP clients and servers can more easily integrate these features, thus better fitting into the MCP ecosystem.

## Specification

This NIP extends MCP with identity authentication and payment capabilities.

### Dependencies

*   **NIP-1: Agent Single DID Multi-Key Model**: Used to define the identity identifiers for MCP clients and servers.
*   **(Conceptual Dependency) NIP-2: DID-Based A2A Authentication**: The authentication principles from NIP-2 (e.g., using DID signatures to verify request origin and integrity) will be adopted and applied to MCP messages.
*   **(Conceptual Dependency) NIP-4: A2A Agent Service Payment Protocol**: The payment flows and message types from NIP-4 will be adapted for payment interactions within MCP.

### MCP Identity Authentication Extension

*   **3.1. Identity Identifiers and Service Declaration**:
    *   Both MCP clients and MCP servers should possess a DID compliant with NIP-1 specifications.
    *   MCP servers must declare their service endpoint(s) and associated DID within their DID Document, following the service endpoint definition guidelines specified in NIP-1. This allows clients to discover MCP services and verify their authenticity.
    *   The service type for MCP services, to be used in the `service.type` field of the DID document, should be `"MCPServiceV1"`. Other service-specific metadata relevant to MCP may also be included as defined by the broader MCP specification.
*   **3.2. Request Signing**:
    *   MCP clients, when initiating tool call requests, must sign critical parts of the request according to the procedures outlined in NIP-2. 
    *   The `contentToSign` (as defined in NIP-2) for an MCP request comprises the canonicalized MCP request payload. This payload **must** include a `timestamp` and a `nonce` as top-level fields for replay protection, adhering to NIP-2 requirements. The specific canonicalization method for the MCP request payload should be defined by the MCP standard.
    *   The `domainSeparator` for NIP-2 signatures in the context of NIP-10 should be `"MCP_NIP10_AUTH_V1:"`.
    *   Signature information (including client DID, `key_id` used, and signature value) can be transmitted via MCP request metadata or as part of the request body. 
    *   When MCP is layered over HTTP, the signature information should be transmitted using the HTTP Header mechanism defined in NIP-2 (e.g., `Authorization: DIDAuthV1 <credentials>`). This NIP does not define a new signature header.
*   **3.3. Server-side Verification**:
    *   Upon receiving a request, the MCP server parses the signature information to obtain the client's DID, following NIP-2.
    *   It resolves the client's DID to retrieve its DID document and the corresponding public key.
    *   It verifies the signature's validity, ensuring the request is untampered and originates from a legitimate client.
    *   (Optional) Access control can be enforced based on the client's DID.

### MCP Payment Extension

*   **4.1. Service Pricing and Quotation**:
    *   MCP servers should be able to declare whether their tools (or specific operations) require payment and their pricing strategy. This can be achieved by extending MCP's tool descriptions.
    *   **MCP Message Extension - `ToolQuotationRequest`**: Client requests a quotation for a specific tool call from the server.
        *   Includes: `tool_id`, `tool_input` (for precise server-side quotation).
    *   **MCP Message Extension - `ToolQuotationResponse`**: Server replies with the quotation.
        *   Includes: `quotation_id`, `price` (amount, currency unit), `payment_instructions`. 
        *   `payment_instructions` should detail the required payment method (e.g., direct on-chain transfer to a specified address, or indication if an NIP-4 payment channel can be used for settlement). If direct payment, it includes necessary details like `chain_id`, `asset_id`, `recipient_address`.
*   **4.2. Payment Confirmation and Service Execution (Pre-payment Model)**:
    *   The client completes the payment according to the `payment_instructions` in the `ToolQuotationResponse`.
    *   **MCP Message Extension - `PaymentConfirmation`**: Client sends payment confirmation to the server.
        *   Includes: `quotation_id`, `payment_proof`. 
        *   `payment_proof` contains evidence of the payment (e.g., for a direct on-chain payment, this would include `transaction_hash`, `block_number`). If an NIP-4 channel is used by agreement after quotation, this message might be superseded or augmented by NIP-4 channel update messages.
    *   The server verifies the payment proof.
    *   Upon successful verification, the server executes the actual MCP tool call and returns the result.
*   **4.3. State Channel Payment Model (Optional, for frequent/streaming interactions)**:
    *   For frequent or streaming interactions, MCP can leverage the state channel payment mechanisms defined in NIP-4. This includes protocols for channel establishment, funding, state updates, and closure.
    *   Implementations should refer to NIP-4 for specific A2A message definitions and flows for channel management. 
    *   If MCP interactions are layered over HTTP, the `X-Payment-Channel-Data` header mechanism defined in NIP-4 should be used to convey channel payment information (proposals and confirmations) within HTTP requests and responses.
    *   Once a payment channel is established according to NIP-4, subsequent MCP tool calls can indicate billing via this specific payment channel in their requests, using a mechanism compatible with both MCP and NIP-4 (e.g., by referencing a channel ID in MCP request metadata or as part of the `X-Payment-Channel-Data` header if over HTTP).

### MCP Protocol Modifications and Message Definitions

*   Detail the structure of the newly introduced MCP message types (e.g., `ToolQuotationRequest`, `PaymentConfirmation`).
*   Specify how existing MCP messages (like tool call requests) need to be extended to carry authentication information or payment intent.
*   Consider how MCP services can discover if a client supports and expects to use these extensions.

## Rationale

This section explains the "why" behind the design choices in the "Specification" section. 
*   Integrating authentication and payment directly into MCP, rather than relying solely on an A2A wrapper, simplifies adoption for existing MCP implementations.
*   The choice of DID-based authentication (NIP-1, NIP-2) provides a decentralized and robust identity layer.
*   Adapting payment flows from NIP-4 ensures consistency within the broader Nuwa ecosystem.
*   Alternative designs, such as using OAuth2 for authentication or a completely separate payment sidecar protocol, were considered but deemed to add more complexity for this specific MCP extension.

## Backwards Compatibility

*   Existing MCP clients and servers will need to be upgraded to support these new authentication and payment fields/messages.
*   Servers implementing these extensions should clearly signal their capabilities.
*   Maintain backward compatibility by allowing clients that do not support these extensions to continue interacting with servers offering only free tools or tools that do not require authentication. Servers can choose to reject unauthenticated/unpaid requests for protected resources.
*   New message types (`ToolQuotationRequest`, etc.) are additive and will be ignored by older clients/servers. Optional fields in existing messages for signature/payment info should not break parsing for implementations unaware of them.

## Test Cases

Test cases are highly recommended for all NIPs, and mandatory for NIPs proposing changes to consensus-critical or core protocol components.
*   **Test Case 1: Successful Authenticated Tool Call (No Payment)**
    *   Client signs request with its DID.
    *   Server verifies signature, executes tool, returns result.
*   **Test Case 2: Failed Authentication (Invalid Signature)**
    *   Client sends request with an invalid signature.
    *   Server rejects request with an authentication error.
*   **Test Case 3: Successful Paid Tool Call (Pre-payment)**
    *   Client requests quotation.
    *   Server provides quotation.
    *   Client makes payment, sends confirmation.
    *   Server verifies payment, executes tool, returns result.
*   **Test Case 4: Failed Payment (Payment Verification Fails)**
    *   Client sends payment confirmation with invalid proof.
    *   Server rejects tool execution due to payment failure.
*   **Test Case 5: Interaction with non-supporting server/client**
    *   Client supporting auth/payment calls a server that does not. Server processes as a normal MCP call if the tool is free/public.
    *   Client not supporting auth/payment calls a server that requires it for a specific tool. Server returns an error indicating authentication/payment is required.

## Reference Implementation


## Security Considerations

All NIPs must include a section discussing security implications.
*   **Authentication Security**:
    *   Relies on the security of the underlying DID infrastructure (NIP-1) and signature schemes (NIP-2). Key management by clients and servers is critical.
    *   Protection against replay attacks for signed requests (e.g., using timestamps and nonces) must be clearly defined in the signature scheme.
*   **Payment Security**:
    *   Relies on the security of the payment mechanisms defined in NIP-4.
    *   Risk of double-spending or payment disputes needs to be handled by the referenced payment protocol.
    *   Quotation and payment confirmation messages must be protected against tampering.
*   **Data Privacy**:
    *   Client DIDs will be exposed to servers. Implications for privacy should be considered.
*   **Denial of Service**:
    *   Servers need to protect against resource exhaustion from unauthenticated clients making quotation requests or attempting signature verification. Rate limiting or other DoS protection mechanisms may be needed.
*   **New Attack Surfaces**:
    *   The new message types and signature verification logic introduce new potential attack surfaces that must be carefully implemented and tested.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
