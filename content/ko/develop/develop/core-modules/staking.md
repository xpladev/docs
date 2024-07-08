---
title: Staking
weight: 170
type: docs
---

{{< alert >}}
**Note**

XPLA Chain's staking module inherits from the Cosmos SDK's [`staking`](https://docs.cosmos.network/v0.45/modules/staking/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

The staking module enables XPLA Chain's proof-of-stake functionality by requiring validators to bond XPLA, the native staking asset.

## State

### Pool

Pool is used for tracking bonded and not-bonded token supply of the bond denomination.

### LastTotalPower

LastTotalPower tracks the total amounts of bonded tokens recorded during the previous end block.
Store entries prefixed with "Last" must remain unchanged until EndBlock.

- LastTotalPower: `0x12 -> ProtocolBuffer(sdk.Int)`

### Params

Params is a module-wide configuration structure that stores system parameters
and defines overall functioning of the staking module.

- Params: `Paramsspace("staking") -> legacy_amino(params)`

### Validator

Validators can have one of three statuses

- `Unbonded`: The validator is not in the active set. They cannot sign blocks and do not earn
  rewards. They can receive delegations.
- `Bonded`: Once the validator receives sufficient bonded tokens they automtically join the
  active set during [`EndBlock`]({{< ref "staking#validator-set-changes" >}}) and their status is updated to `Bonded`.
  They are signing blocks and receiving rewards. They can receive further delegations.
  They can be slashed for misbehavior. Delegators to this validator who unbond their delegation
  must wait the duration of the UnbondingTime, a chain-specific param, during which time
  they are still slashable for offences of the source validator if those offences were committed
  during the period of time that the tokens were bonded.
- `Unbonding`: When a validator leaves the active set, either by choice or due to slashing, jailing or
  tombstoning, an unbonding of all their delegations begins. All delegations must then wait the UnbondingTime
  before their tokens are moved to their accounts from the `BondedPool`.

Validators objects should be primarily stored and accessed by the
`OperatorAddr`, an SDK validator address for the operator of the validator. Two
additional indices are maintained per validator object in order to fulfill
required lookups for slashing and validator-set updates. A third special index
(`LastValidatorPower`) is also maintained which however remains constant
throughout each block, unlike the first two indices which mirror the validator
records within a block.

- Validators: `0x21 | OperatorAddrLen (1 byte) | OperatorAddr -> ProtocolBuffer(validator)`
- ValidatorsByConsAddr: `0x22 | ConsAddrLen (1 byte) | ConsAddr -> OperatorAddr`
- ValidatorsByPower: `0x23 | BigEndian(ConsensusPower) | OperatorAddrLen (1 byte) | OperatorAddr -> OperatorAddr`
- LastValidatorsPower: `0x11 | OperatorAddrLen (1 byte) | OperatorAddr -> ProtocolBuffer(ConsensusPower)`

`Validators` is the primary index - it ensures that each operator can have only one
associated validator, where the public key of that validator can change in the
future. Delegators can refer to the immutable operator of the validator, without
concern for the changing public key.

`ValidatorByConsAddr` is an additional index that enables lookups for slashing.
When Tendermint reports evidence, it provides the validator address, so this
map is needed to find the operator. Note that the `ConsAddr` corresponds to the
address which can be derived from the validator's `ConsPubKey`.

`ValidatorsByPower` is an additional index that provides a sorted list of
potential validators to quickly determine the current active set. Here
ConsensusPower is validator.Tokens/10^18 by default. Note that all validators
where `Jailed` is true are not stored within this index.

`LastValidatorsPower` is a special index that provides a historical list of the
last-block's bonded validators. This index remains constant during a block but
is updated during the validator set update process which takes place in [`EndBlock`]({{< ref "staking#end-block" >}}).

Each validator's state is stored in a `Validator` struct:

```go
// Validator defines a validator, together with the total amount of the
// Validator's bond shares and their exchange rate to coins. Slashing results in
// a decrease in the exchange rate, allowing correct calculation of future
// undelegations without iterating over delegators. When coins are delegated to
// this validator, the validator is credited with a delegation whose number of
// bond shares is based on the amount of coins delegated divided by the current
// exchange rate. Voting power can be calculated as total bonded shares
// multiplied by exchange rate.
type Validator struct {
	// operator_address defines the address of the validator's operator; bech encoded in JSON.
	OperatorAddress string `protobuf:"bytes,1,opt,name=operator_address,json=operatorAddress,proto3" json:"operator_address,omitempty" yaml:"operator_address"`
	// consensus_pubkey is the consensus public key of the validator, as a Protobuf Any.
	ConsensusPubkey *types1.Any `protobuf:"bytes,2,opt,name=consensus_pubkey,json=consensusPubkey,proto3" json:"consensus_pubkey,omitempty" yaml:"consensus_pubkey"`
	// jailed defined whether the validator has been jailed from bonded status or not.
	Jailed bool `protobuf:"varint,3,opt,name=jailed,proto3" json:"jailed,omitempty"`
	// status is the validator status (bonded/unbonding/unbonded).
	Status BondStatus `protobuf:"varint,4,opt,name=status,proto3,enum=cosmos.staking.v1beta1.BondStatus" json:"status,omitempty"`
	// tokens define the delegated tokens (incl. self-delegation).
	Tokens github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,5,opt,name=tokens,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"tokens"`
	// delegator_shares defines total shares issued to a validator's delegators.
	DelegatorShares github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,6,opt,name=delegator_shares,json=delegatorShares,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"delegator_shares" yaml:"delegator_shares"`
	// description defines the description terms for the validator.
	Description Description `protobuf:"bytes,7,opt,name=description,proto3" json:"description"`
	// unbonding_height defines, if unbonding, the height at which this validator has begun unbonding.
	UnbondingHeight int64 `protobuf:"varint,8,opt,name=unbonding_height,json=unbondingHeight,proto3" json:"unbonding_height,omitempty" yaml:"unbonding_height"`
	// unbonding_time defines, if unbonding, the min time for the validator to complete unbonding.
	UnbondingTime time.Time `protobuf:"bytes,9,opt,name=unbonding_time,json=unbondingTime,proto3,stdtime" json:"unbonding_time" yaml:"unbonding_time"`
	// commission defines the commission parameters.
	Commission Commission `protobuf:"bytes,10,opt,name=commission,proto3" json:"commission"`
	// min_self_delegation is the validator's self declared minimum self delegation.
	MinSelfDelegation github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,11,opt,name=min_self_delegation,json=minSelfDelegation,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"min_self_delegation" yaml:"min_self_delegation"`
}

// Commission defines commission parameters for a given validator.
type Commission struct {
	// commission_rates defines the initial commission rates to be used for creating a validator.
	CommissionRates `protobuf:"bytes,1,opt,name=commission_rates,json=commissionRates,proto3,embedded=commission_rates" json:"commission_rates"`
	// update_time is the last time the commission rate was changed.
	UpdateTime time.Time `protobuf:"bytes,2,opt,name=update_time,json=updateTime,proto3,stdtime" json:"update_time" yaml:"update_time"`
}

// CommissionRates defines the initial commission rates to be used for creating
// a validator.
type CommissionRates struct {
	// rate is the commission rate charged to delegators, as a fraction.
	Rate github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,1,opt,name=rate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"rate"`
	// max_rate defines the maximum commission rate which validator can ever charge, as a fraction.
	MaxRate github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=max_rate,json=maxRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"max_rate" yaml:"max_rate"`
	// max_change_rate defines the maximum daily increase of the validator commission, as a fraction.
	MaxChangeRate github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=max_change_rate,json=maxChangeRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"max_change_rate" yaml:"max_change_rate"`
}

// Description defines a validator description.
type Description struct {
	// moniker defines a human-readable name for the validator.
	Moniker string `protobuf:"bytes,1,opt,name=moniker,proto3" json:"moniker,omitempty"`
	// identity defines an optional identity signature (ex. UPort or Keybase).
	Identity string `protobuf:"bytes,2,opt,name=identity,proto3" json:"identity,omitempty"`
	// website defines an optional website link.
	Website string `protobuf:"bytes,3,opt,name=website,proto3" json:"website,omitempty"`
	// security_contact defines an optional email for security contact.
	SecurityContact string `protobuf:"bytes,4,opt,name=security_contact,json=securityContact,proto3" json:"security_contact,omitempty" yaml:"security_contact"`
	// details define other optional details.
	Details string `protobuf:"bytes,5,opt,name=details,proto3" json:"details,omitempty"`
}
```

### Delegation

Delegations are identified by combining `DelegatorAddr` (the address of the delegator)
with the `ValidatorAddr` Delegators are indexed in the store as follows:

- Delegation: `0x31 | DelegatorAddrLen (1 byte) | DelegatorAddr | ValidatorAddrLen (1 byte) | ValidatorAddr -> ProtocolBuffer(delegation)`

Stake holders may delegate coins to validators; under this circumstance their
funds are held in a `Delegation` data structure. It is owned by one
delegator, and is associated with the shares for one validator. The sender of
the transaction is the owner of the bond.

```go
// Delegation represents the bond with tokens held by an account. It is
// owned by one delegator, and is associated with the voting power of one
// validator.
type Delegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	// validator_address is the bech32-encoded address of the validator.
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	// shares define the delegation shares received.
	Shares github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=shares,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"shares"`
}
```

#### Delegator Shares

When one Delegates tokens to a Validator they are issued a number of delegator shares based on a
dynamic exchange rate, calculated as follows from the total number of tokens delegated to the
validator and the number of shares issued so far:

`Shares per Token = validator.TotalShares() / validator.Tokens()`

Only the number of shares received is stored on the DelegationEntry. When a delegator then
Undelegates, the token amount they receive is calculated from the number of shares they currently
hold and the inverse exchange rate:

`Tokens per Share = validator.Tokens() / validatorShares()`

These `Shares` are simply an accounting mechanism. They are not a fungible asset. The reason for
this mechanism is to simplify the accounting around slashing. Rather than iteratively slashing the
tokens of every delegation entry, instead the Validators total bonded tokens can be slashed,
effectively reducing the value of each issued delegator share.

### UnbondingDelegation

Shares in a `Delegation` can be unbonded, but they must for some time exist as
an `UnbondingDelegation`, where shares can be reduced if Byzantine behavior is
detected.

`UnbondingDelegation` are indexed in the store as:

- UnbondingDelegation: `0x32 | DelegatorAddrLen (1 byte) | DelegatorAddr | ValidatorAddrLen (1 byte) | ValidatorAddr -> ProtocolBuffer(unbondingDelegation)`
- UnbondingDelegationsFromValidator: `0x33 | ValidatorAddrLen (1 byte) | ValidatorAddr | DelegatorAddrLen (1 byte) | DelegatorAddr -> nil`

The first map here is used in queries, to lookup all unbonding delegations for
a given delegator, while the second map is used in slashing, to lookup all
unbonding delegations associated with a given validator that need to be
slashed.

A UnbondingDelegation object is created every time an unbonding is initiated.

```go
// UnbondingDelegation stores all of a single delegator's unbonding bonds
// for a single validator in an time-ordered list.
type UnbondingDelegation struct {
	// delegator_address is the bech32-encoded address of the delegator.
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	// validator_address is the bech32-encoded address of the validator.
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	// entries are the unbonding delegation entries.
	Entries []UnbondingDelegationEntry `protobuf:"bytes,3,rep,name=entries,proto3" json:"entries"`
}

// UnbondingDelegationEntry defines an unbonding object with relevant metadata.
type UnbondingDelegationEntry struct {
	// creation_height is the height which the unbonding took place.
	CreationHeight int64 `protobuf:"varint,1,opt,name=creation_height,json=creationHeight,proto3" json:"creation_height,omitempty" yaml:"creation_height"`
	// completion_time is the unix time for unbonding completion.
	CompletionTime time.Time `protobuf:"bytes,2,opt,name=completion_time,json=completionTime,proto3,stdtime" json:"completion_time" yaml:"completion_time"`
	// initial_balance defines the tokens initially scheduled to receive at completion.
	InitialBalance github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,3,opt,name=initial_balance,json=initialBalance,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"initial_balance" yaml:"initial_balance"`
	// balance defines the tokens to receive at completion.
	Balance github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,4,opt,name=balance,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"balance"`
}
```

### Redelegation

The bonded tokens worth of a `Delegation` may be instantly redelegated from a
source validator to a different validator (destination validator). However when
this occurs they must be tracked in a `Redelegation` object, whereby their
shares can be slashed if their tokens have contributed to a Byzantine fault
committed by the source validator.

`Redelegation` are indexed in the store as:

- Redelegations: `0x34 | DelegatorAddrLen (1 byte) | DelegatorAddr | ValidatorAddrLen (1 byte) | ValidatorSrcAddr | ValidatorDstAddr -> ProtocolBuffer(redelegation)`
- RedelegationsBySrc: `0x35 | ValidatorSrcAddrLen (1 byte) | ValidatorSrcAddr | ValidatorDstAddrLen (1 byte) | ValidatorDstAddr | DelegatorAddrLen (1 byte) | DelegatorAddr -> nil`
- RedelegationsByDst: `0x36 | ValidatorDstAddrLen (1 byte) | ValidatorDstAddr | ValidatorSrcAddrLen (1 byte) | ValidatorSrcAddr | DelegatorAddrLen (1 byte) | DelegatorAddr -> nil`

The first map here is used for queries, to lookup all redelegations for a given
delegator. The second map is used for slashing based on the `ValidatorSrcAddr`,
while the third map is for slashing based on the `ValidatorDstAddr`.

A redelegation object is created every time a redelegation occurs. To prevent
"redelegation hopping" redelegations may not occur under the situation that:

- the (re)delegator already has another immature redelegation in progress
  with a destination to a validator (let's call it `Validator X`)
- and, the (re)delegator is attempting to create a _new_ redelegation
  where the source validator for this new redelegation is `Validator X`.

```go
// RedelegationEntryResponse is equivalent to a RedelegationEntry except that it
// contains a balance in addition to shares which is more suitable for client
// responses.
type RedelegationEntryResponse struct {
	RedelegationEntry RedelegationEntry                      `protobuf:"bytes,1,opt,name=redelegation_entry,json=redelegationEntry,proto3" json:"redelegation_entry"`
	Balance           github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,4,opt,name=balance,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"balance"`
}

// RedelegationEntry defines a redelegation object with relevant metadata.
type RedelegationEntry struct {
	// creation_height  defines the height which the redelegation took place.
	CreationHeight int64 `protobuf:"varint,1,opt,name=creation_height,json=creationHeight,proto3" json:"creation_height,omitempty" yaml:"creation_height"`
	// completion_time defines the unix time for redelegation completion.
	CompletionTime time.Time `protobuf:"bytes,2,opt,name=completion_time,json=completionTime,proto3,stdtime" json:"completion_time" yaml:"completion_time"`
	// initial_balance defines the initial balance when redelegation started.
	InitialBalance github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,3,opt,name=initial_balance,json=initialBalance,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"initial_balance" yaml:"initial_balance"`
	// shares_dst is the amount of destination-validator shares created by redelegation.
	SharesDst github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,4,opt,name=shares_dst,json=sharesDst,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"shares_dst"`
}
```

### Queues

All queues objects are sorted by timestamp. The time used within any queue is
first rounded to the nearest nanosecond then sorted. The sortable time format
used is a slight modification of the RFC3339Nano and uses the the format string
`"2006-01-02T15:04:05.000000000"`. Notably this format:

- right pads all zeros
- drops the time zone info (uses UTC)

In all cases, the stored timestamp represents the maturation time of the queue
element.

#### UnbondingDelegationQueue

For the purpose of tracking progress of unbonding delegations the unbonding
delegations queue is kept.

- UnbondingDelegation: `0x41 | format(time) -> []DVPair`

```go
// DVPair is struct that just has a delegator-validator pair with no other data.
// It is intended to be used as a marshalable pointer. For example, a DVPair can
// be used to construct the key to getting an UnbondingDelegation from state.
type DVPair struct {
	DelegatorAddress string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress string `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
}
```

#### RedelegationQueue

For the purpose of tracking progress of redelegations the redelegation queue is
kept.

- RedelegationQueue: `0x42 | format(time) -> []DVVTriplet`

```go
// DVVTriplet is struct that just has a delegator-validator-validator triplet
// with no other data. It is intended to be used as a marshalable pointer. For
// example, a DVVTriplet can be used to construct the key to getting a
// Redelegation from state.
type DVVTriplet struct {
	DelegatorAddress    string `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorSrcAddress string `protobuf:"bytes,2,opt,name=validator_src_address,json=validatorSrcAddress,proto3" json:"validator_src_address,omitempty" yaml:"validator_src_address"`
	ValidatorDstAddress string `protobuf:"bytes,3,opt,name=validator_dst_address,json=validatorDstAddress,proto3" json:"validator_dst_address,omitempty" yaml:"validator_dst_address"`
}
```

#### ValidatorQueue

For the purpose of tracking progress of unbonding validators the validator
queue is kept.

- ValidatorQueueTime: `0x43 | format(time) -> []sdk.ValAddress`

The stored object as each key is an array of validator operator addresses from
which the validator object can be accessed. Typically it is expected that only
a single validator record will be associated with a given timestamp however it is possible
that multiple validators exist in the queue at the same location.

### HistoricalInfo

HistoricalInfo objects are stored and pruned at each block such that the staking keeper persists
the `n` most recent historical info defined by staking module parameter: `HistoricalEntries`.

```go
// HistoricalInfo contains header and validator information for a given block.
// It is stored as part of staking module's state, which persists the `n` most
// recent HistoricalInfo
// (`n` is set by the staking module's `historical_entries` parameter).
type HistoricalInfo struct {
	Header types.Header `protobuf:"bytes,1,opt,name=header,proto3" json:"header"`
	Valset []Validator  `protobuf:"bytes,2,rep,name=valset,proto3" json:"valset"`
}
```

At each BeginBlock, the staking keeper will persist the current Header and the Validators that committed
the current block in a `HistoricalInfo` object. The Validators are sorted on their address to ensure that
they are in a determisnistic order.
The oldest HistoricalEntries will be pruned to ensure that there only exist the parameter-defined number of
historical entries.


## Message Types

### MsgCreateValidator

```go
// MsgCreateValidator defines a SDK message for creating a new validator.
type MsgCreateValidator struct {
	Description       Description                            `protobuf:"bytes,1,opt,name=description,proto3" json:"description"`
	Commission        CommissionRates                        `protobuf:"bytes,2,opt,name=commission,proto3" json:"commission"`
	MinSelfDelegation github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,3,opt,name=min_self_delegation,json=minSelfDelegation,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"min_self_delegation" yaml:"min_self_delegation"`
	DelegatorAddress  string                                 `protobuf:"bytes,4,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress  string                                 `protobuf:"bytes,5,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	Pubkey            *types.Any                             `protobuf:"bytes,6,opt,name=pubkey,proto3" json:"pubkey,omitempty"`
	Value             types1.Coin                            `protobuf:"bytes,7,opt,name=value,proto3" json:"value"`
}
```

### MsgEditValidator

```go
// MsgEditValidator defines a SDK message for editing an existing validator.
type MsgEditValidator struct {
	Description      Description `protobuf:"bytes,1,opt,name=description,proto3" json:"description"`
	ValidatorAddress string      `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"address"`
	// We pass a reference to the new commission rate and min self delegation as
	// it's not mandatory to update. If not updated, the deserialized rate will be
	// zero with no way to distinguish if an update was intended.
	// REF: #2373
	CommissionRate    *github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=commission_rate,json=commissionRate,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"commission_rate,omitempty" yaml:"commission_rate"`
	MinSelfDelegation *github_com_cosmos_cosmos_sdk_types.Int `protobuf:"bytes,4,opt,name=min_self_delegation,json=minSelfDelegation,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Int" json:"min_self_delegation,omitempty" yaml:"min_self_delegation"`
}
```

### MsgDelegate

```go
// MsgDelegate defines a SDK message for performing a delegation of coins
// from a delegator to a validator.
type MsgDelegate struct {
	DelegatorAddress string      `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress string      `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	Amount           types1.Coin `protobuf:"bytes,3,opt,name=amount,proto3" json:"amount"`
}
```

### MsgUndelegate

```go
// MsgUndelegate defines a SDK message for performing an undelegation from a
// delegate and a validator.
type MsgUndelegate struct {
	DelegatorAddress string      `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress string      `protobuf:"bytes,2,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	Amount           types1.Coin `protobuf:"bytes,3,opt,name=amount,proto3" json:"amount"`
}
```

### MsgBeginRedelegate

```go
// MsgBeginRedelegate defines a SDK message for performing a redelegation
// of coins from a delegator and source validator to a destination validator.
type MsgBeginRedelegate struct {
	DelegatorAddress    string      `protobuf:"bytes,1,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorSrcAddress string      `protobuf:"bytes,2,opt,name=validator_src_address,json=validatorSrcAddress,proto3" json:"validator_src_address,omitempty" yaml:"validator_src_address"`
	ValidatorDstAddress string      `protobuf:"bytes,3,opt,name=validator_dst_address,json=validatorDstAddress,proto3" json:"validator_dst_address,omitempty" yaml:"validator_dst_address"`
	Amount              types1.Coin `protobuf:"bytes,4,opt,name=amount,proto3" json:"amount"`
}
```


## Transitions

### Begin-Block

Each abci begin block call, the historical info will get stored and pruned
according to the `HistoricalEntries` parameter.

#### Historical Info Tracking

If the `HistoricalEntries` parameter is 0, then the `BeginBlock` performs a no-op.

Otherwise, the latest historical info is stored under the key `historicalInfoKey|height`, while any entries older than `height - HistoricalEntries` is deleted.
In most cases, this results in a single entry being pruned per block.
However, if the parameter `HistoricalEntries` has changed to a lower value there will be multiple entries in the store that must be pruned.

##### GetLastValidators

This function gets the bonded validators of the previous block. the number of validators in the return value is the same as `params.MaxValidators` in the cosmos-sdk module, and the XPLA Chain provides as many as the number of Volunteer Validators in addition.

### End-Block

Each abci end block call, the operations to update queues and validator set
changes are specified to execute.

#### Validator Set Changes

The staking validator set is updated during this process by state transitions
that run at the end of every block. As a part of this process any updated
validators are also returned back to Tendermint for inclusion in the Tendermint
validator set which is responsible for validating Tendermint messages at the
consensus layer. Operations are as following:

- the new validator set is taken as the top `params.MaxValidators` number of
  validators retrieved from the `ValidatorsByPower` index
    - regardless of power, [Volunteer Validators]({{< ref "volunteer#volunteer-validator" >}}) are always included.
- the previous validator set is compared with the new validator set:
    - missing validators begin unbonding and their `Tokens` are transferred from the
    `BondedPool` to the `NotBondedPool` `ModuleAccount`
    - new validators are instantly bonded and their `Tokens` are transferred from the
    `NotBondedPool` to the `BondedPool` `ModuleAccount`

In all cases, any validators leaving or entering the bonded validator set or
changing balances and staying within the bonded validator set incur an update
message reporting their new consensus power which is passed back to Tendermint.

The `LastTotalPower` and `LastValidatorsPower` hold the state of the total power
and validator power from the end of the last block, and are used to check for
changes that have occured in `ValidatorsByPower` and the total new power, which
is calculated during `EndBlock`.

#### Queues

Within staking, certain state-transitions are not instantaneous but take place
over a duration of time (typically the unbonding period). When these
transitions are mature certain operations must take place in order to complete
the state operation. This is achieved through the use of queues which are
checked/processed at the end of each block.

##### Unbonding Validators

When a validator is kicked out of the bonded validator set (either through
being jailed, or not having sufficient bonded tokens) it begins the unbonding
process along with all its delegations begin unbonding (while still being
delegated to this validator). At this point the validator is said to be an
"unbonding validator", whereby it will mature to become an "unbonded validator"
after the unbonding period has passed.

Each block the validator queue is to be checked for mature unbonding validators
(namely with a completion time <= current time and completion height <= current
block height). At this point any mature validators which do not have any
delegations remaining are deleted from state. For all other mature unbonding
validators that still have remaining delegations, the `validator.Status` is
switched from `types.Unbonding` to
`types.Unbonded`.

##### Unbonding Delegations

Complete the unbonding of all mature `UnbondingDelegations.Entries` within the
`UnbondingDelegations` queue with the following procedure:

- transfer the balance coins to the delegator's wallet address
- remove the mature entry from `UnbondingDelegation.Entries`
- remove the `UnbondingDelegation` object from the store if there are no
  remaining entries.

##### Redelegations

Complete the unbonding of all mature `Redelegation.Entries` within the
`Redelegations` queue with the following procedure:

- remove the mature entry from `Redelegation.Entries`
- remove the `Redelegation` object from the store if there are no
  remaining entries.

### Hooks

Other modules may register operations to execute when a certain event has
occurred within staking.  These events can be registered to execute either
right `Before` or `After` the staking event (as per the hook name). The
following hooks can registered with staking:

- `AfterValidatorCreated(Context, ValAddress)`
    - called when a validator is created
- `BeforeValidatorModified(Context, ValAddress)`
    - called when a validator's state is changed
- `AfterValidatorRemoved(Context, ConsAddress, ValAddress)`
    - called when a validator is deleted
- `AfterValidatorBonded(Context, ConsAddress, ValAddress)`
    - called when a validator is bonded
- `AfterValidatorBeginUnbonding(Context, ConsAddress, ValAddress)`
    - called when a validator begins unbonding
- `BeforeDelegationCreated(Context, AccAddress, ValAddress)`
    - called when a delegation is created
- `BeforeDelegationSharesModified(Context, AccAddress, ValAddress)`
    - called when a delegation's shares are modified
- `BeforeDelegationRemoved(Context, AccAddress, ValAddress)`
    - called when a delegation is removed


## Parameters

The subspace for the Staking module is `staking`.

```go
// Params defines the parameters for the staking module.
type Params struct {
	// unbonding_time is the time duration of unbonding.
	UnbondingTime time.Duration `protobuf:"bytes,1,opt,name=unbonding_time,json=unbondingTime,proto3,stdduration" json:"unbonding_time" yaml:"unbonding_time"`
	// max_validators is the maximum number of validators.
	MaxValidators uint32 `protobuf:"varint,2,opt,name=max_validators,json=maxValidators,proto3" json:"max_validators,omitempty" yaml:"max_validators"`
	// max_entries is the max entries for either unbonding delegation or redelegation (per pair/trio).
	MaxEntries uint32 `protobuf:"varint,3,opt,name=max_entries,json=maxEntries,proto3" json:"max_entries,omitempty" yaml:"max_entries"`
	// historical_entries is the number of historical entries to persist.
	HistoricalEntries uint32 `protobuf:"varint,4,opt,name=historical_entries,json=historicalEntries,proto3" json:"historical_entries,omitempty" yaml:"historical_entries"`
	// bond_denom defines the bondable coin denomination.
	BondDenom string `protobuf:"bytes,5,opt,name=bond_denom,json=bondDenom,proto3" json:"bond_denom,omitempty" yaml:"bond_denom"`
}
```

### UnbondingTime

- type: `time.Duration`
- default: 3 weeks

Time duration of unbonding.

### MaxValidators

- type: `uint32`
- default: `40`

Maximum number of active validators.

### MaxEntries

- type: `uint32`
- default: `7`

Maximum entries for either unbonding delegation or redelegation per pair or trio. Be aware of potential overflow because this is user-determined.

### HistoricalEntries

- type: `uint32`
- default: `10000`

The number of historical entries to persist.

### BondDenom

- type: `string`
- default: `axpla`

Denomination of the asset required for staking.
