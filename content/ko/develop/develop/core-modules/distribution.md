---
title: Distribution
weight: 70
type: docs
---

{{< alert >}}
**Note**

XPLA Chain's distribution module inherits from Cosmos SDK's [`distribution`](https://docs.cosmos.network/v0.45/modules/distribution/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

The distribution module describes a mechanism that tracks collected fees and passively distributes them to validators and delegators. Additionally, the distribution module defines the community pool, which is a pool of funds under the control of on-chain governance.

## Concepts

In Proof of Stake (PoS) blockchains, rewards gained from transaction fees are paid to validators. The fee distribution module fairly distributes the rewards to the validators' constituent delegators.

Rewards are calculated per period. The period is updated each time a validator's delegation changes, for example, when the validator receives a new delegation.
The rewards for a single validator can then be calculated by taking the total rewards for the period before the delegation started, minus the current total rewards.
To learn more, see the [F1 Fee Distribution paper](https://github.com/cosmos/cosmos-sdk/blob/v0.45.9/docs/spec/fee_distribution/f1_fee_distr.pdf).

The commission to the validator is paid when the validator is removed or when the validator requests a withdrawal.
The commission is calculated and incremented at every `BeginBlock` operation to update accumulated fee amounts.

The rewards to a delegator are distributed when the delegation is changed or removed, or a withdrawal is requested.
Before rewards are distributed, all slashes to the validator that occurred during the current delegation are applied.

### Reference Counting in F1 Fee Distribution

In F1 fee distribution, the rewards a delegator receives are calculated when their delegation is withdrawn. This calculation must read the terms of the summation of rewards divided by the share of tokens from the period which they ended when they delegated, and the final period that was created for the withdrawal.

Additionally, as slashes change the amount of tokens a delegation will have (but we calculate this lazily,
only when a delegator un-delegates), we must calculate rewards in separate periods before / after any slashes
which occurred in between when a delegator delegated and when they withdrew their rewards. Thus slashes, like
delegations, reference the period which was ended by the slash event.

All stored historical rewards records for periods which are no longer referenced by any delegations
or any slashes can thus be safely removed, as they will never be read (future delegations and future
slashes will always reference future periods). This is implemented by tracking a `ReferenceCount`
along with each historical reward storage entry. Each time a new object (delegation or slash)
is created which might need to reference the historical record, the reference count is incremented.
Each time one object which previously needed to reference the historical record is deleted, the reference
count is decremented. If the reference count hits zero, the historical record is deleted.

## State

> This section was taken from the official Cosmos SDK docs, and placed here for your convenience to understand the distribution module's parameters and genesis variables.

### FeePool

All globally tracked parameters for distribution are stored within
`FeePool`. Rewards are collected and added to the reward pool and
distributed to validators/delegators from here.

Note that the reward pool holds decimal coins (`DecCoins`) to allow
for fractions of coins to be received from operations like inflation.
When coins are distributed from the pool they are truncated back to
`sdk.Coins` which are non-decimal.

### Validator Distribution

Validator distribution information for the relevant validator is updated each time:

1. delegation amount to a validator is updated,
2. a validator successfully proposes a block and receives a reward,
3. any delegator withdraws from a validator, or
4. the validator withdraws its commission.

### Delegation Distribution

Each delegation distribution only needs to record the height at which it last
withdrew fees. Because a delegation must withdraw fees each time its
properties change (aka bonded tokens etc.), its properties will remain constant
and the delegator's accumulation factor can be calculated passively knowing
only the height of the last withdrawal and its current properties.

## Message Types

### MsgSetWithdrawAddress

```go
// MsgSetWithdrawAddress sets the withdraw address for
// a delegator (or validator self-delegation).
type MsgSetWithdrawAddress struct {
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	WithdrawAddress  string `protobuf:"bytes,2,opt,name=withdraw_address,json=withdrawAddress,proto3" json:"withdraw_address,omitempty" yaml:"withdraw_address"`
}
```

### MsgWithdrawDelegatorReward

```go
// MsgWithdrawDelegatorReward represents delegation withdrawal to a delegator
// from a single validator.
type MsgWithdrawDelegatorReward struct {
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
}
```

### MsgWithdrawValidatorCommission

```go
// MsgWithdrawValidatorCommission withdraws the full commission to the validator
// address.
type MsgWithdrawValidatorCommission struct {
	ValidatorAddress string `protobuf:"bytes,1,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
}
```

### MsgFundCommunityPool

```go
// MsgFundCommunityPool allows an account to directly
// fund the community pool.
type MsgFundCommunityPool struct {
	Amount    github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,1,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
	Depositor string                                   `protobuf:"bytes,2,opt,name=depositor,proto3" json:"depositor,omitempty"`
}
```

## Proposals

### CommunityPoolSpendProposal

The distribution module defines a special proposal that, upon being passed, disburses the coins specified in `Amount` to the `Recipient` account using funds from the community pool.

```go
// CommunityPoolSpendProposal details a proposal for use of community funds,
// together with how many coins are proposed to be spent, and to which
// recipient account.
type CommunityPoolSpendProposal struct {
	Title       string                                   `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description string                                   `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	Recipient   string                                   `protobuf:"bytes,3,opt,name=recipient,proto3" json:"recipient,omitempty"`
	Amount      github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,4,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

## Transitions

### Begin Block

At each `BeginBlock`, all fees received in the previous block are transferred to
the distribution `ModuleAccount` account. When a delegator or validator
withdraws their rewards, they are taken out of the `ModuleAccount`. During begin
block, the different claims on the fees collected are updated as follows:

- The block proposer of the previous height and its delegators receive between 1% and 5% of fee rewards.
- The reserve community tax is charged.
- The remainder is distributed proportionally by voting power to all bonded validators

To incentivize validators to wait and include additional pre-commits in the block, the block proposer reward is calculated from Tendermint pre-commit messages.

#### The Distribution Scheme

See [params]({{< ref "params" >}}) for description of parameters.

Let `fees` be the total fees collected in the previous block, including
inflationary rewards to the stake. All fees are collected in a specific module
account during the block. During `BeginBlock`, they are sent to the
`"distribution"` `ModuleAccount`. No other sending of tokens occurs. Instead, the
rewards each account is entitled to are stored, and withdrawals can be triggered
through the messages `FundCommunityPool`, `WithdrawValidatorCommission` and
`WithdrawDelegatorReward`.

##### Reward to the Community Pool

The community pool gets `community_tax * fees`, plus any remaining dust after
validators get their rewards that are always rounded down to the nearest
integer value.

##### Reward To the Validators

The proposer receives a base reward of `fees * baseproposerreward` and a bonus
of `fees * bonusproposerreward * P`, where `P = (total power of validators with
included precommits / total bonded validator power)`. The more precommits the
proposer includes, the larger `P` is. `P` can never be larger than `1.00` (since
only bonded validators can supply valid precommits) and is always larger than
`2/3`.

Any remaining fees are distributed among all the bonded validators, including
the proposer, in proportion to their consensus power.

```
powFrac = validator power / total bonded validator power
proposerMul = baseproposerreward + bonusproposerreward * P
voteMul = 1 - communitytax - proposerMul
```

In total, the proposer receives `fees  * (voteMul * powFrac + proposerMul)`.
All other validators receive `fees * voteMul * powFrac`.

##### Rewards to Delegators

Each validator's rewards are distributed to its delegators. The validator also
has a self-delegation that is treated like a regular delegation in
distribution calculations.

The validator sets a commission rate. The commission rate is flexible, but each
validator sets a maximum rate and a maximum daily increase. These maximums cannot be exceeded and protect delegators from sudden increases of validator commission rates to prevent validators from taking all of the rewards.

The outstanding rewards that the operator is entitled to are stored in
`ValidatorAccumulatedCommission`, while the rewards the delegators are entitled
to are stored in `ValidatorCurrentRewards`. The [F1 fee distribution
scheme]({{< ref "distribution#concepts" >}}) is used to calculate the rewards per delegator as they
withdraw or update their delegation, and is thus not handled in `BeginBlock`.

##### Example Distribution

For this example distribution, the underlying consensus engine selects block proposers in
proportion to their power relative to the entire bonded power.

All validators are equally performant at including pre-commits in their proposed
blocks. Then hold `(precommits included) / (total bonded validator power)`
constant so that the amortized block reward for the validator is `( validator power / total bonded power) * (1 - community tax rate)` of
the total rewards. Consequently, the reward for a single delegator is:

```
(delegator proportion of the validator power / validator power) * (validator power / total bonded power)
  * (1 - community tax rate) * (1 - validator commision rate)
= (delegator proportion of the validator power / total bonded power) * (1 -
community tax rate) * (1 - validator commision rate)
```

### Hooks

Available hooks that can be called by and from this module.

#### Create or modify delegation distribution

- triggered-by: `staking.MsgDelegate`, `staking.MsgBeginRedelegate`, `staking.MsgUndelegate`

##### Before

- The delegation rewards are withdrawn to the withdraw address of the delegator.
  The rewards include the current period and exclude the starting period.
- The validator period is incremented.
  The validator period is incremented because the validator's power and share distribution might have changed.
- The reference count for the delegator's starting period is decremented.

##### After

The starting height of the delegation is set to the previous period.
Because of the `Before`-hook, this period is the last period for which the delegator was rewarded.

#### Validator created

- triggered-by: `staking.MsgCreateValidator`

When a validator is created, the following validator variables are initialized:

- Historical rewards
- Current accumulated rewards
- Accumulated commission
- Total outstanding rewards
- Period

By default, all values are set to a `0`, except period, which is set to `1`.

#### Validator removed

- triggered-by: `staking.RemoveValidator`

Outstanding commission is sent to the validator's self-delegation withdrawal address.
Remaining delegator rewards get sent to the community fee pool.

Note: The validator gets removed only when it has no remaining delegations.
At that time, all outstanding delegator rewards will have been withdrawn.
Any remaining rewards are dust amounts.

#### Validator is slashed

- triggered-by: `staking.Slash`
  
- The current validator period reference count is incremented.
  The reference count is incremented because the slash event has created a reference to it.
- The validator period is incremented.
- The slash event is stored for later use.
  The slash event will be referenced when calculating delegator rewards.

## Parameters

The subspace for the distribution module is `distribution`.

```go
// Params defines the set of params for the distribution module.
type Params struct {
	CommunityTax        github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,1,opt,name=community_tax,json=communityTax,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"community_tax" yaml:"community_tax"`
	BaseProposerReward  github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=base_proposer_reward,json=baseProposerReward,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"base_proposer_reward" yaml:"base_proposer_reward"`
	BonusProposerReward github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=bonus_proposer_reward,json=bonusProposerReward,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"bonus_proposer_reward" yaml:"bonus_proposer_reward"`
	WithdrawAddrEnabled bool                                   `protobuf:"varint,4,opt,name=withdraw_addr_enabled,json=withdrawAddrEnabled,proto3" json:"withdraw_addr_enabled,omitempty" yaml:"withdraw_addr_enabled"`
}
```

### CommunityTax

- type: `Dec`

### BaseProposerReward

- type: `Dec`

### BonusProposerReward

- type: `Dec`

### WithdrawAddrEnabled

- type: `bool`
