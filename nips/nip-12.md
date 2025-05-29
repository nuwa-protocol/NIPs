---
nip: 12
title: Opentimestamps Attestation Protocol
author: ZHANG, HENGMING <22887031+ghpZ54K8ZRwU62zGVSePPs97yAv9swuAY0mVDR4@users.noreply.github.com>
discussions-to: <URL of the discussion thread, e.g., a GitHub issue or forum post>
status: Draft
type: Informational
category: Interface
created: 2025-05-29
updated: <date of last significant update, in ISO 8601 (yyyy-mm-dd) format> // Optional, but recommended
requires: https://github.com/opentimestamps
replaces: <NIP number(s)>
superseded-by: <NIP number(s)>
---

## Abstract

This NIP provides general interface for timestamping transactions using Opentimestamps where ordinary timestamp interfaces are not provided upon creation of transactions.

## Motivation

<!-- This section is critical. It should clearly explain:
*   The problem or opportunity the NIP addresses.
*   Why the existing protocol/system is inadequate.
*   The benefits of adopting this NIP.
NIP submissions without sufficient motivation may be rejected. -->

## Specification

<!-- The technical specification should describe the syntax, semantics, and behavior of any new feature or change. It must be detailed enough to allow for independent, interoperable implementations.
This section may include:
*   Definitions of new terms.
*   Data structures and formats.
*   Algorithm descriptions.
*   API specifications.
*   State machine diagrams, if applicable. -->

## Rationale

<!-- This section explains the "why" behind the design choices in the "Specification" section. It should:
*   Describe alternative designs that were considered and why they were not chosen.
*   Discuss related work or prior art.
*   Provide evidence of community consensus or address significant objections raised during discussions. -->

The proofs generated using above method may be encoded using formal encoding schemes, e.g. base64, to reduce its size produced by the above method. When doing this way, the encoded proofs must be the same with the originals that exist in a .ots file.

In a stand of verifying the generated proofs, a third party service provider, e.g. a blockchain node, a blockchain oracle, or a http server, may involve in the verification process.

## Backwards Compatibility

<!-- All NIPs that introduce changes must address backwards compatibility.
*   If the NIP is fully backwards compatible, explain how.
*   If there are incompatibilities, describe them, their severity, and how the author proposes to manage them (e.g., migration paths, versioning).
NIPs with insufficient backwards compatibility considerations may be rejected. -->

## Test Cases

<!-- Test cases are highly recommended for all NIPs, and mandatory for NIPs proposing changes to consensus-critical or core protocol components.
*   Provide concrete examples and expected outcomes.
*   Link to test suites if available. -->

## Reference Implementation

<!-- A reference implementation is highly recommended, and mandatory for NIPs proposing changes to consensus-critical or core protocol components.
*   Link to the reference implementation (e.g., a GitHub repository or branch). -->

## Security Considerations

<!-- All NIPs must include a section discussing security implications. This should cover:
*   Potential vulnerabilities introduced by the NIP.
*   How the NIP mitigates these vulnerabilities.
*   Any new attack surfaces.
*   Security-relevant design decisions.
NIPs without adequate security considerations will be rejected and cannot reach "Final" status. -->

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
