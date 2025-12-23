---
title: Overview
weight: 10
type: docs
---

The CONX Chain Core is the official Golang reference implementation of the CONX Chain.
The CONX Chain Core is built using the [Cosmos SDK](https://v1.cosmos.network/sdk), which provides a robust framework for blockchains that run atop the [Tendermint](https://tendermint.com/) consensus protocol.

Before diving into the core modules, it may be useful to familiarize yourself with the [Cosmos](https://docs.cosmos.network/v0.45/) and [Tendermint](https://docs.tendermint.com/v0.34/) documentation.

## How to Use the CONX Chain Core Module Specifications

Each module specification begins with a short description of the module's main function within the architecture of the system and an explanation of how it contributes to implementing CONX Chain's features.

The body of each module specification provides a more detailed description of its main processes and algorithms alongside any concepts you might need to know. The body of each module specification also contains links to more granular information, such as specific state variables, message handlers, and other functions.

These specifications are not an exhaustive reference and are provided as a companion guide for users who need to work directly with the CONX Chain Core codebase or understand it. Though all the important functions in each module are described, more trivial functions, such as getters and setters, are omitted for clarity. Module logic is also located in either the message handler or block transitions, such as begin-blocker and end-blocker.

The end of each module specification includes lists of various module parameters alongside their default values with a brief explanation of their purpose, associated events / tags, and errors issued by the module.

## Module Architecture

The CONX Chain Core is organized into the following individual modules that implement different parts of the CONX Chain. They are listed in the order in which they are initialized during genesis:

1. `genaccounts` - import & export genesis account
2. [`distribution`]({{< ref "distribution" >}}): distribute rewards between validators and delegators
   - reward distribution
   - community pool
3. [`staking`]({{< ref "staking" >}}): validators and XPLA
4. [`auth`]({{< ref "auth" >}}): ante handler
   - vesting accounts
5. [`bank`]({{< ref "bank" >}}) - sending funds from account to account
6. [`slashing`]({{< ref "slashing" >}}) - low-level Tendermint slashing (double-signing, etc)
7. [`gov`]({{< ref "governance" >}}): on-chain governance
    - proposals
    - parameter updating
11. `crisis` - reports consensus failure state with proof to halt the chain
12. `genutil` - handles `gentx` commands
    - filter and handle `MsgCreateValidator` messages

### Inherited Modules

Many of the modules in CONX Chain Core inherit from the Cosmos SDK and are configured to work with CONX Chain through customization in either genesis parameters or by augmenting their functionality with additional code.

## Block Lifecycle

The following processes are executed during each block transition:

### Begin Block

1\. Distribution: Issuance of rewards for the previous block.

2\. Slashing: Checking of infraction evidence or downtime of validators for double-signing and downtime penalties.

### Process Messages

3\. Messages are routed to their appropriate modules and then processed by the appropriate message handlers.

### End Block

4\. Crisis: Check all registered invariants and assert that they remain true.

5\. Governance: Remove inactive proposals, check active proposals whose voting periods have ended for passes, and run the registered proposal handler of the passed proposal.

6\. Staking: The new set of active validators is determined from the top 130 XPLA stakers. Validators that lose their spot within the set start the unbonding process.

## Conventions

### Currency Denominations

- XPLA is the CONX Chain's native staking asset. Delegators earn mining rewards when they stake their XPLA to an active validator. XPLA is also used to make and vote on governance proposals.

The attounit {{< katex >}}\times 10^{-18}{{< /katex >}} is the smallest atomic unit of XPLA.

| Denomination | Atto-Unit | Code    | Value                     |
|:-------------|:----------|:--------|:--------------------------|
| XPLA         | axpla     | `axpla` | 0.000000000000000001 XPLA |
