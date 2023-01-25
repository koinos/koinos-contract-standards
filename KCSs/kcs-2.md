---
KCS: 2
title: NFT Collection Standard
description: A token contract standard for NFT collections
authors: Ederlang<https://github.com/ederaleng>, Von Looten<https://github.com/vonlooten>, Justin W<https://github.com/jredbeard>, Dokterkraakbeen<https://github.com/Dokterkraakbeen>
status: Draft
---

A token contract standard for NFT collections on the Koinos blockchain.

## Long Description

This standard is to define how NFT collections can work on the Koinos blockchain. The functionality is setup to closely mimic the [ERC-721](https://eips.ethereum.org/EIPS/eip-721) standard on Ethereum. We believe that NFTs on Ethereum have become a common standard and our goal is to provide similar functionality here for users and developers who have come to expect this basic layer of functionality.

NFTs using this standard may include additional utility and functionality beyond this standard. This is just the base layer of functionality that is expected.

## Why

By mimicing the Ethereum NFT collection standard this makes it easy to onboard NFT projects who may have already launched collections on other chains. Many tools and resources already available for working with NFT collections will already be compatible here and easy to implement on Koinos.

Further, by setting forth a standard for NFT collections developers can be sure that NFT marketplaces (including [Kollection](https://kollection.app)) will be able to aggregate and display these NFT collections.

## Specification

At a minimum, a NFT contract using this standard will include the following methods and unique data:

### name

This simply returns the name of the collection.

### symbol

This returns a symbol representing this collection of tokens (NFTs).

### uri

A universal resource identifier that is a link to metadata formatted in `JSON` for individual items within the collection (each NFT). This is usually an `https` or `ipfs` link. The URI is the base path and the individual metadata contained is at `/<item number>`. As an example, if `https://somekoinosexample.com/mycollection` were the URI, the 1st item's metadata would be located at `https://somekoinosexample.com/mycollection/0` and the 2nd would be at `/1` and so forth.

The metadata should include at a minimum a name, description, and a URI where the associated image can be found. Other properties are arbitrary and can be any properties you deem fit for your collection. Attributes are also optional but will often be used.

The JSON located at these URI's should follow this format (similar and compatible with ERC-721 format):

```
{
   "name":"Collection Nft #1",
   "description":"The first of this collection",
   "attributes":[
      {
         "trait_type":"Awesome",
         "value":"Yes"
      },
      {
         "trait_type":"Likes Standards",
         "value":"Loves Them"
      }
   ],
   "my_arbitrary_custom_property": "totally ok!",
   "image":"https://somekoinosexample.com/mycollection/images/0.png"
}
```

### total_supply

This method gets the current total supply of a collection.

### royalties

This method returns the amount of royalties (if any).

### set_royalties

This method allows the contract owner to set royalties (optional). The max allowed is 10% in our implementation.

### owner

Returns current owner of the collection contract.

### transfer_ownership

Allows updating the owner address of a collection contract.

### balance_of

Returns how many tokens (NFTs) a specific address holds of this collection.

### get_approved

Returns if a specific token is approved for transfer

### is_approved_for_all

Checks if the entire collection is approved for transfer

### mint

This method is used by the contract owner to initially mint NFTs from this collection to new owners.

### transfer

This will transfer tokens (NFTs) to a new owner if that token is approved for transfer. As part of the process it will clear the approval.

### approve

Updates the approval for transfer for a specific token (NFT).

### set_approval_for_all

Updates the approval for transfer of the collection contract to a new owner

## Expected Unique Data and Types

With our proposed implementation you would set the following constants before uploading your contract:

-`NAME` - a string for the name of the collection
-`SYMBOL` - a string for the symbol used for this collection
-`PRICE` - a u64 for the initial price (can be zero)
-`URI` - a string that contains the metadata URI
-`OWNER` - A Base58 decoded Unit8Array which is the address of the initial contract owner
-`FEE_MINT` - a boolean (true/false). If set, there will be a set fee for minting new NFTs in this collection
-`TOKEN_PAY` - if `FEE_MINT` is true, this is the address where the mint fee will be sent
-`ADDRESS_PAY` - if `FEE_MINT` is true, this is who will pay the mana for the mint event

## Where to find and how to use

There is a reference implementation for launching a NFT collection following this standard [available here](https://github.com/kollection-nft/collection-base). It uses the AssemblyScript SDK which you can read more about [here](https://docs.koinos.io/quickstart/contract-developer-guide.html) in the official Koinos smart contract developer documentation.

## References

Our implementation is built and extended based on the original example from [Roamin](https://github.com/roaminro)'s NFT example contract [located here](https://github.com/roaminro/koinos-sdk-as-examples/tree/main/nft). Thanks Roamin!