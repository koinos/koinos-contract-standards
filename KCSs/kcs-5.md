---
KCS: 5
title: NFT Standard that mimics ERC-721 and supports Koinos authority system
description: A standard interface for NFTs
authors: Julián González (https://github.com/joticajulian)
status: Final
---

A contract standard for NFT collections on the Koinos blockchain.

## Long Description

This standard is to define how NFT collections can work on the Koinos blockchain. The functionality mimic the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) standard on Ethereum and extend it with new features. At the same time, it supports the Koinos authority system, which is useful for smart wallets.

## Why

This standard introduces new features with respect to the standard KCS-2. Here is a summary of the principal changes:

1. **Possibility to store metadata onchain**. When the `uri` is defined then the metadata is stored offchain. However, if the `uri` is undefined is because the metadata should be obtained from the contract itself, onchain. Thanks to this option, the developers don't have to setup an API for the contract.
2. **Function to get a paginated list of NFTs**. One of the main barriers of the ERC-721 and KCS-2 is that it's not possible know what is the list of NFTs in the collection, unless you setup a microservice that listen changes in the contract, or you rely in third parties offering this functionality.
3. **Function to get NFTs by owner**. Same feature as the previous one but filtered by owner.
4. **Function to list approvals**. This feauture allows users to know what are the approvals they have set to third parties. In this sense, they will have more control on their assets.
5. **Koinos System Authority**. Introduction of the new system call in Koinos to be able to classify accounts by type, in order to give safety to the users that don't have smart wallets and at the same time be able to call users' contracts to resolve authorities in case they use smart wallets.

## Allowance and check authority

The function to verify authorizations is extended in this way:

1. Check if the caller of the operation is allowed to do it by checking the allowances (approval of a single NFT, or approval for all NFTs).
2. If the contract caller is the owner of the NFT then the operation is accepted.
3. If there is a contract caller AND the owner doesn't have a contract to resolve authorizations then the operation is rejected.
4. If the points above do not apply then use the native check authority function:
  a. If the owner has a smart contract to resolve authorizations then call it.
  b. If the owner does not have a smart contract then check if the trasaction is signed by the owner.

Consider the case where the owner uses a normal account (not smart wallet). If he interacts directly with the NFT contract then the signature will be validated (point 4.b). If he interacts with a DEX and the DEX makes a transfer in name of the owner then the allowance will be validated (point 1), otherwise it will be rejected (point 3). In this way allowances protect the assets of the users, because they have to set them in advance before a third contract tries to execute a transfer.

Now consider the case where the owner has a smart wallet. Consider also that he does not set any allowance in the NFT contract. If he interacts directly with the NFT contract then the smart wallet will be called (point 4.a). If he interacts with a DEX and the DEX makes a transfer in name of the owner then the smart wallet will be called as well (point 4.a). This means that the NFT contract supports the Koinos authority system, which is useful for smart wallets.

## Specification

At a minimum, a NFT contract using this standard will include the following methods and unique data:

### Read methods

#### name

Returns the name of the NFT. No arguments required.

```ts
name(): nft.str {}
```

Protobuffers:

```proto
// Return
message str {
   string value = 1;
}
```

#### symbol

Returns the symbol for the NFT. No arguments required.

```ts
symbol(): nft.str {}
```

Protobuffers:

```proto
// Return
message str {
   string value = 1;
}
```

#### uri

Returns the endpoint to resolve the metadata. If the uri is empty then the metadata should be searched onchain by using the method `metadata_of`.

```ts
uri(): nft.str {}
```

Protobuffers:

```proto
// Return
message str {
   string value = 1;
}
```

#### get_info (optional)

Returns name, symbol, decimals, and description in a single call.

```ts
get_info(): nft.info {}
```

Protobuffers:

```proto
// Return
message info {
   string name = 1;
   string symbol = 2;
   uint32 uri = 3;
   string description = 4;
};
```

#### owner

Returns the owner of the collection.

```ts
owner(): nft.address {}
```

Protobuffers:

```proto
// Return
message address {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### total_supply

Returns the total supply of the NFT collection.

```ts
total_supply(): nft.uint64 {}
```

Protobuffers:

```proto
// Return
message uint64 {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### royalties

Returns the royalties configuration.

```ts
royalties(): nft.royalties {}
```

Protobuffers:

```proto
message royalty {
   uint64 percentage = 1 [jstype = JS_STRING];
   bytes address = 2 [(koinos.btype) = ADDRESS];
}

// Return
message royalties {
   repeated royalty value = 1;
}
```

#### balance_of

Returns how many NFTs a specific address holds.

```ts
balance_of(args: nft.balance_of_args): nft.uint64 {}
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

#### owner_of

Returns the owner of a specific NFT.

```ts
owner_of(args: nft.token): nft.address {}
```

Protobuffers:

```proto
// Arguments
message token {
   bytes token_id = 1 [(koinos.btype) = HEX];
}

// Return
message address {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### metadata_of

Returns the metadata of a specific NFT.

```ts
metadata_of(args: nft.token): nft.str {}
```

Protobuffers:

```proto
// Arguments
message token {
   bytes token_id = 1 [(koinos.btype) = HEX];
}

// Return
message str {
   string value = 1;
}
```

#### get_tokens

Returns a paginated list of NFTs in the collection.

```ts
get_tokens(args: nft.get_tokens_args): nft.token_ids {}
```

Protobuffers:

```proto
enum direction {
   ascending = 0;
   descending = 1;
}

// Arguments
message get_tokens_args {
   bytes start = 1 [(koinos.btype) = HEX];
   int32 limit = 2;
   direction direction = 3;
}

// Return
message token_ids {
   repeated bytes token_ids = 1 [(koinos.btype) = HEX];
}
```

#### get_tokens_by_owner

Returns a paginated list of NFTs owned by a specific account.

```ts
get_tokens_by_owner(args: nft.get_tokens_by_owner_args): nft.token_ids {}
```

Protobuffers:

```proto
enum direction {
   ascending = 0;
   descending = 1;
}

// Arguments
message get_tokens_by_owner_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = HEX];
   int32 limit = 3;
   direction direction = 4;
}

// Return
message token_ids {
   repeated bytes token_ids = 1 [(koinos.btype) = HEX];
}
```

#### get_approved

Returns the account allowed to operate a specific NFT (apart from the owner).

```ts
get_approved(args: nft.token): nft.address {}
```

Protobuffers:

```proto
// Arguments
message token {
   bytes token_id = 1 [(koinos.btype) = HEX];
}

// Return
message address {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### is_approved_for_all

Returns if an account is authorized to operate all NFTs of a specific owner.

```ts
is_approved_for_all(args: nft.is_approved_for_all_args): nft.boole {}
```

Protobuffers:

```proto
// Arguments
message is_approved_for_all_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes operator = 2 [(koinos.btype) = ADDRESS];
}

// Return
message boole {
   bool value = 1;
}
```

#### get_operator_approvals

Returns a paginated list of all accounts allowed to operate all NFTs of a specific owner.

```ts
get_operator_approvals(
  args: nft.get_operators_args
): nft.get_operators_return {}
```

Protobuffers:

```proto
enum direction {
   ascending = 0;
   descending = 1;
}

// Arguments
message get_operators_args {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = ADDRESS];
   int32 limit = 3;
   direction direction = 4;
}

// Return
message get_operators_return {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   repeated bytes operators = 2 [(koinos.btype) = ADDRESS];
}
```

### Write methods

#### transfer_ownership

Function to transfer the ownership of the collection to other account. Only the owner of the collection can perform this operation.

```ts
transfer_ownership(args: nft.address): void {}
```

Protobuffers:

```proto
// Arguments
message address {
   bytes value = 1 [(koinos.btype) = ADDRESS];
}
```

#### set_royalties

Function to set the royalties. Only the owner of the collection can perform this operation.

```ts
set_royalties(args: nft.royalties): void {}
```

Protobuffers:

```proto
message royalty {
   uint64 percentage = 1 [jstype = JS_STRING];
   bytes address = 2 [(koinos.btype) = ADDRESS];
}

// Arguments
message royalties {
   repeated royalty value = 1;
}
```

#### set_metadata

Function to set the metadata of the tokens. It should be a JSON converted to string.

```ts
set_metadata(args: nft.metadata_args): void {
```

Protobuffers:

```proto
// Arguments
message metadata_args {
   bytes token_id = 1 [(koinos.btype) = HEX];
   string metadata = 2;
}
```

#### approve

Grant permissions to other account to manage the NFTs owned by the user.

```ts
approve(args: nft.approve_args): void {}
```

Protobuffers:

```proto
// Arguments
message approve_args {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
}
```

#### set_approval_for_all

Grant permissions to other account to manage all Tokens owned by the user.

```ts
set_approval_for_all(args: nft.set_approval_for_all_args): void {}
```

Protobuffers:

```proto
// Arguments
message set_approval_for_all_args {
   bytes approver_address = 1 [(koinos.btype) = ADDRESS];
   bytes operator_address = 2 [(koinos.btype) = ADDRESS];
   bool approved = 3;
}
```

#### mint

Used by the contract owner to initially mint the NFT to a given address.

```ts
mint(args: nft.mint_args): void {}
```

Protobuffers:

```proto
// Arguments
message mint_args {
   bytes to = 1 [(koinos.btype) = ADDRESS];
   bytes token_id = 2 [(koinos.btype) = HEX];
}
```

#### transfer

This will transfer an NFT to a new owner. The authorization is checked using allowances and/or smart wallets as explained above.

```ts
transfer(args: nft.transfer_args): void {}
```

Protobuffers:

```proto
// Arguments
message transfer_args {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   bytes token_id = 3 [(koinos.btype) = HEX];
   string memo = 100;
}
```

### burn (optional)

Burns an amount of NFT from an address. The authorization is checked using allowances and/or smart wallets as explained above.

```ts
burn(args: nft.burn_args): void {}
```

Protobuffers:

```proto
// Arguments
message burn_args {
   bytes token_id = 1 [(koinos.btype) = HEX];
}
```

## Expected Unique Data and Types

With the proposed implementation developers would set the following constants before uploading their NFT contract:

- `NAME` - a string for the human readable name of the NFT collection.
- `SYMBOL` - a string for the symbol or ticker used for the NFT (all uppercase).
- `URI` - a string that contains the metadata URI (can be ipfs or https). When the metadata is stored onchain it should be empty.

## Implementation

The implementation of this NFT contract can be found at:

- [Nft contract v2.0.2 - @koinosbox/contracts@v2.0.2](https://github.com/joticajulian/koinos-contracts-as/blob/v2.0.2/contracts/nft/assembly/Nft.ts) (check also latest updates).
