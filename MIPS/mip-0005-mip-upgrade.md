---
mip: MIP5
title: MIP upgrade
description: This MIP adds several improvements to the MIP process
authors: Joaquin López (@joaquin.lopez1) <joaquin.lopez@minafoundation.com>, Remigiusz Antczak (@remiantczak) <remigiusz.antczak@minafoundation.com>, Ben Koppelman (benk0543) <ben.koppelman@minafoundation.com>, Cristina Echeverry (@cristinaecheverry) <cristina.echeverry@minafoundation.com>, Evan Shapiro (@evan02048) <evan@minafoundation.com>
discussions-to: https://forums.minaprotocol.com/t/upgrading-the-mip-process/6464 
status: Review
type: Meta
category: Governance
created: 2024-11-04
---

## Abstract

The [Next Steps blog post](https://minaprotocol.com/blog/next-steps-mina-protocol-governance) outlines a strategy for Mina Protocol’s governance, emphasizing an experimental and incremental approach that allows the community to iteratively test, improve, and expand decision-making processes. As the long-term vision is to enable substantial community participation in protocol decision-making, this document details how we can gradually increase that involvement, especially considering the current limited expertise and experience, which we expect will grow over time.

We propose the following initial changes to the MIP process:

- Creating new categories for future MIPs: engineering, economics, and governance.
- Introducing new roles and responsibilities to review MIPs and assist MIP authors throughout the process.
- Enhancing transparency by introducing mass deliberation tools and public debates.
- Establishing clear guidelines for implementing future MIPs.

These proposed changes were identified after collecting feedback from the community, including Mina Research Forum and Discord related channels, and learning from the experience of previous MIPs. This feedback was informed by the blog post, “Next Steps for Mina Protocol’s Governance” published on May 2, 2024, and discussed with the community in several town halls, fireside chats, and surveys.

The plan is to incorporate community feedback on this document and propose these changes in a new MIP, which would require approval through an on-chain vote. The expectation is that these changes will empower community members to present and discuss proposals, making Mina Protocol’s governance system more transparent, effective, and aligned with the community.


## Motivation

The Mina Improvement Proposal (MIP) process is the main mechanism for protocol decision making and documenting design decisions that have gone into Mina. The process provides a structured, transparent and participatory way for community members to propose, discuss and agree (or not) to implement these changes.

The MIP process is inspired by the successful governance models of projects like Bitcoin and Ethereum. It’s also inspired by the [Swiss democratic system](https://www.eda.admin.ch/aboutswitzerland/en/home/politik-geschichte/politisches-system/direkte-demokratie.html), embracing the principles of decentralization and direct democracy. Like the Swiss model, which empowers citizens to propose and vote on initiatives, the MIP process allows community members to propose and vote on major changes to the Mina Protocol, this ensures that decision-making power is distributed and community-driven, fostering a robust and participatory governance model.

Over the last few years, Mina’s community has grown substantially and an increasing number of companies and individuals are joining and contributing to it. Consequently, there is also a rising need to further develop Mina’s governance to support this growth and help to realize Mina’s vision of a future powered by participants so that its decision-making processes remain effective and aligned with the wishes of the community.


## Rationale

This challenge stems from the current MIP process, where authorship has been limited and relies on a few organizations and individuals perhaps due to the limited expertise in the community. The informal nature of the current process allows anyone to provide comments on GitHub but there is a lack of a structured process for experts to sufficiently review MIPs that could benefit the deliberation of other community members. For example, non-expert community members who vote on proposals often have to evaluate them without fully understanding the risks and benefits, increasing the likelihood of risky proposals passing with low participation.

To address these challenges, we propose the following desired properties for MIP decision making:
- **Informed and Engaged:** Community members, especially non-experts, should be able to understand the potential risks and benefits of a proposal. Additionally, there should be a high level of participation in deliberations, debates, and voting. 
- **Scalable:** The process should have a clear path to scaling with the number of MIPs.
- **Decentralized:** There should be a path for all dependencies to be managed on-chain, avoiding reliance on centralized individuals and organizations.

To achieve these desired properties, we propose the following mechanisms before and during voting to be discussed by the community.
- Introduce MIP Facilitators to assist MIP Authors in the process, coordinate collaboration and help manage the flow of MIPs.
- Experts form a Review Group to evaluate each MIP and provide insights for the broader community about the risks, benefits and tradeoffs. 
- This Review Group publishes a Risk Assessment - a summary of the risks, benefits and tradeoffs, and an assessment of the MIP’s risk to the long-term health of the protocol. 
- The Risk Assessment would be informative to help community members understand the MIP when they participate in deliberations, debates and the On-Chain Vote.
- Following the publication of the Risk Assessment, there should be opportunities for community members to discuss it with the MIP Review Group, for example at a community call.

**Potential Next Steps**

In line with the vision outlined in the [Next Steps for Mina Protocol’s Governance blog post](https://minaprotocol.com/blog/next-steps-mina-protocol-governance), we anticipate that this MIP proposal represents only the beginning of an evolving governance structure. Below are potential next steps that could be proposed in future MIPs, refined through community feedback and direction. These ideas are shared here to provide a broader context for where this initiative could lead over time:

- Risk-Based Participation Thresholds: Introduce minimum participation thresholds for high-risk or complex MIPs, ensuring robust community engagement and a higher level of scrutiny before implementation.
- Automation of the Facilitator Role: As the system scales, the MIP Facilitator role could be partially or fully automated, reducing manual intervention and ensuring efficiency while maintaining transparency.
- Randomized Reviewer Selection: MIP Reviewers could be randomly selected from a larger pool of qualified individuals to ensure fairness and distribute responsibilities more evenly across the community.
- Enhanced Community Onboarding: Develop comprehensive information and onboarding materials to empower community members, providing them with the tools and knowledge needed to actively participate in governance processes.
- On-Chain Implementation of Governance Processes: Transition governance operations to an on-chain system (after thorough testing), automating workflows to reduce dependency and further decentralized decision-making.

These next steps are not part of the immediate changes proposed in this MIP Upgrade but are shared here to give context to the overarching goals of scalability, decentralization, and inclusivity for Mina Protocol’s governance. Community feedback will play a pivotal role in shaping these potential directions as the governance framework evolves.



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
- Implementation Team


#### Clarifying Responsibilities of Existing Roles: 

**MIP Author:** any community member can present a proposal and start the MIP process. Multiple authors can contribute to one MIP, but at least one person must be identified to submit and edit it.

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

**Implementation Team:** core development team, contributors and any other active participants that are able to execute the implementation plan, making updates, parameter changes, and upgrades to the protocol itself. 

Responsibilities:
- Review MIP’s implementation plan.
- Execute the implementation plan of approved MIPs.

Functions:
- Assess the feasibility of the MIP’s implementation plan and provide feedback to the MIP Author.
- Ensure correct execution of the implementation plan in a timely manner.
- Communicate and explain any delays and other important decisions about the implementation.


#### Introducing New Roles and Responsibilities:

**MIP Facilitator:** Elected community members with a deep understanding of Mina Protocol’s governance to oversee and ensure that the MIP process is followed correctly.

Responsibilities:
- Ensure a smooth progression of MIPs through the process.
- Facilitate cooperation and collaboration between all participants and ensure they follow the rules.
- Assist and support MIP Authors as necessary.

Functions:
- Accept or reject an MIP submission based on the established approval criteria.
- Accept or reject a status change request ensuring that each step of the process is completed correctly.
- Schedule and facilitate meetings between MIP Author and MIP Review Group, and other stakeholders.
- Produce and share meeting minutes, and follow the Chatham House Rule when necessary for security reasons.
- Identify key stakeholders affected by the MIP and invite them to review the proposal and provide feedback.
- Coordinate communications with the Mina Foundation to raise community awareness and promote participation.

**MIP Reviewer:** Elected community members who have expertise to review a MIP in a specific category.

Responsibilities:
- Review the assigned MIP.
- Provide feedback to the Author to improve the MIP.

Functions:
- Identify risks, benefits and tradeoffs of the proposed changes.
- Assess the risk of an MIP to the long-term health of the protocol.
- Publish a Risk Assessment with a rationale.
- Join the meetings scheduled by the MIP Facilitator.


#### New Role Frameworks

To ensure accountability and transparency for MIP Facilitators and Reviewers, we propose developing frameworks that clearly explain the processes for role selection, assignment, removal, resignation, and compensation. 

These frameworks will share common principles, including:

- Any community member can propose themselves as a candidate for MIP Facilitator or MIP Reviewer by submitting a new MIP.
- Any community member can propose the removal of an existing MIP Facilitator or MIP Reviewer by submitting a new MIP.
- Any community member can vote to approve or reject the selection or removal of MIP Facilitators and MIP Reviewers.
- Facilitators and Reviewers can voluntarily resign, following a clearly defined resignation process.
- The roles of MIP Author, Reviewer, and Facilitator must be held by different individuals for any specific MIP to ensure impartiality and avoid conflicts of interest.
- As the community builds its capacity to effectively take on these responsibilities, key ecosystem players will continue to fulfill these roles as needed, supporting the bootstrapping and testing of the new processes.


### Introducing Mass Deliberation Tools and Public Debates

We propose to implement **new tools** to collect and analyze the comments and sentiments from the community at different phases in the MIP process. These include:

- Polling on MinaResearch. The community is encouraged to experiment with different methods to increase effectiveness.
- Survey bot on Discord. The development and usage of open source AI bots on Discord to gather, organize and summarize mass feedback. The community is encouraged to review and propose improvements to the development:
https://github.com/MinaFoundation/gptSurveySummarizer 
- On-Chain Voting. The community is encouraged to suggest any improvements that could be proposed in a future MIP:
https://github.com/MinaFoundation/mina-on-chain-voting/tree/main 
https://github.com/MinaFoundation/govbot 
- Governance Dashboards. The community is encouraged to experiment with different methods and tools to increase participation in the MIP process:
https://minascan.io/mainnet/governance/MIPs 

We propose to introduce **new types of public debates:**

- During the expert part of the Review Phase: The MIP Facilitator will schedule, livestream and record open and public meetings between the MIP Author, the MIP Review Group and other stakeholders. The meeting minutes will be posted and shared in the designated channels for public consumption, allowing the community members who are interested to follow the process and contribute with comments. 
- During the community part of the Review Phase: There will be a period of time for the community to carefully read and consider the MIP and the Review Group’s Risk Assessment. The MIP Facilitator will coordinate with the Mina Foundation to organize a community call where the MIP Author and the MIP Review Group can explain the MIP, including its risks, benefits and tradeoffs, in a way that is comprehensible to non-expert audiences and respond to community questions. Dates for the on-chain vote will be set, along with instructions on how to participate.

The Mina Foundation's role in the Review Phase is primarily supportive and logistical, ensuring that the necessary resources, communication channels, and coordination are in place to maximize community awareness and participation.


### Changes in the Drafting and Review Phases

**Drafting Phase - MIP Submission**

During the Drafting Phase, the MIP Author initiates the process by submitting a new MIP as a Pull Request in GitHub. From this point, any community member is allowed and welcome to informally and voluntarily review the PR and provide comments. At this stage, a MIP Facilitator is randomly assigned to assist the MIP Author. The responsibilities of the assigned MIP Facilitator include:
- **Supporting the MIP Author:** Providing guidance throughout the drafting process, offering feedback to refine the proposal, and ensuring it aligns with the MIP template and standards.
- **Identifying Key Stakeholders:** Reviewing the proposal to determine stakeholders who may be significantly impacted by the proposed changes and encouraging their participation in the review process.
- **Managing Conflicts of Interest:** If a potential conflict of interest is identified, the assigned MIP Facilitator has the option to step down and remove themselves from the assignment. In such cases, another Facilitator will be randomly reassigned to ensure impartiality.

The MIP Facilitator will consult with other Facilitators, relevant Reviewers, and identified stakeholders to gather input and assess the proposal. Based on these consultations, the Facilitator will decide whether to approve or reject the Pull Request according to the following **acceptance criteria:**
- The proposal is original or new.
- The proposal is clearly articulated.
- The proposal uses the MIP Template correctly.
- The proposal is not missing any required parts.

If the pull request is rejected, the MIP Facilitator will communicate the decision to the MIP Author explaining the reasons for the rejection and a list of changes that could be made for the proposal to be accepted, so that the MIP Author can make these changes.

If the pull request is accepted, the assigned MIP Facilitator will:
- Approve the MIP and assign it a formal MIP number.
- Change the status of the MIP.
- Merge in the PR.
- Communicate the decision to the MIP author.

**Review Phase - Part 1: Risk Assessment**

To enhance the rigor and transparency of the review process, we propose the creation of a pool of expert reviewers across the three MIP categories. Participation in this pool will be voluntary, with community elected individuals offering their expertise to ensure a thorough evaluation of each proposal. 

Once a MIP submission is accepted and assigned a formal MIP number, the Author submits a Request for Comments (RFC) to the whole community and the MIP Facilitator will notify the expert pool for the relevant category. This open call requires all qualified experts assigned from the pool to provide their feedback and Risk Assessment. The experts in the pool will also reach out and invite other potential subject matter experts to review the proposal and provide their insights. This approach ensures flexibility and leverages the expertise of a wide range of contributors without requiring mandatory participation.

The responsibilities and safeguards during this stage include:

- Independence and Impartiality: Reviewers must disclose any potential conflicts of interest and step down if they believe their impartiality is compromised. In such cases, a new Reviewer will be assigned.
- Collaborative Feedback: The assigned Reviewers provide detailed feedback, which the Author can use to refine the MIP. While incorporating feedback is encouraged, the process continues even if the Author does not integrate every suggestion.
- Risk Assessment: Once the MIP Author finalizes the draft, a one-week period begins where Reviewers independently write Risk Assessments. These assessments evaluate the risks, benefits, and tradeoffs of the proposal and are included in a designated section of the MIP template.

The MIP Facilitator plays a critical oversight role during this phase by ensuring:

- All participants adhere to the process guidelines.
- There are no conflicts of interest among Reviewers, Authors, or other stakeholders.
- The MIP has received a sufficient number of Risk Assessments to proceed: at least 3 if there is consensus, or 5 in cases where significant disagreements arise.

**Review Phase - Part 2: Community Conversation with Mass Deliberation**

Following the expert review, the focus shifts to maximizing community awareness and participation.

- Community Review Period: The Facilitator sets a review period for the community to read and consider the MIP, including the Risk Assessment.
- Structured Community Call: During this phase, the MIP Facilitator coordinates a community call in collaboration with the Mina Foundation. The call serves to:
    - Present the MIP in detail, with input from the Author and Reviewers.
    - Discuss risks, benefits, and tradeoffs in accessible language for non-expert audiences.
    - Provide an opportunity for community members to ask questions and express concerns.
    - Share instructions for the on-chain voting process, including key dates.

To further encourage deliberation, the community is given additional time post-call to digest the information before the on-chain vote begins. This ensures a balance between thoroughness and efficiency, avoiding rushed decision-making.


**Facilitator’s Role in Managing Throughput**
To prevent network congestion and voter fatigue:

- MIP Facilitators group proposals for periodic votes (e.g., quarterly or bi-annual voting cycles) where feasible, allowing multiple MIPs to be voted on in a single transaction.
- Proposals that require urgent consideration are flagged and processed separately, with clear justification for any changes in the recommended timelines.
- Facilitators also monitor the volume of proposals to ensure a manageable flow, coordinating closely with Reviewers and the community to balance efficiency and inclusivity.


**Community Feedback Loop**
The community has multiple opportunities to provide feedback during this phase:

- Informal Reviews and Comments: Open to anyone at any stage of the MIP process, from the Research Phase to the On-Chain Voting phase, fostering inclusivity and diverse perspectives.
- Informed Deliberation: The Risk Assessment and community call materials equip members with a clear understanding of the proposal.
- Comment Period: Community members can comment publicly on the MIP to highlight unresolved issues or suggest refinements.
- Facilitated Discussions: Facilitators track common concerns raised by the community, ensuring they are addressed in the final discussions before voting.

By integrating these measures, the Review Phase strengthens both the inclusivity and effectiveness of Mina’s governance process, setting the foundation for scalable and decentralized decision-making.

![Upgraded MIP process compared to current](https://drive.google.com/file/d/1RWpwc08Oj43zE25oKRdSmh-Gud_8RNfl/view?usp=sharing)


### Guidelines for Implementation

With the introduction of the new MIP categories—Engineering, Tokenomics, and Governance—the proposal broadens the range of initiatives that can be pursued through the Mina Improvement Proposal process. This evolution reflects a deliberate shift towards a more inclusive and collaborative framework, enabling contributions that go beyond technical protocol upgrades.

Under this proposal, the roles of the MIP Author and the Implementation Team are distinct but interdependent. While the MIP Author is responsible for drafting the proposal and outlining a feasible implementation plan, the Implementation Team focuses on executing the plan once it has been approved. This separation fosters independence and autonomy in execution while ensuring collaboration between the two parties.

MIP Facilitators play a key role in enabling this collaboration by:

- Helping MIP Authors refine their proposals and align their implementation plans with the available expertise and resources.
- Coordinating between the MIP Author and the Implementation Team to ensure mutual understanding of the scope, goals, and expectations.
- Facilitating the exchange of information, particularly in cases where the MIP Author lacks the technical expertise required for execution.

This framework ensures that MIP Authors with diverse skill sets—whether technical or non-technical—can contribute to Mina’s governance and participate in the proposal process effectively.

The Implementation Team is also involved during the Review Phase to assess the feasibility of the proposed changes and their implementation plan. Their input provides:

- Feasibility Assessment: An evaluation of whether the proposed changes are technically, economically, and logistically viable.
- Implementation Insights: Identification of potential challenges or adjustments to the plan to ensure smoother execution.

By involving the Implementation Team at this stage, the process ensures that proposals are grounded in practical considerations before they move forward to voting and implementation.

During the Implementation Phase, the Implementation Team is tasked with maintaining transparency and accountability by:

- Providing Updates: Sharing regular progress reports with the community, detailing milestones achieved and ongoing work.
- Rationale for Deviations: Explaining any deviations from the original plan, including justifications for delays or changes.
- Open Communication: Ensuring the community is informed and engaged throughout the process.

This collaborative model, supported by the new MIP categories, enhances inclusivity by lowering the barrier to entry for non-technical contributors. It also sets the foundation for scalable governance, as the implementation of proposals becomes a collective effort involving a broad spectrum of expertise and perspectives.


## Backward (and Forward) Compatibility

No backward incompatibilities are derived from this proposal because this MIP doesn’t introduce changes to the protocol that need an update. 

To go back to the previous version of the MIP process: 

- Update GitHub pages to the previous version.
- Assign old reviewer roles.

The changes proposed in this MIP are designed to be compatible with the goals and the strategy presented in the blogpost [“Next Steps for Mina Protocol’s Governance”](https://minaprotocol.com/blog/next-steps-mina-protocol-governance) while at the same time keeping it open and flexible to changing conditions and direction from the community.


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
- **As a community member,** I want to be able to present myself as a candidate for the role of MIP Facilitator and MIP Reviewer
- **As a community member,** I want to be able to vote to accept or reject candidates for MIP Facilitator and/or MIP Reviewer
- **As a community member,** I want to be able to propose the removal of any MIP Facilitator and/or MIP Reviewer
- **As a MIP Facilitator and/or a MIP Reviewer,** I want to resign from my position.


## Reference Implementation

The changes proposed in this MIP don’t require any protocol update but their implementation requires the coordinated collaboration of different stakeholders and community members:

**Dissolving the MIP Editor Group:**
The MIP Editor group currently oversees the MIP process. It checks that MIPs are drafted according to the MIP template and correctly formatted, and then ensures that the process is followed correctly. We propose to dissolve this group and transfer the responsibility for overseeing the process to the Protocol Governance Facilitators, while the responsibility for reviewing the content of the proposal would be transferred to the MIP Review Group.

**Update MIPs README file in GitHub:**
The Mina Foundation will create a pull request to update the Readme file in the MIPs section in GitHub.
- [MIP Upgrade_GitHub README.md](https://docs.google.com/document/d/1WgQpnwSX70uGyuZ1I-n6ETV9FyejujcTTtxoNSWvKjw/edit?usp=drive_link)

**Add new pages in GitHub for both MIP Facilitators and MIP Reviewers:** 
The Mina Foundation will create a pull request to create the new pages for these MIP roles inside the MIPs section in GitHub. 
- [MIP Upgrade_GitHub Facilitator Framework](https://docs.google.com/document/d/1T3Ea0SSlgd0rBsTK-G3Tb9hzLo_qkon_ROjm1_h4nXc/edit?usp=drive_link)
- [MIP Upgrade_GitHub Reviewer Framework](https://docs.google.com/document/d/1zjPghSyDM0PR6d98gFInONvIztUPainRxD8Pe7wTlV4/edit?usp=sharing)

**Add a new page in GitHub called MIP template:**
The Mina Foundation will create a pull request to create a new page to host the MIP template inside the MIPs section in GitHub.
- [MIP Upgrade_GitHub Template](https://docs.google.com/document/d/15v8Gd0pd_aw1klEdhYJthfOa2dIxOBRPdxjEJP9lWn0/edit?usp=drive_link)

**Add new communication channels in Discord for MIP Facilitators and MIP Reviewers:**

The Admins of the Mina Discord Server will create new private channels to coordinate the operations and internal discussion for both of these roles, granting MIP Facilitators the necessary permissions to effectively facilitate communications and moderate these channels as needed. 

**Implementation plan**

To ensure a structured and effective rollout of the proposed governance improvements, we propose the introduction of a group of initial reviewers that ensures that the review process can function immediately upon approval of this MIP, providing a bridge until the community has built sufficient capacity. These individuals will be listed within this MIP to provide a reliable starting point, ensuring that there is sufficient expertise to maintain the quality and integrity of the review process from the outset. By providing this initial foundation, we reduce the risk of delays or bottlenecks, reinforcing trust in the governance system and creating a strong starting point for future scalability.

As a safeguard during this bootstrapping phase, the Mina Foundation is readily available to temporarily step in to fulfill the roles of MIP Facilitators and MIP Reviewers if needed. Working in collaboration with o1Labs and other Mina ecosystem members, the Foundation’s involvement ensures stability while processes are tested and community capacity is cultivated.

During this bootstrapping phase, the Mina Foundation will focus on:

- Capacity Building: Providing onboarding resources and training to enable community members to step into these roles as quickly as possible.
- Ensuring Structure: Defining clear guidelines and expectations for both Facilitators and Reviewers to streamline their workflows and reduce ambiguity.
- Encouraging Participation: Actively engaging with the community to cultivate a larger pool of qualified individuals for these roles.

The Foundation's involvement during this phase is designed to ensure that the governance process remains effective and aligned while transitioning to a more decentralized model where community members independently manage these responsibilities.

These changes should be implemented, and the new roles filled by elected community members, no later than one year after approval through an on-chain vote. The Mina Foundation will provide regular updates and coordinate communications throughout the implementation phase.

**Initial List of Facilitators (WIP)**

- Name1 (Email, Discord username)
- Name2 (Email, Discord username)
- Name3 (Email, Discord username)
- Name4 (Email, Discord username)


**Initial List of Reviewers (WIP)**

Core Protocol Engineering
- Name1 (Email, Discord username)
- Name2 (Email, Discord username)
- Name3 (Email, Discord username)
- Name4 (Email, Discord username)

Economics
- Name1 (Email, Discord username)
- Name2 (Email, Discord username)
- Name3 (Email, Discord username)
- Name4 (Email, Discord username)

Governance
- Name1 (Email, Discord username)
- Name2 (Email, Discord username)
- Name3 (Email, Discord username)
- Name4 (Email, Discord username)


## Security Considerations

**Dangerous proposals passing with low turnout**

The proposal introduces Risk Assessments for informational purposes, allowing the community to review risks before voting. However, the proposal depends on the community to stay informed and reject dangerous proposals through on-chain voting. To further mitigate this, future upgrades could consider linking risk levels to higher participation thresholds for high-risk MIPs.

**Concentrating too much power in the hands of a few experts**

The risk of centralizing authority in the hands of a few reviewers is mitigated through transparency and broader community involvement. Public debates, deliberation tools, and well-defined reviewer roles ensure decisions reflect diverse perspectives and are not overly influenced by a small group. Opening the possibility of experts in the pool reaching out and inviting other experts to review the proposal and provide their insights, provides further flexibility and leverages the expertise of a wide range of contributors without requiring mandatory participation.

**Risk of not having enough experts**

The proposal addresses the potential lack of expertise by assigning MIP Facilitators to manage the process, ensuring that a sufficient number of expert reviews are collected. If expertise is lacking, the MIP may be put on hold until necessary input is secured.

**Conflict of Interest between MIP Author and Reviewers**

To prevent conflicts of interest, the proposal establishes separate roles for authors and reviewers. MIP Facilitators ensure that authors do not review their own proposals, and to promote objectivity and impartiality in the evaluation process.

**Managing capacity for MIP reviews**

MIP Facilitators are responsible for managing the flow of MIPs, anticipating future volumes, and requesting resources to ensure efficiency during high activity periods. This may include recruiting additional facilitators or reviewers, adopting tools like project management software for tracking progress, and implementing prioritization frameworks to handle overloads effectively. Facilitators may also develop training programs to expand the pool of capable contributors, fostering a more resilient process. The framework is intentionally flexible to adapt to evolving needs, allowing resources and strategies to scale as Mina grows and the MIP process becomes more complex. Data from early phases will guide refinements, ensuring a responsive and robust system.

**Safeguarding against manipulation by MIP Facilitators**

The proposal ensures transparency by implementing clear rules and guidelines for MIP Facilitators. The community can monitor facilitators' actions, and future MIPs can propose new mechanisms to enhance accountability and prevent bias, keeping the process fair and inclusive.


## Copyright

This document is published under the Creative Commons Attribution 4.0 International License (CC BY 4.0), which allows you to share, copy, modify, and redistribute the content in any medium or format, even for commercial purposes, as long as you provide appropriate credit to original author and indicate if any changes were made.

If you choose to modify the content, please highlight the changes you’ve made. For more information about this license, please see the full terms at https://creativecommons.org/licenses/by/4.0/ 
