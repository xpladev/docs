---
title: XATP
weight: 200
---

The XATP(XPLA Alternative Transaction Protocol) module supports fee payment in CW20 tokens.

## Concepts

The XATP module allows tokens other than XPLA to be paid as transaction fees on the XPLA Chain. This feature aims to help users who do not hold XPLA to participate in the network and new projects to get onboarded into the ecosystem more seamlessly.

The module works through the following steps:

1. The token is registered through a proposal
2. The XATP module owns an account(hereafter the module account) holding XPLA to pay fees
3. When a user generates a transaction, the module pays its fee in a registered token that is higher value than the original one by `TaxRate`
4. The paid fees are sent to the XATP module, and the module pays the fee in XPLA by the module account
5. The remaining fee(equal to the user paid fee multiplies (1 - `FeePoolRate` + `TaxRate`)) is distributed to the `community pool`, `reserve account`, and `reward pool` according to the ratios in [parameters]({{< ref "xatp#parameters" >}}).

## State

### XATP

XATP objects are stored and accessed by the denom.

- Xatps: `0x11 | DenomLen (1 byte) | Denom -> ProtocolBuffer(XATP)`

```go
type XATP struct {
	Denom    string `protobuf:"bytes,1,opt,name=denom,proto3" json:"denom,omitempty"`
	Token    string `protobuf:"bytes,2,opt,name=token,proto3" json:"token,omitempty"`
	Pair     string `protobuf:"bytes,3,opt,name=pair,proto3" json:"pair,omitempty"`
	Decimals int32  `protobuf:"varint,4,opt,name=decimals,proto3" json:"decimals,omitempty"`
}
```

#### Denom

- type: `string`

`Denom` is highly recommended using a prefix that matches the number of decimal places. e.g. CTXT, which decimal point is 6, is uCTXT.

#### Token

- type: `string`

`Token` is the contract address of CW20.

#### Pair

- type: `string`

AMMs(Automated Market Makers)' pair contract address should be a trading pair including XPLA and be able to query the ratio of its pool.

#### Decimals

- type: `int32`

Indicates the number of decimal places and must be the same as the token information.

## Proposals

### RegisterXatpProposal
```go
type RegisterXatpProposal struct {
	Title       string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	Xatp        *XATP  `protobuf:"bytes,3,opt,name=xatp,proto3" json:"xatp,omitempty"`
}
```

Register XATP Proposal is a special type of proposal which, once passed, will automatically go into effect by allowing `Xatp` to be used as a transaction fee.

{{< alert context="warning" >}}
**Note**

The denom already registered to the module will be overwritten.
{{< /alert >}}


### UnregisterXatpProposal
```go
type UnregisterXatpProposal struct {
	Title       string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	Denom       string `protobuf:"bytes,3,opt,name=denom,proto3" json:"denom,omitempty"`
}
```

Unregister XATP Proposal is a special type of proposal which, once passed, will automatically go into effect by preventing the token that `Denom` represents to be used as a transaction fee.


## Transitions

### InitGenesis

`InitGenesis` initializes the XATP module genesis state by setting the `GenesisState` fields to the store. In particular, it sets the [parameters]({{< ref "xatp#parameters" >}}) and XATPs.

### ExportGenesis

The `ExportGenesis` ABCI function exports the genesis state of the XATP module. In particular, it retrieves all the XATPs information.

## Parameters

The XATP module contains the following parameters:

### Params
```go
type Params struct {
	TaxRate           github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,1,opt,name=tax_rate,json=taxRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"tax_rate" yaml:"tax_rate"`
	FeePoolRate       github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=fee_pool_rate,json=feePoolRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"fee_pool_rate" yaml:"fee_pool_rate"`
	CommunityPoolRate github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=community_pool_rate,json=communityPoolRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"community_pool_rate" yaml:"community_pool_rate"`
	ReserveRate       github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,4,opt,name=reserve_rate,json=reserveRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"reserve_rate" yaml:"reserve_rate"`
	RewardPoolRate    github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,5,opt,name=reward_pool_rate,json=rewardPoolRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"reward_pool_rate" yaml:"reward_pool_rate"`
	ReserveAccount    string                                 `protobuf:"bytes,6,opt,name=reserve_account,json=reserveAccount,proto3" json:"reserve_account,omitempty"`
}
```

### TaxRate

- type: `sdk.Dec`
- default: `0.2`

The ratio of paying in XATP denom compared to the default denom, generally with a higher value than default.

### FeePoolRate

- type: `sdk.Dec`
- default: `1`

The distribution ratio of received fees to be sent to the fee pool

### CommunityPoolRate

- type: `sdk.Dec`
- default: `0.158`

The distribution ratio of received fees to be sent to the community pool

### ReserveRate

- type: `sdk.Dec`
- default: `0.04`

The distribution ratio of received fees to be sent to the reserve account

### RewardPoolRate

- type: `sdk.Dec`
- default: `0.002`

The distribution ratio of received fees to be sent to the reward pool

### ReserveAccount

- type: `string`
- default: ``

The reserve account address
