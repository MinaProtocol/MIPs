---
mip: <to be assigned>
title: Increase Events&Actions Limit
description: This MIP proposes raising zkApp event and action limits to 1024 field elements per transaction, enabling more complex applications while maintaining performance.
authors: Yihang Liu <corvo@o1labs.org>
discussions-to: TODO
status: Draft
type: Meta
category: Core
created: 2025-08-26
---

## Abstract

This MIP proposes increasing the current limit on events and actions that can be included in a zkApp transaction. The proposed change aims to enhance the expressiveness and utility of zkApps by allowing developers to create more complex applications that require a higher number of on-chain events and actions while maintaining the protocol’s performance guarantees.

## Motivation

The current Mina protocol restricts the number of events and actions that can be emitted or executed within a single zkApp transaction to 16 per account update. While these limits were initially established to ensure low impact on performance, they have become increasingly restrictive as zkApp development progressed.

This limitation creates several problems:

1. **Development Constraints**: Developers are forced to split complex logic across multiple transactions, increasing complexity and limiting capabilities
2. **User Experience**: End users must approve and pay for multiple transactions to complete what is logically a single operation.
3. **Competitive Disadvantage**: Other smart contract platforms allow for more complex operations within a single transaction, creating a barrier to adoption for Mina.

By increasing these limits, we can enable more sophisticated zkApps that better serve user needs while maintaining Mina’s commitment to succinct blockchain architecture.

## Specification

A zkApp command comprises multiple account updates, each containing multiple events and actions. Every event or action can include up to 16 field elements as its constituent parts.

The system enforces two key constraints: zkApp commands are limited to a maximum of 100 field elements for events (`max_event_elements`) and 100 field elements for actions (`max_action_elements`). This limit differs from the previously mentioned 16, since a zkApp command can contains multiple account updates, each containing multiple events and/or actions. 

### This MIP proposes the following changes:

1. **Event Limit Increase**:
    - The number of event field elements that can be emitted in a single zkApp transaction shall be capped at 1024. The original limit that each event can have at most 16 field elements shall be removed.
    - Currently, the limit was 100 field elements per transaction, and each event can have at most 16 event elements.
2. **Actions Limit Increase**:
    - The number of action field elements that can be emitted in a single zkApp transaction shall be capped at 1024. The original limit that each action can have at most 16 field elements shall be removed.
    - Currently, the limit was 100 field elements per transaction, and each event can have at most 16 event elements.
3. **Implementation Requirements**:
    - The Mina node must be updated to validate transactions with the new limits.
    - The o1js library must be updated to support the new limits in transaction creation and verification.
    - Block producers must apply the new limits during transaction validation and block creation.

## Rationale

The specific limits proposed in this MIP were determined through a combination of:

1. **Benchmarking**: Performance testing to measure the impact of increased events and actions on block production, validation time, and network propagation (see “Benchmark for this MIP” in the Appendix). 
2. **Developer Feedback**: Input from the zkApp developer community regarding their needs and current constraints, so that the new design will allow for several actions to be dispatched, supporting increasingly complex data structures.

The chosen limits represent a balance between enabling more complex applications and maintaining the protocol’s performance characteristics.

## Backwards Compatibility

This change introduces a backwards incompatibility as it modifies a core protocol constraint. To address this:

1. **Hard Fork**: This change will require a hard fork of the network.
2. **Developer Tools**: The o1js library will be updated to support a larger limit on the number of actions and events.

## Test Cases

Test cases for this implementation will cover the following scenarios:

1. **Maximum Utilization**: Transactions using exactly the new maximum number of events and actions.
    - Measure memory, CPU, and bandwidth impact of transactions.
2. **Boundary Testing**: Transactions slightly above and below the new limits.
3. **Mixed Environment**: Blocks containing transactions with verification keys created both before and after the update should verify with all transactions being applied.
    - The expected result of the test before release is that verification keys do not depend on the limit, and post-upgrade, everything will function without the need to redeploy the zkApps.
    - However, if App State Size Increase MIP is included into the build, update on verification keys and redeployment of zkApps is still required.

## Reference Implementation

Implementation of this MIP can be found at https://github.com/MinaProtocol/mina/pull/17573.

## Security Considerations

Increasing the events and actions limits introduces a security consideration that, while noteworthy, is estimated not to affect overall system operation:

- **Resource Consumption**: Higher limits could lead to increased resource consumption during transaction verification and block processing. The specific impact on block processing has been measured and determined to be a minor cost (see Appendix).

The proposed limits have been set at levels that provide significant functional benefits while maintaining acceptable security margins. Load testing before release will be utilized to confirm this in practice.

## Appendix

### Benchmark for this MIP

We estimate increased resource consumption to be at most 40s per block. This estimation is derived from comparing runs of the ledger application benchmark (see [PR description](https://github.com/MinaProtocol/mina/pull/16968#issue-3000286307)) and experimental [data](https://mina-block-trace-viewer.netlify.app/). The estimation suggests that the proposal is compatible with [the Slot Reduction MIP](https://forums.minaprotocol.com/t/reduce-slot-time-to-90s). There’s also some further benchmark data available in the reference implementation PR.  We plan to perform the precise measurement after the MIP is fully implemented and deployed together with other concurrent MIPs.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
