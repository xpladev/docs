---
title: Upgrade
weight: 170
---

{{< alert >}}
**Note**

XPLA Chain's upgrade module inherits from the Cosmos SDK's [`upgrade`](https://docs.cosmos.network/v0.45/modules/upgrade/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

`x/upgrade` is an implementation of a Cosmos SDK module that facilitates smoothly upgrading a live XPLA Chain to a new (breaking) software version. It accomplishes this by providing a `BeginBlocker` hook that prevents the blockchain state machine from proceeding once a pre-defined upgrade block height has been reached.

The module does not prescribe anything regarding how governance decides to do an upgrade, but just the mechanism for coordinating the upgrade safely. Without software support for upgrades, upgrading a live chain is risky because all of the validators need to pause their state machines at exactly the same point in the process. If this is not done correctly, there can be state inconsistencies which are hard to recover from.

## Concepts

### Plan

The `x/upgrade` module defines a `Plan` type in which a live upgrade is scheduled
to occur. A `Plan` can be scheduled at a specific block height.
A `Plan` is created once a (frozen) release candidate along with an appropriate upgrade
`Handler` (see below) is agreed upon, where the `Name` of a `Plan` corresponds to a
specific `Handler`. Typically, a `Plan` is created through a governance proposal
process, where if voted upon and passed, will be scheduled. The `Info` of a `Plan`
may contain various metadata about the upgrade, typically application specific
upgrade info to be included on-chain such as a git commit that validators could
automatically upgrade to.

#### Sidecar Process

If an operator running the application binary also runs a sidecar process to assist
in the automatic download and upgrade of a binary, the `Info` allows this process to
be seamless. Namely, the `x/upgrade` module fulfills the
[cosmosd Upgradeable Binary Specification](https://github.com/regen-network/cosmosd#upgradeable-binary-specification)
specification and `cosmosd` can optionally be used to fully automate the upgrade
process for node operators. By populating the `Info` field with the necessary information,
binaries can automatically be downloaded. See [here](https://github.com/regen-network/cosmosd#auto-download)
for more info.

```go
type Plan struct {
  Name   string
  Height int64
  Info   string
}
```

### Handler

The `x/upgrade` module facilitates upgrading from major version X to major version Y. To
accomplish this, node operators must first upgrade their current binary to a new
binary that has a corresponding `Handler` for the new version Y. It is assumed that
this version has fully been tested and approved by the community at large. This
`Handler` defines what state migrations need to occur before the new binary Y
can successfully run the chain. Naturally, this `Handler` is application specific
and not defined on a per-module basis. Registering a `Handler` is done via
`Keeper#SetUpgradeHandler` in the application.

```go
type UpgradeHandler func(Context, Plan, VersionMap) (VersionMap, error)
```

During each `EndBlock` execution, the `x/upgrade` module checks if there exists a
`Plan` that should execute (is scheduled at that height). If so, the corresponding
`Handler` is executed. If the `Plan` is expected to execute but no `Handler` is registered
or if the binary was upgraded too early, the node will gracefully panic and exit.

### StoreLoader

The `x/upgrade` module also facilitates store migrations as part of the upgrade. The
`StoreLoader` sets the migrations that need to occur before the new binary can
successfully run the chain. This `StoreLoader` is also application specific and
not defined on a per-module basis. Registering this `StoreLoader` is done via
`app#SetStoreLoader` in the application.

```go
func UpgradeStoreLoader (upgradeHeight int64, storeUpgrades *store.StoreUpgrades) baseapp.StoreLoader
```

If there's a planned upgrade and the upgrade height is reached, the old binary writes `UpgradeInfo` to the disk before panic'ing.

```go
type UpgradeInfo struct {
  Name    string
  Height  int64
}
```

This information is critical to ensure the `StoreUpgrades` happens smoothly at correct height and
expected upgrade. It eliminiates the chances for the new binary to execute `StoreUpgrades` multiple
times everytime on restart. Also if there are multiple upgrades planned on same height, the `Name`
will ensure these `StoreUpgrades` takes place only in planned upgrade handler.

### Proposal

Typically, a `Plan` is proposed and submitted through governance via a `SoftwareUpgradeProposal`.
This proposal prescribes to the standard governance process. If the proposal passes,
the `Plan`, which targets a specific `Handler`, is persisted and scheduled. The
upgrade can be delayed or hastened by updating the `Plan.Height` in a new proposal.

```go
type SoftwareUpgradeProposal struct {
  Title       string
  Description string
  Plan        Plan
}
```

#### Cancelling Upgrade Proposals

Upgrade proposals can be cancelled. There exists a `CancelSoftwareUpgrade` proposal type, which can be voted on and passed and will remove the scheduled upgrade `Plan`. Of course this requires that the upgrade was known to be a bad idea well before the upgrade itself, to allow time for a vote.

If such a possibility is desired, the upgrade height is to be `2 * (VotingPeriod + DepositPeriod) + (SafetyDelta)` from the beginning of the upgrade proposal. The `SafetyDelta` is the time available from the success of an upgrade proposal and the realization it was a bad idea (due to external social consensus).

A `CancelSoftwareUpgrade` proposal can also be made while the original `SoftwareUpgradeProposal` is still being voted upon, as long as the `VotingPeriod` ends after the `SoftwareUpgradeProposal`.

## State

The internal state of the `x/upgrade` module is relatively minimal and simple. The state contains the currently active upgrade `Plan` (if one exists) by key `0x0` and if a `Plan` is marked as "done" by key `0x1`. The state contains the consensus versions of all app modules in the application. The versions are stored as big endian `uint64`, and can be accessed with prefix `0x2` appended by the corresponding module name of type `string`. The state maintains a `Protocol Version` which can be accessed by key `0x3`.

- Plan: `0x0 -> Plan`
- Done: `0x1 | byte(plan name)  -> BigEndian(Block Height)`
- ConsensusVersion: `0x2 | byte(module name)  -> BigEndian(Module Consensus Version)`
- ProtocolVersion: `0x3 -> BigEndian(Protocol Version)`

The `x/upgrade` module contains no genesis state.


