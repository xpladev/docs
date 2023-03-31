---
title: XATP
weight: 200
---

The XATP(XPLA Alternative Transaction Protocol) module supports accept cw20 fee payment.

## Concepts

The XATP module is allows for the use of tokens other than XPLA to pay transaction fees on the XPLA Chain. This makes it easier for users who do not hold XPLA to participate in the network, easier in the onboarding of new projects.

The XATP is used through the following steps:

- Token to be used as XATP is registered through a proposal
- The XATP module owns XPLA to paid as fees
- When a user generates a transaction, they pay a fee using XATP that is higher than the original fee by `TaxRate`
- The paid XATP fee is sent by the XATP module and the module paid the fee using XPLA it owns to the network
- The remaining fee, equal to Tax, is distributed according to [parameters]({{< ref "xatp#parameters" >}}) to the `community pool`, `reserve account`, and `reward pool`.

## State

### XATP

XATPs objects should be primarily stored and accessed by the denom.

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

Denom recommends using a prefix that matches the number of decimal places. For example, if the decimal point of CTXT is 6, use uCTXT.

#### Token

- type: `string`

Token is the contract address of cw20.

#### Pair

- type: `string`

AMM's pair contract address must be a pair that can query the ratio with a pool including XPLA.

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

Register XATP Proposal is a special type of proposal which, once passed, will automatically go into effect by allow XATP to be used as a Tx fee.

{{< alert context="warning" >}}
Note: If XATP of the same denom is registered, it will be overwritten.
{{< /alert >}}


### UnregisterXatpProposal
```go
type UnregisterXatpProposal struct {
	Title       string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	Denom       string `protobuf:"bytes,3,opt,name=denom,proto3" json:"denom,omitempty"`
}
```

Unregister XATP Proposal is a special type of proposal which, once passed, will automatically go into effect by prevent XATP to be used as a Tx fee.


## Transitions

### InitGenesis

`InitGenesis` initializes the XATP module genesis state by setting the `GenesisState` fields to the store. In particular, it sets the parameters and XATPs.

### ExportGenesis

The `ExportGenesis` ABCI function exports the genesis state of the XATP module. In particular, it retrieves all the XATPs information.

## Parameters

The xatp module contains the following parameters:

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

Ratio of paying more when paying with XATP than when paying with default denom.

### FeePoolRate

- type: `sdk.Dec`
- default: `1`

Ratio of fees sent to the fee pool when fees are received with XATP

### CommunityPoolRate

- type: `sdk.Dec`
- default: `0.158`

Ratio of fees sent to the community pool when fees are received with XATP

### ReserveRate

- type: `sdk.Dec`
- default: `0.04`

Ratio of fees sent to the reserve account when fees are received with XATP

### RewardPoolRate

- type: `sdk.Dec`
- default: `0.002`

Ratio of fees sent to the reward pool when fees are received with XATP

### ReserveAccount

- type: `string`
- default: ``

Reserve account address
