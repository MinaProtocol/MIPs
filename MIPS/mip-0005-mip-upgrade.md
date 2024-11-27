---
mip: MIP5
title: MIP upgrade
description: This MIP adds several improvements to the MIP process
authors: Joaquin López (@criptowaco)
discussions-to: https://forums.minaprotocol.com/t/upgrading-the-mip-process/6464 
status: Draft
type: Process
category: Governance
created: 2024-11-04
---

## Abstract

The Next Steps blog post (https://minaprotocol.com/blog/next-steps-mina-protocol-governance) outlines a strategy for Mina Protocol’s governance, emphasizing an experimental and incremental approach that allows the community to iteratively test, improve, and expand decision-making processes. As the long-term vision is to enable substantial community participation in protocol decision-making, this document details how we can gradually increase that involvement, especially considering the current limited expertise and experience, which we expect will grow over time.

We propose the following initial changes to the MIP process:

- Creating new categories for future MIPs: engineering, economics, and governance.
- Introducing new roles and responsibilities to review MIPs and assist MIP authors throughout the process.
- Enhancing transparency by introducing mass deliberation tools and public debates.
- Establishing clear guidelines for implementing future MIPs.

These proposed changes were identified after collecting feedback from the community, including Mina Research Forum and Discord related channels, and learning from the experience of previous MIPs. This feedback was informed by the blog post, “Next Steps for Mina Protocol’s Governance” published on May 2, 2024, and discussed with the community in several town halls, fireside chats, and surveys.

The plan is to incorporate community feedback on this document and propose these changes in a new MIP, which would require approval through an on-chain vote. The expectation is that these changes will empower community members to present and discuss proposals, making Mina Protocol’s governance system more transparent, effective, and aligned with the community.


## Motivation

The Mina Improvement Proposal (MIP) process is the main mechanism for protocol decision making and documenting design decisions that have gone into Mina. The process provides a structured, transparent and participatory way for community members to propose, discuss and agree (or not) to implement these changes.

The MIP process is inspired by the successful governance models of projects like Bitcoin and Ethereum. It’s also inspired by the Swiss democratic system (https://www.eda.admin.ch/aboutswitzerland/en/home/politik-geschichte/politisches-system/direkte-demokratie.html), embracing the principles of decentralization and direct democracy. Like the Swiss model, which empowers citizens to propose and vote on initiatives, the MIP process allows community members to propose and vote on major changes to the Mina Protocol, this ensures that decision-making power is distributed and community-driven, fostering a robust and participatory ecosystem.

Over the last few years, Mina’s ecosystem has grown substantially and an increasing number of companies and individuals are joining and contributing to the ecosystem. Consequently, there is also a rising need to develop Mina’s governance to support this growth and help to realize Mina’s vision of a future powered by participants so that its decision making processes remain effective and aligned with the wishes of the community.


## Rationale

This challenge stems from the current MIP process, where authorship has been limited and relies on a few organizations and individuals perhaps due to the limited expertise in the community. The informal nature of the current process allows anyone to provide comments on GitHub but there is a lack of structured process for experts to review MIPs that could benefit the deliberation of other community members. For example, non-expert community members who vote on proposals often have to evaluate them without fully understanding the risks and benefits, increasing the likelihood of risky proposals passing with low participation.

To address these challenges, we propose the following desired properties for MIP decision making:
- **Informed and Engaged:** Community members, especially non-experts, should be able to understand the potential risks and benefits of a proposal. Additionally, there should be a high level of participation in deliberations, debates, and voting. 
- **Scalable:** The process should have a clear path to scaling with the number of MIPs.
- **Decentralized:** There should be a path for all dependencies to be managed on-chain, avoiding reliance on centralized individuals and organizations.

To achieve these desired properties, we propose the following mechanisms before and during voting to be discussed by the community.
- Experts form a Review Group to evaluate each MIP and provide insights for the broader community about the risks, benefits and tradeoffs. 
- This Review Group publishes a Risk Assessment - a summary of the risks, benefits and tradeoffs, and an assessment of the MIP’s risk to the long-term health of the protocol. 
- The Risk Assessment would be informative to help community members understand the MIP when they participate in deliberations, debates and the On-Chain Vote.
- Following the publication of the Risk Assessment, there should be opportunities for community members to discuss it with the MIP Review Group, for example at a community call.

For greater scalability, we propose the following:
- Short-term: Introduce MIP Facilitators to assist MIP Authors in the process, coordinate collaboration and help manage the flow of MIPs.
- Long-term: As the number of MIPs and experts increases, the role of MIP Facilitator could be automated so that it’s no longer needed (or within a lower capacity). MIP Reviewers could be randomly selected from a larger pool of qualified individuals.

For greater decentralization, we propose the following:
- Provide information and onboarding materials to the community to empower members and give them more agency in the decision-making processes.
- Implement the system on-chain (eventually, once tested) so that processes are automated, eliminating dependency on the Mina Foundation to manage these processes manually.


## Specification

The following is a description of the proposed changes to the MIP process:

### New MIP Categories

Currently, the MIP process is narrowly focused on technical types of protocol changes and decisions. To support more flexible decision making, we propose to broaden the MIP process to include the following three categories. 

**Engineering MIPs** would continue to focus on Mina's core protocol technical foundations and what feature upgrades the protocol should undergo. For example:
- The network protocol
- The consensus mechanism
- Block validity rules
- Proof system

**Economics MIPs** would consider Mina's financial parameters, the levels and sources of funding to support development, including how a decentralized treasury should be configured and managed. For example:
- Mina tokenomics
- Transaction fees
- Treasury management

**Governance MIPs** would consider the rules and processes for making changes to the protocol and funding its development, and how they can be improved over time. For example:
- MIP process
- MIP roles and processes
- On-Chain Voting

These categories will need to be clearly defined, specifying their parameters and what kinds of major changes could be made.

### New Roles and Responsibilities

To encourage greater community participation, we propose to create the following roles while ensuring that ultimate decision-making authority remains with the community through on-chain voting:
- MIP Author
- MIP Facilitator
- MIP Reviewer
- Dev Team

#### Clarifying Responsibilities of Existing Roles: 

**MIP Author:** any community member can present a proposal and start the MIP process.

Responsibilities: 
- Articulate the proposed changes clearly and concisely.
- Explain the expected effects of the proposed changes.
- Encouraging community participation and engaging in discussions around the proposal.
- Actively seek feedback from other community members.
- Check alignment with the community.

Functions:
- Draft, edit and refine the MIP using the provided template.
- Follow the instructions provided in the GitHub/MIPs documentation.
- Collect and incorporate feedback from the community.
- Document alternative opinions, the options considered and other discussed topics.
- Champion the MIP through each stage of the process.
- Designate, coordinate and oversee a Dev Team for the MIP implementation.

**Dev Team:** core development team, contributors and any other active participants that are making updates, parameter changes, and upgrades to the protocol itself. 

Responsibilities:
- Review MIP’s implementation plan.
- Execute the implementation plan of approved MIPs.

Functions:
- Assess the feasibility of the MIP’s implementation plan and provide feedback to the MIP Author.
- Ensure correct execution of the implementation plan in a timely manner.
- Communicate and explain any delays and other important decisions about the implementation.

#### Introducing New Roles and Responsibilities:

**MIP Facilitator:** Elected community member with a deep understanding of Mina Protocol’s governance to oversee that the MIP process is followed correctly.

Responsibilities:
- Ensure a smooth progression of MIPs through the process.
- Facilitate cooperation and collaboration between all participants and ensure they follow the rules.
- Assist and support MIP Authors as necessary.

Functions:
- Accept or reject a MIP submission based on the established approval criteria.
- Accept or reject a status change request ensuring that each step of the process is completed correctly.
- Schedule and facilitate meetings between MIP Author and MIP Review Group, and other stakeholders.
- Produce and share meeting minutes under the Chatham House Rule.
- Identify key stakeholders affected by the MIP.
- Coordinate communications with the Mina Foundation.

**MIP Reviewer:** Elected community members who have expertise to review a MIP in a specific category.

Responsibilities:
- Review the assigned MIP.
- Provide feedback to the Author to improve the MIP.

Functions:
- Identify risks, benefits and tradeoffs of the proposed changes.
- Assess the risk of a MIP to the long-term health of the protocol.
- Publish a Risk Assessment with a rationale.
- Join the meetings scheduled by the MIP Facilitator.

#### New Role Frameworks

To ensure accountability and transparency for MIP Facilitators and Reviewers, we propose developing frameworks that clearly explain the processes for role selection, assignment, removal, resignation, and compensation. 

These frameworks will share common principles, including:

- Any community member can propose themselves as a candidate for MIP Facilitator or MIP Reviewer by submitting a new MIP.
- Any community member can propose the removal of an existing MIP Facilitator or MIP Reviewer by submitting a new MIP.
- Any community member can vote to approve or reject the selection or removal of MIP Facilitators and MIP Reviewers.
- As the community builds its capacity to effectively take on these responsibilities, key ecosystem players will continue to fulfill these roles as needed, supporting the bootstrapping and testing of the new processes.

#### Bootstrapping the Process

However, due to the limited current capacity we propose that the Mina Foundation, in collaboration with o1 Labs and other members of the Mina ecosystem, will temporarily fill these new roles as needed while the processes are tested, and the capacity and expertise grows in the community.

This bootstrapping period would last no longer than 12 months.

### Introducing Mass Deliberation Tools and Public Debates

We propose to implement **new tools** to collect and analyze the comments and sentiments from the community at different phases in the MIP process. These include:

- Polling on MinaResearch. The community is encouraged to experiment with different methods to increase effectiveness.
- Survey bot on Discord. The development and usage of open source AI bots on Discord to gather, organize and summarize mass feedback. The community is encouraged to review and propose improvements to the development:
https://github.com/MinaFoundation/gptSurveySummarizer 
- On-Chain Voting. The community is encouraged to suggest any improvements that could be proposed in a future MIP:
https://github.com/MinaFoundation/mina-on-chain-voting/tree/main 
https://github.com/MinaFoundation/govbot 
- Governance Dashboards. The community is encouraged to experiment with different methods and tools to increase participation in the MIP process.
https://minascan.io/mainnet/governance/MIPs 

We propose to introduce new types of public debates:

- During the expert part of the Review Phase: The MIP Facilitator will schedule, livestream and record open and public meetings between the MIP Author, the MIP Review Group and other stakeholders. The meeting minutes will be posted and shared in the designated channels for public consumption, allowing the community members who are interested to follow the process and contribute with comments. 
- During the community part of the Review Phase: There will be a period of time for the community to carefully read and consider the MIP and the Review Group’s Risk Assessment. The MIP Facilitator will coordinate with the Mina Foundation to organize a community call where the MIP Author and the MIP Review Group can explain the MIP, including its risks, benefits and tradeoffs, in a way that is comprehensible to non-expert audiences and respond to community questions. Dates for the on-chain vote will be set, along with instructions on how to participate.

### Changes in the Drafting and Review Phases

**Drafting Phase - MIP Submission**

During the Drafting phase, when the MIP Author submits a new MIP as a pull request in GitHub, a MIP Facilitator is assigned to assist the MIP Author, review the MIP and (if necessary, consult with the MIP Reviewers to) decide whether to approve or reject the pull request according to the following acceptance criteria:
- The proposal is original or new.
- The proposal is clearly articulated.
- The proposal uses the MIP Template correctly.
- The proposal is not missing any required parts.

If the pull request is rejected, the MIP Facilitator will communicate the decision to the MIP Author explaining the reasons for the rejection and a list of changes that could be made for the proposal to be accepted, so that the MIP Author can make these changes.

If the pull request is accepted, the MIP Facilitator:
- Approves the MIP and assigns it a formal MIP number.
- Changes the status of the MIP.
- Merges in the PR.
- Communicates the decision to the MIP author.

**Review Phase - Part 1: Risk Assessment**
We propose to create a pool of experts to review MIPs across the 3 categories. For each MIP, the Author reaches out to the pool of experts for the relevant category and sends a Request for Comments. To maintain independence, the Author does not choose who the Reviewers are. This MIP Review Group has a period to review the MIP and provide their feedback. The MIP Author can incorporate the feedback into the MIP until a final draft is completed. If the MIP Author does not incorporate the feedback, the MIP process still continues.

When the MIP Author considers the draft ready for the final review, editing stops for at least 1 week to allow MIP Reviewers to write individually a Risk Assessment that is incorporated into the designated section of the MIP template. 

The MIP Facilitator must check that every MIP has enough reviews to complete the Review phase (at least 3 if there is agreement, 5 otherwise). The MIP Facilitator can also coordinate communications and collaborate with the MIP Author to schedule any necessary meetings, and to define a roadmap with target dates for each milestone in the MIP process. 

**Review Phase - Part 2: Community Conversation with Mass Deliberation**
The MIP Facilitator sets a review period for the community to review the MIP, including the Risk Assessment, and schedules an open call where the MIP Author and MIP Review Group can present the proposal in detail. During this call, they will discuss the risks, benefits, and tradeoffs, address questions from the community, and provide instructions for on-chain voting, including the voting dates.

After this meeting, the community will have some time again to reconsider the proposal before the voting takes place.

https://drive.google.com/file/d/1RWpwc08Oj43zE25oKRdSmh-Gud_8RNfl/view?usp=drive_link

### Guidelines for Implementation

Every MIP should include an implementation plan that explicitly states what exactly needs to be done to implement the proposed changes, who is responsible for this implementation (in agreement with the Dev Team) and what are the target dates for each milestone. 

MIP Authors are responsible for writing the implementation plan, collaborating and establishing connections with the Dev Team and other stakeholders during the implementation. 

As a general rule, unless the MIP says otherwise, the implementation should be finished no later than one year after the approval via on-chain vote, and follow an iterative process with short delivery cycles (less than 3 months). 

The Dev Team designated for the implementation of each MIP must provide transparency about how the implementation is being executed, so that the community can verify that the changes are implemented in the way and time that they were intended. 

A comprehensive suite of tests is mandatory. When necessary, they will properly communicate the rationale for potential changes and delays.


## Backward (and Forward) Compatibility

No backwards incompatibilities derived from this proposal because this MIP doesn’t introduce changes to the protocol that need an update. 

To go back to the previous version of the MIP process: 

- Update GitHub pages to the previous version.
- Assign old reviewer roles.

The changes proposed in this MIP are designed to be compatible with the goals and the strategy presented in the blogpost “Next Steps for Mina Protocol’s Governance” while at the same time keeping it open and flexible to changing conditions and direction from the community.


## Test Cases

The changes described in this document require thorough end-to-end testing to ensure that the process works properly:

- **As a MIP author,** I want to submit a new MIP via a GitHub pull request using the new template so that I can initiate the proposal process.
- **As a MIP Facilitator,** I want to ensure that the MIP Review Group has enough experts with the right expertise to review MIPs effectively.
- **As a MIP Facilitator,** I want to approve or reject pull requests with a clear rationale to provide feedback on proposals.
- **As a community member,** I want to use SurveyBot to provide feedback in Discord to participate in MIP discussions.
- **As a MIP Facilitator,** I want to schedule, livestream, and record meetings on different platforms to maintain transparency in the process.
- **As a MIP Facilitator,** I want to use AI bots to accurately collect and organize meeting minutes.
- **As a community member,** I want to access meeting minutes in different formats to stay informed.
- **As a MIP Reviewer,** I want to manage my time efficiently to review MIPs and draft Risk Assessments within a reasonable timeframe.
- **As a MIP Facilitator,** I want to handle multiple MIPs simultaneously and evaluate how long reviewers need to process them in different scenarios, so I can set realistic expectations.


## Reference Implementation

The changes proposed in this MIP don’t require any protocol update but their implementation requires the coordinated collaboration of different stakeholders and community members:

**Dissolving the MIP Editor Group:**
The MIP Editor group currently oversees the MIP process. It checks that MIPs are drafted according to the MIP template and correctly formatted, and then ensures that the process is followed correctly. We propose to dissolve this group and transfer the responsibility for overseeing the process to the Protocol Governance Facilitators, while the responsibility for reviewing the content of the proposal would be transferred to the MIP Review Group.

**Update MIPs README file in GitHub:**
The Mina Foundation will create a pull request to update the Readme file in the MIPs section in GitHub.
https://docs.google.com/document/d/1ywIopypVE7tRLAD3909Tu5jjxxC1qmH7amUj2GJPHNM/edit

**Add new pages in GitHub for both MIP Facilitators and MIP Reviewers:** 
The Mina Foundation will create a pull request to create the new pages for these MIP roles inside the MIPs section in GitHub. 
https://docs.google.com/document/d/1HPHJ77VCbvUL0nG8jC4o2B8D_ly-r4tOPGuOwAqx9Rs/edit
https://docs.google.com/document/d/1PouLedjwCV0m3Qi-Kfe-oibkNmJMRQCjXWWSF2Yk2v8/edit

**Add a new page in GitHub called MIP template:**
The Mina Foundation will create a pull request to create a new page to host the MIP template inside the MIPs section in GitHub.
https://docs.google.com/document/d/1WUORti_GyXikuGPIf9Xxr7otauDzidDhk__ElDVAfC0/edit

**Add new communication channels in Discord for MIP Facilitators and MIP Reviewers:**
The Admins of the Mina Discord Server will create new private channels to coordinate the operations and internal discussion for both of these roles, granting MIP Facilitators the necessary permissions to effectively facilitate communications and moderate these channels as needed. 

**Implementation plan**
The Mina Foundation, in collaboration with o1 Labs and other members of the Mina ecosystem, will support the implementation of the proposed changes and help initiate this new process by temporarily filling the new roles as needed while the processes are tested, and the community develops the capacity to effectively assume these roles.

These changes should be implemented, and the new roles filled by elected community members, no later than one year after approval through an on-chain vote. The Mina Foundation will provide regular updates and coordinate communications throughout the implementation phase.


## Security Considerations

**Dangerous proposals passing with low turnout**
The proposal introduces Risk Assessments for informational purposes, allowing the community to review risks before voting. However, the proposal depends on the community to stay informed and reject dangerous proposals through on-chain voting. To further mitigate this, future upgrades could consider linking risk levels to higher participation thresholds for high-risk MIPs.

**Concentrating too much power in the hands of a few experts**
The risk of centralizing authority in the hands of a few reviewers is mitigated through transparency and broader community involvement. Public debates, deliberation tools, and well-defined reviewer roles ensure decisions reflect diverse perspectives and are not overly influenced by a small group.

**Risk of not having enough experts**
The proposal addresses the potential lack of expertise by assigning MIP Facilitators to manage the process, ensuring that a sufficient number of expert reviews are collected. If expertise is lacking, the MIP may be put on hold until necessary input is secured.

**Conflict of Interest between MIP Author and Reviewers**
To prevent conflicts of interest, the proposal establishes separate roles for authors and reviewers. MIP Facilitators ensure that authors do not review their own proposals, promoting objectivity and impartiality in the evaluation process.

**Managing capacity for MIP reviews**
MIP Facilitators are responsible for managing MIP flow, estimating future volumes, and requesting additional resources as needed. They develop strategies to handle capacity overload, ensuring the process remains efficient during periods of high activity.

**Safeguarding against manipulation by MIP Facilitators**
The proposal ensures transparency by implementing clear rules and guidelines for MIP Facilitators. The community can monitor facilitators' actions, and future MIPs can propose new mechanisms to enhance accountability and prevent bias, keeping the process fair and inclusive.


## Copyright

Copyright and related rights waived via CC0.
https://creativecommons.org/publicdomain/zero/1.0/
