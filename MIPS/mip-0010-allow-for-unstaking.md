---
mip: <to be assigned>
title: Allow for Unstaking
description: Allow mina holders to designate their accounts as non-delegating.
author: Martin Allen (martyall)
discussions-to: <URL>
status: Draft
type: Standard
category (*only required for Standards Track): Core
created: February 2, 2026
requires (*optional): <MIP number(s)>
---

## Abstract

According to the [mina consensus algorithm](https://minaprotocol.com/blog/what-is-ouroboros-samasika), block producers are eligible to propose blocks in proportion with the amount of stake they are delegated. Mina holders delegate their tokens via submitting a `Delegate Transaction`, for example in the case where they would like to earn block rewards without producing blocks themselves.
If an account has never submitted such a transaction, they are set to delegate to themselves, which is the default setting upon account creation. **As a result, every single mina account is staking**.

The proposal consists of two parts:
- Maintain a distinct `total_stake` protocol parameter. Allow accounts to set their delegation status as "not participating", which would withhold their tokens from the `total_stake`.
- Change the VRF threshold check for block production eligibility -- rather than evaluating the inequality based on `total_currency`, evaluate based on the `total_stake`.

## Motivation

The primary motivation for this proposal is the current low fill-rate. Despite incentivization through block rewards, it is an empirical fact that not all accounts want to participate in "active staking", defined as delegating to a block-producing account. This could be for completely legitimate reasons, e.g. a large account such as an exchange cannot stake for regulatory reasons.

There is currently no way to designate this desire to "opt out" in the protocol. As mentioned in the abstract, all accounts are staking and all stake is contributing to the VRF threshold check. Together these imply a mismatch between the VRF threshold check and the target fill-rate, resulting in the low fill rates seen on the network today.

While the _ability_ to opt out of staking does not guarantee that relevant accounts _will_ opt out on their own, it is a prerequisite for any other updates to the protocol that desire to improve this situation.

## Specification

### A Sentinel Value for Unstaked Accounts

The `Account` type in the mina codebase has the field

```ocaml
  delegate: Public_key.Compressed.t option
```

where `Public_key.Compressed.t` is defined by an elliptic curve point on the `Pallas` curve, represented by

```ocaml
  {x : field; is_odd: boolean }
```

with the obvious interpretation for the OCaml vs circuit-level versions. We propose to use the assignment

```ocaml
  delegate = Public_key.Compressed.empty (* == {x: zero, is_odd: false}  *)
```

as a sentinel value to designate that the account is non-staking. This value is the most natural for two reasons:

1. It is an invalid public key for the signature schema on the Pallas curve.
2. The mina ocaml codebase already treats this value the way we would expect for this proposal in many places.

There is validation logic currently in the place governing a `Delegate Transaction` that would prevent you from delegating to such an account, so updating that logic is part of this proposal.

### Unstaked by default
When new accounts are created, their delegate field will be set to `empty` rather then themselves.

### Allow Unstaking Txs
Currently the system does not allow a `Delegation Transaction` where the recipient is an "invalid" account. The `empty` account is considered an invalid recipient, so this will need to be changed.

### Maintaining the total_stake
The `total_stake` is an accumulated value that is maintained by the protocol level. The current proposal is to maintain it to part of the block headers, though there are other ways to do this. Transactions will update this value as a side-effect (ie they do not declare it explicitly), and the updates will depend on 
1. The type of transaction
2. The state of `Account`s invovled, particularly their delegation status
3. The current `total_stake`

See the [Reference Implementation](#reference-implementation) section for precise details.

### Changing the VRF Threshold Check

The VRF threshold check will change to use the `total_stake` rather than the `total_currency` to compute
the threshold inequality. Roughly speaking it will change from

```
  vrf_output / 2^256 <= c * (1 - (1 - f)^(my_stake / total_currency))
```
to
```
  vrf_output / 2^256 <= c * (1 - (1 - f)^(my_stake / total_stake))
```

where `c` and `f` are protocol constants that will remain unchanged. [NB: The current codebase may use different variable names in this equation that obscure the point we're trying to make]

## Rationale

The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

## Backwards Compatibility

This change is backwards incompatible. The new `total_stake` protocol parameter must be maintained, and the VRF threshold check is computed differently. The rules around the recipient of a `Delegate Transaction` will actually relax to allow transferring to the `empty` public key.

## Test Cases

### Core Functionality Tests

**Test Case 1: Basic Opt-Out Mechanism**

- Create account with significant balance
- Delegate to `empty` address
- Verify `total_stake` decreases by account balance
- Verify account no longer participates in consensus slot calculations

**Test Case 2: Opt-In Process**

- Start with account delegated to `empty` address
- Delegate to active validator
- Verify `total_stake` increases by account balance
- Verify account can participate in block production (if running validator)

**Test Case 3: Balance Change Propagation**

- Account delegated to `empty` address receives additional tokens from a delegating account
- Verify `total_stake` decreases by the amount received
- Account delegated to `empty` address sends tokens to an account that **doesn’t** delegate to the `empty` address
- Verify `total_stake` increases by the amount sent

**Test Case 4: Zero Balance Accounts**

- Account with small balance delegates to `empty` address, using all of the funds for the delegation transaction fee
- Verify `total_stake` decreases by the fee amount
- Account receives tokens while delegated to `empty` address
- Verify `total_stake` decreases by the amount of tokens received

**Test Case 5: Rapid Delegation Changes**

- Account rapidly switches between `empty` address and active validator
- Verify `total_stake` updates correctly for each change
- Verify no calculation errors

**Test Case 6: Large Holder Opt-Out**

- Account with >30% of total supply opts out
- Verify network continues operating normally
- Verify slot assignment calculations adjust correctly

### Network Health Tests

**Test Case 7: Post-Migration Block Production**

- After migration, verify consistent block production
- Verify no degradation in network performance

**Test Case 8: Consensus Safety**

- With reduced total participating stake, verify consensus safety properties
- Test network behavior under various participation rates
- Verify epoch transition handling with new stake calculations

### Integration Tests

**Test Case 9: GraphQL API Consistency**

- Query total stake vs total supply via GraphQL
- Verify delegation status queries return correct `empty` address information

**Test Case 10: Multi-Ledger Consistency**

- Verify all three ledgers (staking, next, genesis) updated consistently
- Test epoch transitions with modified ledger states
- Verify no inconsistencies between ledger stake calculations

## Reference Implementation

### Maintaining total_stake through transactions

The `total_stake` parameter will need to be properly maintained when processing transactions. Transactions in the mina codebase have the following pseudo-code description:

```ocaml
type transaction =
  | Command of user_command
  | Fee_transfer of fee_transfer
  | Coinbase of coinbase

and user_command =
  | Signed_command of signed_command
  | Zkapp_command of zkapp_command

and signed_command = {
  fee_payer : Public_key.Compressed.t;
  fee : Fee.t;
  body : signed_command_body;
  ...
}

and signed_command_body =
  | Payment of { receiver_pk : Public_key.Compressed.t; amount : Amount.t }
  | Stake_delegation of { new_delegate : Public_key.Compressed.t }

and zkapp_command = {
  fee_payer : { public_key : Public_key.Compressed.t; fee : Fee.t; ... };
  account_updates : account_update list;
  ...
}

and account_update = {
  public_key : Public_key.Compressed.t;
  balance_change : Amount.Signed.t;
  ...
}

and fee_transfer = {
  receiver_pk : Public_key.Compressed.t;
  fee : Fee.t;
  ...
} One_or_two.t

and coinbase = {
  receiver : Public_key.Compressed.t;
  amount : Amount.t;
  fee_transfer : fee_transfer option;
  ...
}
```

the proposed semantics are as follows

```ocaml
(* Assumes:
   - get_delegate : Public_key.Compressed.t -> Public_key.Compressed.t option
   - get_balance : Public_key.Compressed.t -> Amount.t
   - is_opted_out pk = get_delegate pk |> Option.is_none

* All three functions are assumed to return account's information based on ledger **before** applying the transaction being processed.
  
*)

let rebalance_stake_for_transaction tx initial_total_stake =

  let adjust pk amount_delta total_state =
    if is_opted_out pk then total_stake
    else total_stake + amount_delta
  in

  match tx with
  | Command (Signed_command { fee_payer; fee; body = Payment { receiver_pk; amount } }) ->
      initial_total_stake
      |> adjust fee_payer -(fee + amount)
      |> adjust receiver_pk amount

  | Command (Signed_command { fee_payer; fee; body = Stake_delegation { new_delegate } }) ->
      let old_delegate = get_delegate fee_payer in
      let old_balance = get_balance fee_payer in
      match old_delegate, new_delegate with
        | Some _, None -> total_stake - old_balance
        | None, Some _ -> total_stake + old_balance - fee
        | _ -> total_stake - fee

  | Command (Zkapp_command { fee_payer; account_updates; _ }) ->
      let total_stake = adjust fee_payer.public_key (-fee_payer.fee) initial_total_stake in
      List.fold account_updates ~init:total_stake ~f:(fun acc update ->
        adjust update.public_key update.balance_change acc
      )

  | Fee_transfer transfers ->
      One_or_two.fold transfers ~init:initial_total_stake ~f:(fun acc transfer ->
        adjust transfer.receiver_pk transfer.fee acc
      )

  | Coinbase { receiver; amount; fee_transfer = None } ->
      adjust receiver amount initial_total_stake

  | Coinbase { receiver; amount; fee_transfer = Some ft } ->
      let total_stake = adjust receiver amount initial_total_stake in
      One_or_two.fold ft ~init:initial_total_stake ~f:(fun acc transfer ->
        adjust transfer.receiver_pk transfer.fee acc 
      )
```

these updates should be run as each transaction is process. The flow will likely mimic the way we currently track `supply_increase`.

### Calculating total_stake at genesis

The total stake at the HF genesis is simply the sum over all mina-based accounts where the `delegate` field is not the `empty` account.

```ocaml
let genesis_ledger_total_stake ~ledger =
  Mina_ledger.Ledger.foldi ~init:Currency.Amount.zero (Lazy.force ledger)
    ~f:(fun _addr sum (account : Mina_base.Account.t) ->
      if Mina_base.(
          Token_id.equal account.token_id Token_id.default && 
          not (Account.equal account.delegate Account.empty)
        ) then
        Currency.Amount.add sum (Currency.Balance.to_amount @@ account.balance)
        |> Base.Option.value_exn ?here:None ?error:None
             ~message:"failed to calculate total stake in genesis ledger"
      else sum )
```

Without considering further related proposals, at first HF this would be simply the `total_currency` (See implementation of `genesis_ledger_total_currency`).

## Security Considerations

### Positive Security Impacts

- **Clearer Security Model**: Enables precise estimation of attack costs and network security
- **Improved Consensus Assumptions**: Aligns actual participation with protocol assumptions
- **Better Monitoring**: Facilitates accurate assessment of network health

**Risk: Reduced Total Participating Stake**

- *Analysis*: Does not reduce security compared to current state where inactive stake provides no consensus value
- *Mitigation*: Better reflects actual network security posture
- *Mitigation*: Incentivizes active participation through clearer metrics

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
