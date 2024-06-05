---
KCS: 3
title: Token Standard that mimics ERC-20
description: A standard interface for tokens
authors: Julián González (https://github.com/joticajulian)
status: Final
---

A contract standard for tokens on the Koinos blockchain.

## Long Description

This standard is to define how tokens can work on the Koinos blockchain. The functionality is setup to closely mimic the [ERC-20](https://eips.ethereum.org/EIPS/eip-20) standard on Ethereum. Tokens on Ethereum have become a common standard and our goal is to provide similar functionality here for Koinos users and developers who have come to expect this basic layer of functionality in a token contract.

Tokens using this standard may include additional utility and functionality beyond this standard in their smart contracting. This is only a base layer of functionality that is expected.

## Why

The KCS-1 token standard is a good solution for smart wallets but it puts in risks the normal accounts (typical accounts without a contract linked to them) because there are no allowances.

Let's see an example:
- Alice interacts with a scam contract.
- In the popup to sign the transaction she only sees the operation of this contract: get_airdrop(). She signs the transaction and broadcast it to the blockchain.
- Now this contract is called.
- The scam contract now calls the KOIN contract requesting a transfer of all funds from Alice.
- The KOIN contract is called.
- The KOIN contract calls the `check_authority` system call to verify if Alice has approved this operation.
- Alice doesn't have a smart contract to resolve authorizations then the blockchain checks the signatures.
- As the transaction is signed by Alice then the KOIN transfer is accepted, causing as a result the execution of the scam.

The KCS-1 contains a section explaining the authorization process and functionality of `check_authority`. The problem with this system call is that it is useful only for accounts that have smart wallets to resolve authorizations, but normal accounts are in risk and people should not interact with contracts they don't trust because the popup to sign the transaction will not be able to alert them what will happen behind the scenes (the blockchain also has the `broadcast:false` option to be able to check events without broadcasting the transaction, but this option has also security risks not covered in this article).

Take into account that we just mentioned the KOIN contract in the example, but the scam could call all assets at the same time (tokens using KCS-1 standard) and steal them all.

## How this is solved on ethereum?

The [ERC-20 token](https://eips.ethereum.org/EIPS/eip-20) from ethereum solves this by the introduction of allowances. Following the previous example, Alice must approve the transfer before calling the scam contract. In simple terms, the popup to sign the transaction will show to Alice that she is authorizing an `spender` to transfer her tokens. In this way she will know in advance what will happen during the execution of the transaction.

## What changes between KCS-1 and KCS-3?

KCS-3 introduces allowances. At the same time the `check_authority` system call is not used anymore. As consecuence, if the user has a smart wallet to resolve authorizations it will not be called. If the user wants to use smart wallets then the smart wallet should call the token contract in order to set the allowance.

For reference, the KCS-4 token standard resolves the security risks mentioned here and takes back the `check_authority` to be able to use smart wallets to resolve authorizations, but for the present KCS-3 standard this option is removed.

## Specification

At a minimum, a token contract using this standard will include the following methods and unique data:

### Read methods

#### name

Returns the name of the token. No arguments required.

```ts
name(): token.str {}
```

Protobuffers:

```proto
// Return
message str {
   string value = 1;
}
```

#### symbol

Returns the symbol for the token. No arguments required.

```ts
symbol(): token.str {}
```

Protobuffers:

```proto
// Return
message str {
   string value = 1;
}
```

#### decimals

Returns the decimal precision of the token. No arguments required.

```ts
decimals(): token.uint32 {}
```

Protobuffers:

```proto
// Return
message uint32 {
   uint32 value = 1;
}
```

#### get_info (optional)

Returns name, symbol, and decimals in a single call.

```ts
get_info(): token.info {}
```

Protobuffers:

```proto
// Return
message info {
   string name = 1;
   string symbol = 2;
   uint32 decimals = 3;
};
```

#### total_supply

Returns the total supply of the token. No arguments required.

```ts
total_supply(): token.uint64 {}
```

Protobuffers:

```proto
// Return
message uint64 {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### balance_of

Returns how many tokens a specific address holds.

```ts
balance_of(args: token.balance_of_args): token.uint64 {}
```

Protobuffers:

```proto
// Arguments
message balance_of_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
}

// Return
message uint64 {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### allowance

Returns the allowance defined for a specific account.

```ts
allowance(args: token.allowance_args): token.uint64 {}
```

Protobuffers:

```proto
// Arguments
message allowance_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes spender = 2 [(koinos.btype) = ADDRESS];
}

// Return
message uint64 {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### get_allowances

Returns a paginated list of allowances defined in an account.

```ts
get_allowances(args: token.get_allowances_args): token.get_allowances_return {}
```

Protobuffers:

```proto
enum direction {
   ascending = 0;
   descending = 1;
}

// Arguments
message get_allowances_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = ADDRESS];
   int32 limit = 3;
   direction direction = 4;
}

// Return
message get_allowances_return {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   repeated spender_value allowances = 2;   
}
```

### Write methods

#### approve

Grant permissions to other account to manage the tokens owned by the user.

```ts
approve(args: token.approve_args): void {}
```

Protobuffers

```proto
// Arguments
message approve_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes spender = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
}
```

#### mint

Used by the contract owner to initially mint the token to a given address.

```ts
mint(args: token.mint_args): void {}
```

Protobuffers:

```proto
// Arguments
message mint_args {
   bytes to = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
}
```

#### transfer

This will transfer tokens to a new owner. The authorization is checked with the native `check_authority` system call.

```ts
transfer(args: token.transfer_args): void {}
```

Protobuffers:

```proto
// Arguments
message transfer_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
   string memo = 4;
}
```

#### burn (optional)

Burns an amount of token from an address. The authorization is checked with the native `check_authority` system call.

```ts
burn(args: token.burn_args): void {}
```

Protobuffers:

```proto
// Arguments
message burn_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
}
```

## Expected Unique Data and Types

With the proposed implementation developers would set the following constants before uploading their token contract:

- `NAME` - a string for the human readable name of the token.
- `SYMBOL` - a string for the symbol or ticker used for the token (all uppercase).
- `DECIMALS` - a u32 for the decimal precision of the token.

## Implementation

The implementation of this token contract can be found at:

- [Token contract v1.0.3 - @koinosbox/contracts@v1.2.4](https://github.com/joticajulian/koinos-contracts-as/blob/v1.2.4/contracts/token/assembly/Token.ts).

## References

- [Improve security in koinos (by @jga)](https://peakd.com/koinos/@jga/improve-security-koinos).
