---
title: Governance
weight: 100
---

{{< alert >}}
**Note**

XPLA Chain's Governance module inherits from Cosmos SDK's [`gov`](https://docs.cosmos.network/v0.45/modules/gov/) module. This document is a stub, and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /alert >}}

Governance is the process through which members of the XPLA Chain community can effect change on the protocol by submitting petitions known as "proposals" and arriving at a popular consensus when a threshold amount of support has been reached. The proposal structure is versatile and allows for holders of staked XPLA (those who have an interest in the long-term viability of the network) to voice their opinion on both blockchain parameter updates and the future development of the XPLA Chain.

Check the [Governance section of the `xplad` reference]({{< ref "subcommands#tx-gov-submit-proposal" >}}) to see examples of how to participate in the Governance process.

To learn how to vote with your staked XPLA or submit proposals, visit the [XPLA Vault governance guide]({{< ref "/docs/learn/xpla-vault/governance" >}}).

## Concepts

_Disclaimer: This is work in progress. Mechanisms are susceptible to change._

The governance process is divided in a few steps that are outlined below:

- **Proposal submission:** Proposal is submitted to the blockchain with a
  deposit.
- **Vote:** Once deposit reaches a certain value (`MinDeposit`), proposal is
  confirmed and vote opens. Bonded Xpla holders can then send `TxGovVote`
  transactions to vote on the proposal.
- If the proposal involves a software upgrade:
    - **Signal:** Validators start signaling that they are ready to switch to the
    new version.
    - **Switch:** Once more than 75% of validators have signaled that they are
    ready to switch, their software automatically flips to the new version.

### Proposal submission

#### Right to submit a proposal

Any Xpla holder, whether bonded or unbonded, can submit proposals by sending a
`TxGovProposal` transaction. Once a proposal is submitted, it is identified by
its unique `proposalID`.

#### Proposal types

In the initial version of the governance module, there are five types of
proposals:

- `TextProposal` All the proposals that do not involve a modification of
  the source code go under this type. For example, an opinion poll would use a
  proposal of type `TextProposal`.
- `SoftwareUpgradeProposal`. If accepted, validators are expected to update
  their software in accordance with the proposal. They must do so by following
  a 2-steps process described in the [Software Upgrade](#software-upgrade)
  section below. Software upgrade roadmap may be discussed and agreed on via
  `TextProposals`, but actual software upgrades must be performed via
  `SoftwareUpgradeProposals`.
- `CommunityPoolSpendProposal` details a proposal for use of community funds,
  together with how many coins are proposed to be spent, and to which recipient account.
- `ParameterChangeProposal` defines a proposal to change one or
  more parameters. If accepted, the requested parameter change is updated
  automatically by the proposal handler upon conclusion of the voting period.
- `CancelSoftwareUpgradeProposal` is a gov Content type for cancelling a software upgrade.

Other modules may expand upon the governance module by implementing their own
proposal types and handlers. These types are registered and processed through the
governance module (eg. `ParamChangeProposal`), which then execute the respective
module's proposal handler when a proposal passes. This custom handler may perform
arbitrary state changes.

### Deposit

To prevent spam, proposals must be submitted with a deposit in the coins defined in the `MinDeposit` param. The voting period will not start until the proposal's deposit equals `MinDeposit`.

When a proposal is submitted, it has to be accompanied by a deposit that must be strictly positive, but can be inferior to `MinDeposit`. The submitter doesn't need to pay for the entire deposit on their own. If a proposal's deposit is inferior to `MinDeposit`, other token holders can increase the proposal's deposit by sending a `Deposit` transaction. The deposit is kept in an escrow in the governance `ModuleAccount` until the proposal is finalized (passed or rejected).

Once the proposal's deposit reaches `MinDeposit`, it enters voting period. If proposal's deposit does not reach `MinDeposit` before `MaxDepositPeriod`, proposal closes and nobody can deposit on it anymore.

#### Deposit refund and burn

When a the a proposal finalized, the coins from the deposit are either refunded or burned, according to the final tally of the proposal:

- If the proposal is approved or if it's rejected but _not_ vetoed, deposits will automatically be refunded to their respective depositor (transferred from the governance `ModuleAccount`).
- When the proposal is vetoed with a supermajority, deposits be burned from the governance `ModuleAccount`.

### Vote

#### Participants

_Participants_ are users that have the right to vote on proposals. On the
XPLA Chain, participants are bonded Xpla holders. Unbonded Xpla holders and
other users do not get the right to participate in governance. However, they
can submit and deposit on proposals.

Note that some _participants_ can be forbidden to vote on a proposal under a
certain validator if:

- _participant_ bonded or unbonded Xplas to said validator after proposal
  entered voting period.
- _participant_ became validator after proposal entered voting period.

This does not prevent _participant_ to vote with Xplas bonded to other
validators. For example, if a _participant_ bonded some Xplas to validator A
before a proposal entered voting period and other Xplas to validator B after
proposal entered voting period, only the vote under validator B will be
forbidden.

#### Voting period

Once a proposal reaches `MinDeposit`, it immediately enters `Voting period`. We
define `Voting period` as the interval between the moment the vote opens and
the moment the vote closes. `Voting period` should always be shorter than
`Unbonding period` to prevent double voting. The initial value of
`Voting period` is 2 weeks.

#### Option set

The option set of a proposal refers to the set of choices a participant can
choose from when casting its vote.

The initial option set includes the following options:

- `Yes`
- `No`
- `NoWithVeto`
- `Abstain`

`NoWithVeto` counts as `No` but also adds a `Veto` vote. `Abstain` option
allows voters to signal that they do not intend to vote in favor or against the
proposal but accept the result of the vote.

_Note: from the UI, for urgent proposals we should maybe add a ‘Not Urgent’
option that casts a `NoWithVeto` vote._

#### Weighted Votes

[ADR-037](https://docs.cosmos.network/v0.45/architecture/adr-037-gov-split-vote.html) introduces the weighted vote feature which allows a staker to split their votes into several voting options. For example, it could use 70% of its voting power to vote Yes and 30% of its voting power to vote No.

Often times the entity owning that address might not be a single individual. For example, a company might have different stakeholders who want to vote differently, and so it makes sense to allow them to split their voting power. Currently, it is not possible for them to do "passthrough voting" and giving their users voting rights over their tokens. However, with this system, exchanges can poll their users for voting preferences, and then vote on-chain proportionally to the results of the poll.

To represent weighted vote on chain, we use the following Protobuf message.

```go
// Vote defines a vote on a governance proposal.
// A Vote consists of a proposal ID, the voter, and the vote option.
type Vote struct {
	ProposalId uint64 `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty" yaml:"proposal_id"`
	Voter      string `protobuf:"bytes,2,opt,name=voter,proto3" json:"voter,omitempty"`
	// Deprecated: Prefer to use `options` instead. This field is set in queries
	// if and only if `len(options) == 1` and that option has weight 1. In all
	// other cases, this field will default to VOTE_OPTION_UNSPECIFIED.
	Option VoteOption `protobuf:"varint,3,opt,name=option,proto3,enum=cosmos.gov.v1beta1.VoteOption" json:"option,omitempty"` // Deprecated: Do not use.
	// Since: cosmos-sdk 0.43
	Options []WeightedVoteOption `protobuf:"bytes,4,rep,name=options,proto3" json:"options"`
}

// WeightedVoteOption defines a unit of vote for vote split.
//
// Since: cosmos-sdk 0.43
type WeightedVoteOption struct {
	Option VoteOption                             `protobuf:"varint,1,opt,name=option,proto3,enum=cosmos.gov.v1beta1.VoteOption" json:"option,omitempty"`
	Weight github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=weight,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"weight" yaml:"weight"`
}
```

For a weighted vote to be valid, the `options` field must not contain duplicate vote options, and the sum of weights of all options must be equal to 1.

#### Quorum

Quorum is defined as the minimum percentage of voting power that needs to be
casted on a proposal for the result to be valid.

#### Threshold

Threshold is defined as the minimum proportion of `Yes` votes (excluding
`Abstain` votes) for the proposal to be accepted.

Initially, the threshold is set at 50% with a possibility to veto if more than
1/3rd of votes (excluding `Abstain` votes) are `NoWithVeto` votes. This means
that proposals are accepted if the proportion of `Yes` votes (excluding
`Abstain` votes) at the end of the voting period is superior to 50% and if the
proportion of `NoWithVeto` votes is inferior to 1/3 (excluding `Abstain`
votes).

#### Inheritance

If a delegator does not vote, it will inherit its validator vote.

- If the delegator votes before its validator, it will not inherit from the
  validator's vote.
- If the delegator votes after its validator, it will override its validator
  vote with its own. If the proposal is urgent, it is possible
  that the vote will close before delegators have a chance to react and
  override their validator's vote. This is not a problem, as proposals require more than 2/3rd of the total voting power to pass before the end of the voting period. If more than 2/3rd of validators collude, they can censor the votes of delegators anyway.

#### Validator’s punishment for non-voting

At present, validators are not punished for failing to vote.

#### Governance address

Later, we may add permissioned keys that could only sign txs from certain modules. For the MVP, the `Governance address` will be the main validator address generated at account creation. This address corresponds to a different PrivKey than the Tendermint PrivKey which is responsible for signing consensus messages. Validators thus do not have to sign governance transactions with the sensitive Tendermint PrivKey.

### Software Upgrade

If proposals are of type `SoftwareUpgradeProposal`, then nodes need to upgrade
their software to the new version that was voted. This process is divided in
two steps.

#### Signal

After a `SoftwareUpgradeProposal` is accepted, validators are expected to
download and install the new version of the software while continuing to run
the previous version. Once a validator has downloaded and installed the
upgrade, it will start signaling to the network that it is ready to switch by
including the proposal's `proposalID` in its _precommits_.(_Note: Confirmation
that we want it in the precommit?_)

Note: There is only one signal slot per _precommit_. If several
`SoftwareUpgradeProposals` are accepted in a short timeframe, a pipeline will
form and they will be implemented one after the other in the order that they
were accepted.

#### Switch

Once a block contains more than 2/3rd _precommits_ where a common
`SoftwareUpgradeProposal` is signaled, all the nodes (including validator
nodes, non-validating full nodes and light-nodes) are expected to switch to the
new version of the software.

_Note: Not clear how the flip is handled programmatically_

## Data

### Proposal

`Proposal` objects are used to account votes and generally track the proposal's state. They contain `Content` which denotes
what this proposal is about, and other fields, which are the mutable state of
the governance process.

```go
// Proposal defines the core field members of a governance proposal.
type Proposal struct {
	ProposalId       uint64                                   `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"id" yaml:"id"`
	Content          *types1.Any                              `protobuf:"bytes,2,opt,name=content,proto3" json:"content,omitempty"`
	Status           ProposalStatus                           `protobuf:"varint,3,opt,name=status,proto3,enum=cosmos.gov.v1beta1.ProposalStatus" json:"status,omitempty" yaml:"proposal_status"`
	FinalTallyResult TallyResult                              `protobuf:"bytes,4,opt,name=final_tally_result,json=finalTallyResult,proto3" json:"final_tally_result" yaml:"final_tally_result"`
	SubmitTime       time.Time                                `protobuf:"bytes,5,opt,name=submit_time,json=submitTime,proto3,stdtime" json:"submit_time" yaml:"submit_time"`
	DepositEndTime   time.Time                                `protobuf:"bytes,6,opt,name=deposit_end_time,json=depositEndTime,proto3,stdtime" json:"deposit_end_time" yaml:"deposit_end_time"`
	TotalDeposit     github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,7,rep,name=total_deposit,json=totalDeposit,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"total_deposit" yaml:"total_deposit"`
	VotingStartTime  time.Time                                `protobuf:"bytes,8,opt,name=voting_start_time,json=votingStartTime,proto3,stdtime" json:"voting_start_time" yaml:"voting_start_time"`
	VotingEndTime    time.Time                                `protobuf:"bytes,9,opt,name=voting_end_time,json=votingEndTime,proto3,stdtime" json:"voting_end_time" yaml:"voting_end_time"`
}

type Content interface {
	GetTitle() string
	GetDescription() string
	ProposalRoute() string
	ProposalType() string
	ValidateBasic() sdk.Error
	String() string
}
```

The `Content` on a proposal is an interface which contains the information about
the `Proposal` such as the tile, description, and any notable changes. Also, this
`Content` type can by implemented by any module. The `Content`'s `ProposalRoute`
returns a string which must be used to route the `Content`'s `Handler` in the
governance keeper. This allows the governance keeper to execute proposal logic
implemented by any module. If a proposal passes, the handler is executed. Only
if the handler is successful does the state get persisted and the proposal finally
passes. Otherwise, the proposal is rejected.

## Message Types

### MsgSubmitProposal

```go
// MsgSubmitProposal defines an sdk.Msg type that supports submitting arbitrary
// proposal Content.
type MsgSubmitProposal struct {
	Content        *types.Any                               `protobuf:"bytes,1,opt,name=content,proto3" json:"content,omitempty"`
	InitialDeposit github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=initial_deposit,json=initialDeposit,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"initial_deposit" yaml:"initial_deposit"`
	Proposer       string                                   `protobuf:"bytes,3,opt,name=proposer,proto3" json:"proposer,omitempty"`
}
```

### MsgDeposit

```go
// MsgDeposit defines a message to submit a deposit to an existing proposal.
type MsgDeposit struct {
	ProposalId uint64                                   `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id" yaml:"proposal_id"`
	Depositor  string                                   `protobuf:"bytes,2,opt,name=depositor,proto3" json:"depositor,omitempty"`
	Amount     github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,3,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

### MsgVote

```go
// MsgVote defines a message to cast a vote.
type MsgVote struct {
	ProposalId uint64     `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id" yaml:"proposal_id"`
	Voter      string     `protobuf:"bytes,2,opt,name=voter,proto3" json:"voter,omitempty"`
	Option     VoteOption `protobuf:"varint,3,opt,name=option,proto3,enum=cosmos.gov.v1beta1.VoteOption" json:"option,omitempty"`
}
```

## Proposals

### Text Proposal

```go
type TextProposal struct {
	Title       string `json:"title" yaml:"title"`
	Description string `json:"description" yaml:"description"`
}
```

Text Proposals are used to create general-purpose petitions, such as asking core developers or community members to implement a specific feature. The community can reference a passed Text Proposal to the core developers or community members to indicate that a feature that potentially requires a soft or hard fork is in significant demand.

### Parameter Change Proposals

```go
// ParameterChangeProposal defines a proposal to change one or more parameters.
type ParameterChangeProposal struct {
	Title       string        `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description string        `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	Changes     []ParamChange `protobuf:"bytes,3,rep,name=changes,proto3" json:"changes"`
}

// ParamChange defines an individual parameter change, for use in
// ParameterChangeProposal.
type ParamChange struct {
	Subspace string `protobuf:"bytes,1,opt,name=subspace,proto3" json:"subspace,omitempty"`
	Key      string `protobuf:"bytes,2,opt,name=key,proto3" json:"key,omitempty"`
	Value    string `protobuf:"bytes,3,opt,name=value,proto3" json:"value,omitempty"`
}
```

Parameter Change Proposals are a special type of proposal which, once passed, will automatically go into effect by directly altering the network's specified parameter.

### Software Upgrade Proposals

This type of proposal requires validators to update their node software to a new version at a specified block height.

{{< alert context="danger" >}}
**Danger**

Software upgrade proposals can be difficult to execute. Exercise caution when using this proposal type, as you may lose your deposit due to an incorrect proposal.
{{< /alert >}}

## Proposal Processing Queue

### End-Block

> This section was taken from the official Cosmos SDK docs and placed here for your convenience to understand the Governance process.

`ProposalProcessingQueue`: A queue `queue[proposalID]` containing all the `ProposalIDs` of proposals that reached `MinDeposit`. During each `EndBlock`, all the proposals that have reached the end of their voting period are processed. To process a finished proposal, the application tallies the votes, computes the votes of each validator and checks if every validator in the validator set has voted. If the proposal is accepted, deposits are refunded. Finally, the proposal content `Handler` is executed.

And the pseudocode for the `ProposalProcessingQueue`:

```GO
for finishedProposalID in GetAllFinishedProposalIDs(block.Time)
	proposal = load(Governance, <proposalID|'proposal'>) // proposal is a const key

	validators = Keeper.getAllValidators()
	tmpValMap := map(sdk.AccAddress)ValidatorGovInfo

	// Initiate mapping at 0. This is the amount of shares of the validator's vote that will be overridden by their delegator's votes
	for each validator in validators
	tmpValMap(validator.OperatorAddr).Minus = 0

	// Tally
	voterIterator = rangeQuery(Governance, <proposalID|'addresses'>) //return all the addresses that voted on the proposal
	for each (voterAddress, vote) in voterIterator
	delegations = stakingKeeper.getDelegations(voterAddress) // get all delegations for current voter

	for each delegation in delegations
		// make sure delegation. Shares does NOT include shares being unbonded
		tmpValMap(delegation.ValidatorAddr).Minus += delegation.Shares
		proposal.updateTally(vote, delegation.Shares)

	_, isVal = stakingKeeper.getValidator(voterAddress)
	if (isVal)
		tmpValMap(voterAddress).Vote = vote

	tallyingParam = load(GlobalParams, 'TallyingParam')

	// Update tally if validator voted they voted
	for each validator in validators
	if tmpValMap(validator).HasVoted
		proposal.updateTally(tmpValMap(validator).Vote, (validator.TotalShares - tmpValMap(validator).Minus))

	// Check if proposal is accepted or rejected
	totalNonAbstain := proposal.YesVotes + proposal.NoVotes + proposal.NoWithVetoVotes
	if (proposal.Votes.YesVotes/totalNonAbstain > tallyingParam.Threshold AND proposal.Votes.NoWithVetoVotes/totalNonAbstain  < tallyingParam.Veto)
	//  proposal was accepted at the end of the voting period
	//  refund deposits (non-voters already punished)
	for each (amount, depositor) in proposal.Deposits
		depositor.XplaBalance += amount

	stateWriter, err := proposal.Handler()
	if err != nil
		// proposal passed but failed during state execution
		proposal.CurrentStatus = ProposalStatusFailed
		else
		// proposal pass and state is persisted
		proposal.CurrentStatus = ProposalStatusAccepted
		stateWriter.save()
	else
	// proposal was rejected
	proposal.CurrentStatus = ProposalStatusRejected

	store(Governance, <proposalID|'proposal'>, proposal)
```

## Parameters

The subspace for the Governance module is `gov`.

```go
// DepositParams defines the params for deposits on governance proposals.
type DepositParams struct {
	//  Minimum deposit for a proposal to enter voting period.
	MinDeposit github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,1,rep,name=min_deposit,json=minDeposit,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"min_deposit,omitempty" yaml:"min_deposit"`
	//  Maximum period for Xpla holders to deposit on a proposal. Initial value: 2
	//  months.
	MaxDepositPeriod time.Duration `protobuf:"bytes,2,opt,name=max_deposit_period,json=maxDepositPeriod,proto3,stdduration" json:"max_deposit_period,omitempty" yaml:"max_deposit_period"`
}

// TallyParams defines the params for tallying votes on governance proposals.
type TallyParams struct {
	//  Minimum percentage of total stake needed to vote for a result to be
	//  considered valid.
	Quorum github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,1,opt,name=quorum,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"quorum,omitempty"`
	//  Minimum proportion of Yes votes for proposal to pass. Default value: 0.5.
	Threshold github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,2,opt,name=threshold,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"threshold,omitempty"`
	//  Minimum value of Veto votes to Total votes ratio for proposal to be
	//  vetoed. Default value: 1/3.
	VetoThreshold github_com_cosmos_cosmos_sdk_types.Dec `protobuf:"bytes,3,opt,name=veto_threshold,json=vetoThreshold,proto3,customtype=github.com/cosmos/cosmos-sdk/types.Dec" json:"veto_threshold,omitempty" yaml:"veto_threshold"`
}

// VotingParams defines the params for voting on governance proposals.
type VotingParams struct {
	//  Length of the voting period.
	VotingPeriod time.Duration `protobuf:"bytes,1,opt,name=voting_period,json=votingPeriod,proto3,stdduration" json:"voting_period,omitempty" yaml:"voting_period"`
}
```


### MinDeposit

- type: `Coins`
- denom: `axpla`
- amount: `10000000000000000000`


The minimum deposit amount for a proposal to enter a voting period. Currently 10 XPLA.

### MaxDepositPeriod

- type: `time.Duration` (seconds)
- `"max_deposit_period": "172800s"`

Maximum period for XPLA holders to deposit on a proposal. Currently 2 days.

### Quorum

- type: `Dec`
- `"quorum": "0.334000000000000000",`

Minimum percentage of total stake needed to vote for a result to be considered valid.

### Threshold

- type: `Dec`
- `"threshold": "0.500000000000000000"`

Minimum proportion of Yes votes for a proposal to pass.

### Veto

- type: `Dec`
- `"veto_threshold": "0.334000000000000000"`

Minimum value of Veto votes to Total votes ratio for proposal to be vetoed.

### VotingPeriod

- type: `time.Duration` (seconds)
-  `"voting_period": "604800s"`

Length of the voting period. Currently 1 week.
