---
title: Bank
weight: 40
type: docs
---

{{< alert >}}
**Note**

XPLA Chain's bank module inherits from the Cosmos SDK's [`bank`](https://docs.cosmos.network/v0.45/modules/bank/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

The bank module is the base transactional layer of the XPLA Chain. This module allows assets to be sent from one `Account` to another. The bank module defines the following types of send transactions: `MsgSend` and `MsgMultiSend`.

The bank module is responsible for handling multi-asset coin transfers between accounts and tracking special-case pseudo-transfers which must work differently with particular kinds of accounts (notably delegating/undelegating for vesting accounts). It exposes several interfaces with varying capabilities for secure interaction with other modules which must alter user balances.

In addition, the bank module tracks and provides query support for the total supply of all assets used in the application.

This module will be used in the XPLA Chain.

### Supply

The `supply` functionality:

- passively tracks the total supply of coins within a chain,
- provides a pattern for modules to hold/interact with `Coins`, and
- introduces the invariant check to verify a chain's total supply.

#### Total Supply

The total `Supply` of the network is equal to the sum of all coins from the
account. The total supply is updated every time a `Coin` is minted (eg: as part
of the inflation mechanism) or burned (eg: due to slashing or if a governance
proposal is vetoed).

### Module Accounts

The supply functionality introduces a new type of `auth.Account` which can be used by
modules to allocate tokens and in special cases mint or burn tokens. At a base
level these module accounts are capable of sending/receiving tokens to and from
`auth.Account`s and other module accounts. This design replaces previous
alternative designs where, to hold tokens, modules would burn the incoming
tokens from the sender account, and then track those tokens internally. Later,
in order to send tokens, the module would need to effectively mint tokens
within a destination account. The new design removes duplicate logic between
modules to perform this accounting.

The `ModuleAccount` interface is defined as follows:

```go
type ModuleAccount interface {
  auth.Account               // same methods as the Account interface

  GetName() string           // name of the module; used to obtain the address
  GetPermissions() []string  // permissions of module account
  HasPermission(string) bool
}
```

{{< alert context="warning" >}}
**WARNING**

Any module or message handler that allows either direct or indirect sending of funds must explicitly guarantee those funds cannot be sent to module accounts (unless allowed).
{{< /alert>}}

The supply `Keeper` also introduces new wrapper functions for the auth `Keeper`
and the bank `Keeper` that are related to `ModuleAccount`s in order to be able
to:

- Get and set `ModuleAccount`s by providing the `Name`.
- Send coins from and to other `ModuleAccount`s or standard `Account`s
  (`BaseAccount` or `VestingAccount`) by passing only the `Name`.
- `Mint` or `Burn` coins for a `ModuleAccount` (restricted to its permissions).

#### Permissions

Each `ModuleAccount` has a different set of permissions that provide different
object capabilities to perform certain actions. Permissions need to be
registered upon the creation of the supply `Keeper` so that every time a
`ModuleAccount` calls the allowed functions, the `Keeper` can lookup the
permissions to that specific account and perform or not the action.

The available permissions are:

- `Minter`: allows for a module to mint a specific amount of coins.
- `Burner`: allows for a module to burn a specific amount of coins.
- `Staking`: allows for a module to delegate and undelegate a specific amount of coins.

## Message Types

### MsgSend

`MsgSend` transfers funds from a source account to a destination account.

```go
// MsgSend represents a message to send coins from one account to another.
type MsgSend struct {
	FromAddress string                                   `protobuf:"bytes,1,opt,name=from_address,json=fromAddress,proto3" json:"from_address,omitempty" yaml:"from_address"`
	ToAddress   string                                   `protobuf:"bytes,2,opt,name=to_address,json=toAddress,proto3" json:"to_address,omitempty" yaml:"to_address"`
	Amount      github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,3,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

The Bank module is used to send coins from one XPLA Chain account to another. `MsgSend` is constructed to facilitate the transfer. If the balance of coins in the sender `Account` is insufficient or the recipient `Account` is unable to receive the funds via the bank module, the transaction fails. Fees already paid through failed transactions are not refunded.

### MsgMultiSend

```go
// MsgMultiSend represents an arbitrary multi-in, multi-out send message.
type MsgMultiSend struct {
	Inputs  []Input  `protobuf:"bytes,1,rep,name=inputs,proto3" json:"inputs"`
	Outputs []Output `protobuf:"bytes,2,rep,name=outputs,proto3" json:"outputs"`
}

// Input models transaction input.
type Input struct {
	Address string                                   `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	Coins   github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=coins,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"coins"`
}

// Output models transaction outputs.
type Output struct {
	Address string                                   `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	Coins   github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=coins,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"coins"`
}
```

To send multiple transactions at once, use `MsgMultiSend`. For each transaction, `Inputs` contains the incoming transactions, and `Outputs` contains the outgoing transactions. The `Inputs` coin balance must exactly match the `Outputs` coin balance. Batching transactions via `MsgMultiSend` conserves gas fees and network bandwidth. Fees already paid through failed transactions are not refunded.

## CW20 & ERC20 Support

XPLA Chain's bank module has been extended to support both CosmWasm cw20 tokens and EVM erc20 tokens, allowing users to interact with these tokens using standard bank module commands without additional configuration.

### Supported Token Types

- **cw20 tokens**: Use denom format `xcw20/<contract address>`
- **erc20 tokens**: Use denom format `xerc20/<contract address>`

### How to Use

#### 1. Balance Queries

Query the balance of a specific token for an account:

```bash
# cw20 token balance
xplad q bank balance <account_address> xcw20/<contract_address>

# erc20 token balance  
xplad q bank balance <account_address> xerc20/<contract_address>
```

Example:
```bash
xplad q bank balance xpla1xr3rq8yvd7qplsw5yx90ftsr2zdhg4e9z60h5duusgxpv72hud3s5c7wrn xerc20/B99a63f7e1BD195f65e2EBFcFC393897D73F24c9
```

#### 2. Total Supply Queries

Query the total supply of a specific token:

```bash
# cw20 token total supply
xplad q bank total-supply-of xcw20/<contract_address>

# erc20 token total supply
xplad q bank total-supply-of xerc20/<contract_address>
```

Example:
```bash
xplad q bank total-supply-of xerc20/B99a63f7e1BD195f65e2EBFcFC393897D73F24c9
```

#### 3. Token Transfers

Send tokens between accounts using the standard bank send command:

```bash
# cw20 token transfer
xplad tx bank send <sender> <recipient> <amount>xcw20/<contract_address> --from <sender> --gas auto --gas-adjustment 1.4

# erc20 token transfer
xplad tx bank send <sender> <recipient> <amount>xerc20/<contract_address> --from <sender> --gas auto --gas-adjustment 1.4
```

Example:
```bash
xplad tx bank send sender1 xpla14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s525s0h 1xerc20/B99a63f7e1BD195f65e2EBFcFC393897D73F24c9 --from sender1 --gas auto --gas-adjustment 1.4
```

### Advantages

- **Unified Interface**: Use the same bank module commands for native XPLA tokens, cw20 tokens, and erc20 tokens
- **No Additional Configuration**: Works out of the box without requiring special setup
- **Standard Operations**: All standard bank module operations (balance, send, multi send, supply of) are supported
