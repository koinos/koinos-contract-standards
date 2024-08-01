---
KCS: 2
title: NFT Collection Standard
description: A token contract standard for NFT collections
authors: Ederlang<https://github.com/ederaleng>, Von Looten<https://github.com/vonlooten>, Justin W<https://github.com/jredbeard>, Dokterkraakbeen<https://github.com/Dokterkraakbeen>
status: Final
---

A token contract standard for NFT collections on the Koinos blockchain.

## Long Description

This standard is to define how NFT collections can work on the Koinos blockchain. The functionality is setup to closely mimic the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) standard on Ethereum. We believe that NFTs on Ethereum have become a common standard and our goal is to provide similar functionality here for users and developers who have come to expect this basic layer of functionality.

NFTs using this standard may include additional utility and functionality beyond this standard. This is just the base layer of functionality that is expected.

## Why

By mimicking the Ethereum NFT collection standard this makes it easy to onboard NFT projects who may have already launched collections on other chains. Many tools and resources already available for working with NFT collections will already be compatible here and easy to implement on Koinos.

Further, by setting forth a standard for NFT collections developers can be sure that NFT marketplaces (including [Kollection](https://kollection.app)) will be able to aggregate and display these NFT collections.

## Why the concept of "owner"?

On Koinos contracts can be either upgradeable or immutable. A contract address is also a standard wallet address. This default behavior on Koinos is different than Ethereum and ERC-721 contracts.

By default, the wallet that can make changes to a contract is the address that uploaded the contract. If a contract is set to immutable (by blocking authorities) then the wallet used to upload the contract can no longer make changes. In our implementation of KCS-2 there is the concept of an "owner" address and the ability to transfer that ownership. That means that a collection owner could transfer the ownership of the collection to another individual or organization even if the contract is set to be immutable. Even though this is "unnecessary" to be default behavior on Koinos because standard wallet addresses can be contract addresses it's useful to understand why the concept of `owner` is used here.

In addition to having the ability to transfer ownership, having an `owner` means that front-ends can verify the collection owner in order to set certain things like off-chain metadata for a collection (YouTube link, Discord, etc).

## Specification

At a minimum, an NFT contract using this standard will include the following methods and unique data:

### Read methods

#### name

Returns the name of the NFT. No arguments required.

Protobuf definition:

```proto
// Arguments
message name_arguments {}
// Result
message name_result {
   string value = 1;
}
```

#### symbol

Returns the symbol for the NFT. No arguments required.

Protobuf definition:

```proto
// Arguments
message symbol_arguments {}
// Result
message symbol_result {
   string value = 1;
}
```

#### uri

Returns the endpoint to resolve the metadata.

Protobuf definition:

```proto
// Arguments
message uri_arguments {}
// Result
message uri_result {
   string value = 1;
}
```

#### owner

Returns the owner of the collection.

Protobuf definition:

```proto
// Arguments
message owner_arguments {}
// Result
message owner_result {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### total_supply

Returns the total supply of the NFT collection.

Protobuf definition:

```proto
// Arguments
message total_supply_arguments {}
// Result
message total_supply_result {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### royalties

Returns the royalties configuration.

Protobuf definition:

```proto
message royalty {
   uint64 percentage = 1 [jstype = JS_STRING];
   bytes address = 2 [(koinos.btype) = ADDRESS];
}
// Arguments
message royalties_arguments {}
// Result
message royalties_result {
   repeated royalty value = 1;
}
```

#### balance_of

Returns how many NFTs a specific address holds.

Protobuf definition:

```proto
// Arguments
message balance_of_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
}
// Result
message balance_of_result {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### owner_of

Returns the owner of a specific NFT.

Protobuf definition:

```proto
// Arguments
message owner_of_arguments {
   bytes token_id = 1 [(koinos.btype) = HEX];
}
// Result
message owner_of_result {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### get_approved

Returns the account allowed to operate a specific NFT (apart from the owner).

Protobuf definition:

```proto
// Arguments
message get_approved_arguments {
   bytes token_id = 1 [(koinos.btype) = HEX];
}
// Result
message get_approved_result {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### is_approved_for_all

Returns if an account is authorized to operate all NFTs of a specific owner.

Protobuf definition:

```proto
// Arguments
message is_approved_for_all_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes operator = 2 [(koinos.btype) = ADDRESS];
}
// Return
message is_approved_for_all_result {
   bool value = 1;
}
```

### Write methods

#### transfer_ownership

Function to transfer the ownership of the collection to other account. Only the owner of the collection can perform this operation.

Protobuf definition:

```proto
// Arguments
message transfer_ownership_arguments {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
// Result
message transfer_ownership_result {}
```

The method should emit `owner_event` upon success with the name `collections.owner_event`. The event should indicate the new owner and then the previous owner as impacted accounts.

```proto
// Event
message owner_event {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
}

#### set_royalties

Function to set the royalties. Only the owner of the collection can perform this operation.

Protobuf definition:

```proto
message royalty {
   uint64 percentage = 1 [jstype = JS_STRING];
   bytes address = 2 [(koinos.btype) = ADDRESS];
}

// Arguments
message set_royalties_argument {
   repeated royalty value = 1;
}
// Result
message set_royalties_result {}
```

The method should emit `royalties_event` upon success with the name `collections.royalties_event`. The event should indicate the addresses of the royalties as impacted accounts.

```proto
message royalty {
   uint64 percentage = 1 [jstype = JS_STRING];
   bytes address = 2 [(koinos.btype) = ADDRESS];
}

// Event
message royalties_event {
   repeated royalty value = 1;
}
```

#### approve

Grant permissions to other account to manage the NFTs owned by the user.

Protobuf definition:

```proto
// Arguments
message approve_arguments {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
}
// Result
message approve_result {}
```

The method should emit `token_approval_event` upon success with the name `collections.token_approval_event`. The event should indicate the approved address and then the approver address as impacted accounts.

```proto
// Event
message token_approval_event {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
}
```

#### set_approval_for_all

Grant permissions to other account to manage all Tokens owned by the user.


Protobuf definition:

```proto
// Arguments
message set_approval_for_all_arguments {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes operator_address = 2 [(koinos.btype) = ADDRESS];
   bool approved = 3;
}
// Result
message set_approval_for_all_result {}
```

The method should emit `operator_approval_event` upon success with the name `collections.operator_approval_event`. The event should indicate the operator address and then the approver address as impacted accounts.

```proto
// Event
message operator_approval_event {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes operator_address = 2 [(koinos.btype) = ADDRESS];
   bool approved = 3;
}
```

#### mint

Used by the contract owner to initially mint the NFT to a given address.

Protobuf definition:

```proto
// Arguments
message mint_arguments {
   bytes to = 1 [(koinos.btype) = ADDRESS];
   bytes token_id = 2 [(koinos.btype) = HEX];
}
// Result
message mint_result {}
```

The method should emit `mint_event` upon success with the name `collections.mint_event`. The event should indicate the recipient of the mint as an impacted account.

```proto
// Event
message mint_event {
   bytes to = 1 [(koinos.btype) = ADDRESS];
   bytes token_id = 2 [(koinos.btype) = HEX];
}
```

#### transfer

This will transfer an NFT to a new owner. The authorization is checked using allowances and/or smart wallets as explained above.

Protobuf definition:

```proto
// Arguments
message transfer_arguments {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
}
message transfer_result {}
```

The method should emit `transfer_event` upon success with the name `collections.transfer_event`. The event should indicate the receiver and then the sender as impacted accounts.

```proto
// Event
message transfer_event {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
}
```

### burn (optional)

Burns an amount of NFT from an address. The authorization is checked using allowances and/or smart wallets as explained above.

Protobuf definition:

```proto
// Arguments
message burn_arguments {
   bytes token_id = 1 [(koinos.btype) = HEX];
}
// Result
message burn_result {}
```

The method should emit `burn_event` upon success with the name `collections.burn_event`. The event should indicate the previous owner as an impacted account.

```proto
// Event
message burn_event {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes token_id = 2 [(koinos.btype) = HEX];
}
```

## Expected Unique Data and Types

With our proposed implementation you would set the following constants before uploading your contract:

- `NAME` - a string for the human readable name of a collection
- `SYMBOL` - a string for the symbol used for this collection (all uppercase)
- `MINT_PRICE` - a u64 for the price to mint an NFT for a collection in satoshis (KOIN has a precision of 8)
- `MINT_FEE` - a boolean (true/false). If set, addresses other than the owner can mint NFTs for the `MINT_PRICE`
- `MAX_SUPPLY` - this is the maximum amount of mintable tokens (NFTs) in your collection.
- `URI` - a string that contains the metadata URI (can be ipfs or https)
- `OWNER` - A Base58 decoded Unit8Array which is the address of the initial contract owner
- `TOKEN_PAY` - this is the type of token accepted for minting. If using KOIN, for harbinger testnet, this would be `19JntSm8pSNETT9aHTwAUHC5RMoaSmgZPJ`. For mainnet, this would be `15DJN4a8SgrbGhhGksSBASiSYjGnMU8dGL`.
- `ADDRESS_PAY` - this is the address that will receive the funds for minting.

### Token ID's

The token ID is stored as a type `bytes` and is serialized as hex. On Ethereum with ERC-721, a token ID is of type `uint256` - this specific type is not available in protobuf so `bytes` is used instead because it can fit the same data that you could in a `uint256` to try to mimic this standard.

Any hex will work, however, we recommend using hex string representation for token ID's. As an example, a string of `"1"` would be `0x31`. In the case of non-numerical token ID's, such as [KAP domains](https://kap.domains/), an example string of `"kollection.koin"` would be `0x6b6f6c6c656374696f6e2e6b6f696e`.

The supplied example implementation contract will automatically mint in numerical order as hex strings.

An example NodeJS implementation for encoding/decoding hex strings:

```
function encodeHex(str) {
    return `0x${Buffer.from(str, 'utf-8').toString('hex')}`;
}

function decodeHex(str) {
    if (str.startsWith('0x')) {
        str = str.slice(2);
    }
    return `${Buffer.from(str, 'hex').toString('utf-8')}`;
}

module.exports = { encodeHex, decodeHex}
```

NodeJS can handle this using the built-in `Buffer.from` in any recent version.

If you need to do this decoding/encoding at the browser level, both `web3.js` (for Ethereum) and [koilib](https://github.com/joticajulian/koilib) for Koinos provide browser safe helper functions.

A front end implementation using Koilib might look something like this:

```
import { utils } from "koilib";

function encodeHex(str) {
    let buffer = new TextEncoder().encode(str);
    return `0x${utils.toHexString(buffer)}`;
}

function decodeHex(str) {
    if (str.startsWith('0x')) {
        str = str.slice(2);
    }
    let buffer = utils.toUint8Array(str);
    return new TextDecoder().decode(buffer);
}
```

## Events

An NFT collection contract is expected to emit certain events.

`collections.royalties_event` - emitted when `set_royalties` is used to set royalties
`collections.owner_event` - emitted when ownership is transferred to another address
`collections.mint_event` - emitted when a new NFT (or NFTs) is minted
`collections.transfer_event` - emitted when an NFT is transferred
`collections.token_approval_event` - emitted when approval is given to transfer an NFT to another wallet from another contract (such as a marketplace)

## Burning NFTs

While the concept of burning is part of the `KCS-1` token standard, it is not in `KCS-2`. The reason for this is so that the supply stays the same and the `MAX_SUPPLY` is always constant. If you want to "burn" a particular NFT so that it can no longer be accessed and used, you should instead transfer it to the standard burn account address.

## Where to find and how to use

There is a reference implementation for launching a NFT collection following this standard [available here](https://github.com/kollection-nft/collection-base). It uses the AssemblyScript SDK which you can read more about [here](https://docs.koinos.io/quickstart/contract-developer-guide/) in the official Koinos smart contract developer documentation.

## References

Our implementation is built and extended based on the original example from [Roamin](https://github.com/roaminro)'s NFT example contract [located here](https://github.com/roaminro/koinos-sdk-as-examples/tree/main/nft). Thanks Roamin!