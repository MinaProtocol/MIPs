---
mip: <to be assigned>
title: Mina Provider API
description: This MIP standardizes a wallet and public-provider API for Mina applications by defining a unified JSON-RPC request interface.
authors: TheMonkeyCoder
discussions-to: https://forums.minaprotocol.com/t/mip-proposal-mina-provider-api/7033/
status: Draft
type: Standards Track
category: Interface
created: 2026-04-23
---

## Abstract

This MIP standardizes a Mina Provider API for wallets, account providers, and public providers used by zkApp frontends. The API follows an EIP-1193-style `request` interface and defines a common set of RPC methods and events so applications can integrate with Mina providers through one interoperable surface. This MIP introduces a standardized JSON-RPC interface and a consistent transaction request model for Mina providers.

The specification in this proposal is intentionally brief, including only the necessary methods and events to facilitate easy adoption. Once current Mina wallets adopt this specification, more elaborate extensions can be designed in the future.

## Motivation

Mina wallets and provider implementations currently expose different JavaScript APIs, which forces zkApp developers to maintain wallet-specific integration code. This fragmentation increases implementation effort, testing overhead, and the chance of inconsistent behavior across applications.

[RFC-0008](https://github.com/MinaFoundation/Core-Grants/blob/main/RFCs/rfc-0008-wallet-provider-api.md) established a useful starting point for provider standardization by defining a minimal Mina Provider API. However, it leaves several areas underspecified for real-world wallet interoperability, including:

- lack of clear conventions for consistent request parameter structures across implementations;
- absence of standardized methods for account connection and permission revocation;
- ambiguity in naming and network identification (e.g., `chainId` vs existing Mina conventions);
- insufficient guidance for handling different transaction types in a uniform way.

These gaps lead to divergent implementations across wallets and increase the burden on application developers integrating with multiple providers.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

### Definitions

- **Provider**: A JavaScript object exposed to an application and capable of processing Mina RPC requests.
- **Wallet Provider**: A Provider that can access user accounts and perform user-authorized actions such as signing or sending transactions.
- **Public Provider**: A Provider that serves read-only network methods without requiring access to user accounts.
- **dApp**: A frontend or application that consumes the Provider.

### Provider Interface

A compliant Provider MUST implement the following methods:

```ts
interface RequestArguments {
  readonly method: string;
  readonly params?: Record<string, unknown>;
}

interface MinaProvider {
  request(args: RequestArguments): Promise<unknown>;
  on(eventName: string, listener: (...args: any[]) => void): void;
  removeListener(eventName: string, listener: (...args: any[]) => void): void;
}
```

`request` MUST accept a `method` string and MAY accept a `params` object. If a method takes no parameters, the `params` property MAY be omitted or provided as an empty object.

### RPC Conventions

All methods defined by this MIP are invoked through `request`.

- Method names MUST be unique strings.
- Method names standardized by this MIP MUST be implemented with the exact names defined below.
- Standardized methods that take inputs MUST receive those inputs as a JSON object in `params`.
- Providers MAY implement additional non-standard methods, but such methods SHOULD NOT conflict with names standardized by this MIP.

### Error Handling

Providers SHOULD reject failed requests with an error object compatible with the following shape:

```ts
interface ProviderRpcError extends Error {
  message: string;
  code: number;
  data?: unknown;
}
```

### Common Types

#### NetworkId

A `NetworkId` is a string identifier for the active Mina network, for example `mina:mainnet` or `mina:devnet`.

#### TransactionType

```ts
type TransactionType = 'payment' | 'delegation' | 'zkapp';
```

#### PaymentTransactionPayload

```ts
interface PaymentTransactionPayload {
  readonly to: string;
  readonly from?: string;
  readonly fee: string;
  readonly amount: string;
  readonly nonce?: string;
  readonly memo?: string;
  readonly validUntil?: string;
}
```

#### DelegationTransactionPayload

```ts
interface DelegationTransactionPayload {
  readonly to: string;
  readonly from?: string;
  readonly fee: string;
  readonly nonce?: string;
  readonly memo?: string;
  readonly validUntil?: string;
}
```

`to` is the delegate public key.

#### ZkappTransactionPayload

```ts
interface ZkappTransactionPayload {
  readonly zkappCommand: Record<string, unknown>;
}
```

This MIP does not prescribe the full internal schema of a `zkappCommand`. Implementers are advised to look to `o1js` for a reference representation of the `ZkappCommand` object.

#### TransactionRequest

```ts
type TransactionRequest =
  | ({
      readonly type: 'payment';
    } & PaymentTransactionPayload)
  | ({
      readonly type: 'delegation';
    } & DelegationTransactionPayload)
  | ({
      readonly type: 'zkapp';
    } & ZkappTransactionPayload);
```

The `type` field is REQUIRED for both `mina_signTransaction` and `mina_sendTransaction` and determines how the provider interprets the accompanying request payload.

#### SignedTransactionParams

```ts
interface SignedTransactionParams {
  readonly transaction: Record<string, unknown>;
}
```

#### AddChainParams

```ts
interface AddChainParams {
  readonly url: string;
}
```

### Provider Method Groups

This MIP distinguishes between Public Provider methods and Wallet Provider methods.

Public Provider methods are network-facing RPC methods that do not require access to user accounts or wallet authorization. They MAY be implemented by public HTTP providers, wallet providers, or other compatible provider implementations. Wallet Providers MAY implement or proxy Public Provider methods for convenience, but dApps MAY also call them through any compatible Public Provider.

Wallet Provider methods require wallet state, user authorization, or access to user accounts. These methods are intended for wallet providers and include account access, permission management, network management, transaction signing, and transaction signing with submission.

### Public Provider Methods

#### `mina_blockHash`

Returns the hash of the latest block known to the provider.

##### Parameters

None.

##### Returns

`string` — the current block hash.

##### Example

```json
// Request
{
  "method": "mina_blockHash"
}

// Response
{
  "result": "3NLeFzJBrAKh4BhpHFcs1DaPFkTKemgBdTY2W1EFYtrHJHfiC96Q"
}
```

#### `mina_networkId`

Returns the current Mina network identifier.

##### Parameters

None.

##### Returns

`string` — the active network ID.

##### Example

```json
// Request
{
  "method": "mina_networkId"
}

// Response
{
  "result": "mina:mainnet"
}
```

#### `mina_getBalance`

Returns the balance of an account.

##### Parameters

```ts
interface GetBalanceParams {
  readonly publicKey: string;
  readonly tokenId?: string;
}
```

##### Returns

`string` — the balance as a string quantity.

##### Example

```json
// Request
{
  "method": "mina_getBalance",
  "params": {
    "publicKey": "B62qoYJCCwNSGRw73ww5eNCZnKCvjoEtrqr9UxuLnVHamDBEwSXjaw3",
    "tokenId": "1"
  }
}

// Response
{
  "result": "5000000000"
}
```

#### `mina_getTransactionCount`

Returns the current nonce for an account.

##### Parameters

```ts
interface GetTransactionCountParams {
  readonly publicKey: string;
}
```

##### Returns

`string` — the nonce as a string quantity.

##### Example

```json
// Request
{
  "method": "mina_getTransactionCount",
  "params": {
    "publicKey": "B62qoYJCCwNSGRw73ww5eNCZnKCvjoEtrqr9UxuLnVHamDBEwSXjaw3"
  }
}

// Response
{
  "result": "42"
}
```

#### `mina_sendSignedTransaction`

Submits a previously signed transaction to the network.

##### Parameters

`SignedTransactionParams`

##### Returns

`string` — the transaction hash.

##### Example

```json
// Request
{
  "method": "mina_sendSignedTransaction",
  "params": {
    "transaction": {
      "signature": {
        "field": "2270917456437054151866310845889777237190541188364956508055930611671093285487",
        "scalar": "21449516654198770916732742168324673178939547645509705487897779421915836159965"
      },
      "input": {
        "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
        "fee": "10000000",
        "amount": "1000000000",
        "nonce": "33",
        "memo": "Offline Payment",
        "validUntil": "4294967295"
      }
    }
  }
}

// Response
{
  "result": "5Ju...txhash..."
}
```

### Wallet Provider Methods

#### `mina_accounts`

Returns the accounts currently connected to the dApp.

##### Parameters

None.

##### Returns

`string[]` — the connected account public keys.

##### Example

```json
// Request
{
  "method": "mina_accounts"
}

// Response
{
  "result": ["B62q1...", "B62q2..."]
}
```

#### `mina_requestAccounts`

Prompts the user to connect one or more accounts.

##### Parameters

None.

##### Returns

`string[]` — the connected account public keys.

##### Example

```json
// Request
{
  "method": "mina_requestAccounts"
}

// Response
{
  "result": ["B62q..."]
}
```

#### `mina_addChain`

Requests that the wallet add a network configuration.

##### Parameters

`AddChainParams`

##### Returns

`null`

##### Example

```json
// Request
{
  "method": "mina_addChain",
  "params": {
    "url": "https://api.minascan.io/node/devnet/v1/graphql"
  }
}

// Response
{
  "result": null
}
```

#### `mina_switchChain`

Requests that the wallet switch to another Mina network.

##### Parameters

```ts
interface SwitchChainParams {
  readonly networkId: string;
}
```

##### Returns

`null`

##### Example

```json
// Request
{
  "method": "mina_switchChain",
  "params": {
    "networkId": "mina:devnet"
  }
}

// Response
{
  "result": null
}
```

#### `mina_signTransaction`

Requests that the wallet sign a transaction without sending it.

##### Parameters

`TransactionRequest`

##### Returns

A wallet-defined signed transaction representation, such as a signature for payments and delegations or a signed zkApp command for zkApp transactions.

##### Example: payment

```json
// Request
{
  "method": "mina_signTransaction",
  "params": {
    "type": "payment",
    "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
    "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
    "fee": "10000000",
    "amount": "1000000000",
    "nonce": "33",
    "memo": "Offline Payment",
    "validUntil": "4294967295"
  }
}

// Response
{
  "result": {
    "field": "2270917456437054151866310845889777237190541188364956508055930611671093285487",
    "scalar": "21449516654198770916732742168324673178939547645509705487897779421915836159965"
  }
}
```

##### Example: delegation

```json
// Request
{
  "method": "mina_signTransaction",
  "params": {
    "type": "delegation",
    "to": "B62qdelegate...",
    "from": "B62qdelegator...",
    "fee": "100000000",
    "nonce": "12",
    "memo": "Delegate stake",
    "validUntil": "4294967295"
  }
}

// Response
{
  "result": {
    "field": "2270917456437054151866310845889777237190541188364956508055930611671093285487",
    "scalar": "21449516654198770916732742168324673178939547645509705487897779421915836159965"
  }
}
```

##### Example: zkapp

```json
// Request
{
  "method": "mina_signTransaction",
  "params": {
    "type": "zkapp",
    "zkappCommand": {
      "...": "..."
    }
  }
}

// Response
{
  "result": {
    "signedZkappCommand": "..."
  }
}
```

#### `mina_sendTransaction`

Requests that the wallet sign and send a transaction.

##### Parameters

`TransactionRequest`

##### Returns

`string` — the transaction hash.

##### Example: payment

```json
// Request
{
  "method": "mina_sendTransaction",
  "params": {
    "type": "payment",
    "to": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
    "from": "B62qpSphT9prqYrJFio82WmV3u29DkbzGprLAM3pZQM2ZEaiiBmyY82",
    "fee": "100000000",
    "amount": "1000000000",
    "nonce": "33",
    "memo": "Offline Payment",
    "validUntil": "4294967295"
  }
}

// Response
{
  "result": "5Ju...txhash..."
}
```

##### Example: delegation

```json
// Request
{
  "method": "mina_sendTransaction",
  "params": {
    "type": "delegation",
    "to": "B62qdelegate...",
    "from": "B62qdelegator...",
    "fee": "100000000",
    "nonce": "12",
    "memo": "Delegate stake",
    "validUntil": "4294967295"
  }
}

// Response
{
  "result": "5Ju...txhash..."
}
```

##### Example: zkapp

```json
// Request
{
  "method": "mina_sendTransaction",
  "params": {
    "type": "zkapp",
    "zkappCommand": {
      "...": "..."
    }
  }
}

// Response
{
  "result": "5Ju...txhash..."
}
```

#### `wallet_revokePermissions`

Revokes the current dApp connection permissions.

##### Parameters

None.

##### Returns

`null`

##### Example

```json
// Request
{
  "method": "wallet_revokePermissions"
}

// Response
{
  "result": null
}
```

### Events

A Wallet Provider MUST implement `on` and `removeListener` for the events defined below. Public Providers MAY implement event subscription where applicable.

#### `chainChanged`

If the network the Provider is connected to changes, it MUST emit `chainChanged` with:

```ts
type ChainChangedEvent = string;
```

The value is the new `networkId`.

#### `accountsChanged`

If the accounts available to the Provider change, it MUST emit `accountsChanged` with:

```ts
type AccountsChangedEvent = string[];
```

The value is the new list of connected account public keys.

#### `message`

The `message` event is reserved for arbitrary notifications not covered by other standardized events.

```ts
interface ProviderMessage {
  readonly type: string;
  readonly data: unknown;
}
```

### Implementation Requirements

A Provider implementation claiming compliance with this MIP:

- MUST accept omitted `params` or an empty object for methods with no parameters.
- MUST preserve the semantics of transaction submission across supported transaction types.
- SHOULD continue to expose non-standard legacy methods only for backwards compatibility and SHOULD document them separately from this MIP-compliant interface.

## Rationale

A standard provider API improves interoperability between wallets and applications, lowers the barrier for new wallet implementations, and provides developers with a predictable interface for common tasks such as connecting accounts, querying network state, signing transactions, and submitting transactions.

The specification intentionally standardizes only a compact set of widely needed methods and events. This keeps adoption friction low while leaving room for future MIPs to define extensions for message signing, subscriptions, advanced chain metadata, or richer wallet capabilities.

## Backwards Compatibility

The current APIs of Auro Wallet, Pallad, and [RFC-0008](https://github.com/MinaFoundation/Core-Grants/blob/main/RFCs/rfc-0008-wallet-provider-api.md) have been taken into consideration to minimize required changes and ensure a smooth transition and implementation.

Current wallets MAY continue to support both this standard and their existing formats during a transition period, allowing zkApps to incrementally adopt the standardized interface defined in this MIP.

Libraries and abstraction layers MAY be used to bridge differences between wallet implementations and provide a consistent developer experience. For example, tools such as [wagmina](https://github.com/wagmina/wagmina) can normalize provider behavior across wallets, as demonstrated in projects like [Scaffold Mina](https://github.com/wagmina/scaffold-mina).

This MIP does not introduce any protocol-level or consensus changes and is limited to standardizing the wallet-provider interface.

## Security Considerations

Provider objects are exposed in an untrusted JavaScript environment and MUST be treated as adversarial inputs by wallet implementations.

Wallets and providers implementing this MIP SHOULD ensure that:

- all request payloads are validated before processing;
- transaction requests are validated against the declared `type` and rejected if fields are inconsistent;
- unsupported methods and malformed parameters fail predictably rather than being silently coerced;
- permissioned methods such as `mina_requestAccounts`, `mina_signTransaction`, `mina_sendTransaction`, and `wallet_revokePermissions` are gated by explicit user authorization;
- providers do not expose private key material or other sensitive wallet state to the dApp environment.

The `type` discriminator on `mina_signTransaction` and `mina_sendTransaction` reduces one class of implementation risk by preventing ambiguous transaction interpretation. However, wallets MUST NOT rely on `type` alone; they MUST also validate the transaction body against the rules for the declared type.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
