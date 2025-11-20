---
title: Feemarket
weight: 110
type: docs
---

{{< alert >}}
**Note**

CONX Chain's feemarket module inherits from Evmos's [`feemarket`](https://docs.evmos.org/protocol/modules/feemarket) module. This document is a stub and covers mainly important CONX Chain-specific notes about how it is used.
{{< /alert >}}

The feemarket module which allows to define a global transaction fee for the network.

This module has been designed to support EIP1559 in cosmos-sdk.

The `MempoolFeeDecorator` in `x/auth` module needs to be overrided to the `minimal-gas-prices`.

This module will be used in the CONX Chain.

## Concepts

The transaction fee is calculated with

```
fee = gasPrice * gasLimit
```

where `gasPrice` is the price per gas and `gasLimit` describes the amount of gas required to perform the transaction.
The more complex operations a transaction requires, the higher the gasLimit (See [Executing EVM bytecode](https://docs.evmos.org/protocol/modules/evm#concepts)).
To submit a transaction, the signer needs to specify the `gasPrice`.

{{< alert >}}
The Cosmos SDK uses a different terminology for `gas` than Ethereum.
What is called `gasLimit` on Ethereum is called `gasWanted` on Cosmos.
You might encounter both terminologies on CONX Chain since it builds Ethereum on top of the SDK,
e.g. when using different wallets like Keplr for Cosmos and Metamask for Ethereum.
{{< /alert >}}

## Transitions

### End block

The block_gas_used value is updated at the end of each block.

#### Block Gas Used

The total gas used by current block is stored in the KVStore at `EndBlock`.

It is initialized to `block_gas` defined in the genesis.

## Parameters

The `x/feemarket` module contains the following parameters:

```go
// Params defines the EVM module parameters
type Params struct {
	// no_base_fee forces the EIP-1559 base fee to 0 (needed for 0 price calls)
	NoBaseFee bool `protobuf:"varint,1,opt,name=no_base_fee,json=noBaseFee,proto3" json:"no_base_fee,omitempty"`
	// base_fee_change_denominator bounds the amount the base fee can change
	// between blocks.
	BaseFeeChangeDenominator uint32 `protobuf:"varint,2,opt,name=base_fee_change_denominator,json=baseFeeChangeDenominator,proto3" json:"base_fee_change_denominator,omitempty"`
	// elasticity_multiplier bounds the maximum gas limit an EIP-1559 block may
	// have.
	ElasticityMultiplier uint32 `protobuf:"varint,3,opt,name=elasticity_multiplier,json=elasticityMultiplier,proto3" json:"elasticity_multiplier,omitempty"`
	// enable_height defines at which block height the base fee calculation is enabled.
	EnableHeight int64 `protobuf:"varint,5,opt,name=enable_height,json=enableHeight,proto3" json:"enable_height,omitempty"`
	// base_fee for EIP-1559 blocks.
	BaseFee github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,6,opt,name=base_fee,json=baseFee,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"base_fee"`
	// min_gas_price defines the minimum gas price value for cosmos and eth transactions
	MinGasPrice github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,7,opt,name=min_gas_price,json=minGasPrice,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"min_gas_price"`
	// min_gas_multiplier bounds the minimum gas used to be charged
	// to senders based on gas limit
	MinGasMultiplier github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,8,opt,name=min_gas_multiplier,json=minGasMultiplier,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"min_gas_multiplier"`
}
```
