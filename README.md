# Koinos Contract Standard (KCS)

This repository is to track and propose smart contract standards for the Koinos Blockchian. While this is not an "official" Koinos repo, since there was nothing like this available and we [(Kollection Inc.)](https://kollection.app) had a standard to propose we decided to take the initiative to put this in place. It's our hope that this repo will eventually be forked somewhere else and can become part of an "official" process as decided by the Koinos community. Until that time we're happy to get this process rolling.

## Why standards?

For the case of tokens on Koinos (including NFTs) we believe that it's important to give developers an expectation for what format of token contracts would and would not be included on exchanges or marketplaces. If a developer wishes to launch a token contract that is in line with others listed, they would want to follow a standard to be sure that it's compatible with both interfaces and aggregators that use these standards.

It's important to note that some token contracts (including NFTs) can and will have extended utility and functionality beyond a standard, but, the standard exists so that all of these types of tokens will have a base layer of expectations for software interacting with them. If there is additional utility that's intended to be duplicated by other tokens then it may be appropriate to propose a new standard that includes this additional functionality.

## What this is not

This is not a place to submit governance proposals for the Koinos Blockchain. While Ethereum uses the [EIP](https://eips.ethereum.org/EIPS) process for both chain changes and smart contract standards, KCS is specifically for contract standards. Governance proposals can be a separate process that may require a different type of decentralization. Because of this, this process does not need to be the same as EIP and can even be less extensive. Remember, the point here is to help developers and not hinder them - we want to help provide the information they need and this process is not intended to block them.

Developers can launch any smart contract on the Koinos blockchain whether they follow standards or not. The world (and the blockchain) is your oyster.

## How to Contribute

Before submitting a standard you should thoroughly discuss it with other Koinos developers either on the [Koinos discord]() or on the [Koinos forum](https://discourse.koinosforum.com/).

Once you believe that your idea for a standard is sound and should be proposed, next you should write a draft describing the standard using markdown. Open an issue in this repo with simply the title of the standard. The GitHub issue number will become the KCS number (ex. KCS-1). The file name should be in a format like: `kcs-1.md` which includes the GitHub issue number. Open a pull request that will close the issue number. Public discussions can continue on the issue number whether it's in Draft or Final status.

If a standard submitted is filled out in the proper format, it should be merged as a Draft standard.

In order to move to Final status the proposer should be able to point to an actual live implementation of their contract standard in place on Koinos mainnet.

## Formatting a Standard

You can find a template [here](https://template_link)

At a minimum, your standard should include these things:

A header with a Title, the KCS number, Short Description, Author(s), and Status (initially Draft)

Authors can of course be anonymous if they wish.

The body should contain these sections:

### Long Description

This description should be a human readable technical summary of the standard you are proposing.

### Why

You should elaborate on what problem you're trying to solve here. Tell the world why you believe it makes sense for your idea to be a standard. What are your use cases?

### Specification

Provide a complete technical specification for your smart contract standard that includes the minimum criteria for a token contract to be considered using this standard.

### Optional

You are welcome to include any additional information describing your proposed standard. Some of these could include information about backwards compatibility with previous standards, security considerations, links to reference implementation, and anything else you deem necessary.
