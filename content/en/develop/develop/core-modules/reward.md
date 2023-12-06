---
title: Reward
weight: 150
type: docs
---

The reward module supports settlement to validators.

## Concepts

`RewardDistributeAccount` stakes XPLA and a portion of all delegation rewards accrued in `RewardDistributeAccount` are moved to the `reward module account` and devoted to settlement funds, which help validators to settle down on the XPLA Chain.

## Transitions

### Begin-Block

Each abci begin block call, the reward module claims all delegation rewards of `RewardDistributeAccount`. It claims every block, and this claimed reward is distributed to the `reward module account`, `community pool`, and `reserve account`. The ratio among recipients follows the [parameters]({{< ref "reward#parameters" >}}). The amount collected in the `reward module account` is divided by the number of expected blocks per year, and 1 block reward is sent to the `fee pool`. Finally, the rewards in the `fee pool` is distributed to all the validators in the active set and their delegators.

## Parameters

The subspace for the reward module is `reward`.

```go
// Params defines the set of params for the reward module.
type Params struct {
	FeePoolRate             github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,1,opt,name=fee_pool_rate,json=feePoolRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"fee_pool_rate" yaml:"fee_pool_rate"`
	CommunityPoolRate       github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=community_pool_rate,json=communityPoolRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"community_pool_rate" yaml:"community_pool_rate"`
	ReserveRate             github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=reserve_rate,json=reserveRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"reserve_rate" yaml:"reserve_rate"`
	ReserveAccount          string                                 `protobuf:"bytes,4,opt,name=reserve_account,json=reserveAccount,proto3" json:"reserve_account,omitempty"`
	RewardDistributeAccount string                                 `protobuf:"bytes,5,opt,name=reward_distribute_account,json=rewardDistributeAccount,proto3" json:"reward_distribute_account,omitempty"`
}
```
