---
title: Bank
weight: 40
---

{{< hint info >}}
**Note**
XPLA Chain's bank module inherits from the Cosmos SDK's [`bank`](https://docs.cosmos.network/master/modules/bank/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /hint >}}

The bank module is the base transactional layer of the XPLA Chain. This module allows assets to be sent from one `Account` to another. The bank module defines the following types of send transactions: `MsgSend` and `MsgMultiSend`.

## Message Types

### MsgSend

`MsgSend` transfers funds from a source account to a destination account.

```go
// MsgSend - high level transaction of the coin module
type MsgSend struct {
    FromAddress sdk.AccAddress `json:"from_address"`
    ToAddress   sdk.AccAddress `json:"to_address"`
    Amount      sdk.Coins      `json:"amount"`
}
```

The Bank module is used to send coins from one XPLA Chain account to another. `MsgSend` is constructed to facilitate the transfer. If the balance of coins in the sender `Account` is insufficient or the recipient `Account` is unable to receive the funds via the bank module, the transaction fails. Fees already paid through failed transactions are not refunded.

### MsgMultiSend

```go
// MsgMultiSend - high level transaction of the coin module
type Input struct {
  Address sdk.AccAddress `json:"address" yaml:"address"`
  Coins   sdk.Coins      `json:"coins" yaml:"coins"`
}

type Output struct {
  Address sdk.AccAddress `json:"address" yaml:"address"`
  Coins   sdk.Coins      `json:"coins" yaml:"coins"`
}

type MsgMultiSend struct {
  Inputs  []Input  `json:"inputs" yaml:"inputs"`
  Outputs []Output `json:"outputs" yaml:"outputs"`
}
```

To send multiple transactions at once, use `MsgMultiSend`. For each transaction, `Inputs` contains the incoming transactions, and `Outputs` contains the outgoing transactions. The `Inputs` coin balance must exactly match the `Outputs` coin balance. Batching transactions via `MsgMultiSend` conserves gas fees and network bandwidth. Fees already paid through failed transactions are not refunded.
