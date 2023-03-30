---
title: Slashing
weight: 130
---

{{< alert >}}
**Note**

XPLA Chain's slashing module inherits from the Cosmos SDK's [`slashing`](https://docs.cosmos.network/v0.45/modules/slashing/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

The slashing module enables XPLA Chain to disincentivize any attributable action by a protocol-recognized actor with value at stake by penalizing them. The penalty is called [slashing]({{< ref "glossary#slashing" >}}). XPLA Chain mainly uses the [`Staking`]({{< ref "staking" >}}) module to slash when violating validator responsibilities. This module manages lower-level penalties at the Tendermint consensus level, such as double-signing.

## Concepts

### States

At any given time, there are any number of validators registered in the state
machine. Each block, the top `MaxValidators` (defined by `x/staking`) validators
who are not jailed become _bonded_, meaning that they may propose and vote on
blocks. Validators who are _bonded_ are _at stake_, meaning that part or all of
their stake and their delegators' stake is at risk if they commit a protocol fault.

For each of these validators we keep a `ValidatorSigningInfo` record that contains
information partaining to validator's liveness and other infraction related
attributes.

### Tombstone Caps

In order to mitigate the impact of initially likely categories of non-malicious
protocol faults, the XPLA Chain implements for each validator
a _tombstone_ cap, which only allows a validator to be slashed once for a double
sign fault. For example, if you misconfigure your HSM and double-sign a bunch of
old blocks, you'll only be punished for the first double-sign (and then immediately tombstombed). This will still be quite expensive and desirable to avoid, but tombstone caps
somewhat blunt the economic impact of unintentional misconfiguration.

Liveness faults do not have caps, as they can't stack upon each other. Liveness bugs are "detected" as soon as the infraction occurs, and the validators are immediately put in jail, so it is not possible for them to commit multiple liveness faults without unjailing in between.

### Infraction Timelines

To illustrate how the `x/slashing` module handles submitted evidence through
Tendermint consensus, consider the following examples:

**Definitions**:

_[_ : timeline start  
_]_ : timeline end  
_C<sub>n</sub>_ : infraction `n` committed  
_D<sub>n</sub>_ : infraction `n` discovered  
_V<sub>b</sub>_ : validator bonded  
_V<sub>u</sub>_ : validator unbonded

#### Single Double Sign Infraction

<----------------->
[----------C<sub>1</sub>----D<sub>1</sub>,V<sub>u</sub>-----]

A single infraction is committed then later discovered, at which point the
validator is unbonded and slashed at the full amount for the infraction.

#### Multiple Double Sign Infractions

<--------------------------->
[----------C<sub>1</sub>--C<sub>2</sub>---C<sub>3</sub>---D<sub>1</sub>,D<sub>2</sub>,D<sub>3</sub>V<sub>u</sub>-----]

Multiple infractions are committed and then later discovered, at which point the
validator is jailed and slashed for only one infraction. Because the validator
is also tombstoned, they can not rejoin the validator set.


## Message Types

### MsgUnjail

```go
// MsgUnjail defines the Msg/Unjail request type
type MsgUnjail struct {
	ValidatorAddr string `protobuf:"bytes,1,opt,name=validator_addr,json=validatorAddr,proto3" json:"address" yaml:"address"`
}
```

## Transitions

### BeginBlock

#### Liveness Tracking

At the beginning of each block, we update the `ValidatorSigningInfo` for each
validator and check if they've crossed below the liveness threshold over a
sliding window. This sliding window is defined by `SignedBlocksWindow` and the
index in this window is determined by `IndexOffset` found in the validator's
`ValidatorSigningInfo`. For each block processed, the `IndexOffset` is incremented
regardless if the validator signed or not. Once the index is determined, the
`MissedBlocksBitArray` and `MissedBlocksCounter` are updated accordingly.

Finally, in order to determine if a validator crosses below the liveness threshold,
we fetch the maximum number of blocks missed, `maxMissed`, which is
`SignedBlocksWindow - (MinSignedPerWindow * SignedBlocksWindow)` and the minimum
height at which we can determine liveness, `minHeight`. If the current block is
greater than `minHeight` and the validator's `MissedBlocksCounter` is greater than
`maxMissed`, they will be slashed by `SlashFractionDowntime`, will be jailed
for `DowntimeJailDuration`, and have the following values reset:
`MissedBlocksBitArray`, `MissedBlocksCounter`, and `IndexOffset`.

**Note**: Liveness slashes do **NOT** lead to a tombstombing.

```go
height := block.Height

for vote in block.LastCommitInfo.Votes {
  signInfo := GetValidatorSigningInfo(vote.Validator.Address)

  // This is a relative index, so we counts blocks the validator SHOULD have
  // signed. We use the 0-value default signing info if not present, except for
  // start height.
  index := signInfo.IndexOffset % SignedBlocksWindow()
  signInfo.IndexOffset++

  // Update MissedBlocksBitArray and MissedBlocksCounter. The MissedBlocksCounter
  // just tracks the sum of MissedBlocksBitArray. That way we avoid needing to
  // read/write the whole array each time.
  missedPrevious := GetValidatorMissedBlockBitArray(vote.Validator.Address, index)
  missed := !signed

  switch {
  case !missedPrevious && missed:
    // array index has changed from not missed to missed, increment counter
    SetValidatorMissedBlockBitArray(vote.Validator.Address, index, true)
    signInfo.MissedBlocksCounter++

  case missedPrevious && !missed:
    // array index has changed from missed to not missed, decrement counter
    SetValidatorMissedBlockBitArray(vote.Validator.Address, index, false)
    signInfo.MissedBlocksCounter--

  default:
    // array index at this index has not changed; no need to update counter
  }

  if missed {
    // emit events...
  }

  minHeight := signInfo.StartHeight + SignedBlocksWindow()
  maxMissed := SignedBlocksWindow() - MinSignedPerWindow()

  // If we are past the minimum height and the validator has missed too many
  // jail and slash them.
  if height > minHeight && signInfo.MissedBlocksCounter > maxMissed {
    validator := ValidatorByConsAddr(vote.Validator.Address)

    // emit events...

    // We need to retrieve the stake distribution which signed the block, so we
    // subtract ValidatorUpdateDelay from the block height, and subtract an
    // additional 1 since this is the LastCommit.
    //
    // Note, that this CAN result in a negative "distributionHeight" up to
    // -ValidatorUpdateDelay-1, i.e. at the end of the pre-genesis block (none) = at the beginning of the genesis block.
    // That's fine since this is just used to filter unbonding delegations & redelegations.
    distributionHeight := height - sdk.ValidatorUpdateDelay - 1

    Slash(vote.Validator.Address, distributionHeight, vote.Validator.Power, SlashFractionDowntime())
    Jail(vote.Validator.Address)

    signInfo.JailedUntil = block.Time.Add(DowntimeJailDuration())

    // We need to reset the counter & array so that the validator won't be
    // immediately slashed for downtime upon rebonding.
    signInfo.MissedBlocksCounter = 0
    signInfo.IndexOffset = 0
    ClearValidatorMissedBlockBitArray(vote.Validator.Address)
  }

  SetValidatorSigningInfo(vote.Validator.Address, signInfo)
}
```

### Hooks

This section contains a description of the module's `hooks`. Hooks are operations that are executed automatically when events are raised.

#### Staking hooks

The slashing module implements the `StakingHooks` defined in `x/staking` and are used as record-keeping of validators information. During the app initialization, these hooks should be registered in the staking module struct.

The following hooks impact the slashing state:

+ `AfterValidatorBonded` creates a `ValidatorSigningInfo` instance as described in the following section.
+ `AfterValidatorCreated` stores a validator's consensus key.
+ `AfterValidatorRemoved` removes a validator's consensus key.

#### Validator Bonded

Upon successful first-time bonding of a new validator, we create a new `ValidatorSigningInfo` structure for the
now-bonded validator, which `StartHeight` of the current block.

```
onValidatorBonded(address sdk.ValAddress)

  signingInfo, found = GetValidatorSigningInfo(address)
  if !found {
    signingInfo = ValidatorSigningInfo {
      StartHeight         : CurrentHeight,
      IndexOffset         : 0,
      JailedUntil         : time.Unix(0, 0),
      Tombstone           : false,
      MissedBloskCounter  : 0
    }
    setValidatorSigningInfo(signingInfo)
  }

  return
```


## Parameters

The subspace for the slashing module is `slashing`.

```go
// Params represents the parameters used for by the slashing module.
type Params struct {
	SignedBlocksWindow      int64                                  `protobuf:"varint,1,opt,name=signed_blocks_window,json=signedBlocksWindow,proto3" json:"signed_blocks_window,omitempty" yaml:"signed_blocks_window"`
	MinSignedPerWindow      github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=min_signed_per_window,json=minSignedPerWindow,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"min_signed_per_window" yaml:"min_signed_per_window"`
	DowntimeJailDuration    time.Duration                          `protobuf:"bytes,3,opt,name=downtime_jail_duration,json=downtimeJailDuration,proto3,stdduration" json:"downtime_jail_duration" yaml:"downtime_jail_duration"`
	SlashFractionDoubleSign github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,4,opt,name=slash_fraction_double_sign,json=slashFractionDoubleSign,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"slash_fraction_double_sign" yaml:"slash_fraction_double_sign"`
	SlashFractionDowntime   github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,5,opt,name=slash_fraction_downtime,json=slashFractionDowntime,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"slash_fraction_downtime" yaml:"slash_fraction_downtime"`
}
```

### SignedBlocksWindow

- type: `int64`
- `"signed_blocks_window": "100"`

### MinSignedPerWindow

- type: `Dec`
- `"min_signed_per_window": "0.500000000000000000"`

### DowntimeJailDuration

- type: `time.Duration` (seconds)
- `"downtime_jail_duration": "600s"`

### SlashFractionDoubleSign

- type: `Dec`
- `"slash_fraction_double_sign": "0.050000000000000000"`

### SlashFractionDowntime

- type: `Dec`
- `"slash_fraction_downtime": "0.010000000000000000"`
