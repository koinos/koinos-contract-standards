---
KCS: 5
title: NFT Standard that mimics ERC-721 and supports Koinos authority system
description: A standard interface for NFTs
authors: Julián González (https://github.com/joticajulian)
status: Pending
---

A contract standard for NFT collections on the Koinos blockchain.

## Long Description

This standard is to define how NFT collections can work on the Koinos blockchain. The functionality mimic the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) standard on Ethereum and extend it with new features. At the same time, it supports the Koinos authority system, which is useful for smart wallets.

## Why

This standard introduces new features with respect to the standard KCS-2. Here is a summary of the principal changes:

1. **Possibility to store metadata onchain**. When the `uri` is defined then the metadata is stored offchain. However, if the `uri` is undefined is because the metadata should be obtained from the contract itself, onchain. Thanks to this option, the developers don't have to setup an API for the contract.
2. **Function to get a paginated list of NFTs**. One of the main barriers of the ERC-721 and KCS-2 is that it's not possible know what is the list of NFTs in the collection, unless you setup a microservice that listen changes in the contract, or you rely in third parties offering this functionality.
3. **Function to get NFTs by owner**. Same feature as the previous one but filtered by owner.
4. **Function to list approvals**. This feature allows users to know what are the approvals they have set to third parties. In this sense, they will have more control on their assets.
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

Returns the endpoint to resolve the metadata. If the uri is empty then the metadata should be searched onchain by using the method `metadata_of`.

Protobuf definition:

```proto
// Arguments
message uri_arguments {}
// Result
message uri_result {
   string value = 1;
}
```

#### get_info (optional)

Returns name, symbol, decimals, and description in a single call.

Protobuf definition:

```proto
// Arguments
message info_arguments {}
// Result
message info_results {
   string name = 1;
   string symbol = 2;
   uint32 uri = 3;
   string description = 4;
};
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

#### metadata_of

Returns the metadata of a specific NFT.

Protobuf definition:

```proto
// Arguments
message metadata_of_arguments {
   bytes token_id = 1 [(koinos.btype) = HEX];
}
// Result
message metadata_of_result {
   string value = 1;
}
```

#### get_tokens

Returns a paginated list of NFTs in the collection.

Protobuf definition:

```proto
// Arguments
message get_tokens_arguments {
   bytes start = 1 [(koinos.btype) = HEX];
   int32 limit = 2;
   bool descending = 3;
}
// Result
message get_tokens_result {
   repeated bytes values = 1 [(koinos.btype) = HEX];
}
```

#### get_tokens_by_owner

Returns a paginated list of NFTs owned by a specific account.

Protobuf definition:

```proto
// Arguments
message get_tokens_by_owner_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = HEX];
   int32 limit = 3;
   bool descending = 4;
}
// Result
message get_tokens_by_owner_result {
   repeated bytes values = 1 [(koinos.btype) = HEX];
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

#### get_operator_approvals

Returns a paginated list of all accounts allowed to operate all NFTs of a specific owner.

Protobuf definition:

```proto
// Arguments
message get_operator_approvals_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = ADDRESS];
   int32 limit = 3;
   bool descending = 4;
}
// Return
message get_operator_approvals_result {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   repeated bytes operators = 2 [(koinos.btype) = ADDRESS];
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
```

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
// Event
message royalties_event {
   repeated royalty value = 1;
}
```

#### set_metadata

Function to set the metadata of the tokens. It should be a JSON converted to string.

Protobuf definition:

```proto
// Arguments
message set_metadata_arguments {
   bytes token_id = 1 [(koinos.btype) = HEX];
   string metadata = 2;
}
// Result
message set_metadata_result {}
```

The method should emit `set_metadata_event` upon success with the name `collections.set_metadata_event`. The event should indicate the source as an impacted account.

```proto
// Event
message set_metadata_event {
   bytes token_id = 1 [(koinos.btype) = HEX];
   string metadata = 2;
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
   string memo = 4;
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
   string memo = 4;
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
With the proposed implementation developers would set the following constants before uploading their NFT contract:
- `NAME` - a string for the human readable name of the NFT collection.
- `SYMBOL` - a string for the symbol or ticker used for the NFT (all uppercase).
- `URI` - a string that contains the metadata URI (can be ipfs or https). When the metadata is stored onchain it should be empty.

## Implementation
The implementation of this NFT contract can be found at:
- [Nft contract v2.0.2 - @koinosbox/contracts@v2.0.2](https://github.com/joticajulian/koinos-contracts-as/blob/v2.0.2/contracts/nft/assembly/Nft.ts) (check also latest updates).