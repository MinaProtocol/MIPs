---
mip: <to be assigned>
title: Fungible Token Standard
description: This standard provides a unified way how to define custom fungible tokens on Mina.
author: Matej Šima (@maht0rz), Philipp Kant (@kantp)
discussions-to: https://forums.minaprotocol.com/t/draft-fungible-token-standard-zkapps/6142
status: Draft
type: Standards Track
category: ERC
created: 2023-11-30
requires: MIP4
---

## Abstract

This MIP defines a standard for implementing fungible tokens on Mina. It builds on the custom tokens feature defined in [defined in MIP4](https://github.com/MinaProtocol/MIPs/blob/main/MIPS/mip-zkapps.md#custom-tokens), and adds a uniform API for common operations on fungible tokens, such as minting, transferring, and burning.

## Motivation

Fungible tokens are a common feature on programmable blockchain systems. Establishing a standard will help developers to create their own fungible tokens, and to interface their application with tokens that others have created.

The token standard is designed in a modular fashion, leveraging [zkApp composability](https://github.com/o1-labs/o1js/issues/303). As long as the contracts involved in the token standard contract suite follow the predefined interfaces, the interoperability aspects of the token standard will be maintained. This allows for rich integrations with third party smart contracts, without the need to upgrade the token contracts themselves.

## Specification

Accounts on Mina are defined by a public key and a token id. For Mina, the token id is  [MIP4](https://github.com/MinaProtocol/MIPs/blob/main/MIPS/mip-zkapps.md#custom-tokens) allows creating new, non-Mina tokens. This requires creating an _owner_ account for the token[^1].
A user can hold some of these custom tokens when there is an account that

- is a child account of the token owner account
- the token id of the account is the token id of the custom token
- the private key of the account is controlled by user

[^1]: This is not to be confused by a user "owning" tokens in the sense of having access to the secret key of an account and thus being able to transfer those tokens. To avoid confusion, we will refer to this as _controlling_ or _holding_ tokens.

In particular, for any user that wants to hold any amount of a new custom token, a new account needs to be created, and the default account creation fee of 1 Mina needs to be paid.

We propose the following interfaces for fungible tokens:

### Transferable
Transfers are an essential feature of every token standard. In the case of o1js and its account update based smart contracts, a simple `transfer()` method would not be sufficient. Recall from [MIP4: Permissions](https://github.com/MinaProtocol/MIPs/blob/main/MIPS/mip-zkapps.md#permissions) that each Mina account can require a different kind of permission (`None`, `Either`, `Proof`, `Signature`, or `Impossible`), both for sending and receiving tokens. Therefore, the token contract cannot make any assumptions about the authorization required for the `to` and `from` accounts.

In order to be compatible with the account permission system, the Transferable interface is split into multiple methods, which allow the token contract and anyone interacting with it to issue account updates in a layout that suits their use case.
#### Interface
```TypeScript
type MayUseToken =
  | typeof AccountUpdate.MayUseToken.InheritFromParent
  | typeof AccountUpdate.MayUseToken.ParentsOwnToken;

interface TransferOptions {
  from?: PublicKey;
  to?: PublicKey;
  amount: UInt64;
  mayUseToken?: MayUseToken;
}
class TransferFromToOptions extends Struct({
  from: PublicKey,
  to: PublicKey,
  amount: UInt64,
}) {}

type FromTransferReturn = [AccountUpdate, undefined];
type ToTransferReturn = [undefined, AccountUpdate];
type FromToTransferReturn = [AccountUpdate, AccountUpdate];
type TransferReturn =
  | FromToTransferReturn
  | FromTransferReturn
  | ToTransferReturn;


interface Transferable {
  transferFromTo: (options: TransferFromToOptions) => FromToTransferReturn;

  transfer: (options: TransferOptions) => TransferReturn;
  transferFrom: (
    from: PublicKey,
    amount: UInt64,
    mayUseToken: MayUseToken
  ) => FromTransferReturn;
  transferTo: (
    to: PublicKey,
    amount: UInt64,
    mayUseToken: MayUseToken
  ) => ToTransferReturn;
}
```
#### Usage
Deciding which method to call is based on the underlying permissions of the `from` and `to` accounts. For convenience, all the available interface methods are wrapped under `transfer(...)`, which decides which underlying implementation to call based on the parameters provided.

In the following, we give a breakdown of what parameters should be used, depending on the account permissions.

##### Simple transfer between non-zkApp accounts
```TypeScript
// from: `signature`, to: `none`
token.transfer({ from, to, amount });
```

##### Withdrawal from a zkApp
```TypeScript
// from: `proof`, to: `none`

// from
// zkApp -> issue a child account update to decrease its own balance, which can be proven
token.transfer({ from, amount });

// to
// Mina.transaction -> issue a top-level account update to increase the balance of the recipient
token.transfer({ to, amount });
```

##### Transfer between two zkApps
```TypeScript
// from: `proof`, to: `proof`

// from
// zkApp -> issue a child account update to decrease its own balance, which can be proven
token.transfer({ from, amount });

// to
// zkApp -> issue a child account update to increase its own balance, which can be proven
token.transfer({ to, amount });
```

### Approvable (WIP)

> This section is currently work in progress, as we are converging on an efficient way of implementing `Approvable`.

In addition to the individual account updates providing a proof or signature for that account, the token owner contract needs to approve the set of account updates for a given zkApp transaction. That allows the token owner to define a set of rules that must be obeyed in all transactions involving their token.

A common example for a rule that many fungible tokens will want to enforce is balancedness of transactions: for Mina, the system itself checks that for every transaction, the overall sum of Mina tokens in the system does not change. Without this enforcement, users would be able to implicitly mint Mina in their transactions. For custom tokens, no such rule is automatically enforced by the system. Rather, such a rule will need to be enforced in the implementation of `Approvable` for a given token.

### Administration Interfaces: Mintable, Burnable, Pausable, Upgradeable
We define the following interfaces for administrative actions on the token: changing the supply through minting and burning, temporarily pausing transactions involving the token, and changing the token owner verification key.

#### Interfaces
```TypeScript
interface Mintable {
  totalSupply: State<UInt64>;
  circulatingSupply: State<UInt64>;
  mint: (to: PublicKey, amount: UInt64) => AccountUpdate;
  setTotalSupply: (amount: UInt64) => void;
}

interface Burnable {
  burn: (from: PublicKey, amount: UInt64) => AccountUpdate;
}

interface Pausable {
  paused: State<Bool>;
  setPaused: (paused: Bool) => void;
}

interface Upgradable {
  setVerificationKey: (verificationKey: VerificationKey) => void;
}
```

### Viewable
The Viewable interface provides a set of non-method functions on the token (owner) contract, which can be used to inspect state of the token (owner) contract itself, or its token accounts. Each viewable function offers an option to enable certain assertions, to support a case where the view function may be used in a third party smart contract.

#### Interface
```TypeScript
interface Viewable {
  getAccountOf: (address: PublicKey) => ReturnType<typeof Account>;
  getBalanceOf: (address: PublicKey, options: ViewableOptions) => UInt64;
  getTotalSupply: (options: ViewableOptions) => UInt64;
  getCirculatingSupply: (options: ViewableOptions) => UInt64;
  getDecimals: () => UInt64;
  getPaused: (options: ViewableOptions) => Bool;
  getHooks: (options: ViewableOptions) => PublicKey;
}
```

#### Usage
```TypeScript
// zkApp

@method runIfEnoughBalance(address: PublicKey) {
  const token = new Token(tokenAddress);

  // creates a precondition, that the balance of the given address
  // is the same as when this method was proven
  const balance = token.getBalanceOf(address);
  const minimalBalance = UInt64.from(1000);

  balance.assertGreaterThanOrEqual(minimalBalance);
}
```

### Hookable
The Hookable interface provide a non-invasive way to extend the behavior of the token’s implementation. Hooks can be used to intercept certain features, such as transfers or admin-ish actions. Hooks are implemented as a standalone contract, which is called by the token owner contract during certain points of execution. The hooks contract is referenced by an address in the token owner contract itself.

#### Interface

```TypeScript
// for the Token (owner) contract
interface Hookable {
  hooks: State<PublicKey>;
}

// for the Hooks contract
interface Hooks {
  canAdmin: (action: AdminAction) => Bool;
  canTransfer: ({ from, to, amount }: TransferFromToOptions) => Bool;
}
```

## Rationale

> TODO: Write an explanation of how this is different from ERC-20, and why.

## Backwards Compatibility

This is the first version of a fungible token standard on Mina. The MIP does not propose changes to Mina itself. Consequently, backwards compatibility is not an issue we need to address.

## Test Cases

The below reference implementation contains a test suite.

> TODO: ensure the test suite is comprehensive.

## Reference Implementation

The complete reference implementation is contained in the repository https://github.com/stove-labs/mip-token-standard

## Security Considerations

All MIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. E.g. include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. MIP submissions missing the "Security Considerations" section will be rejected. An MIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
