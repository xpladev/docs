---
title: Auth
weight: 20
type: docs
---

{{< alert >}}
**Note**

CONX Chain's auth module inherits from Cosmos SDK's [`auth`](https://docs.cosmos.network/v0.45/modules/auth/) module. This document is a stub and covers mainly important CONX Chain-specific notes about how it is used.
{{< /alert >}}

CONX Chain's Auth module extends the functionality from Cosmos SDK's `auth` module with a modified ante handler, which applies basic transaction validity checks, such as signatures, nonces, and auxiliary fields. This module also defines a special vesting account type that handles the logic for token vesting from the XPLA presale.

## Concepts

**Note:** The auth module is different from the [authz module]({{< ref "authz" >}}).

The differences are:

* `auth` - authentication of accounts and transactions for Cosmos SDK applications and is responsible for specifying the base transaction and account types.
* `authz` - authorization for accounts to perform actions on behalf of other accounts and enables a granter to grant authorizations to a grantee that allows the grantee to execute messages on behalf of the granter.

### Gas & Fees

Fees serve two purposes for an operator of the network.

Fees limit the growth of the state stored by every full node and allow for general purpose censorship of transactions of little economic value. Fees are best suited as an anti-spam mechanism where validators are disinterested in the use of the network and identities of users.

Fees are determined by the gas limits and gas prices transactions provide, where `fees = ceil(gasLimit * gasPrices)`. Txs incur gas costs for all state reads/writes, signature verification, as well as costs proportional to the tx size. 


## Parameters

The subspace for the Auth module is `auth`.

```go
type Params struct {
  MaxMemoCharacters      uint64 `json:"max_memo_characters" yaml:"max_memo_characters"`
  TxSigLimit             uint64 `json:"tx_sig_limit" yaml:"tx_sig_limit"`
  TxSizeCostPerByte      uint64 `json:"tx_size_cost_per_byte" yaml:"tx_size_cost_per_byte"`
  SigVerifyCostED25519   uint64 `json:"sig_verify_cost_ed25519" yaml:"sig_verify_cost_ed25519"`
  SigVerifyCostSecp256k1 uint64 `json:"sig_verify_cost_secp256k1" yaml:"sig_verify_cost_secp256k1"`
}
```

### MaxMemoCharacters

The maximum permitted number of characters in the memo of a transaction.

- type: `uint64`
- genesis: `512`

### TxSigLimit

The maximum number of signers in a transaction. A single transaction can have multiple messages and multiple signers.

- type: `uint64`
- default: `7`

### TxSizeCostPerByte

The cost per byte used to compute the gas consumption of a transaction. `TxSizeCostPerByte * txsize`.

- type: `uint64`
- default: `10`

### SigVerifyCostED25519

The gas cost for verifying ED25519 signatures.

- type: `uint64`
- default: `590`

### SigVerifyCostSecp256k1

The gas cost for verifying Secp256k1 signatures.

- type: `uint64`
- default: `1000`
