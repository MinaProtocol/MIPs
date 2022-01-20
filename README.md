# Mina Improvement Proposals

## What is a MIP?

MIP stands for Mina Improvement Proposal. A MIP is a design document providing information to the Mina community, or describing a new feature for Mina or changes to the parameters of the protocol. The MIP should provide a concise technical specification of the feature and a rationale for its inclusion in the protocol. The MIP author is responsible for championing their proposal, building consensus within the community and incorporate feedback.

## MIP Rationale

The Mina Foundation intends MIPs to be the primary mechanism for proposing new features, collecting community input on an issue, and for documenting design decisions that have gone into Mina. It is intended to be a highly participatory and transparent process for complex decision making on changes to the Mina Protocol. In the long term, the vision for Mina is for more decentralized decision making, and during every iteration, we will work towards that goal.

## The MIP Process

This document is not intended to be the end state process for MIPs, but rather serves as a draft proposal to encourage discussion and feedback from commnuity and all stakeholders. The process will be evaluated and adapted on a regular basis, based on participant feedback and robustly testing in beta mode by running a series of decisions through the process.

The process described here is inspired by the long track record in this area by other projects, particurlarly Bitcoin and Ethereum.

##

## Types of MIP

A **Standards Track** MIP describes any change that affects most or all Mina protocol, such as a change to the network protocol, a change in consensus or block validity rules, to the zero-knowledge cryptography or any change in the accounts or rules around MINA. Standards Track MIPs, as well as serving as the design document, should include a reference implementation where appropriate.

**Process** discuss requirement for this type

**Meta** discuss requirement for this type

## MIP format and structure

Template and Github repo for the beta version of MIP to be created once the 'Idea' has been vetted

## MIP Process

![](https://github.com/MinaProtocol/MIPs/blob/process-image-update/assets/MIP%20Process%20Diagram.jpg?raw=true)

**Idea** - The MIP is put forward by the author(s) for community discussion and sentiment gathering on Mina Research and via Polling e.g., (Pol.is). The MIP is not tracked within the MIP repository.

**Draft** - The MIP has been formally drafted and submitted to the Github repository. The MIP is checked by the MIP Editors for compliance and is merged when it has been validated.

**Review** - Dev Teams, Community and Advisory Group provide feedback on the MIP

**Last call** This is the final review window for an MIP before moving to Finalization. The MIP Editors will assign Last Call status and set a review end date, typically 2 weeks later.

If this period results in the identification of high impact issues the MIP will be moved back to Review.

**Finalization** The proposal is deemed to have met all the appropriate criterias to be considered and the Dev Teams are in a position to incorporate the changes into the implementation schedule

**Inactive** A MIP that has been deemed to be inactive for a period of 1 month.

**Withdrawn** - The MIP Author(s) have withdrawn the proposed MIP. The proposal can no longer be resurrected using this MIP number. If the idea is pursued at later date it is considered a new proposal.

## MIP Workflow

### How to create a MIP

#### 1. Submit a MIP Idea to MinaResearch

If you have an idea you would like to propose, first search through previously proposed or discussed ideas in MinaResearch. If your idea is original or sufficiently distinct, start a new conversation thread to enable a discussion with the community on MinaResearch (see guidelines [here](https://forums.minaprotocol.com/t/how-to-engage-in-minaresearch-mip-process/4661). You should seek feedback and try to build informal consensus, as well as keep track of any critical feedback so that you can address it later.

For complex decisions, it is recommended that the Author start with gathering criteria and information from the community for complicated solutions. The Author is encouraged to poll the community at this point for the criteria that a good solution/proposal would fulfil, or to gather sentiment for the idea generally.

#### 2. Create a Draft MIP and submit it to the MIP repository

Once you've allowed enough time for feedback and a properly formatted (see below section) draft MIP has been written, it should be submitted to the MIPs GitHub repository as a pull request, accompanied by a summary of the discussion so far in the comment section, with links where possible.

A member of the MIP Working Group will assign a MIP number, select the appropriate MIP type (Standards, Informational, or Meta) and begin the validation process.

#### 3. Wait for approval and/or feedback from the MIP Editors

A MIP editor will assign a MIP number, select the appropriate MIP type and merge the pull request if the conditions have been met. Otherwise, the Author will receive a written notification and reasons why their draft was not merged.

## Roles

#### Authors

Responsible for vetting the idea, gathering sentiment, writing the MIP, championing MIP through process, gathering and incorporating community feedback including alternative and dissenting views. Multiple authors can contribute to one MIP, but one person must be identified to submit and edit it.

#### MIP Editors

The MIP Editors are here to ensure the MIP is complaint to the process at all times and that process is fluid and there is progression of MIPs through the process.  If you have questions regarding the MIP process, they can point you in the right direction.

#### Dev Teams

The team responsible for checking technical specifications and implementing the MIP once finalized.

#### Advisory Group

A group comprised of subject matter experts representative of impacted developers and community members. It is important that impacted stakeholders for a particular MIP are identified and included in the process. This is not a fixed group and impacted participants may vary for each MIP.

## What belongs in a successful MIP?

Each should have the following parts:

**Preamble** - RFC 822 style headers containing metadata about the MIP, including the MIP number, a short descriptive title (limited to a maximum of 44 characters), a description (limited to a maximum of 140 characters), and the author details. Irrespective of the category, the title and description should not include MIP number. See below for details.

**Abstract** - Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.

**Motivation** (optional) - A motivation section is critical for MIPs that want to change the Mina protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the MIP solves. MIP submissions without sufficient motivation may be rejected outright.

**Specification** - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Mina implementations.

**Rationale** - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other systems. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.

**Backwards Compatibility** - All MIP that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity.

The MIP must explain how the author proposes to deal with these incompatibilities. MIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

**Test Cases** - Test cases for an implementation are mandatory for MIPs.

**Reference Implementation** - An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification.

**Security Considerations** - All MIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life-cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. MIP submissions missing the “Security Considerations” section will be rejected. An MIP cannot proceed to status “Final” without a Security Considerations discussion deemed sufficient by the reviewers.

Copyright Waiver - All MIPs must be in the public domain.

## MIP Formats and Templates

MIPs should be written in markdown format.

## MIP Header Preamble

Each MIP must begin with an RFC 822 style header preamble, preceded and followed by three hyphens (---). This header is also termed “front matter” by Jekyll. The headers must appear in the following order.

mip: MIP number (this is determined by the MIP editor)

`title`: The MIP title is a few words, not a complete sentence

`description`: Description is one full (short) sentence

`author`: The list of the author’s or authors’ name(s) and/or username(s), or name(s) and email(s). Details are below.

`discussions-to`: The url pointing to the official discussion thread on MinaResearch

`status`: Draft, Review, Last Call, Finalization, Inactive, Withdrawn

`last-call-deadline`: The date last call period ends on (Optional field, only needed when status is Last Call)

`type`: One of Standards Track, Meta, or Informational

`category`: One of Core, Networking, Interface, or ERC (Optional field, only needed for Standards Track MIPs)

`created`: Date the MIP was created on

`requires`: MIP number(s) (Optional field)

`withdrawal-reason`: A sentence explaining why the MIP was withdrawn. (Optional field, only needed when status is Withdrawn)

Headers that permit lists must separate elements with commas.

Headers requiring dates will always do so in the format of ISO 8601 (yyyy-mm-dd).

author header
The author header lists the names, email addresses or usernames of the authors/owners of the MIP. Those who prefer anonymity may use a username only, or a first name and a username. The format of the author header value must be:

Random J. User <address@dom.ain>

or

Random J. User (@username)

if the email address or GitHub username is included, and

Random J. User

if the email address is not given.

It is not possible to use both an email and a GitHub username at the same time. If important to include both, one could include their name twice, once with the GitHub username, and once with the email.

At least one author must use a GitHub username, in order to get notified on change requests and have the capability to approve or reject them.

discussions-to header
While a MIP is a draft, a discussions-to header will indicate the URL where the MIP is being discussed.

The preferred discussion URL is a topic on MinaResearch. The URL cannot point to Github pull requests, any URL which is ephemeral, and any URL which can get locked over time (i.e. Reddit topics).

` type header`
The type header specifies the type of MIP: Standards Track, Meta, or Informational. If the track is Standards please include the subcategory (core, networking, interface, or ERC).

`category header`
The category header specifies the MIP’s category. This is required for standards-track MIPs only.

`created header`
The created header records the date that the MIP was assigned a number. Both headers should be in yyyy-mm-dd format, e.g. 2001-08-14.

`requires header`
MIPs may have a requires header, indicating the MIP numbers that this MIP depends on.
---
