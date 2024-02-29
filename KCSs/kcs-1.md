---
KCS: 1
title: Token Standard
description: A standard interface for tokens
authors: q t, otherfolks <https://github.com/otherfolks>
status: Draft
---

A contract standard for tokens on the Koinos blockchain.

## Long Description

This standard is to define how tokens can work on the Koinos blockchain. The functionality is setup to closely mimic the [ERC-20](https://eips.ethereum.org/EIPS/eip-20) standard on Ethereum. Tokens on Ethereum have become a common standard and our goal is to provide similar functionality here for Koinos users and developers who have come to expect this basic layer of functionality in a token contract.

Tokens using this standard may include additional utility and functionality beyond this standard in their smart contracting. This is only a base layer of functionality that is expected.

## Why

By mimicking the Ethereum token standard this makes it easy to onboard tokens who may have already launched projects on other chains. Many tools and resources already available for working with tokens will already be compatible here and easy to implement on Koinos.

Further, by setting forth a standard for Koinos tokens, developers can be sure that token exchanges (including [KoinDX](https://app.koindx.com)) will be able to aggregate and display these tokens as intended.

## Specification

The token contract use the following proto messages:

```proto
message balance_of_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
}

message transfer_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
}

message mint_args {
   bytes to = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
}

message burn_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
}

message str {
   string value = 1;
}

message uint32 {
   uint32 value = 1;
}

message uint64 {
   uint64 value = 1 [jstype = JS_STRING];
}
```

At a minimum, a token contract using this standard will include the following methods and unique data:

### Read methods

#### name

Returns the name of the token. No arguments required.

```ts
// entry_point: 0x82a3537f
// read_only: true
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
// entry_point: 0xb76a7ca1
// read_only: true
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
// entry_point: 0xee80fd2f
// read_only: true
decimals(): token.uint32 {}
```

Protobuffers:

```proto
// Return
message uint32 {
   uint32 value = 1;
}
```

#### total_supply

Returns the total supply of the token. No arguments required.

```ts
// entry_point: 0xb0da3934
// read_only: true
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
// entry_point: 0x5c721497
// read_only: true
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

### Write methods

### mint

Used by the contract owner to initially mint the token to a given address.

```ts
// entry_point: 0xdc6f17bb
// read_only: false
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

### transfer

This will transfer tokens to a new owner. The authorization is checked with the native `check_authority` system call.

```ts
// entry_point: 0x27f576ca
// read_only: false
transfer(args: token.transfer_args): void {}
```

Protobuffers:

```proto
// Arguments
message transfer_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
}
```

### burn (optional)

Burns an amount of token from an address. The authorization is checked with the native `check_authority` system call.

```ts
// entry_point: 0x859facc5
// read_only: false
burn(args: token.transfer_args): void {}
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

There are already a handful of KCS-1 compliant tokens deployed on the Koinos network of nodes. They can be found at [KoinDX](https://app.koindx.com/swap) or [Koiner.app](https://koiner.app/). While their methods can be read/written through Koinos blockchain explorers like [KoinosBlocks](https://koinosblocks.com/) or directly through the [Koinos-CLI](https://github.com/koinos/koinos-cli).

## Where to find and how to use

There is a reference implementation for launching a token following this standard [available here](https://github.com/roaminro/koinos-sdk-as-examples/tree/main/token). It uses the AssemblyScript SDK which you can read more about [here](https://docs.koinos.io/quickstart/contract-developer-guide/) in the official Koinos smart contract developer documentation. There is also a module at [Learn Koinos](https://learnkoinos.xyz/docs/modules/) that covers Koinos developer environment set-up and token deployment to this standard.

## References

This implementation is built and extended based on the original example from [Roamin](https://github.com/roaminro)'s token contract [located here](https://github.com/roaminro/koinos-sdk-as-examples/tree/main/token).
