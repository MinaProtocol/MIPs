---
mip: <to be assigned>
title: Remove supercharged rewards
description: This MIP removes the short-term incentive of supercharged rewards
author: Gareth Davies (@garethtdavies)
discussions-to: https://forums.minaprotocol.com/t/reduce-supercharged-rewards-in-line-with-initial-tokenomics/4540
status: Draft
type: Standards Track
category: Core
created: 2022-06-01
---

## Abstract

This MIP proposes to remove [supercharged rewards](https://minaprotocol.com/blog/mina-token-distribution-and-supply), which were planned as a short-term incentive to provide additional staking rewards to unvested tokens to increase staking adoption at the earliest stages of mainnet.

## Motivation

Supercharged rewards were introduced to provide additional incentives to stake during the early periods of mainnet launch. According to the [initial tokenomics](https://minaprotocol.com/blog/mina-token-distribution-and-supply):

> Mina Protocol is designed to provide extra block rewards( i.e., Supercharged Rewards) to block producers that stake with unlocked tokens during the first 15 months after launch. Mechanically, the Mina Protocol will pay out extra block rewards via coinbase transactions whenever a block is produced by an address that does not have any time-locked tokens. If a fully-unlocked address delegates to an address that has time-locked tokens, the address which delegated the tokens is still eligible for Supercharged Rewards.

The supercharged block reward rate of Mina was planned to follow a step-wise schedule, with changes made at hard forks. However, no hard forks have been completed to this point, so supercharged rewards have only [declined relative to the increase in total currency](https://minaprotocol.com/blog/update-on-minas-supercharged-rewards-schedule). This is demonstrated by the following chart showing the actual annualized expected protocol returns for supercharged rewards, the planned return for supercharged rewards, and the vesting tokens (locked) expected returns.

![Supercharged Yields](https://storage.googleapis.com/mina-explorer-data/supercharged_yields_epoch40.png)

Supercharged rewards [are implemented](https://github.com/MinaProtocol/mina/pull/5867) via a constant `supercharged_coinbase_factor`, which is currently `2` such that the coinbase rewards for supercharged rewards are always twice the coinbase for locked tokens (720 & 1440 respectively). Once this MIP is implemented, the return for locked and unlocked tokens will be the same. Without this MIP, supercharged rewards will continue to accrue to unlocked token holders, thereby increasing the planned inflation rate of the network relative to that initially proposed.

Supercharged rewards were proposed to incentivize staking. Mina's block production is probabilistic, so we cannot precisely know the staking participation rate. However, we can determine the lower bound based on actual blocks produced for slots won and the amount of stake delegated. For example, using the public BigQuery archive data source for epoch 40 we can determine the amount of stake delegated that won a slot and produced a block (802,958,437).

```sql
SELECT SUM(l.balance) as stake
FROM minaexplorer.archive.ledgers as l
         JOIN (SELECT DISTINCT creator
               FROM minaexplorer.archive.blocks as b
               WHERE b.protocolstate.consensusState.epoch = 40) as bps
              ON bps.creator = l.delegate
WHERE l.epoch = 40
```

We can also determine the total stake in the ledger (966,042,291) for the epoch.

```sql
SELECT SUM(l.balance) as staking_ledger
FROM minaexplorer.archive.ledgers as l
WHERE l.epoch = 40
```

This yields the following table for early and recent epochs, indicating the lower bound for the staking rate is close to 90%. This is a lower bound as it does not include stake that did not win a slot or failed to produce a block.

| Epoch | Active Stake | Total Currency | Staking Rate |
| ----- | ------------ | -------------- | ------------ |
| 0     | 752,536,881  | 805,385,693    | 93.4%        |
| 1     | 751,863,439  | 805,385,693    | 93.4%        |
| 2     | 755,814,399  | 808,975,534    | 93.4%        |
| ...   | ...          | ...            | ...          |
| 37    | 777,807,308  | 951,002,968    | 81.8%        |
| 38    | 794,336,958  | 956,037,561    | 83.1%        |
| 39    | 799,057,567  | 961,075,604    | 83.1%        |
| 40    | 802,958,437  | 966,042,291    | 83.1%        |

## Specification

Supercharged rewards, as implemented by the `supercharged_coinbase_factor` MUST be removed, ensuring that all tokens, both locked and unlocked, receive only the `coinbase_fee` without any additional weighting.

## Rationale

As outlined in the [Motivation](#motivation) section, a step change to reduce and eventually remove supercharged rewards over the first 15 months was proposed but never implemented. As the first hard fork will be (well) past the 15-month date, this MIP simply implements removing them at the time of the first hard fork. The staking participation rate is currently high, so there is unlikely a need to continue to additionally incentivize stakers with unlocked tokens at the cost of an increased inflation rate.

## Backwards Compatibility

This MIP is not backward compatible as it is a change to the consensus rules and requires a hard fork to implement.

## Security Considerations

Supercharged rewards were an initial incentive to increase the staking participation rate on Mina. As with any POS system, the security of the protocol is dependent on the majority of the active stake being honest. Given the expected returns for staking after this MIP is implemented will still be close to 10%, and with MINA tokens not subject to bonding or slashing risk, it is not anticipated this will result in a dramatic decrease in staking percentage rates.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
