---
mip: 0002
title: Add zkApps to the protocol
description: This Mina Improvement Proposal (MIP) adds programmable smart contracts (zkApps) to the Mina protocol
authors: Deepthi Kumar (deepthiskumar), Paul Steckler (@psteckler), Brett Carter (@carterbrett), Brandon Kase (@bkase)
discussions-to:
status: Draft
type: Standards Track
category: Core
created: 2023-02-14
---

## Abstract

This MIP proposes adding programmable smart contracts, called zkApps, to the Mina protocol.

The reference implementation is located on the `develop` branch of the
MinaProtocol/mina repo.

## Motivation

The zkApps protocol aims to add a more fleshed out smart contract layer to Mina
to make it much more convenient and flexible to add custom applications.
These changes extend the potential of zero-knowledge cryptography by enabling
the following characteristics while preserving the succinctness of Mina:

  **General programmability** - able to execute and settle arbitrary programs,
  not constrained to a particular VM model, but instead is flexibly designed to
  support any number of execution models, VM-like or otherwise

  **Programmable privacy** - the privacy of inputs to smart contracts and
  their state can be programmed by their developers

  **Constant in verification time** - individual transactions are executed, or
  more accurately “proven” asynchronously off-chain and verified on-chain in
  time proportional to the account updates, independent of computational
  complexity of any individual proof inside of each account update

This MIP represents the first iteration of such a protocol.

## Specification

The following description summarizes the planned changes to the protocol. Not
all technical details are described here.

### Ledger account changes

Recall that accounts are identified by a public key and token id pair.

To accommodate zkApps, Mina ledger accounts have the following new field:

- `token_symbol`: a string, up to six characters long, that can provide a name
     for a `token_id` whose owner is the public key and token id in this
     account. In the existing mainnet, only the default MINA `token_id` is
     used. New `token_id`s are derived from the addresses of the
     smart contract that defines them.

The following account field is removed:

- `token_permissions`: this field was meant for non-default tokens that were
never introduced

The `snapp` field in accounts was never instantiated and is
replaced by the `zkapp` field.

The `zkapp` field is a record with the following fields:

- `app_state` : an array of eight field elements, the current state of a zkApp
associated with the account
- `verification_key` : an optional key to use when verifying transaction
SNARKs for zkApps using this account; when not provided, the account is a
typical Mina account that supports payments.
- `zkapp_version` : a nonnegative integer, the zkApps version that determined
the `app_state`
- `action_state` : an array of five field elements, each representing an
"action" (see below) contained in a zkApp
- `last_action_slot` : the global slot when the `action_state` was last updated
- `proved_state` : a boolean, whether the application state was set from a
proof-driven zkapp transaction.
- `zkapp_uri` : a string, of max size=255, either empty, or denoting a URI that
contains information about the zkApp

An "action" is a list of special "events" that is an array of field elements
representing information chosen by a zkApp developer. See the published
zkApps documentation on
[events](https://docs.minaprotocol.com/zkapps/advanced-snarkyjs/events) and
[actions](https://docs.minaprotocol.com/zkapps/advanced-snarkyjs/actions-and-reducer)
for more information on the mental model of these primitives and how they're
used in practice.

Initially, the `action_state` contains default values; it's updated at most once
per block. Each time a new element is added, the existing elements are shifted,
and the rightmost element discarded. The max size of the queue is 5.

### zkApp transactions

A zkApp transaction contains an account update for a fee payer, and zero or more
account updates for other accounts (might include the fee payer account).

The fee payer's account update contains:
- a signature
- a public key
- an optional `valid_until` slot and
- a nonce

The other account updates contain more information:

- `authorization` : a proof, a signature, or `None_given`
- `public_key`
- `token_id` : the default MINA token or a custom token
- `update` : a specification of how the account must be updated
- `balance_change` : a signed quantity, how much this account's balance will
change
- `increment_nonce` : a boolean, whether this account's nonce will increment
- `events` : an array of field elements the zkApp can choose to provide
- `actions` : a list of events the zkApp can choose to provide (see above re
`sequence_state` for a primer on actions)
- `call_data` : a hash used when composing zkApps
- `call_depth` : a nonnegative integer, used to establish parent/child
relationships between account updates (see more information on tokens)
- `preconditions` : preconditions that must hold for the account update to
succeed
- `use_full_commitment` : whether to include hashes of the fee payer and memo in
the commitment for a signature authorization
- `implicit_account_creation_fee` : a boolean, whether to pay an account
creation fee from the `balance_change`; otherwise, the fee is taken from the
"fee excess" across all account updates
- `may_use_token` : See below for an explanation of this field's semantics
alongside other information about tokens

If the authorization is a proof, the account update contains a hash of the
verification key.

Like payments and delegations, zkApps contain a memo, up to 32 bytes long.

#### Token Mechanics

Mina builds custom token processing affordance directly into the protocol.

Accounts are referenced by the pair of public key and token_id as stated above.

Custom token contracts define logic for "children" in the "call forest" of
account updates. The token_id for the logic specified by this custom contract is
derived (via hashing) from the address of the contract.

Whenever you use a custom token in an account update, you must grant permission
using the `may_use_token` field. This grants the token owner information about
the specific token transfer by using a signature or proof. It has the
possibility to make assertions about what happens to token accounts. Tokens can
be used as direct or indirect children within a local account update tree.

Parent account updates receive a commitment to the entire subtree as a
parameter in their account update logic. This update allows token contracts to
enforce rules about their usages.

`may_use_token` can take one of three values:
* `No` -- only default MINA token can be used
* `Parents_own_token` -- the direct parent account update owns the token
* `Inherit_from_parent` -- copy the direct parent account update's permission

Valid transactions can contain arbitrarily deep trees, though the network can
process only the transactions that have a total size within the constraints of
the "size heuristic" defined later in this document.

This structure is flexible to support all sorts of token interactions that we
see on other blockchains. For example, you can check arbitrary depth subtrees
with recursive proofs and sums up all balances and checks that all the balances
sum to zero in order to allow any token interactions that don't create new
tokens. Using the parenting system, you can also implement complex interactions
like liquidity tokens in a DEX.

Note that this structure enables extremely powerful usages of tokens. The entire
Mina ecosystem benefits with a collective agreement to agree on more restrictive
token standards built on top of these systems such that groups of contracts can
compose with one-another more easily.

#### Call Forest Encoding

Construct a call forest from a list of account updates, where the
call-depth field of an account update encodes the Node-Left-Right depth-first
traversal order of the update within a tree. A new tree within a forest is
indicated by a call-depth of 0. Each successive account update can
increase by 1, stay the same, or decrease by an arbitrary amount. Call-depth 0
defines a new tree in the forest, and a non-zero number defines a child of the
most recently defined tree.

Here is an example of a forest constructed from a list of account updates with
call-depths `0 0 1 2 3 1 0` :

```
 *  *     *
    | \
    *  *
    |
    *
    |
    *
```

#### Updates in Account Updates

An account update can contain a request to update certain fields in an account.
Those fields are:

- `app_state`
- `delegate`
- `verification_key`
- `permissions`
- `zkapp_uri`
- `token_symbol`
- `timing`
- `voting_for`

Note that some of these items are not in the in the `zkapp` part of an account
(for example, the `delegate` field). A given account update can request to
update some or all of these items.

The `permissions` field of accounts is expanded to include permissions to update
each of these items. Besides requiring permission, the `timing` field of an
account can be set only if the account is currently untimed. The complete set of
permissions is described.

#### Preconditions in account updates

Three kinds of preconditions can be checked before applying an account update:

- account preconditions
- network preconditions
- a `validWhile` precondition

The account preconditions are:

- `balance` : a range of account balances
- `nonce` : a specific nonce, or a range of nonces
- `receipt_chain_hash` : a specific receipt chain hash
- `delegate` : a specific public key
- `state` : a specific application state
- `sequence_state` : a specific sequence state
- `proved_state` : a specific proved state
- `is_new` : whether the account is new

An account update can include some or all of these
preconditions. These preconditions are checked against the account
from the previous protocol state. These conditions must be true at the
moment the transaction is included in a block, otherwise the
transaction fails.

The network preconditions are:

- `snarked_ledger_hash` : a specific ledger hash
- `blockchain_length` : a range of block heights
- `min_window_density` : a range of window densities
- `total_currency` : a range of amounts
- `global_slot_since_genesis` : a range of slots
- `staking_epoch_data` : epoch data
- `next_epoch_data` : epoch_data

where `epoch_data` contains:

- a specific ledger hash
- a range for total currency
- a specific epoch seed
- start and lock checkpoints
- a range of epoch lengths

Among these network precondition items, any item can be checked or ignored as a
precondition. That also holds for the individual items in `epoch_data`. As for
account preconditions, these items are checked against the previous protocol
state.

The initial implementation proposes that network preconditions are to be true at
the moment the transaction is included in a block, otherwise the transaction
fails. This failure is known to be a bit frustrating as many of the network
preconditions . In a future hardfork, there will likely be changes to allow
network preconditions to be true in a range of slots/blocks rather than the
exact one where the transaction is added.

The `validWhile` precondition is like the `global_slot_since_genesis`
precondition. It is a range of global slots, except that it is checked against
the current slot, rather than the slot from the previous protocol state.


### Custom tokens

zkApps allow the creation of custom, non-default tokens. Every token, except the
default MINA token, has an owner that is an account identifier (a public key
and token pair from a ledger account).

To mint a custom token, a zkApp contains an account update for an account that
owns the token to be minted; a child account update specifies that token and the
number of tokens to be minted.

This code, adapted from a Mina unit test, shows the pattern:

```ocaml
let token_minting =
  mk_forest
    [ mk_tree
        (mk_account_update_body Signature No token_owner default_token
           (- account_creation_fee) )
        [ mk_tree
           (mk_account_update_body None_given Parents_own_token
             custom_token_account custom_token 100 )
           []
        ]
    ]
```

`No` and `Parents_own_token` are the token permissions for the parent
and child, respectively.  If these account updates succeed, a new
account is created with a balance of 100 in the custom token.

An account update using a custom token requires permission to use
that token. In this example, the parent account is the the account id
given by the public key `token_owner` and the token `default_token`;
that account id owns `custom_token`. The permission
`Parents_own_token` allows the new account to receive the minted
`custom token`.

### Permissions

An account update can provide new values for certain account
fields. Whether those changes are permitted is governed by a set of permissions.
A permission can take on values `None`, `Either`, `Proof`, `Signature`, or
`Impossible`, of OCaml type `Auth_required.t`.

There is a set of associated tags `Proof`, `Signature`, and `None_given`. An
account update that has a proof that verifies yields the tag `Proof`. A
signature that verifies yields the tag `Signature`. If neither a proof nor a
signature verifies, you get the `None_given` tag.

Given a permission and a tag, an account update can perform an operation
according to this OCaml function:

```ocaml
let check (t : Auth_required.t) (c : Control.Tag.t) =
  match (t, c) with
  | Impossible, _ ->
      false
  | None, _ ->
      true
  | Proof, Proof ->
      true
  | Signature, Signature ->
      true
  | Either, (Proof | Signature) ->
      true
  | Signature, Proof ->
      false
  | Proof, Signature ->
      false
  | (Proof | Signature | Either), None_given ->
      false
```

The permissions that govern individual field updates are:

- `edit_state`
- `set_delegate`
- `set_permissions`
- `set_verification_key`
- `set_zkapp_uri`
- `edit_sequence_state`
- `set_token_symbol`
- `increment_nonce`
- `set_voting_for`
- `set_timing`

There are permissions for sending and receiving tokens:

- `send`
- `receive`

Finally, there is an `access` permission that allows an account
update to access a ledger account. The `access` permission disallows all
access to a ledger account if the authorization does not check against
the permission, or the proof or signature authorization does not
verify.

The `access` permission introduced here affects not only zkApps, but
also the payment and delegation transactions in the existing mainnet.
For those transactions, the fee payer and source accounts (that are
always the same accounts, in any case) need to have an `access`
permission compatible with a signature authorization.

### Processing zkApp transactions

#### zkApp Transaction Entrypoints

A user can create a zkApp transaction using SnarkyJS or other tooling, and sign
it with a wallet key. The daemon provides a GraphQL endpoint to accept the
zkApp. If the zkApp has a sufficient fee and passes validity checks (see below),
it's queued to the transaction pool for broadcast to the network.

#### Fees and queueing

The fee payer's account update specifies a transaction fee and a
nonce. The fee payer nonce must agree with the nonce in the fee
payer's ledger account, or the zkApp is not added to the transaction
pool. All signatures and proofs in the zkApp must verify before adding
it to the pool. There is a size heuristic for zkApps: The zkApps
that are too large according to that heuristic are not accepted into
the pool.

The size heuristic involves three limits: a limit on the number of
field elements in actions, a limit on the number of field elements in
events, and a limit on the cost of the account updates.  These three
limits are fixed numbers in the protocol. The cost of the account
updates is calculated from the number of proofs and signatures
contained in them, subject to a grouping used to minimize the number
of SNARKs needed to prove the transaction. That grouping sometimes
pairs signatures as one element contributing to the cost.  The number
of proofs, signatures, and signature pairs are multiplied by factors
determined empirically to yield a valid cost metric.

The transaction pool maintains a queue of pending transactions for each
fee payer, and checks the applicability of transactions considering nonces
and balances, before accepting transactions into the pool.

If added to the pool, zkApp transactions are selected according
to fee, just as for payments and delegations.

#### Two pass system

zkApps are processed in two passes. The first pass processes only the
fee payer's account update. That account update contains a fee,
nonce, a `valid_until` slot, and a signature.  The signature can fail
to verify, and the current global slot can be past the `valid_until`
value. There are no preconditions to check for the fee payer. If the
first pass fails, there is no more processing for that zkApp. Whether
or not it succeeds, the fee payer's nonce in the ledger is
incremented, and the transaction fee deducted from the fee payer's
account balance.

The second pass processes the forest of account updates contained in
the zkApp. The account updates in the forest are processed in
depth-first order.

#### Account Update Checking

An account update is checked in several ways:

- if a custom token is involved, the token permissions are checked
- if the account update is authorized by using a proof, the verification key
   hash in the account update matches that in the ledger account
- proofs and signatures are verified
- the preconditions, including `validWhile` are checked
- the sender's balance change is checked against an account balance
- the balance change is checked against the account timing, if that exists

As soon as a check fails, processing of that account update stops, no
more account updates are processed, and the zkApp fails.

When all account updates have been processed, the sum of their
balance changes, including the fee payer's balance change, must be zero.
Otherwise, the zkApp fails.

Payments and delegations are also processed over two passes, though
the second pass does not change the result from the first pass.
The first-pass processing is identical to what it was in the protocol
before the changes proposed in this document.

Proofs inside of account updates are checked when zkApp transactions are
added to the transaction pool against some known potential future verification
key (see the "Mitigation of Attack 2: Verification Key Superposition" section
for more details). When a block is created, the proofs are not re-checked
because they were already checked when added to the pool. When a block is
received, all checks required to verify that the sender hasn't manipulated the
payload are re-verified, but the proof is not explicitly checked in all cases.

#### Transaction Application

If the zkApp succeeds, the balance changes from all the account
updates are applied to the ledger; the nonces are incremented for
those account updates with `implicit_increment_nonce`; updates to
account fields are applied, and `app_state` and `proved_state` are
updated.

As with failing payments, failing zkApps are eligible to be included in
blocks. For a failing zkApp, the only affected account is the fee payer.

### Transaction SNARK

The transaction SNARK has been changed to implement the above zkapp logic in a
Kimchi circuit.

### Impact On Other Protocol Components

#### Snarked Ledger

A snarked ledger corresponds to the ledger state after all transactions in a
scan-state tree are fully snarked. For example, after the sequence of
transactions are recursively proven. As the chain progresses and transactions
are snarked, there is a new snarked ledger everytime a sequence of transactions
is fully snarked and this state is updated in the protocol state. With the two-
pass application model there are two ledger states that correspond to a scan
state tree-first pass ledger and second pass ledger. When there are transactions
from a block overflowing to a new scan state tree (as shown below), the second
pass ledger of the first tree is assumed to be true provided ledger state after
applying the remainder of the block's transaction in the second tree are true.
Due to this impending assumption that is validated only in the SNARK proof of
the second tree, first pass ledger is deemed as fully snarked and used as the
snarked ledger everywhere else in the protocol.


Having first-pass ledger as snarked ledger means that a snarked ledger can
correspond to a set of transactions that aren't fully applied and/or it may
include transactions that were applied partially (first-pass) in the previous
snarked ledger and fully applied (first-pass and second-pass) in the current
one.

For example, say, transactions `T1`s from a block split across two scan-state
trees. `T10` and `T11` from the first tree are applied partially (first pass) to
`Tree1`'s snarked ledger. The snarked ledger for `Tree2` extend `Tree1`'s
snarked ledger by applying first pass of `T12`, `T13` and then second pass of
`T10`, `T11`, `T12`, and `T13`.

            Tree1                  Tree2

             M4                      M5
       M0          M1           M2        M3
    T00   T01  T10   T11    T12   T13  T20   T21

#### Epoch Ledgers

Snarked ledgers are used as epoch ledgers or staking ledgers for vrf
evaluations. These now are first-pass ledgers as described
[above](#snarked-ledger).

#### Bootstrap

The data downloaded during bootstrap now includes the newly added fields to
the protocol state. To validate the protocol state at the root block (finalized
as per the consensus constant `k`) by generating staged ledger (second pass
ledger) from the root's snarked ledger (first pass ledger) now requires
partially applied transactions from the previous snarked ledger. This data can
be stored as part of the scan state data structure.

#### Persisted transition frontier data

Transition frontier data persisted in the config directory includes newly
added zkapp transactions in blocks, partially applied transactions from previous
snarked ledger, and statement of the most recent snarked ledger added to the
blockchain state.

### Performance Impact

#### Transaction pool

To keep the transaction pool simple, only fee payers of zkApp transactions are
checked for balance and nonce validity. Signatures and proofs of all account
updates must be verified before adding a zkApp transaction to the pool. This
introduces an additional snark verification step in the transaction pool which
until now checked only signatures. After verified in the pool, the proofs are
assumed to be valid during block creation and don't require checking again,
similar to signatures for signed commands. Proofs within a transaction and
across multiple transactions can be batched to make the verification step
slightly more efficient. Failing batch verification involves additional
verification to identify the faulty proof or transaction and so, worst case each
transaction are actually a bit slower.
Additionally, hashing zkApp transactions impact the pool's performance.
Care should be taken to hash a transaction only once and use it everywhere in
the pool and throughout the protocol.

#### Block production

ZkApp transactions in blocks can increase block production time. For example, a
zkApp transaction with four account updates is equivalent to two payments. So
essentially with zkApp transactions there can be more accounts to be updated
(or more updates per account) with the same number of transactions per blocks as
mainnet is currently at. The overhead mainly comes from additional steps
involved in updating the scan state which includes witness generation.

#### Block Validation

Block verification now involves additional zkApp proof verification. The
proofs can be efficiently batch verified since a failing batch means the block
is simply rejected.

#### Snark workers

A ZkApp transaction is essentially a list of updates to accounts. Generating
transaction snark for a zkApp transaction involves proving each account
update and the recursively merging the proofs. Thus proving time of a zkApp
transaction depends on the number of account updates. Additionally, account
updates with proofs are more expensive than account updates with signatures or
no authorization. Because the transaction application logic circuit will
increase, more memory is required to create transaction proofs. This MIP
requires that Snark Workers utilize more compute and memory than they had before
this change.

#### Memory impact

In addition to the impact to Snark Workers, all participating nodes in the
network must now store larger transactions through different
subsystems. Blocks will be larger because transactions can be larger. The
mempool can be tweaked to store fewer max transactions at a time, but other
components that store blocks must expand in size. This has some effect on
memory, but initial prototypes confirm node requirements remain reasonable.

#### Limiting zkApp transaction size

There is a "size heuristic" described earlier in this document to ensure the
number of account updates inside zkApp transactions is limited. There are also
limits on other parts of a transaction: The memo has a fixed size and events
and actions are capped to enforce that the network doesn't need to do too much
work to process a transaction.

## Backwards Compatibility

This MIP is not backwards-compatible; it is an extensive change to the
consensus rules and requires a hard fork to implement. Nonetheless,
processing of payments and delegations has not changed.

Backwards compatibility for zkApps is an important consideration for future
hardforks but has been deemed out of scope for this hardfork, and, moreover,
those proposing to upgrade to any part of zkApps at a later time are in a better
position to make concrete suggestions.

To assuage concerns that compatibility are impossible: If it is deemed important
to maintain compatibility on an upgrade, upgrades of the transaction structure,
proof circuit, proof system, or hash function can be done by wrapping the old
system in the new one or providing maps from the old to the new.

## Security Considerations

### Permissions Guarantees

Some of the permissions associated with ledger accounts provide a
data-integrity guarantee. For a ledger account, an account field
guarded by a permission in that account can be updated only if the
authorization specified by the permission is provided in an account
update that contains the field update. That account update contains a
public key and token id; the governing permissions are in the ledger
account with the same public key and token id.

The `access` permission prevents a particular kind of attack that
would allow an account update to create an arbitrary amount of a
custom token.  Without it, a user could create an empty account
update, without providing an authorization, which will allow
transfers of a custom token owned by the account identifier in the
account update in children account updates.

### Transaction Replay Attacks

To prevent replaying zkapp transactions, account updates are successfully
applied if at least one of the following conditions is true:

1. An account update increments its account's nonce and applies a precondition
   to constrain the previous nonce to some fixed value
2. An account uses the full transaction commitment (essentially depending on the
   fee payer's nonce) and isn't the fee payer themselves
3. An account update doesn't use a signature at all and therefore allows zkApp
   applications to specify their own replay protection logic

### DoS Vulnerability Mitigations

The zkApps upgrade is fairly complex and so if care is not taken, it would be
susceptible to denial of service attacks. This section outlines two such DoS
vulnerabilities that came up design of zkApps and how these attack vectors that
were revealed during design were mitigated.

#### Attack 1: Fee hijacking

Consider a hypothetical zkApps implementation where fees and account updates
are applied in their entirety before moving on to the next transaction.

In this world, consider the following scenario:

* Account A has a balance of 10 MINA tokens
* Account A has granted a smart contract access to drain his funds
* Account A sends a transaction, `T1`, with a high fee, 5 MINA, such that it
will likely be included in the next or soon to be upcoming block
* This transaction is accepted in the mempool because Account A has access to
these funds
* Another Account B triggers the smart contract call in another transaction,
`T2`, that drains Account A
* `T2` is sent with a higher fee such that it will be selected first by block
producers
* `T2` is chosen before `T1`, but then `T1` does not have the funds to pay for
the fee!

Such processes can be repeated to completely fill the mempool with "broken"
transacitons such that honest transactions cannot be processed; the economic
incentives for filling the mempool are bypassed.

#### Mitigation of Attack 1: Two-phase Transaction Application

The following mitigation sufficiently protects against fee hijacking attacks by
ensuring that for all potential transaction mempool states, the fee portion of
every transaction in the mempool can always be applied in the next block under
any circumstance.

How? Introduce two-phase transaction application (as described in earlier
sections). zkApp transactions are applied in the folowing manner: First fees are
payed for all transactions in the block, then the other account updates are
applied in sequence (and may optionally fail).

Two-phase application enables the transaction mempool implementation to only
look at the fee-payment part of a transaction when keeping a sorted queue of
transactions to-be-selected for a block. It also ensures that no other
components need to consider smart sequencing mechanisms.

Alternative mitigations include adding sophisticated sequencing requirements to
honest mempools or block producers, but this comes with other complexities, so
the two-phase application was chosen.

#### Attack 2: Verification key denial of service

Consider a hypothetical zkApps implementation where transaction mempools
consider transactions valid under a _current verification key_ model; ie. the
current verification key in the ledger is used as a source of truth when
considering the validity of subsequent zkApp transactions.

In this world, consider the following scenario:

1. A zkApp transaction that deploys a zkApp (with verification key `vk0` is
accepted to the mempool)
2. All new incoming transactions are verified against `vk0` in the ledger (from
step1)
3. A transaction that updates the verification key to `vk1` is enqueued,
followed by many zkapp transactions with proofs verifying against `vk0`. These
transactions will be invalid after the the verification key is updated to `vk1`
however, they get accepted into the pool because they are verified against the
key in the best tip ledger `vk0`. (Proofs in the same transaction against `vk1`
will succeed)
4. During block production, we check if the verification key the proofs were
verified with matches the one in the account at the time of application. All
the invalid transactions, which rely on `vk0`, will be skipped  after setting
`vk1` and remain in the pool and thereby successfully DoSing the network.

#### Mitigation of Attack 2: Verification Key Superposition

Attack 2 is mitigated by weakening the notion of valid transactions for the
mempool from valid under _latest ordering_ to one valid under
_verification key superposition_.

We say a transaction is valid under _verification key superposition_ if it is
valid given _any_ possible ordering of the current verification key or deploy
account updates within any pending transaction in the mempool.

Since a lookup in the current ledger state alone cannot determine which
verification key is being used within an account update, a hash of the
verification key is also added to all account updates that use proofs.

Changes in other components as a consequence:

* When applying transactions during block production, proofs are not verified
again (since they were verified in the pool). The only check is that an account
update's verification key hash matches the key in the account when the update is
applied, setting the status as failed if they do not match.
* In the transaction snark, again compare the account update's verification key
hash against the ledger and if they don’t match, do not require proofs to
validate in the snark (the snark succeeds)
* When validating blocks, proofs in blocks are split based on transaction
statuses: all transactions that failed for invalid proofs are batched vertically
(within each command), and all other transactions are batched horizontally (one
batch for all commands).

Note that it does not suffice to consider only the _latest_ verification key
pending in the mempool when checking if a transaction is valid because it is
impossible to know when a smart contract will deploy.

## Testing

### zkApps Protocol Testing

The changes described in this document are extensive, and require thorough
testing.

Test transaction application for zkApp transactions performing all possible
account updates, with and without permission for each update. Unless the fee
payer permission is invalid, all transactions whether or not are permitted to
make an update should be included in a block. This is to prevent transaction
pool DDOS attack. For each of these cases, test both in-snark and out-of-snark
logic.

| Arbitrary zkApp transactions|
|-----------|
| zkApps based payments (no proofs)|
| zkApp transaction with 1 proof|
| Transactions with multiple proofs|
| Create new zkApp accounts|
| Create new non-zkApp account|
| Update account data (permissions, verification key, zkApp state, action state, zkApp uri, token symbol)|
|Update all the fields in one transaction|
| Send actions |
| Change a non-zkApp account to zkApp account|
| Update account data(delegate)|
| Account updates using full commitment|
| Account updates using account predicate (fields other than nonce)|
| Account updates Parties using protocol state predicate|
| Expired transactions are rejected (using protocol state predicate for zkApps)|
| Transaction with one zkApp calling the other|


| Time locked accounts|
|---------------------|
| Create time locked accounts using zkApp transactions|
| Cannot change the vesting schedule when one exists|
| Allow setting vesting schedule in the genesis ledger and using zkApp transactions|

|Invalid transactions are rejected by the mempool|
|---------------------------------|
| For insufficient_funds|
| For insufficient replace fee|
| For invalid signature|
| Duplicate transactons (already submitted)|
| For sender account not existing|
| For invalid nonce|
| For insufficient fee|
| Expired transactions (using protocol state predicate)|
| Invalid proofs|
| Invalid signatures|

|Tokens|
|------|
| Create new token accounts|
| Transfer between custom token accounts|
| Transaction with multiple token transfers|
| Mint custom tokens|
| Burn custom tokens|
| Verify snarked ledger is emitted. This is to check that the fee excess is zero and is in Mina token when there are transaction with custom tokens|

|Transaction pool|
|----------------|
| Valid zkApp transactions are added to the pool|
| Invalid zkApp transactions are rejected (for all the rejected cases as listed above)|
| zkApp transaction with incrementing fee payer nonces with sufficient funds for fee payment and valid authorization are added to the pool|
|zkApp transaction with non-incrementing fee payer nonces are rejected|
| A zkApp transaction with a fee payer and empty update should replace one previously submitted into the pool with the same fee payer and nonce|
| zkApp transaction with incrementing nonce but insufficient funds to pay fees are rejected| 
| zkApp transactions with incrementing nonce, sufficient funds but unauthorized fee payer update are rejected|
| Payments and zkApp transactions from the same account are accepted|
| Multiple valid zkApp transactions from a single account are accepted|
| Allow cancelling/replacing a zkApp transaction|
| Transactions are ordered fee per weight unit for inclusion in blocks|
| After a block is added, invalid transactions are removed from the pool|

|Smart contract deployment and upgrades|
| Deploy a smart contract by setting verification key|
| zkApp transaction that updates verification key and refers to it in subsequent account updates (in the same transaction) should be accepted into the pool and included in a block with `Applied` status|
| zkApp transaction that updates verifcation but refers to the previous verification key in subsequent account updates should be accepted into the pool and included in a block but with `Failed` status|

| Block Production |
| -----------------|
| All zkApp transactions with valid fee payers are included in blocks|
| Snark work for zkApp transactions from snark pool is included in blocks|
| All user commands (Legacy signed transactions and zkApp transactions) get included in a block|
| Account updates that sets fee payer permission that could invalidate further enqueued transactions should fail and not cause the enqueued transactions to drop|
| Be able to generate blocks with maximum allowed transactions|

| Other protocol tests|
|---------------------|
| Snark workers: Run the network until a few snarked ledgers with zkApp transactions are generated|
| Persistence: Restart the node with zkApp transactions in the persisted frontier|
| Catchup: Restart a node before 2*K blocks to trigger catchup|
| Bootstrap: Restart a node after 2*k blocks to trigger full bootstrap|
| Genesis ledger: zkApp accounts in genesis ledger|

These cases can be covered in unit tests or integration tests defaulting to unit
tests when applicable since the unit tests run relatively quickly and are meant
to test specific features. The integration tests take more time, and test
combined features of the protocol.

### zkApps Composite Logic Testing

Besides testing the zkApp features of the protocol in isolation, we can increase
our confidence in the correctness of those features by constructing applications
that combine such features. As an example of such a feature, the SnarkyJS
toolkit lets us specify reducers that correspond to "actions" in the protocol, 
that can be included in an application.

Since SnarkyJS includes a LocalBlockchain runtime which contains the full
zkApps implementation compiled to JavaScript, this was used as a target backend.

Goals of this effort were to define the minimal scope of realistic
applications that exercised all parts of the zkApps logic such that:

1. With very high confidence, it is believed that funds won't be at risk from
any bugs (see application 2 for complex token interactions that stress test
this).

2. With medium confidence, we believe that all features work as expected.

It suffices to create two realistic applications to cover the surface area of
features within zkApps.

### Application 1: Timed Voting

We constructed an app that allows voters to choose candidates within a
certain time period. The app supports more than eight candidates as to
warrant the use of off-chain merkle tree. It also uses another app that keeps
track of voters, number of votes per voter, and whether or not they are
qualified to vote.

Additional requirements:

* An election commission can deploy the voting zkapp and makes the candidate
list public in the form of events
* Voters can generate transactions that specify who/what they want to vote for
* The election commission ensures that the zkapp records the votes from accepted
transactions and stores in its state
* The election commission can use the zkapp to generate transactions that
commits to the latest vote count (what it has recorded so far)
* After the voting window closes, the election commision can commit to the final
vote count.
* The "count votes" method can be called at any moment

Components of the zkCPU tested:

* Smart contract composition
* Actions/Reducers + edge cases
* Account + Network Preconditions
* Live upgrading of contract mid-transaction
* Permissions
* State overflows (use merkle trees)

[Source code](https://github.com/o1-labs/snarkyjs/tree/main/src/examples/zkapps/voting)

### Application 2: DEX

The DEX is an automated market maker app like [Uniswap](https://uniswap.org/).
The app allows swaps between two tokens, and adding liquidity to a pair of two
types of tokens (note: this mints a new "liquidity pair token"). Additionally,
there is a
[constant product market making algorithm (x*y=k)](https://docs.uniswap.org/contracts/v2/concepts/protocol-overview/how-uniswap-works#:~:text=This%20formula%2C%20most%20simply%20expressed,referred%20to%20as%20the%20invariant.)
for determining the price. Liquidity can only be removed after it "vests".

Additional requirements:

* Contracts must include tokens X and Y structured as typical assets
* Developers can initially deploy the contract with a liquidity pool 
* Users can supply liquidity to the exchange. Liquidity is a pair of tokens X
and Y added in proportion to one another.
* Users can supply liquidity that allows the contract to take custody of tokens
X and Y and mint a lqXY token
* At any point, users can remove liquidity from the pool by burning lqXY tokens
to retrieve a proportional amount of X and Y tokens
* The developer can force liquidity tokens to vest such that lqXY cannot be
immediately redeemed for X and Y tokens (use timing feature)
* Users are able to swap X for Y or Y for X and the liquidity pool
maintains the X*Y=K proportion
* The "developer" can upgrade the contract
* The "developer" can change permissions on the contract for writing new account
fields.

Components of zkCPU tested:

* Complex Token Interactions (multiple tokens, token ownership, timing, etc)
* Permissions
* Atomic features: Permissions+Changes
* Contract Upgrading

[Source code](https://github.com/o1-labs/snarkyjs/tree/main/src/examples/zkapps/dex)


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

