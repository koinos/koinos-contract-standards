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

### name

This simply returns the name of the collection.

### symbol

This returns a symbol representing this collection of tokens (NFTs).

### uri

A universal resource identifier that is a link to metadata formatted in `JSON` for individual items within the collection (each NFT). This is usually an `https` or `ipfs` link. The URI is the base path and the individual metadata contained is at `/<item number>`. As an example, if `https://somekoinosexample.com/mycollection` were the URI, and the first token is the hex string representation of '1', then the 1st item's metadata would be located at `https://somekoinosexample.com/mycollection/0x31` and the 2nd would be at `/0x32` and so forth.

This is one area where the implementation differs slightly from ERC-721. With ERC-721 a URI is set per-token, here it is set for the whole collection and the full path for a token's metadata is assumed based on the constructed path.

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

This method allows the contract owner to set royalties (optional). The max allowed is 10% in this implementation.

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