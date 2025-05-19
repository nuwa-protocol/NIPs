---
nip: 0
title: NIP Process
author: Nuwa community
discussions-to: <URL of the NIPs discussion forum or GitHub issues>
status: Living
type: Meta
category: Process
created: 2025-05-13
---

## Abstract

This NIP defines the Nuwa Improvement Proposal (NIP) process. It outlines the stages a NIP goes through, the roles and responsibilities of participants, and the criteria for moving a NIP forward. The goal is to provide a clear, transparent, and community-driven framework for proposing, discussing, and integrating changes to the Nuwa Protocol and its ecosystem.

## Motivation

A well-defined process for managing improvement proposals is crucial for the healthy evolution of the Nuwa Protocol. It ensures that:
*   Proposals are thoroughly vetted by the community.
*   Decisions are made transparently.
*   The history of design choices is documented.
*   All stakeholders have an opportunity to contribute.
This NIP aims to establish such a process, drawing inspiration from successful improvement proposal systems in other open-source projects.

## Specification

### NIP Types

NIPs are categorized into the following types:

*   **Standards Track:** Describes any change that affects most or all Nuwa implementations, such as a change to the network protocol, a change in block or transaction validity rules, or any change or addition that affects the interoperability of applications using Nuwa. Standards Track NIPs consist of two parts, a design document and a reference implementation.
*   **Informational:** Describes a Nuwa design issue, or provides general guidelines or information to the Nuwa community, but does not propose a new feature. Informational NIPs do not necessarily represent a Nuwa community consensus or recommendation, so users and implementers are free to ignore Informational NIPs or follow their advice.
*   **Meta:** Describes a process surrounding Nuwa or proposes a change to (or an event in) a process. Meta NIPs, like Standards Track NIPs, consist of two parts, a design document and a reference implementation. They require community consensus and should not be ignored once adopted.

### NIP Statuses

A NIP progresses through the following statuses:

1.  **Draft:** The initial state of a NIP. A NIP is merged by NIP editors into the NIP repository when it is properly formatted and meets the basic requirements.
2.  **Proposed:** A NIP that is under active consideration by the community. This status indicates that the NIP is well-formed, the idea is clear, and it is ready for broader discussion and review.
3.  **Active:** A NIP that has been accepted for implementation. This means the core concepts have been approved, and a reference implementation is being developed or is already available.
4.  **Final:** A Standards Track NIP that has been implemented and adopted by the community. For a NIP to reach "Final" status, it must have a reference implementation and have been adopted by major clients or components of the Nuwa ecosystem.
5.  **Living:** A NIP that is continuously updated and maintained (e.g., NIP-0 itself, or NIPs defining core interfaces that may evolve).
6.  **Stagnant:** A NIP that has been inactive for a significant period (e.g., 6+ months) without progressing. It may be moved back to Draft or Withdrawn.
7.  **Withdrawn:** A NIP that has been withdrawn by its author(s).
8.  **Rejected:** A NIP that has been formally rejected by the NIP editors or community consensus after discussion.

### NIP Workflow

1.  **Idea:** Propose your idea and discuss it with the Nuwa community (e.g., on a designated forum, mailing list, or GitHub discussions). This helps gauge interest and gather initial feedback.
2.  **Drafting:**
    *   Fork the NIP repository.
    *   Copy `nip-template.md` to `NIPs/nip-XXXX.md` (where `XXXX` is a number you will be assigned later).
    *   Fill in the NIP. Pay attention to the NIP header fields.
    *   Ensure the NIP is well-motivated, clearly specified, and considers backwards compatibility and security.
3.  **Submission:**
    *   Submit a Pull Request (PR) to the NIPs repository.
    *   The NIP editors will review the PR for formatting, clarity, and completeness. They will assign a NIP number.
    *   The NIP editors may request revisions before merging the PR.
    *   Once merged, the NIP is in **Draft** status.
4.  **Discussion & Review:**
    *   The NIP author (or a champion) should actively solicit feedback and lead discussions.
    *   The NIP may be updated based on community feedback.
    *   If the NIP gains traction and addresses feedback appropriately, it can be moved to **Proposed** status by the NIP editors.
5.  **Implementation (for Standards Track & relevant Meta NIPs):**
    *   A reference implementation should be developed.
    *   Test cases are crucial.
    *   Once a stable implementation exists and community consensus is strong, the NIP can be moved to **Active**.
6.  **Adoption & Finalization (for Standards Track):**
    *   For a NIP to become **Final**, it must be adopted by the Nuwa ecosystem and demonstrate real-world use.
    *   This typically involves inclusion in official releases or widespread usage.
7.  **Maintenance (for Living NIPs):**
    *   Living NIPs are updated as needed to reflect the current state of the process or standard they define.

### NIP Editors

NIP Editors are individuals responsible for the administrative and editorial aspects of the NIP process. Their responsibilities include:
*   Assigning NIP numbers and merging NIPs.
*   Ensuring NIPs are well-formatted and readable.
*   Facilitating discussions and ensuring the process is followed.
*   Updating NIP statuses based on community consensus and NIP progress.
*   Maintaining the NIP repository.

The current NIP editors are:
*   [List NIP Editor Names/GitHub Handles Here]

### NIP Template

All NIPs should follow the structure defined in `nip-template.md`. Key sections include:
*   Preamble (header with metadata)
*   Abstract
*   Motivation
*   Specification
*   Rationale
*   Backwards Compatibility
*   Test Cases
*   Reference Implementation
*   Security Considerations
*   Copyright

## Rationale

This NIP process is designed to be:
*   **Transparent:** All discussions and decisions are public.
*   **Community-driven:** Success depends on community participation and consensus.
*   **Structured:** Provides a clear path for proposals.
*   **Documented:** Creates a historical record of design choices.

The roles and statuses are adapted from common practices in other successful open-source projects (e.g., Ethereum EIPs, Bitcoin BIPs, Python PEPs) to provide a familiar and effective framework.

## Backwards Compatibility

This NIP defines a new process and does not directly impact existing protocol components. Therefore, no backwards compatibility issues are anticipated.

## Security Considerations

Security considerations for the NIP process itself include:
*   **Authenticity of NIPs:** GitHub PRs and commit histories provide a record of authorship and changes.
*   **Integrity of the process:** NIP editors play a crucial role in maintaining the integrity of the NIP repository and process.
*   **Community Review:** Broad community review is the primary defense against malicious or flawed proposals.

Individual NIPs must address their own specific security implications as per the `nip-template.md`.

## Copyright

Copyright and related rights for this NIP are waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
