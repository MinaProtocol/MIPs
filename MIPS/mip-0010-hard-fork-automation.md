---
mip: <to be assigned>
title: Hard Fork Automation
description: This MIP proposes an automated hard fork mechanism for the Mina daemon, improving decentralization and security, and eliminating the need for precisely timed manual intervention during a hard fork
author: George Agapov <george@o1labs.org>, Christian Despres <christian@o1labs.org>
discussions-to: <URL>
status: Draft
type: Meta
category: Core
created: 2025-10-06
---

## Abstract

This proposal introduces an automated hard fork mechanism for the Mina daemon that provides a complete automation infrastructure for protocol upgrades. The solution involves maintaining migrated versions of key ledgers (snarked root and epoch snapshots) alongside their original versions during normal node operation. When a hard fork occurs, nodes can automatically generate the required post-hard fork configuration locally using these pre-migrated ledgers, enabling a fully automated hard fork transition for Mina node operators.

The design minimizes friction during the critical hard fork transition by performing resource-intensive ledger migrations before the hard fork execution rather than during it. The resulting process is more decentralized, more secure, and allows hard forks to occur without requiring precisely timed manual intervention.

## Motivation

The current hard fork process lacks automation infrastructure, requiring manual intervention and precisely-timed steps that create operational complexity for node operators. Manual processes and side-channel ledger distribution are required.

This proposal introduces a comprehensive automation infrastructure to make the actual hard fork transition smoother. The solution enables nodes to produce migrated ledgers and generate the full fork config themselves locally, so that they’re available at the time the hard fork transition is scheduled.

## Specification

This proposal requires an intermediate release before the hard fork to implement the new automation mechanism.

### Core Mechanism

The solution introduces a `--hardfork-handling` flag for the daemon to control how it behaves before and during a planned hard fork:

- `keep-running`: Disables the new automation features (default)
- `migrate-exit`: Enables the new automation features

**When the automation features are enabled (`--hardfork-handling migrate-exit`),** nodes maintain both original and migrated versions of critical ledgers during normal operation:

**Snarked Root Ledger Management:**

- At startup, if no existing snarked root exists, the node bootstraps normally from genesis
- If an existing unmigrated snarked root is found, the node attempts to load the corresponding migrated version
- If the migrated version doesn't exist or fails integrity checks, migration occurs automatically

**Integrity Check Process:**

- **When performed:** Integrity checks run when loading snarked root ledger from disk at startup and immediately before generating the genesis ledgers during the automatic hard fork process
- **Check methodology:** Compare accounts in the old-versioned ledger against accounts in the newly-versioned ledger, accounting for differences imposed by the conversion function
- **Failure handling:** If the integrity check fails, the system trusts the old versioned database and performs the direct migration once again
- **Performance impact:** Successful sync-checks for mainnet ledger take 5-8 seconds with [zkapp state size migration](https://github.com/MinaProtocol/MIPs/blob/main/MIPS/mip-0007-increase-state-size-limit.md) applied, resulting in an expected 15-24 seconds for three ledgers and up to 2 minutes of additional startup time in practice due to hardware variations

**Staged Ledger Synchronization:**

- Account updates applied to the unmigrated snarked root are simultaneously applied to the migrated version, maintaining synchronization
- Both versions remain synchronized throughout normal operation
- Current epoch and next epoch snapshots are taken of both migrated and unmigrated versions (hard forks are scheduled to avoid epoch boundaries, which occur every two weeks)

**Hard Fork Execution:**

- Fork configuration is generated automatically in the existing `$MINA_CONFIG_DIR`
- The process follows the updated hard fork timeline (detailed in Appendix A)
- Post-hard fork genesis ledgers are created by the daemon using pre-migrated data
- Technical implementation details are provided in Appendix B

**When the automation features are disabled (`--hardfork-handling keep-running`):**

- The migrated ledgers will *not* be maintained during normal operation
- After the chain-end slot, the Mina node will continue running rather than shutting down automatically
- A manual shutdown initiated by the node operator will be required
- Before shutdown, the node operator can access the `forkConfig` GraphQL endpoint to retrieve configuration for semi-manual ledger conversion
- The legacy hard fork procedure must be followed: post-hard fork genesis ledgers must be created semi-manually using scripts by the node operator, or the operator must install the post-hard fork node package

To ensure the option **`--hardfork-handling migrate-exit`** is not used by accident, node should print a `WARN` message on startup if it’s used without either of the stop slots defined.

### Account Format Implementations

The intermediate release must include complete implementations for:

- Post-hard fork account format definitions
- Conversion methods from current stable format to post-hard fork format, including
    - Modifications of the data format (e.g., expanding zkApp states from 8 to 32 fields, with zero-padding for existing states)
    - Non-format modifications (e.g., vesting parameter updates for slot time changes)

### Packaging

Packaging for this pre-fork release will differ from regular releases:

- **Docker packaging strategy**
    - With the pre-fork release, Docker containers containing both pre-fork and post-fork executables will be distributed
    - Containers include intelligent startup scripts that detect hard fork phase and launch the appropriate executable
    - Script checks for empty signal file created by the automatic hard fork config generation mechanism at a predefined location `chain-state/mesa-{network}/activated` to determine which executable to run
- **Manual deployment considerations**
    - Debian packages and manually built executables require an additional startup script for proper hard fork transition, to manage which daemon executable should start up at any particular moment during the hard fork
    - Single executable deployment is insufficient for automated hard fork execution: daemons running in `migrate-exit` hard fork mode will shut down at the transition point, but will not automatically restart on their own
    - Hard fork mode `migrate-exit` should be used with caution outside of Docker containers provided with the release
- **Documentation and release artifacts**
    - Instructions for executing `migrate-exit` mode with Debian/manual builds will be included in pre-hard fork release artifacts
    - Release documentation will specify stop slots and provide deployment guidance

## Rationale

This design accepts additional storage overhead to achieve optimal hard fork execution performance, which aligns with the critical requirements of reliable protocol upgrades. The key rationale is to shift resource-intensive operations in the daemon away from the critical hard fork moment, so daemons can generate the hard fork genesis ledgers by themselves during the hard fork. This allows packaged nodes to transition to the hard fork chain without manual intervention.

Note that the archive and rosetta mina components will still need some manual reinstallation as part of the overall hard fork process, but this will not need to be timed to the precise moment of the hard fork.

### **Core Technical Challenges Addressed**

Ledger migrations are computationally expensive because the ledger is implemented as a Merkle tree of accounts. When protocol changes require modifications to all accounts (format changes, field updates), the entire Merkle tree structure must be regenerated.

While modifying a few accounts is cheap—requiring only a few paths to the root to be re-hashed—migrating entire ledgers causes all account hashes to change, necessitating complete hash structure regeneration. This process takes approximately 70 seconds for a mainnet ledger with 273,166 accounts on hardware that matches the recommendations for running a Mina daemon node (see Appendix C), but timing will vary across different hardware configurations.

During hard forks, this operation must be repeated three times, creating substantial timing uncertainty and blocking node startup during the critical hard fork period.

### **Design Trade-offs**

**Storage Cost vs. Execution Speed:** Maintaining three additional ledger databases (263MB each on mainnet) represents acceptable overhead given modern storage capacities and the operational benefits. Pre-migration eliminates computational bottlenecks during hard forks, enabling fork configuration generation that completes in seconds rather than minutes, scaling independently of ledger size.

**Timing Predictability:** This design eliminates most of the timing variability during the critical transition: the three critical ledgers (the root snarked ledger and two epoch ledger snapshots) will be migrated fully just once, the first time a node starts up with the automatic hard fork mode enabled, and they will be kept in sync by performing incremental migrations during normal operation.

**Alternative Approaches Considered:** Direct optimization of ledger migration performance was considered but rejected because:

- The proposed solution provides superior speed improvements during the critical hard fork window
- The approach scales better with increasing account numbers
- Direct optimizations could complement this proposal for optimizing the startup time for an intermediate release (preliminary testing shows potential for reducing initial migration time from 70s to 45s or less through parallel hashing optimizations)

### **Operational Advantages**

Local generation eliminates external dependencies, improves decentralization, removes timing-critical manual steps, and provides better failure resilience.

## Backwards Compatibility

This proposal maintains full backwards compatibility for regular node operations. The new automation features are opt-in via the `--hardfork-handling` flag, ensuring existing workflows remain unaffected.

For hard fork procedures, a legacy mode `--hardfork-handling keep-running` ****based on the Berkeley hard fork process will be supported for this transition and future upgrades, allowing operators to use existing manual procedures if needed. This ensures a smooth transition path and provides fallback options. The legacy flow also enables operators to join the network after the hard fork, even if they didn't have a node set up before the upgrade.

## Reference Implementation

The implementation will be integrated into the main Mina node codebase. Key areas requiring modification include:

- **Ledger management subsystems for dual-version maintenance**
    - The converting Merkle tree abstraction to maintain the databases in sync was added in PR [#16853](https://github.com/MinaProtocol/mina/pull/16853)
    - The ability to create root ledgers and snapshots with a converting Merkle tree backing was added in PR [#17669](https://github.com/MinaProtocol/mina/pull/17669)
- **Bootstrap and initialization logic for migrated ledger handling**
    - The experimental flag (off by default) to enable the ledger syncing was added in PR [#17684](https://github.com/MinaProtocol/mina/pull/17684)
- **Finalizing ledger migration**
    - The ability to capture the diff from a staged ledger to its root snarked ledger and to apply it to another root database was added in PR [#17675](https://github.com/MinaProtocol/mina/pull/17675)
- **Fork configuration generation system**
    - Combines existing `forkConfig` GraphQL query implementation with `runtime_genesis_ledger` executable functionality
    - Utilizes ledger sync feature to efficiently create migrated databases using runtime information instead of slow account-by-account recreation
- **Packaging implementation**
    - Dockerfile and script to switch between executables are available at `dockerfiles/Dockerfile-mina-daemon-hardfork`  `dockerfiles/scripts/daemon-hardfork-entrypoint.sh`
    - Introduced in PR [#17359](https://github.com/MinaProtocol/mina/pull/17359)

All changes would be included in the release preceding the Mesa hard fork.

## Test Cases

### 1. Integration into Dry Run and Network Testing

**Objective**: Validate that nodes running in the automatic hard fork mode will behave properly when integrated into our existing testnet hard fork dry run procedure for the upcoming Mesa hard fork, and ensure network stability and consensus integrity in mixed-mode hard fork execution

**Test Scenarios**:

- Network with mixed `keep-running`/`migrate-exit` mode nodes
- Block production continuity during transition
- Validation behavior across fork boundary
- Network recovery after transition completion

### 2. Dual-Mode Hard Fork Integration Test

**Objective**: Validate that both legacy and automatic hard fork modes function correctly during a coordinated network transition.

**Setup**:

- Node A configured with `--hardfork-handling keep-running`
- Node B configured with `--hardfork-handling migrate-exit`
    - Node B is configured to be a seed node
- Both nodes initially running on pre-fork network

**Test Phases**:

*Pre-Fork Phase*:

- Verify both nodes maintain consensus on old network
- Confirm transaction processing on both nodes
- Validate epoch ledger synchronization between nodes

*Transition Phase*:

- Monitor Node A continues operation past chain-end slot
- Verify Node B automatically shuts down at chain-end slot
- Validate Node B data extraction to configured dump directory
- Confirm Node A remains on old network while Node B transitions

*Post-Fork Phase*:

- Verify Node B automatic restart with post-fork executable
- Restart Node A using crafted transition artifacts
- Validate both nodes achieve consensus on new network
- Confirm transaction processing resumes on both nodes

**Caution:** Node B might accidentally block Node A after the hardfork if Node A attempts to contact it before being restarted. A mitigation might be: to pass a command to Node A to block communication with Node B, and the trust scores will be reset after restart.

More details here: [New Hard Fork Integration Test](https://www.notion.so/New-Hard-Fork-Integration-Test-216e79b1f9108052be67f81b28433bce?pvs=21) 

### 3. zkApp Transaction Version Fallback Testing

**Objective**: Ensure zkApp transactions gracefully handle version mismatches during and after hard fork transitions.

**Test Scenarios**:

- Pre-fork zkApp transactions submitted during transition window
    - Must be rejected
- Post-fork zkApp transactions with outdated version specifications
    - Must be rejected
- Post-fork zkApp transactions submitted before the transition window
    - Must be rejected
- Post/pre-fork zkApp transactions submitted at the appropriate time windows
- zkApp state migration validation across fork boundary

**Validation Points**:

- Transaction acceptance/rejection behavior
- Error message clarity for version mismatches
- zkApp state consistency after upgrades

### 4. Ledger Synchronization Integrity Test

**Objective**: Verify that pre-migrated ledgers remain synchronized with their unmigrated counterparts throughout normal operation.

**Test Components**:

- Long-running node operation with high transaction volume
- Periodic integrity checks comparing migrated vs unmigrated ledger states
    - Implement in the node via a new CLI flag, test on the cluster
- Epoch boundary transitions with snapshot validation
- Recovery from temporary synchronization failures
    - If integrity check fails on start (due to intentionally corrupted migrated DB), state of node is recovered via migration

### 5. Hard Fork Timing Edge Cases

**Objective**: Test hard fork behavior at critical timing boundaries.

**Test Scenarios**:

- Hard fork scheduled immediately before epoch boundary
- Hard fork during high transaction load
- Node restart during hard fork window
    - Node should be able to synchronize using the help of nodes that kept running on the old fork (i.e. were launched with `--hardfork-mode keep-running`)
- Clock synchronization issues across nodes

### 6. Data Extraction and Recovery Testing

**Objective**: Validate fork config extraction mechanism and recovery procedures.

**Test Cases**:

- Successful fork config generation
    - Verify that the height is incremented from the last block's height prior to the transaction stop slot
    - Verify that the slot accounts for the time elapsed between the transaction stop slot and the genesis block of the post-hard fork network
    - Verify that epoch ledgers, epoch seeds and genesis ledger are defined correctly
    - Verify that genesis timestamp is set as expected in the test scenario
- Legacy mode artifact crafting from the fork config

### 7. Performance and Resource Impact Testing

**Objective**: Measure performance implications of dual-ledger maintenance.

**Metrics**:

- Storage overhead measurement (target: 3x 263MB additional storage)
- Startup time comparison (`keep-running` vs `migrate-exit` modes)

### 8. Docker Integration and Deployment Testing

**Objective**: Validate containerized deployment workflows for both hard fork modes.

**Test Coverage**:

- Docker image building with embedded scripts
- Directory-based executable selection logic
- Container restart behavior during hard fork
- Volume mounting for data persistence
- Multi-container orchestration during transition

### 9. Cross-Platform Compatibility Testing

**Objective**: Verify hard fork automation works across different deployment environments.

**Test Environments**:

- Debian packages (bookworm, bullseye)
- Ubuntu packages (noble, focal)
- Docker containers
- Manual binary deployments
- Various hardware configurations and performance levels

### 10. Archive Node and Rosetta Migration Testing

**Objective**: Validate that archive nodes and Rosetta services handle hard fork transitions correctly.

**Test Components**:

- Archive node data continuity across fork boundary
- Rosetta API compatibility during transition
    - API endpoint behavior during hard fork window
- Historical data accessibility post-fork
- Database migration procedures for archived data

## Security Considerations

The automation mechanism introduces several security aspects that require careful consideration:

**Positive Security Impacts:**

- Ledger maintenance and fork configuration generation are safer when done locally, which is an improvement compared to previous upgrade mechanisms
- Elimination of side-channel ledger distribution removes potential attack vectors
- Reduced reliance on precisely-timed manual operations decreases operational risk

**Risk Mitigation:**

- **Legacy Hard Fork Procedure:** The legacy hard fork procedure will be fully supported, tested, and documented. It will remain available if critical errors occur during the automated hard fork process.
- **Testing Strategy:** Extensive end-to-end and component testing of the automation mode in a variety of network scenarios will be conducted to minimize the possibility of failure in this new hard fork mode.

The overall security posture improves through reduced external dependencies and the elimination of manual intervention points that could introduce errors or vulnerabilities.

## Appendix

### Appendix A - Updated Hard Fork Process

The hard fork process follows this sequence from the consensus protocol perspective:

1. **Pre-Hard Fork Phase:** Nodes maintain dynamic consensus over the evolving ledger state according to current protocol rules
2. **Hard Fork Timing Configuration:** Hard fork timing (transaction stop slot and chain stop slot) is announced in advance and compiled into a Mina node release through the node config’s default options.
    - The `slot_tx_end` in the node config will specify the transaction stop slot
    - The `slot_chain_end` in the node config will specify the chain stop slot
    - A new `hard_fork_genesis_slot_delta` option in the node config will specify the slot interval between the `slot_chain_end` and the post-fork genesis slot
3. **Transaction Freeze:** When the transaction stop slot is reached:
    - Transaction processing from clients halts entirely
    - Block producers stop including transactions, snark work, and coinbase fees
    - Validators reject blocks containing transactions, snark work, or non-zero coinbase fees
4. **Block Production Freeze:** When the chain stop slot is reached:
    - Block production ceases
    - Nodes generate their hard fork config automatically based on the ledger state corresponding to the last block produced before the chain stop slot
        - Nodes launched with `--hardfork-handling keep-running` will generate the hard fork config upon a call to `forkConfig` GraphQL endpoint
    - The transaction stop slot and chain stop slot will be chosen so that it is overwhelmingly likely that all nodes will agree on this ledger state
5. **Automatic Restart:** Nodes shut down after generating hard fork configuration, then restart using the new configuration to establish post-fork consensus
    - Nodes launched with `--hardfork-handling keep-running` will keep running and require manual restart
    - Daemons launched manually with `--hardfork-handling migrate-exit` outside of the release container scripting will generate a hard fork configuration and shut down, but will still require a Mesa-compatible executable to be started up manually afterward
6. **Process Completion:** Hard fork finalization occurs when the new consensus protocol becomes active, i.e. first blocks are created on the post-fork blockchain

The `hard_fork_genesis_slot_delta` option defines the slot interval between the `slot_chain_end` and the `proof.fork.global_slot_since_genesis` (the hard fork genesis slot) that will be used in the consensus parameters after the hard fork. The `genesis_state_timestamp` will be set to the Berkeley timestamp of the `proof.fork.global_slot_since_genesis`.

For example, if the `slot_chain_end` were set to occur at `2025-10-08T00:00:00Z` in the Berkeley chain and the `hard_fork_genesis_slot_delta` were set to `4`, then the `proof.fork.global_slot_since_genesis` would be set to `slot_chain_end + 4` and the `genesis_state_timestamp` would be set to `2025-10-08T00:12:00Z`.

### Appendix B - Hard Fork Configuration Generation

The `$MINA_CONFIG_DIR` requires the following data transformations during hard fork execution:

**Required Migrations:**

- **Post-hard fork genesis ledger:** Created by migrating the best tip-staged ledger as it exists one slot before the `slot_chain_end`
- **Current epoch ledger:** Created by migrating the current epoch ledger snapshot as it exists one slot before the `slot_chain_end`
- **Next epoch ledger:** Created by migrating the next epoch ledger snapshot as it exists one slot before the `slot_chain_end`
- **Hard fork genesis slot:** Created by adding the `hard_fork_genesis_slot_delta` to the `slot_chain_end`
- **Genesis state timestamp:** Created by calculating the Berkeley timestamp of the hard fork global slot since genesis
- **Node configuration:** Updated with new genesis timestamp and ledger hashes

**Data Preservation:**

- All existing config directory state will be preserved
- The packaged node will save the hard fork genesis ledgers in the `chain-state/mesa-{network_id}/genesis` subdirectory of its active config directory at the time of the hard fork; the `{network_id}` will be the name of the network (`mainnet` or `devnet`) the node is connected to. The `daemon.json` file in the active config directory will be backed up as `daemon.berkeley.json`, if it exists, and a new `daemon.json` containing the updated protocol constants will be created.
- When the packaged node starts up again, it will use that `chain-state/mesa-{network_id}` to manage its ledger state. A node version will be released after the hard fork that will remove outdated files at the top-level node config directory.
- Future hard forks are intended to follow a similar process for state management: the existing state will be preserved, a new hard fork config will be generated in a new directory in `chain-state/`, that new directory will be used by the post hard fork node as its state directory, and a release after the hard fork (after a suitable waiting period) will remove the old state directory.

The following table describes the components of the config directory and how they will be modified during the hard fork

- Entries that will be moved into `chain-state/mesa-{network}` are marked in the **Moved?** column
- Entries marked **KEEP** will be unchanged, and used by the node as-is after the hard fork
- Entries marked **MIGRATE** will be created by migrating existing data, and saved in the appropriate location for the node to use after the hard fork
- Entries marked **FRESH** will not need special handling during the hard fork. They will be naturally created in their new locations or be overwritten in their old locations after the hard fork.

| **Component** | **Description** | **Current location in mina config directory** | **Operation** | **Moved?** |
| --- | --- | --- | --- | --- |
| Node config file | The consensus parameters and other node config details that can be specified at runtime | `daemon.json` by default, but also sourced from defaults compiled into the node, environment variables, and command-line options  | MIGRATE | No |
| Mina_NET2 | Mina networking information  | `mina_net2/` | FRESH | Yes |
| Trust | P2P trust information | `trust/` | FRESH | Yes |
| Wallets | User wallet information | `wallets/`  | KEEP | No |
| Root | On-disk representation of the consensus root snarked ledger | `root/` | FRESH | Yes |
| Internal tracing | Internal logs | `internal-tracing/`, `trace/` | KEEP | No |
| Genesis | Genesis ledgers for the current consensus | `genesis/` | MIGRATE | Yes |
| Frontier | On-disk representation of the transition frontier, which tracks the state of the node’s knowledge of how the network consensus is evolving | `frontier/` | FRESH | Yes |
| Epoch ledgers | The actual epoch ledger snapshots used by the node | The two `epoch_ledger*` directories containing the snapshot databases, and the `epoch_ledger.json` file that contains metadata related to the snapshots | FRESH | Yes |
| Proof cache | Cache of snark proofs seen by the node during operation  | `proof_cache/` | FRESH | Yes |
| zkApp VK cache | Cache of zkApp verification keys seen by the node during operation | `zkapp_vk_cache/` | FRESH | Yes |
| Mina logs | External logs generated by the node and its child processes | `mina.log*`, `mina-best-tip.log`, `mina-oversized-logs.log`, `mina-prover.log`, `mina-rejected-blocks.log`, `mina-verifier.log`, `mina-vrf-evaluator.log` | FRESH | No |
| Mina version | Records the version of the node (by commit) that most recently used the mina config directory | `mina.version` | FRESH | No |
| Snark pool | Tracks snark work known to the daemon | `snark_pool/` | FRESH | Yes |

Notes:

- The old genesis ledger and genesis epoch ledgers will not be used in the hard fork config. The new genesis ledger and genesis epoch ledgers will be created by migrating the ledgers specified above, and will be saved in the `chain-state/mesa-{network}/genesis/` directory.

**Automation Advantages:**

The main advantage of automation is minimizing the time needed to migrate ledgers during the hard fork process. An additional benefit of strategically organizing directories post-hard fork is that all data from the old network is fully preserved. If something goes wrong during the transition, we have a clear path to recover any data that may have been lost or corrupted.

*Efficient Best Tip Migration:*

- The migrated best tip-staged ledger constructs efficiently from the migrated snarked root and unmigrated best tip
- The best tip exists as a leaf in the transition frontier mask structure
- Only the small difference between best tip and snarked root requires migration
- This difference applies to a copy of the migrated snarked root to create the migrated best tip

*Direct Epoch Ledger Usage:*

- Migrated current and next epoch ledger snapshots already exist and copy directly as genesis epoch ledgers

## Appendix C - Benchmarking and performance monitoring

The preliminary benchmark numbers in this MIP were gathered on a machine with an Intel Core Ultra 7 Processor 258V (an 8-core processor with BMI2 and AVX) and 32GB of RAM, matching the [hardware requirements](https://docs.minaprotocol.com/node-operators/requirements) for running a Mina daemon node.
As development of the hard fork automation feature progresses, the existing benchmarking tests and performance monitoring infrastructure will be updated to cover the new operations introduced in this MIP implementation: migrating a full ledger and generating a full fork config. These tests will detect performance regressions and guide performance improvements for the hard fork automation feature.
