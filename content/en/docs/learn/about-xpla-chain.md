---
title: About XPLA Chain
weight: 10
---

## What is XPLA Chain?

XPLA Chain is an [open-source blockchain](https://github.com/xpladev/xpla) hosting a vibrant ecosystem of decentralized applications (dApps) and top-tier developer tools. XPLA Chain is based on [Tendermint](https://tendermint.com/), a popular blockchain engine based on Byzantine Fault Tolerant (BFT) consensus which is robust against double-spend attacks and is tolerant against a set of up to 1/3. Through [IBC](https://ibc.cosmos.network/) and [COSMOS SDK](https://docs.cosmos.network/), the XPLA Chain can be easily connected with other various chains and is a developer-friendly chain. Using proof-of-stake consensus, [Ethereum Virtual Machine (EVM)](https://ethereum.org/en/developers/docs/evm/) Compatibility, XPLA Chain SDK, a software kit for the game developers, and XPLA Vault would give the users extensive experiences of De-Fi, Play to Own (P2O) gaming, and lastly, would lead to the adoption of Web 2 users to Web 3 space. Also, XPLA, all capital letters of the blockchain name, is the XPLA Chain’s native staking token. XPLA is used for [governance]({{< ref "glossary#governance" >}}) and [staking]({{< ref "glossary#staking" >}}).

## Validators

Validators are the miners of the XPLA Chain. They are responsible for securing the XPLA Chain and ensuring its accuracy. Validators run programs called full nodes which allow them to verify each transaction made on the XPLA Chain. Validators propose blocks, vote on their validity, and add each new block to the chain in exchange for staking rewards from transaction fees. Users can stake their XPLA to validators in exchange for staking rewards. Validators also play an important role in the governance of the XPLA Chain.

For more information on validators, visit the [Validator FAQ]({{< ref "/docs/full-node/manage-a-validator/validator-faq" >}}).

### Consensus

The XPLA Chain is a proof-of-stake blockchain, powered by the [Cosmos SDK](https://cosmos.network/) and secured by a system of verification called the Tendermint consensus.

The following process explains how Tendermint consensus works. For more information on the Tendermint consensus, visit the [official Tendermint documentation](https://docs.tendermint.com/).

1. A validator called a **proposer** is chosen to submit a new block of transactions.
2. Validators vote in two rounds on whether they accept or reject the proposed block. If a block is rejected, a new proposer is selected and the process starts again.
3. If accepted, the block is signed and added to the chain.
4. The transaction fees from the block are distributed as staking rewards to validators and delegators. Proposers get rewarded extra for their participation.

This process repeats, adding new blocks of transactions to the chain. Each validator has a copy of all transactions made on the network, which they compare against the proposed block of transactions before voting. Because multiple independent validators take place in consensus voting, it is infeasible for any false block to be accepted. In this way, validators protect the integrity of the XPLA Chain and ensure the validity of each transaction.

## Staking

Staking is the process of bonding XPLA to a validator in exchange for staking rewards.

The XPLA Chain only allows the top 130 validators to participate in consensus. A validator's rank is determined by their stake or the total amount of XPLA bonded to them. Although validators can bond XPLA to themselves, they mainly amass larger stakes from delegators. Validators with larger stakes get chosen more often to propose new blocks and earn proportionally more rewards.

To learn how to stake your XPLA and earn staking rewards, visit the [XPLA Vault staking guide]({{< ref "xpla-vault/manage-staking" >}})

### Delegators
Delegators are users who want to receive rewards from consensus without running a full node. Any user that stakes XPLA is a delegator. Delegators stake their XPLA to a validator, adding to a validator’s weight, or total stake. In return, delegators receive a portion of transaction fees as staking rewards.

{{< alert context="warning" >}}
**Who owns staked XPLA?**

Staked XPLA never leaves the possession of the delegator. Even though it can’t be traded freely, staked XPLA is never owned by a validator. For more information, visit the [Validator FAQ]({{< ref "/docs/full-node/manage-a-validator/validator-faq#can-a-validator-run-away-with-a-delegators-xpla" >}})
{{< /alert >}}

### Phases of XPLA

To start receiving rewards, delegators bond their XPLA to a validator. The bonding process adds a delegator's XPLA to a validator's stake, which helps validators to participate in consensus.

XPLA exists in the following three phases:

- **Unbonded**: XPLA that can be freely traded and is not staked to a validator.
- **Bonded**: XPLA that is staked to a validator. Bonded XPLA accrues staking rewards. XPLA bonded to validators in XPLA Vault can’t be traded freely.
- **Unbonding**: XPLA that is in the process of becoming unbonded from a validator and does not accrue rewards. This process takes 21 days to complete.

### Bonding, staking, and delegating

Generally, the terms bonding, staking, and delegating can be used interchangeably, as they happen in the same step. A delegator delegates XPLA to a validator, the XPLA gets bonded to the validator, and the bonded XPLA gets added to the validator's stake.

Delegators can bond XPLA to any validator in the [active set]({{< ref "glossary#active-set" >}}) using the delegate function in XPLA Vault. Delegators start earning staking rewards the moment they bond or stake to a validator.

### Unbonding

Delegators can unbond or unstake their XPLA using the undelegate function in XPLA Vault. The unbonding process takes 21 days to complete. During this period, the unbonding XPLA can't be traded, and no staking rewards accrue.

{{< alert context="warning" >}}
**Warning**

Once started, the delegating or undelegating processes can't be stopped.
Undelegating takes 21 days to complete. The only way to undo a delegating or undelegating transaction is to wait for the unbonding process to pass. Alternatively, you can redelegate staked XPLA to a different validator without waiting 21 days.
{{< /alert >}}

The 21-day unbonding process helps the long-term stability of the XPLA Chain. The unbonding period discourages volatility by locking staked XPLA in the system for at least 21 days. In exchange, delegators receive staking rewards, further incentivizing network stability.

### Redelegation

Redelegating instantly sends staked XPLA from one validator to another. Instead of waiting for the 21-day unstaking period, a user can redelegate their staked XPLA at any time using XPLA Vault's redelegate function. Validators receiving redelegations are barred from further redelegating any amount of XPLA to any validator for 21 days.

{{< alert context="warning" >}}
**Warning**

When a user redelegates staked XPLA from one validator to another, the validator receiving the staked XPLA is barred from making further redelegation transactions for 21 days. This requirement only applies to the wallet that made the redelegation transaction.
{{< /alert >}}

## Rewards

The XPLA Chain incentivizes validators and delegators with staking rewards from gas fees and inflation rewards:

- [Gas]({{< ref "fees#gas" >}}): Compute fees added on to each transaction to avoid spamming. Validators set minimum gas prices and reject transactions that have implied gas prices below this threshold.

- [Inflation rewards]({{< ref "docs/develop/core-modules/mint" >}}): Every block, new XPLA is minted and released to validators and delegators as staking rewards only if the inflation on the XPLA Chain exceeds 0%. As the current rate for the minting of new XPLA is fixed at 0% per year, there are no extra rewards from inflation.

- Rewards from XPLA foundation's delegation: 20% of rewards that the foundation earns are released to validators and delegators.

For more information on fees, visit the [fee page]({{< ref "fees" >}}).

At the end of every block, transaction fees and inflation rewards are distributed to each validator and their delegators proportional to their staked amount. Validators can keep a portion of rewards to pay for their services. This portion is called commission. The rest of the rewards are distributed to delegators according to their staked amounts.

### Slashing

Running a validator is a big responsibility. Validators must meet strict standards and constantly monitor and participate in the consensus process. Slashing is the penalty for misbehaving validators. When a validator gets slashed, they lose a small portion of their stake as well as a small portion of their delegator's stake. Slashed validators also get jailed, or excluded, from consensus for a period of time.

{{< alert context="danger" >}}
**The risks of staking**

Slashing affects validators and delegators. When a validator gets slashed, delegators who stake to that validator also get slashed. Slashing is proportional to a delegator's staked amount. Though slashing is rare and usually results in a small penalty, it does occur. Delegators should monitor their validators closely, do their research, and understand the risks of staking XPLA.
{{< /alert >}}

Slashing occurs under the following conditions:

- **Double signing**: When a validator signs two different blocks with the same chain ID at the same height.
- **Downtime**: When a validator is unresponsive or can't be reached for a period of time.
- **Missed votes**: When a validator misses votes in consensus.

Validators monitor each other closely and can submit evidence of misbehavior. Once discovered, the misbehaving validator will have a small portion of their funds slashed. Offending validators will also be jailed or excluded from consensus for a period of time. Even simple issues such as malfunctions or downtimes from upgrading can lead to slashing.

For more information on slashing, visit the [slashing module]({{< ref "/docs/develop/core-modules/slashing" >}}).

## Governance

The XPLA Chain is a decentralized public [blockchain]({{< ref "glossary#blockchain" >}}) governed by community members. Governance is the democratic process that allows users and validators to make changes to the XPLA Chain. Community members submit, vote, and implement proposals.

To learn how to vote with your staked XPLA or submit proposals, visit the [XPLA Vault governance guide]({{< ref "xpla-vault/governance" >}}).

### Proposals

Proposals start as ideas within the community. A community member drafts and submits a proposal alongside an initial deposit.

The most common proposal types include:

- `ParameterChangeProposal`: To change the parameters defined in each module.
- `CommunityPoolSpendProposal`: To spend funds in the community pool.
- `TextProposal` : To handle other issues like large directional changes or any decision requiring manual implementation.

### Voting process

Community members vote with their staked XPLA. One staked XPLA equals one vote. If a user fails to specify a vote, their vote defaults to the validator they are staked to. Validators vote with their entire stake unless specified by delegators. For this reason, it is very important that each delegator votes according to their preferences.

The following is a basic outline of the governance process. Visit the [governance module]({{< ref "/docs/develop/core-modules/governance" >}}) for more details.

1. A user submits a proposal and a two-week deposit period begins.
2. Users deposit XPLA as collateral to back the proposal. This period ends once a minimum threshold of 50 XPLA is deposited. Deposits are to protect against spam.
3. The one-week vote period begins.
    The voting options are:
    - `Yes`: In favor.
    - `No`: Not in favor.
    - `NoWithVeto`: Not in favor, the deposit should be burned.
    - `Abstain`: Voter abstains.
4. The votes are tallied.
    Proposals pass if they meet three conditions:
    - `Quorum` is met: at least 33.4% of all staked XPLA must vote.
    - The total number of `NoWithVeto` votes is less than 33.4% of the total vote.
    - The number of `Yes` votes reaches a 50% majority.
    If the previous conditions are not met, the proposal is rejected.
5. Accepted proposals get put into effect.
6. Deposits get refunded or burned.

Once accepted, the changes described in a governance proposal are automatically put into effect by the proposal handler. Generic proposals, such as a passed `TextProposal`, must be reviewed by the XPLA Chain development team and community, and they must be manually implemented.

### Deposits

Deposits protect against unnecessary proposals and spam. Users can veto any proposal they deem to be spam by voting `NoWithVeto`.

Deposits get refunded if all of the following conditions are met:
- The minimum deposit of 50 XPLA is reached within the two-week deposit period.
- `Quorum` is met: the number of total votes is greater than 33.4% of all staked XPLA
- The total number of `NoWithVeto` votes is less than 33.4% of the total vote.
- A vote returns a majority of `Yes` or `No` votes.

Deposits are not returned under any of the following conditions:
- The minimum deposit of 50 XPLA is not reached within the two-week deposit period.
- `Quorum` is not met: the number of total votes after the one-week voting period is less than 33.4% of all staked XPla.
- the number of `NoWithVeto` votes is above 33.4% of the total vote.
