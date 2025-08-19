---
mip: <number to be provided>
title: Reduce slot time to 90s
description: This MIP proposes to reduce the slot time in Mina's Ouroboros Samasika consensus protocol from 180 seconds to 90 seconds, enabling more frequent block creation and doubling network throughput while maintaining system stability.
authors: George Agapov <george@o1labs.org>, Christian Despres <christian@o1labs.org>
discussions-to: <to be provided> 
status: Last call
type: Meta
category: Core
created: 2025-06-05
---

# Abstract

This MIP proposes reducing Mina's slot time from 180 seconds to 90 seconds, effectively doubling the network's block production frequency. The change requires updating the `block_window_duration_ms` constant to 90,000 milliseconds and adjusting the coinbase reward from 720 to 360 MINA to maintain consistent token emission rates.

Existing accounts with active vesting schedules will have their parameters automatically migrated during the hard fork to preserve real-world vesting timelines despite the change in slot duration. The zkApp soft limit configuration option will be unset by default, allowing blocks to contain more than the current default of 24 zkApp transactions.

Recent performance improvements have reduced maximum block production time from over 100 seconds to under 30 seconds, making this slot time reduction feasible without compromising network stability. The upgrade requires a hard fork and is planned for Fall 2025, with prerequisite SNARK worker optimizations delivered via soft fork beforehand.

# Motivation

Mina uses the Ouroboros Samasika consensus protocol, which divides time into equal-sized chunks called slots. Currently, the slot time is set at 3 minutes, which directly limits how frequently blocks in the Mina blockchain can be created, as consensus ensures at most one block per slot.

The original slot time of 180 seconds was determined primarily by performance factors, including:

1. The time required to create a Kimchi proof-system-based SNARK proof (5-15 seconds depending on hardware)
2. The requirement that SNARK work for certain previous blocks must be completed before creating the current block
3. Ensuring all nodes can receive blocks before the end of a slot
4. Maximum number of account updates per zkapp transaction

Recent protocol releases have dramatically improved block creation and transaction processing performance (see Appendix for details). These improvements now make it feasible to decrease the slot time without compromising system stability or performance.

Reducing the slot time will allow for more frequent block creation, improving the network's throughput and user experience.

# Specification

Following protocol modifications are required to implement this change:

1. Update the `block_window_duration_ms` constant from 180,000 to 90,000 milliseconds
    - Effect: Slot time will be reduced from 3 minutes to 90 seconds
2. Update the `coinbase` constant from 720 to 360
    - Effect: Block creators will receive half of the current MINA amount for creating a block
    - This maintains the same token emission rate as before, ensuring inflation is not affected by the slot time change
3. Update the account vesting schedule logic
    - As part of the Ledger Migration procedure (that will be executed by the Mina node as part of the upgrade), update vesting parameters for **actively vesting** accounts.
    - For the definition of **actively vesting** accounts and formulas for update of vesting parameters, see Appendix.
4. Unset the zkapp soft limit configuration option by default
   - Currently set to 24 by default
   - Not constrained by the protocol, but will be unset by default to allow more zkApp transactions per block

## Prerequisites

This update requires a prerequisite improvement to the SNARK work production code, which will be delivered before the hard fork as part of a soft fork update. This improvement enables parallel processing of SNARK work across multiple workers, addressing one of the bottlenecks that necessitated the current 180-second slot time.

After delivering that update, some community members running SNARK coordinators should deploy multiple workers (at least 4 workers per coordinator) to ensure optimal SNARK work processing for maximum-cost transactions under the new 90-second slot timing.

For detailed technical background on this optimization, including hardware specifications and processing requirements, see the "SNARK Worker Requirements" section in the Appendix.

# Rationale

The primary rationale for this change is to increase the frequency of block production (throughput) in the Mina blockchain while maintaining the economic model. Recent performance improvements make this change viable without sacrificing system stability. 

Additionally, this change will improve latency, resulting in confirmation times for transactions cut in half and enhancing overall network responsiveness.

The specific value of 90 seconds was chosen as it:

1. Doubles the potential throughput of the network
2. Remains safely within the performance capabilities of the improved system

# Backwards Compatibility

This MIP is not backward compatible as it changes consensus rules and requires a hard fork to implement. It is explicitly incompatible with the current "Berkeley" release and must be implemented in the next hard fork.

This MIP doesn't require any changes to the protocol beyond those mentioned in the Specification section. There is no specific hard fork delivery strategy required for implementing this change.

# Security Considerations

The reduction in slot time could potentially impact network security if nodes are unable to keep up with the increased rate of block production. However, the recent performance improvements mitigate this risk.

Some security considerations include:

- Network propagation time for blocks becomes more critical with shorter slots
- Node synchronization may occur more frequently
- Block production failures could increase if hardware cannot keep pace
- Some blocks may contain fewer transactions than they could (or worst-case zero transactions, only coinbase) if SNARK work production for the heaviest transactions is lagging behind

Testing on various hardware configurations prior to implementation was a necessary step to ensure network stability at the new slot time.

Unsetting the zkapp soft limit configuration option by default will increase the RAM consumption of Mina nodes. However, recently implemented [RAM optimizations](https://github.com/MinaProtocol/mina/pull/16966) ensure that RAM usage after this removal remains well below the recommended [RAM specifications](https://docs.minaprotocol.com/berkeley-upgrade/requirements) for Mina node deployment.

## Community Infrastructure Impact

The reduced slot time and epoch duration changes may require adjustments to community infrastructure and tooling:

- **Block explorers**: May need updates to display timing correctly and handle the increased frequency of blocks and epoch transitions
- **Staking pool operations**: Payout processes and reward distribution systems may require adjustment to accommodate weekly epoch transitions instead of twice-weekly
- **Monitoring tools**: Scripts and dashboards that track network metrics or operate on epoch boundaries will need to account for the new timing
- **Third-party integrations**: Applications that rely on specific timing assumptions may need updates to handle the faster block production and epoch cycles

Community members operating infrastructure should evaluate their systems before the hard fork to identify any timing dependencies that may require updates. While the protocol changes are backward compatible in terms of data structures, the timing changes may affect operational procedures and automated systems.

## Epoch Transition Frequency

The reduction in slot time will decrease the real-world duration of epochs while maintaining the same number of slots per epoch. Currently, with 7,140 slots per epoch at 180 seconds per slot, each epoch lasts 357 hours (approximately 14.9 days). With the new 90-second slot time, epochs will last 178.5 hours (approximately 7.4 days).

Impact on Operations:

- **Delegation timing**: The effective "cooling off" period for stake delegation changes will be halved in real-world time
- **Automated systems**: Scripts and automated tools that operate on epoch boundaries will trigger twice as often, potentially requiring operational adjustments
- **Block producer operations**: Any operational procedures tied to epoch transitions will occur approximately twice as frequently

Block producers and stake pool operators should evaluate whether their current operational procedures need adjustment to accommodate the increased frequency of epoch transitions.

# Test cases

The following test cases are designed to validate the correctness of configuration changes and protocol behavior introduced in this MIP. These cases are implemented as part of our existing test framework, including both CI-based tests and cluster deployments. However, they can also be manually reproduced and verified without specialized tooling.

**Test Case 1: Block Timing Consistency**

Blocks are produced on a consistent 90-second cadence. This means that:

- The time difference between any two consecutive blocks is a multiple of 90 seconds.
- There exist at least two blocks with exactly a 90-second interval.
- This can be verified by inspecting the logs of any working node in the network.

**Test Case 2: zkApp Transaction Throughput**

When the network (local or larger-scale) is fed a continuous stream of zkApp transactions:

- Some blocks include more than 24 zkApp transactions.
- Over a longer duration, certain blocks contain 125 zkApp transactions.

**Test Case 3: Coinbase Amount**

Every block contains a coinbase reward of exactly **360 Mina**.

- This can be validated through GraphQL queries inspecting the block payloads.

**Test Case 4: Vesting Schedule Equivalence**

Accounts with vesting parameters that vest over a predictable timeline (e.g., several years) maintain identical *real-world* vesting schedules across the hard fork:

- The vesting start and distribution points remain aligned in real-world time.
- However, due to the change in slot duration, the schedule as measured in slot numbers is adjusted accordingly.

**Test Case 5: Epoch Duration Verification**

Epochs complete in approximately 178.5 hours (7.4 days) of real-world time:

- Measure the real-world time duration between epoch boundaries
- Verify that epochs contain exactly 7,140 slots at the new 90-second slot time
- Confirm that scripts calculating staking rewards work well under the new frequency
- This can be validated by monitoring epoch transitions and measuring intervals between them

# Reference Implementation

This MIP implementation primarily updates configuration files. The exception is migration logic for ledger accounts with vesting parameters, handled in [PR #17297](https://github.com/MinaProtocol/mina/pull/17297) that includes test coverage and serves as a reference implementation. Configuration changes to `mainnet.mlh` are detailed below.

```diff
diff --git a/src/config/mainnet.mlh b/src/config/mainnet.mlh
index 9051eada48..98332e5efa 100644
--- a/src/config/mainnet.mlh
+++ b/src/config/mainnet.mlh
@@ -6,7 +6,7 @@

 (*BEGIN src/config/coinbase/realistic.mlh*)
-[%%define coinbase "720"]
+[%%define coinbase "360"]
 (*END src/config/coinbase/realistic.mlh*)

@@ -57,7 +57,7 @@
 [%%define plugins false]
 [%%define genesis_ledger "testnet_postake"]
 [%%define genesis_state_timestamp "2020-09-16 03:15:00-07:00"]
-[%%define block_window_duration 180000]
+[%%define block_window_duration 90000]
 [%%define print_versioned_types false]
 [%%define test_full_epoch false]

@@ -76,7 +76,7 @@
 (* 2*block_window_duration *)
 [%%define compaction_interval 360000]
 [%%define vrf_poll_interval 5000]
-[%%define zkapp_cmd_limit 24]
+[%%undef zkapp_cmd_limit]
 [%%undef scan_state_transaction_capacity_log_2]

 (* Constants determining sync ledger query/response size*)
```

# Appendix

This section provides additional context relevant to the MIP.

## Vesting parameter updates

Existing accounts with vesting parameters may have been created under the assumption that funds will unlock incrementally after the passage of an interval of *system time*, not an interval of `global_slot` numbers. This was possible to work out thanks to the fact that the slot time was fixed at 3 minutes. The purpose of the vesting parameter update procedure is to preserve this
expectation as much as possible. To keep the update equations concise, they have been written as if the arithmetic will be performed using arbitrary-precision integers.

In the final update, the parameters will be clamped down to their actual ranges. Only accounts with exceptionally long vesting schedules (roughly, accounts whose vesting periods are over 12000 years long at the current 3 minute slot time) will be affected by this clamping; after the update, these accounts will next unlock funds at the maximum `global_slot`. All other accounts will be unaffected, and will:

1. Have the same minimum balance at the point the hard fork occurs as they would
have had without the hard fork occurring, and
2. Unlock funds at the same system time intervals as they would have without the
hard fork occurring.

### Update equations

An account with vesting parameters set is **actively vesting**, if (and only if) either holds:

- `global_slot < cliff_time` (vesting hasn’t started yet)
- `global_slot ≥ cliff_time ∧ vesting_increment > 0 ∧ vesting_iterations * vesting_period + cliff_time > global_slot` (some funds remain to be unlocked)
- where `vesting_iterations` is:
    - `(initial_minimum_balance - cliff_amount) / vesting_increment`, if `mod(initial_minimum_balance - cliff_amount, vesting_increment) = 0`
    - `div(initial_minimum_balance - cliff_amount + vesting_increment, vesting_increment)`, otherwise

For **actively vesting** accounts the following modifications will be applied as part of the Ledger Migration procedure):

- If `global_slot < cliff_time`, then:
    - `cliff_time_new = global_slot + (cliff_time - global_slot) * 2`
    - `vesting_period_new = vesting_period * 2`
- If `global_slot ≥ cliff_time`, then:
    - `initial_minimum_balance_new = initial_minimum_balance - cliff_amount - vesting_increment * div(global_slot - cliff_time, vesting_period)`
    - `cliff_time_new = global_slot + 2 * (vesting_period - mod(global_slot - cliff_time, vesting_period))`
    - `cliff_amount_new = vesting_increment`
    - `vesting_period_new = vesting_period * 2`
    - `vesting_increment_new = vesting_increment`

## Recent performance improvements

We have recently implemented a series of performance improvements that significantly enhance block application performance. These changes were a critical step towards reducing slot time. As a result, the maximum time required to produce a block has decreased from over 100 seconds to under 30 seconds.

**Key improvements to block processing performance:**

- [Wire transaction pool proxy in txn handler](https://github.com/MinaProtocol/mina/pull/17091)
- [Optimize transaction pool behavior under high load](https://github.com/MinaProtocol/mina/pull/16809)
- [Replace `Root` with `Root_(hash, common)` in persistent database](https://github.com/MinaProtocol/mina/pull/16656)

**Additional optimization:**

- [Avoid recomputing hashes during ledger commit](https://github.com/MinaProtocol/mina/pull/15979)
- [Manually unroll x^7 for poseidon for a ~50% speedup](https://github.com/o1-labs/proof-systems/pull/2400)

**Memory usage enhancements:**

- [Enable disk-backed cache for the Mina daemon](https://github.com/MinaProtocol/mina/pull/16966)
- [Reduce memory footprint of the verifier subprocess](https://github.com/MinaProtocol/mina/pull/16201)

## SNARK Worker Requirements

Under the current system, a single SNARK worker processes an entire maximum-cost zkApp transaction sequentially. These transactions comprise multiple account updates that are currently processed one after another by a single worker, taking 70-150 seconds (120s median) on average-specification hardware. The previous 180-second slot time was calibrated to ensure that **two** maximum-cost transactions can be sequentially processed within 2 slots (360 seconds total), as required by the parallel scan state tree architecture to avoid throughput bottlenecks.

### Parallel Processing Solution

The prerequisite optimization enables coordinators to distribute SNARK work across multiple workers, allowing the multiple account updates within zkApp transactions to be processed in parallel rather than sequentially by a single worker. With this change:

- Account updates within a maximum-cost zkApp transaction can be divided among multiple workers
- Processing time decreases significantly with additional workers (up to a certain limit)
- The network can maintain full blocks of maximum-cost transactions without SNARK processing becoming a bottleneck

### Worker Deployment Requirements

After delivering the prerequisite update, some community members running SNARK coordinators should deploy multiple workers to ensure optimal performance:

- **Minimum recommendation**: At least 4 workers per coordinator for processing maximum-cost transactions within the new 90-second slot timing
- **Hardware specification**: Expecting 4 CPU cores per SNARK worker (specification to be confirmed through final rounds of testing)
- **Network coverage**: Not all coordinators need to run 4+ workers, but sufficient network coverage is needed to prevent processing bottlenecks

The critical requirement is that SNARK work for transactions included in a block must be completed before subsequent blocks need to be created (typically within a few slot intervals). If SNARK work lags behind, blocks may contain fewer than the maximum possible transactions (128) even when more transactions are available in the transaction pool, reducing network throughput.

Proposed prerequisite optimization directly addresses the processing constraint that necessitated the current 180-second slot time and is essential for enabling the proposed reduction to 90 seconds.

## SNARK worker optimization

This section provides the technical implementation details for the SNARK worker optimization described in the "SNARK Worker Requirements" section above.

The optimization works by changing how SNARK coordinators communicate with workers. Under the current protocol—particularly for "base proofs"—the entire transaction is sent to the SNARK worker, which then processes all account updates sequentially. The new protocol enables coordinators to distribute individual account updates across multiple workers for parallel processing.

Under the current protocol — particularly for "base proofs" — the entire transaction is sent to the SNARK worker. The worker then processes the transaction, which involves computing up to 11 SNARK proofs sequentially (6 base proofs followed by 5 merge proofs).

```
    update_1   update_2   update_3   update_4   update_5   fee_payer
       |           |           |           |           |       |
    Prove       Prove       Prove       Prove       Prove   Prove
       |           |           |           |           |       |
      p1          p2          p3          p4          p5      pf
       |           |           |           |           |       |
       +--> Merge(p1, p2) = m1
                |
                +--> Merge(m1, p3) = m2
                          |
                          +--> Merge(m2, p4) = m3
                                    |
                                    +--> Merge(m3, p5) = m4
                                              |
                                              +--> Merge(m4, pf)
```

Base proofs and merge proofs require approximately the same amount of time to compute. Consequently, the total computation time for a maximum-cost transaction is approximately `11p`, where `p` denotes the time required to compute a single proof. Even with parallel processing, this cannot be completed in less than `6p`, since there are six base proofs that must be generated before merges can begin.

Now, consider an alternative approach that parallelizes the computation of merge proofs:

```

   update_1   update_2   update_3   update_4   update_5   fee_payer
       |           |           |           |           |       |
    Prove       Prove       Prove       Prove       Prove   Prove
       |           |           |           |           |       |
      p1          p2          p3          p4          p5      pf
       |           |           |           |           |       |
     Merge(p1, p2) = m1    Merge(p3, p4) = m2     Merge(p5, pf) = m3
                    \                    |                     /   
                     \                   |                    /
                     m1                  m2                  /
                      \                /                    /
                       \              /                    /
                       Merge(m1, m2) = m4                m3
                        \                               /
                         \                             /
                          +----   Merge(m4, m3)   ----+
```

This approach improves upon the current scheme: while the total computation time remains `11p`, with a sufficient number of parallel processes, the work can be completed in as little as `4p`.

An important detail is that the SNARK coordinator dispatches work to the workers in pairs, and each individual worker processes tasks sequentially. As a result, the realistic comparison is between the current `22p` (sequential execution by a single worker) and the optimized `4p` (parallelized execution). By modifying the coordinator to utilize a larger pool of workers, we can reduce SNARK work latency by up to a factor of **6.5×**. This improvement is achievable with at least ten SNARK workers per coordinator, though latency improvements begin to materialize with any number greater than one.

This reduction in latency is sufficient to support a halving of slot time, and it also enables a raise in the number of account updates per zkApp transaction (subject of another MIP considered for the upcoming Fall 2025 hardfork). Notably, this optimization does not require any protocol changes and therefore does not constitute a separate MIP.

# Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
