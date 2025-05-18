---
nip: 5
title: Fiat-Proxy Top-Up Protocol (FPTP)
author: jolestar(@jolestar)
discussions-to: <URL of the discussion thread> # To be filled
status: Draft
type: Standards Track
category: Core
created: 2025-05-16
updated: 2025-05-18
requires: NIP-1, NIP-2, NIP-3
--- 

## Abstract

NIP-5 standardises **Fiat-Proxy Top-Up** for AI Agents across different blockchain environments. It defines how Fiat Proxy services declare their offerings within their DID Documents (as per NIP-1's service discovery model) and the on-chain transaction for executing top-ups.

*   **Service Declaration**: Fiat Proxy services (identified by a DID compliant with NIP-1) declare their supported tokens, payment methods, and fees within the `service` section of their DID document, using a standardized `type` of `"FiatProxyServiceNIP5"`.
*   **User Interaction**: Web2 users pay the proxy off-chain (e.g., card, Apple Pay). The proxy then executes an on-chain transaction to transfer the specified token to the user’s Agent DID on the target blockchain.
*   **Discovery**: Front-ends or Agents discover Fiat Proxy services by resolving their DIDs and parsing the `service` information in the DID document.

The protocol standardises **service discovery via DID documents** and the **`fiatTopUp`() on-chain transaction**, leveraging NIP-1 for identity and NIP-3 for potential payment channel interactions post-top-up.

## Motivation

| Need                               | Current gap                                      | FPTP Solution (DID-based)                                     |
| ---------------------------------- | ------------------------------------------------ | ------------------------------------------------------------- |
| Web2 users cannot hold crypto      | Custodian covers DID control, but funding complex | One-click “Pay 5 USD → +5 RGas” via discovered Fiat Proxy     |
| Ecosystem neutrality & Discovery   | Multiple, siloed on-ramp providers/registries    | Unified service discovery via DID; competitive fee market     |
| Proof & audit                      | Off-chain receipt hard to verify                 | `fiatTopUp` tx emits event with hashed receipt; DID provides provenance |
| Service Registration Complexity    | Separate registry for each service type          | Standardized service declaration within provider's DID document |

## Specification

### Terminology

| Term           | Meaning                                                                                                                                                                                             |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Fiat Proxy** | Licensed off-chain service, identified by a DID (NIP-1), that accepts fiat and transfers a specified on-chain gas/utility token (e.g., RGas on Rooch) on the blockchain where it offers the service. |
| **Service Declaration (in DID Document)** | A Fiat Proxy declares its service offerings as an entry in the `service` array of its DID Document. This entry has a `type` of `"FiatProxyServiceNIP5"` and contains metadata about supported tokens, fees, etc. | 
| **Top-Up Tx**  | `fiatTopUp(recipient_did, amount_token, token_symbol, receipt_hash)` on-chain call executed by a registered Fiat Proxy (identified by its DID) on the chain where the token is being transferred. `token_symbol` specifies the token. | 

### Fiat Proxy Service Declaration in DID Document

A Fiat Proxy service provider MUST declare its offerings within the `service` array of its DID document. Each offering for a specific token on a specific chain (if the proxy operates on multiple chains with the same DID) can be a distinct entry, or grouped if the `serviceEndpoint` is the same.

**Service Entry Properties:**

*   `id`: A unique identifier for this service entry within the DID document (e.g., `did:example:fiatproxy#service-1`).
*   `type`: MUST be `"FiatProxyServiceNIP5"`.
*   `serviceEndpoint`: The HTTPS base URL for the Fiat Proxy's payment processing API for this offering.
*   `name`: (Optional) A human-readable display name for the Fiat Proxy service (e.g., "ACME Fiat Top-Up").
*   `description`: (Optional) A brief description of the service offering.
*   `supportedTokens`: An array of objects, where each object describes a token the proxy supports on a specific blockchain.
    *   `blockchainDidMethod`: (Optional, if ambiguous) The DID method prefix for the target blockchain (e.g., `"rooch"`, `"ethr"`). Helps clarify if the proxy supports tokens on multiple chains under the same DID.
    *   `chainId`: (Optional) The specific chain ID if the `blockchainDidMethod` is not sufficient (e.g., for EVM chains).
    *   `tokenSymbol`: The symbol of the token being provided (e.g., `"RGas"`, `"USDC"`). This symbol is interpreted in the context of the specified blockchain.
    *   `feeBps`: Basis points charged as a fee on the exchanged amount for this token (e.g., `50` for 0.5%).
    *   `payinMethods`: An array of objects detailing supported pay-in methods for this token.
        *   `code`: A numeric code from the `payin_codes` enumeration (see below).
        *   `currency`: ISO 4217 currency code for the fiat payment (e.g., `"USD"`, `"EUR"`).
    *   `minTopUpAmount`: (Optional) Minimum top-up amount in the fiat currency.
    *   `maxTopUpAmount`: (Optional) Maximum top-up amount in the fiat currency.
*   `stakeInfo`: (Optional) Information about any stake the proxy has locked, potentially verifiable on-chain.
    *   `stakeAmount`: Amount of stake.
    *   `stakeTokenSymbol`: Symbol of the staked token.
    *   `stakeChainId`: Chain where the stake is held.
    *   `proofUrl`: URL to a verifiable proof of stake.
*   `active`: (Optional, default: `true`) Boolean indicating if this specific service offering is active.

**Example `service` entry in a Fiat Proxy's DID Document:**

```json
{
  // ... other DID document properties ...
  "service": [
    {
      "id": "did:example:fiatproxy#rooch-rgas",
      "type": "FiatProxyServiceNIP5",
      "serviceEndpoint": "https://proxy.example.com/api/v1/payment",
      "name": "Example Proxy RGas Top-Up (Rooch)",
      "supportedTokens": [
        {
          "blockchainDidMethod": "rooch",
          "tokenSymbol": "RGas",
          "feeBps": 50, // 0.5%
          "payinMethods": [
            { "code": 1, "currency": "USD" }, // Credit/Debit Card
            { "code": 2, "currency": "USD" }  // Apple Pay
          ],
          "minTopUpAmount": "5.00",
          "maxTopUpAmount": "1000.00"
        },
        {
          "blockchainDidMethod": "ethr",
          "chainId": "1", // Ethereum Mainnet
          "tokenSymbol": "USDC",
          "feeBps": 75, // 0.75%
          "payinMethods": [
            { "code": 1, "currency": "EUR" } // Credit/Debit Card
          ]
        }
      ],
      "active": true
    }
    // ... other services offered by this DID ...
  ]
}
```

**`payin_codes` enumeration:**

| Code  | Pay-in method                |
| ----- | ---------------------------- |
| `1`   | Credit / Debit Card (e.g., Stripe) |
| `2`   | Apple Pay                    |
| `3`   | Google Pay                   |
| `4`   | Alipay                       |
| `5`   | WeChat Pay                   |
| `6`   | SEPA Transfer                |
| `7`   | PayPal                       |
| `10+` | *reserved*                   |

### Top-Up Transaction

_The following function signature is illustrative and would be adapted for the specific smart contract language of the target blockchain. This transaction occurs on the chain where the token is being transferred._

```move
// Example in Move-like pseudocode
public entry fun fiatTopUp(
    // The signer of this transaction is the Fiat Proxy.
    // Their DID is implicitly known to the contract (e.g., derived from the on-chain account signing the tx).
    // The contract MUST verify that the signer's DID corresponds to a known/authorized Fiat Proxy.
    recipient_did: String,     // The Agent DID (NIP-1) to receive the token
    amount_token: u64,          // Amount of the specific on-chain token to transfer
    token_symbol: String,       // Symbol of the token being transferred (e.g., "RGas", "USDC"), for clarity and matching service declaration
    receipt_hash: vector<u8>    // sha256 of off-chain payment proof/receipt details
) // `acquires` clause would depend on specific chain/language implementation
```

*   **Access control**: The smart contract executing `fiatTopUp` MUST verify that the transaction signer (the Fiat Proxy's on-chain account) is authorized. This authorization might be based on:
    *   The signer's on-chain account being linked to a DID that has declared `"FiatProxyServiceNIP5"` in its DID document (requiring off-chain DID resolution and verification by the contract or an oracle).
    *   A simpler on-chain allowlist of Fiat Proxy accounts, managed by governance (less decentralized but simpler for initial implementations).
*   The contract must verify that the proxy is authorized to provide the specified `token_symbol`.
*   The contract must transfer `amount_token` of the specified `token_symbol` from the proxy's funded pool (or mint, if applicable based on chain capabilities and proxy permissions for that token) to the on-chain account associated with `recipient_did`.
*   An event `FiatTopUp(recipient_did, amount_token, token_symbol, proxy_did_of_caller, receipt_hash)` MUST be emitted.

> **Funding the pool**: Each Fiat Proxy pre-loads the specific on-chain tokens it supports (e.g., RGas, USDC) into its own managed pool or account on this blockchain, according to the rules of this chain and token standard.

### Client & Runtime Behaviour

1.  **Discovery**:
    *   A front-end or Agent obtains the DID of one or more potential Fiat Proxy service providers.
    *   It resolves their DID documents (as per NIP-1).
    *   It parses the `service` array, looking for entries with `type: "FiatProxyServiceNIP5"`.
    *   It filters these services based on desired `tokenSymbol`, `blockchainDidMethod`/`chainId`, supported `payinMethods`, `currency`, and `feeBps`.
    *   The user selects a preferred proxy and offering.
2.  **Pay-in**:
    *   The user interacts with the selected Fiat Proxy's `serviceEndpoint` (e.g., a web page or API) to complete the fiat payment off-chain.
    *   The proxy generates a `receipt_hash` from the payment details.
3.  **On-chain transfer**:
    *   The Fiat Proxy (identified by its DID and signing with an associated on-chain key/account) calls the `fiatTopUp()` function on the target blockchain for the agreed `token_symbol` and `amount_token`, including the `receipt_hash`.
4.  **Agent Runtime**:
    *   The Agent (or its custodian) monitors the blockchain for `FiatTopUp` events where its `recipient_did` is specified.
    *   Upon detecting a successful top-up for a specific `token_symbol`:
        *   The Agent's balance for that `token_symbol` (managed by the Agent or its custodian on that chain) is updated.
        *   The Agent can then use these funds, e.g., for NIP-3 payment channels.
5.  The updated token balance is reflected in the Agent's UI or internal state.

## Rationale

*   **DID-based Service Discovery**: Using NIP-1's DID document `service` entries for discovery simplifies the ecosystem by removing the need for a separate, chain-specific registry contract for Fiat Proxies. It promotes a unified discovery mechanism for all NIP-defined services.
*   **Flexibility**: Fiat Proxies can update their offerings (tokens, fees, payment methods) by updating their DID documents, without needing on-chain transactions for registration changes (unless stake or on-chain allowlists are involved).
*   **Decentralization**: Relies on the decentralization of the chosen DID method for service information.
*   **`receipt_hash`**: Provides an auditable link between the off-chain fiat payment and the on-chain token transfer.
*   **On-chain `fiatTopUp` function**: Remains essential for the actual token transfer and event emission, providing an on-chain record.

## Backwards Compatibility

*   This NIP revises the discovery mechanism previously envisioned for Fiat Proxies (which might have implied a dedicated on-chain registry similar to NIP-5's original illustrative contract). It now aligns with the unified service discovery model in NIP-1.
*   Requires NIP-1 for the Fiat Proxy's identity and service declaration.
*   Requires NIP-3 if topped-up funds are to be used with NIP-3 payment channels.
*   Agents and front-ends previously built for a dedicated NIP-5 registry would need to be updated to use DID-based discovery.

## Test Cases

1.  **Fiat Proxy DID Document**: Create a valid DID document for a Fiat Proxy declaring `FiatProxyServiceNIP5` with multiple `supportedTokens`.
2.  **Client Discovery**: Simulate a client resolving the proxy's DID and successfully parsing its service offerings, filtering by `tokenSymbol` and `payinMethods`.
3.  **Successful `fiatTopUp`**: Proxy calls `fiatTopUp`, event is emitted, recipient's (conceptual) balance is updated.
4.  **Failed `fiatTopUp` (Unauthorized Proxy)**: Call to `fiatTopUp` from an unauthorized account is rejected.
5.  **Service Update**: Proxy updates its `feeBps` in its DID document; client discovers the new fee.

## Reference Implementation

*   A link to an example DID document for a Fiat Proxy.
*   Illustrative (pseudocode or actual) implementation of the `fiatTopUp` function for a target blockchain.

## Security Considerations

*   **DID Document Integrity**: The security of service discovery relies on the integrity and correct resolution of the Fiat Proxy's DID document. Compromise of the proxy's DID control could lead to malicious service declarations.
*   **Off-Chain Payment Security**: The security of the fiat payment process itself is handled by the Fiat Proxy off-chain and is subject to traditional web payment security standards (e.g., PCI-DSS). This NIP does not cover these aspects.
*   **`receipt_hash` Uniqueness**: Proxies should ensure `receipt_hash` values are unique or at least strongly collision-resistant to prevent replay issues if the hash is used for any off-chain reconciliation.
*   **On-chain `fiatTopUp` Access Control**: Robust verification of the caller's authority to act as a Fiat Proxy for the given token is crucial for the `fiatTopUp` function.
*   **Proxy Solvency**: Users rely on the proxy to execute the on-chain transfer after receiving fiat. While `stakeInfo` in the DID can provide some assurance, it's not a full guarantee against a malicious or insolvent proxy.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).