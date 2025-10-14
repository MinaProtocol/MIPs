---
mip: MIP7
title: Increase On-Chain State Size Limit
description: This MIP proposes to increase the on-chain state size of accounts from 8 field elements to 32 field elements, enabling zkApps to store more data directly in zkApp accounts on Mina.
authors: Florian Kluge <florian.kluge@o1labs.org>
discussions-to: https://forums.minaprotocol.com/t/increase-on-chain-state-size-limit/6961
status: Finalization
last-call-deadline: 2025-10-13
type: Meta
category: Core
created: 2025-08-22
---

## Abstract

This MIP proposes increasing the current limit on the number of on-chain state fields that a zkApp account can store on-chain. The change aims to give developers greater flexibility to store and manage larger amounts of application data directly on-chain, enabling more sophisticated zkApp designs while preserving Mina’s succinctness guarantees.

## Motivation

Currently, Mina restricts each zkApp account to store a fixed number of on-chain state (currently 8 field elements). These registers hold application-specific data that can be read and modified by smart contracts. As zkApp complexity grows, this requires developers to use workarounds to pack multiple values into a single field element, splitting state across multiple accounts or falling back to off-chain account storage and only storing the data root on-chain, complicating code and zkApp design. Advanced zkApps such as on-chain order books, multi-step protocols, or rich metadata storage become cumbersome or even infeasible.

By raising the state-size limit, we can support more advanced applications without sacrificing Mina’s succinctness.

## Specification

This MIP specifies the following protocol changes:

1. **On-chain State Limit Increase**
   - Increase the maximum number of on-chain state fields per zkApp account from **8** to **32**.
2. **Node Software Updates**
   - Mina node validation and block production logic must accept and enforce the new state size.
   - Transaction size checks must account for up to 32 fields being serialized.
   - Transaction version number must be incremented.
3. **o1js Library Updates**
   - o1js must be updated so that account updates, accounts, and smart contracts all support up to 32 fields.
4. **Block Producer Enforcement**
   - Producers must enforce the new limit during transaction inclusion. Transactions exceeding 32 field elements are rejected.

## Rationale

The proposed limit of **32** on-chain state fields was chosen based on:

1. **Benchmarking**
   - Network propagation of larger transactions remains within acceptable latency bounds for a 90s block time target. We don’t expect any significant degradation of network performance. The increase from 8 to 32 field elements is small enough and should have no meaningful impact on block production or propagation latency across the network.
2. **Developer Feedback**
   - Community feedback repeatedly cites the 8 fields cap as a blocker for patterns for more advanced zkApps.
     - Developers have already hit Mina’s on-chain state limit of 8 field elements in common patterns such as key-value pairs. Even a modest bump increase would improve, although not fully solve, this bottleneck and simplify zkApp design and reduce reliance on external infrastructure. [Discord Thread #1](https://discord.com/channels/484437221055922177/1197093876146647050/1197093876146647050)
     - Another example illustrates that even a small increase in that limit would already enable developers to build more sophisticated applications. [Discord Thread #1,](https://discord.com/channels/484437221055922177/1048663278513045577/1048693154087456848) [Discord Thread #2](https://discord.com/channels/484437221055922177/927630405555863573/937471263922855966)
     - Developers have explicitly requested to increase the on-chain state size to 16 or 32 field elements since 8 on-chain state field elements are too limiting for most use cases. [Discord Thread #1](https://discord.com/channels/484437221055922177/910549624413102100/988481123455750144)
     - Settlement contracts for rollups such as Protokit or Zeko are limited by only being able to store 8 field elements of on-chain data.
   - Proposals for intermediate workarounds (e.g., packing data into bitfields, off chain storage) introduce complexity.
     - [Off-chain storage with increased complexity](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/offchain-storage)
     - [Packing values into Field elements in order to not exceed the on-chain state limit](https://github.com/45930/o1js-pack)

This balance preserves Mina’s succinctness while unlocking a wide array of new use cases (see Appendix) .

## Backwards Compatibility

Because this MIP changes a core protocol limit, a network upgrade is required:

1. **Hard Fork**
   - Activation via hard fork at a predetermined epoch.
2. **o1js v3.0 compatibility**
   - This release adds support for the increased state limit and is mandatory for compiling zkApps that will run after the upgrade. Contracts built with earlier o1js versions will fail verification once the new protocol is active.
3. **Existing zkApps**
   - **Redeployment required**
     - zkApp Verification keys are tied to the transaction layout and specific protocol features, so contracts need to be recompiled and redeployed to the Mina protocol.
   - **Default behaviour**
     - The 24 added state fields (indices 8-31) are automatically initialized to `Field(0)`. If a contract does not reference them, its logic and constraint count remain identical to the previous version; this way, legacy apps can be migrated with no code changes beyond recompilation and redeployment of the new verification key
4. **Exchanges, Explorers and Indexers**
   - As a result of the updated account layout and schema, exchanges, block explorers, and other indexing infrastructure will be required to migrate their internal data models to ensure compatibility with the new format.

## Reference implementation

Although the change is scattered among a few subsystems like Archive node, o1js, and Mina daemon, the core of it lies in the zkApp transaction logic. For that change, the main idea is to propagate the diff below upwards the call stack:

```diff
diff --git a/src/lib/mina_base/zkapp_state.ml b/src/lib/mina_base/zkapp_state.ml
index 3f4029a69d..879dbde8d4 100644
--- a/src/lib/mina_base/zkapp_state.ml
+++ b/src/lib/mina_base/zkapp_state.ml
@@ -1,10 +1,10 @@
 open Core_kernel
 open Pickles_types
-module Max_state_size = Nat.N8
+module Max_state_size = Nat.N32

 module State_length_vec :
   Vector.VECTOR with type 'a t = ('a, Max_state_size.n) Vector.vec =
-  Vector.Vector_8
+  Vector.Vector_32

 module V = struct
   (* Think about versioning here! These vector types *will* change
```

## Test Cases

To validate the implementation, the following tests should be performed:

1. **Maximum Utilization**
   - zkApps that read and write exactly 32 field elements.
   - zkApps that continue to only use up to 8 field elements.
   - zkApps that try reading and writing more than the limit of 32 field elements.
2. **Resource Profiling**
   - Measure proof generation CPU/memory and block validation latency
3. **o1js**
   - o1js correctly generates valid transactions and proof for on-chain state 32 field elements
4. **Account backwards-compatibility**
   - When an account was registered with a verification key before the hard fork, new transactions won’t be accepted anymore. It must be recompiled to a new verification key and redeployed after the hard fork in order to accept transactions.

## Security Considerations

Increasing on-chain state capacity introduces the following considerations:

1. **Resource Consumption**
   - More state per account can increase memory consumption for nodes and CPU load for prover. Extensive benchmarking indicates acceptable headroom under anticipated network usage. We expect to add about 768 bytes per account and account updates. We will use the caching statistics to confirm these assumptions (see https://github.com/MinaProtocol/mina/blob/fabc9eebe49c9b7cbfb8a96b15f7787947a2ee88/src/app/disk_caching_stats/README.md)
2. **State Bloat**
   - The larger state field slightly increases each account’s footprint, leading to a slight increase in RAM and disk usage for nodes. We expect that the resulting ledger growth remains well within practical limits for standard hardware.
3. **Circuit complexity increase**
   - We must ensure that the complexity of a circuit with a number of state elements raised will not grow unexpectedly and there won't be a huge performance impact on the system because of the increase in circuit complexity. This will be done through load testing as well as a unit test for determining the circuit size impact.

## Appendix

### Application Examples

- **Multisig or MPC Wallets**
  - Storing 10 to 16 signer public keys plus per-signer weight or nonce requires ~20-30 field elements. Eight fields limits you to a 2-of-3 wallet, forcing Merkle trees or additional token contracts for anything larger, whereas 32 fields supports robust multisig in a single call.
- **NFTs or Soul-Bound Token**
  - Increased on-chain state allowed developers to store more information about NFTs or Soul-Bound Token directly on-chain without having to fall back to complex external infrastructure.
- **Turn-based Games and On-chain Gaming**
  - Board state (eg. 64 squares), counters, and turn flag can fit nicely in ~12–20 field elements when packed. Today you need Merkle trees or multiple contracts, 32 field elements enable direct mutation of the game state during each move and transaction without external infrastructure.
- **Attestations**
  - ID arrays for credentials quickly grow past eight fields after only a handful of entries. Thirty two field elements yield roughly 8160 bits of data, enough for many credential sets without off-chain storage.
- Some other example applications include:
  - DAO and on-chain governance
  - Bridge solutions and Rollups that need to store chain state data
  - Small DeFi or lending protocols
  - Decentralised Identity or Account Abstraction applications

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
