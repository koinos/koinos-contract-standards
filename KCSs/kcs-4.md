---
KCS: 4
title: Token Standard that mimics ERC-20 and supports Koinos authority system
description: A standard interface for tokens
authors: Julián González (https://github.com/joticajulian)
status: Pending
---

A contract standard for tokens on the Koinos blockchain.

> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and  "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Long Description

This standard is to define how tokens can work on the Koinos blockchain. The functionality is setup to closely mimic the [ERC-20](https://eips.ethereum.org/EIPS/eip-20) standard on Ethereum. At the same time, it supports the Koinos authority system, which is useful for smart wallets.

## Why

This standard takes the best of the token standards KCS-1 and KCS-3. On one hand, the KCS-1 is useful for smart wallets by using the Koinos authority system, however it puts in risk the normal accounts. On the other hand, the KCS-3 resolves the risks of normal accounts but it ignores the Koinos authority system.

The difficulty of merging both standards is because before 2024-02-12 there was not possible to classify accounts by type (smart wallets / normal accounts) inside the token contract. But now this is possible thanks to `get_contract_metadata` system call, then the KCS-4 token standard have allowances to put normal accounts in safe place and supports the Koinos authority system for smart wallets.

## Allowance and check authority

### Authorization

Authorization is a key component of this standard. There is a more complex authority check that is used by most functions. For ease of implementation, the AS-SDK v1.2.0 and newer contains the function `System.checkAccountAuthority` which implements the new logic. By default

The function to verify authorizations is extended in this way:

1. Check if the caller of the operation is allowed to do it by checking the allowances.
2. If the contract caller is the owner of the tokens then the operation is accepted.
3. If there is a contract caller AND the owner doesn't have a contract to resolve authorizations then the operation is rejected.
4. If the points above do not apply then use the native check authority function:
  a. If the owner has a smart contract to resolve authorizations then call it.
  b. If the owner does not have a smart contract then check if the transaction is signed by the owner.

Consider the case where the owner uses a normal account (not smart wallet). If he interacts directly with the token contract then the signature will be validated (point 4.b). If he interacts with a DEX and the DEX makes a transfer in name of the owner then the allowance will be validated (point 1), otherwise it will be rejected (point 3). In this way allowances protect the assets of the users, because they have to set them in advance before a third contract tries to execute a transfer.

Now consider the case where the owner has a smart wallet. Consider also that he does not set any allowance in the token contract. If he interacts directly with the token contract then the smart wallet will be called (point 4.a). If he interacts with a DEX and the DEX makes a transfer in name of the owner then the smart wallet will be called as well (point 4.a). This means that the token contract supports the Koinos authority system, which is useful for smart wallets.

This authorization is required for `transfer` and `burn`. Steps 2-4 are required for `approve`. For ease of implementation, the AS-SDK v1.2.0 and newer contains the function `System.checkAccountAuthority` which implements steps 2-4 outlines above. In addition, the existing function `System.requireAuthority` has an additional parameter for enhanced security that will call `checkAccountAuthority`. By default, `requireAuthority` will use the enhanced security, so any contract already using `requireAuthority` can take advantage of the improved security outlined above by simply upgrading their SDK version.

## Specification

At a minimum, a token contract using this standard must include the following methods and events. Some methods in this specification are optional and are documented accordingly.

A token contract should check inputs and return meaningful error messages. For example, if a transfer argument does not contain one of the addresses should exit with a meaningful error message.

### Memos

Some functions allow for user specified memos. These are `mint`, `transfer`, `burn`, and `approve`. For the purposes of this specification, they are not intended for internal use by a token, but are there for protocols external to the blockchain to take advantage of. Please follow these guidelines for all `memo` fields.

The memo field is optional for the user, but must be supported by the contract. The contents of the memo must not be error checked as the contents could include non-ASCII characters such as Unicode for international messages or the entire byte range for encrypted/encoded messages.

If a user provides a memo, it must be included in the corresponding event (e.g. If a memo is present on `transfer_arguments` it must be present on the corresponding `transfer_event` if the transfer succeeds).

### Read methods

#### name

Returns the name of the token. No arguments required.

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

Returns the symbol for the token. No arguments required.

Protobuf definition:

```proto
// Arguments
message symbol_arguments {}
// Result
message symbol_result {
   string value = 1;
}
```

#### decimals

Returns the decimal precision of the token. No arguments required.

Protobuf definition:

```proto
// Arguments
message decimals_arguments {}
// Result
message decimals_result {
   uint32 value = 1;
}
```

#### total_supply

Returns the total supply of the token. No arguments required.

Protobuf definition:

```proto
// Arguments
message total_supply_arguments {}
// Result
message total_supply_result {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### balance_of

Returns how many tokens a specific address holds.

Protobuf definition:

```proto
// Arguments
message balance_of_arguments {
   bytes owner = 1 [(btype) = ADDRESS];
}
// Result
message balance_of_result {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### get_info (optional)

Returns name, symbol, and decimals in a single call.

Protobuf definition:

```proto
// Arguments
message get_info_arguments {}
// Result
message get_info_result {
   string name = 1;
   string symbol = 2;
   uint32 decimals = 3;
};
```

#### allowance

Returns the allowance defined for a specific account.

Protobuf definition:

```proto
// Arguments
message allowance_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes spender = 2 [(koinos.btype) = ADDRESS];
}
// Result
message allowance_result {
   uint64 value = 1 [jstype = JS_STRING];
}
```

#### get_allowances

Returns a paginated list of allowances defined in an account.

Protobuf definition:

```proto
message spender_value {
   bytes spender = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
}

// Arguments
message get_allowances_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes start = 2 [(koinos.btype) = ADDRESS];
   int32 limit = 3;
   bool descending = 4;
}

// Result
message get_allowances_result {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   repeated spender_value allowances = 2;
}
```

### Write methods

#### mint

Used by the contract owner to initially mint the token to a given address. Authorization of the mint function is left up to the developer. Most contracts will require the contract itself to authorize (sign with the key associated with the contract) or delegate to another address that is either another contract or an address the developer controls.

Protobuf definition:

```proto
// Arguments
message mint_arguments {
   bytes to = 1 [(btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
   optional string memo = 3;
}
// Result
message mint_result {}
```

The method must emit a `mint_event` upon success. The event must indicate the recipient of the mint as an impacted account. The name of the event must be `token.mint_event`.

```proto
// Event
message mint_event {
   bytes to = 1 [(btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
   optional string memo = 3;
}
```

#### transfer

This will transfer tokens to a new owner. Transfer is authorized by the improved authority outlined [above](#authorization).

Protobuf definition:

```proto
// Arguments
message transfer_arguments {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   bytes to = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
   optional string memo = 4;
}
// Result
message transfer_result {}
```

The transfer event must emit a `transfer_event` upon success. The event must indicate the receiver and then the sender as impacted accounts. The name of the event must be `token.transfer_event`. If a memo is provided by the user, it must be included in the event.

```proto
// Event
message transfer_event {
   bytes from = 1 [(btype) = ADDRESS];
   bytes to = 2 [(btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
   optional string memo = 4;
}
```

#### burn (optional)

Burns an amount of token from an address. Burn is authorized by the improved authority outlined [above](#authorization).

Protobuf definition:

```proto
// Arguments
message burn_arguments {
   bytes from = 1 [(koinos.btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
   optional string memo = 3;
}
// Result
message burn_result {}
```

The method must emit a `burn_event` upon success. The event must indicate the source as an impacted account. The name of the event must be `token.burn_event`.

```proto
// Event
message burn_event {
   bytes from = 1 [(btype) = ADDRESS];
   uint64 value = 2 [jstype = JS_STRING];
   optional string memo = 3;
}
```

#### approve

Grant permissions to other account to manage the tokens owned by the user. Approve is authorized by the improved authority outlined [above](#authorization).

Protobuf definition:

```proto
// Arguments
message approve_arguments {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes spender = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
   optional string memo = 4;
}
// Result
message approve_result {}
```

The method must emit an `approve_event` upon success. The event must indicate the spender and then the owner as the impacted accounts. The name of the event must be `token.approve_event`.

```proto
// Event
message approve_event {
   bytes owner = 1 [(koinos.btype) = ADDRESS];
   bytes spender = 2 [(koinos.btype) = ADDRESS];
   uint64 value = 3 [jstype = JS_STRING];
   optional string memo = 4;
}
```

#### authorize (optional)

Optionally, token contracts can implement an authorize override that defaults to System authority using `System.checkSystemAuthority`. This allows any tokens inadvertently sent to the token contract itself to be retrieved and sent to their rightful owner via a governance proposal.

## Implementation

The implementation of this token contract can be found at:

- [Token contract v1.0.2 - @koinosbox/contracts@v2.0.2](https://github.com/joticajulian/koinos-contracts-as/blob/v2.0.2/contracts/token/assembly/Token.ts) (check also latest updates).

With this example implementation developers would set the following constants before uploading their token contract:

- `NAME` - a string for the human readable name of the token.
- `SYMBOL` - a string for the symbol or ticker used for the token (all uppercase).
- `DECIMALS` - a u32 for the decimal precision of the token.

## References

- [New system call in koinos - part1 (by @jga)](https://peakd.com/koinos/@jga/check-authority-2-testnet).
- [New system call in koinos - part2 (by @jga)](https://peakd.com/koinos/@jga/new-koinos-system-call-live-in-the-testnet)
