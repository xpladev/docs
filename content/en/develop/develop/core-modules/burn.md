---
title: Burn
weight: 210
type: docs
mermaid: true
---

The burn module is responsible for handling token burning operations through governance proposals. This module allows the XPLA Chain community to burn tokens by submitting and passing governance proposals.

## Concepts

The burn module provides a mechanism for token burning through on-chain governance. This allows the community to make decisions about token supply management in a decentralized manner.

### Burn Proposal

Burn proposals are special governance proposals that, when passed, automatically execute token burning operations. These proposals allow the community to vote on whether specific amounts of tokens should be burned from the total supply.

## State

### BurnProposal

`BurnProposal` represents an ongoing burn proposal that is being tracked by the burn module. This structure contains the proposal details and is stored in the module's state.

```go
// BurnProposal defines an ongoing burn proposal
type BurnProposal struct {
	ProposalId uint64                                   `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty"`
	Proposer   string                                   `protobuf:"bytes,2,opt,name=proposer,proto3" json:"proposer,omitempty"`
	Amount     github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,3,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

#### Fields

- **ProposalId**: The unique identifier of the governance proposal
- **Proposer**: The address of the account that submitted the burn proposal
- **Amount**: The amount of tokens to be burned if the proposal passes

## Message Types

### MsgBurn

`MsgBurn` is the message type used to execute token burning operations. This message is submitted as part of a governance proposal and is executed automatically when the proposal passes.

```go
// MsgBurn represents a message to burn coins from an account.
type MsgBurn struct {
    // authority is the address of the governance account.
	Authority string                                   `protobuf:"bytes,1,opt,name=authority,proto3" json:"authority,omitempty" yaml:"authority"`
    // Amount of tokens to burn
    Amount github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

#### Fields

- **Authority**: The address of the governance module account that has permission to execute burn operations
- **Amount**: The amount of tokens to be burned from the total supply

## Proposals



{{< alert context="warning" >}}
**Note**

Burn proposals follow the same governance process as other proposal types. They require a minimum deposit and must pass the voting period to be executed.
{{< /alert >}}

## Transitions

### InitGenesis

`InitGenesis` initializes the burn module genesis state by setting up the initial burn module state.

### ExportGenesis

`ExportGenesis` exports the genesis state of the burn module, including any ongoing burn proposals.

## Hooks

The burn module implements governance hooks to handle the lifecycle of burn proposals. These hooks are automatically triggered by the governance module at specific points during the proposal process.

### GovHooks Implementation

The burn module implements the `govtypes.GovHooks` interface to handle governance proposal events:

```go
type BankGovHooks struct {
    keeper     Keeper
    bankKeeper types.BankKeeper
    govKeeper  types.GovKeeper
}
```

### Hook Types

#### AfterProposalSubmission

This hook is triggered immediately after a proposal is submitted. For burn proposals, it:

1. Extracts the `MsgBurn` from the proposal messages
2. Creates a `BurnProposal` record and stores it in the module state
3. Transfers the burn amount from the proposer to the burn module account

```go
func (h BankGovHooks) AfterProposalSubmission(ctx context.Context, proposalID uint64) error {
    // Extract MsgBurn from proposal
    // Create BurnProposal record
    // Transfer burn amount to module account
}
```

#### AfterProposalFailedMinDeposit

This hook is triggered when a proposal fails to reach the minimum deposit requirement. For burn proposals, it:

1. Retrieves the stored `BurnProposal` record
2. Returns the burn amount to the original proposer
3. Removes the proposal from the ongoing burn proposals state

```go
func (h BankGovHooks) AfterProposalFailedMinDeposit(ctx context.Context, proposalID uint64) error {
    // Return burn amount to proposer
    // Remove from ongoing burn proposals
}
```

#### AfterProposalVotingPeriodEnded

This hook is triggered when the voting period for a proposal ends. For burn proposals, it:

1. Removes the proposal from the ongoing burn proposals state
2. If the proposal passed, the burn amount remains in the module account (will be burned)
3. If the proposal failed, returns the burn amount to the proposer

```go
func (h BankGovHooks) AfterProposalVotingPeriodEnded(ctx context.Context, proposalID uint64) error {
    // Remove from ongoing burn proposals
    // If passed: keep amount for burning
    // If failed: return amount to proposer
}
```

### Hook Registration

The burn module hooks are registered with the governance module during application initialization to ensure proper handling of burn proposal lifecycle events.

## Integration with Governance

The burn module integrates with the governance module to handle burn proposals. When a burn proposal is submitted:

1. The proposal enters the deposit period
2. Once the minimum deposit is reached, the proposal enters the voting period
3. If the proposal passes, the burn operation is automatically executed
4. The specified amount of tokens is burned from the total supply

### Burn Proposal Flow

The following sequence diagram illustrates the complete flow of a burn proposal from submission to execution:

{{< mermaid >}}
sequenceDiagram
    participant Proposer
    participant GovModule
    participant BurnModule
    participant BankModule
    participant Validator

    Note over Proposer: 110 XPLA
    Note over GovModule: 0 XPLA (deposit)
    Note over BurnModule: 0 XPLA (burn amount)
    Note over BankModule: 1000 XPLA (supply)

    Proposer->>GovModule: Submit Proposal with MsgBurn
    
    Note over Proposer, GovModule: AfterProposalSubmission Hook
    GovModule->>BurnModule: Extract MsgBurn & Create BurnProposal
    Proposer->>BurnModule: Transfer burnAmount 100 XPLA 
    Note over Proposer: 10 XPLA
    Note over BurnModule: 100 XPLA (burn amount)

    Proposer->>GovModule: Transfer Deposit Amount 10 XPLA 
    Note over Proposer: 0 XPLA
    Note over GovModule: 10 XPLA (deposit amount)

    Proposer->>Validator: Request Vote
    Validator->>GovModule: Vote YES
    
    Note over GovModule: Voting Period Ends
    Note over GovModule: Proposal Status: PASSED

    Note over GovModule: AfterProposalVotingPeriodEnded Hook
    GovModule->>BurnModule: Remove from OngoingBurnProposals
    Note over GovModule: Execute Burn Proposal
    GovModule->>BurnModule: Execute MsgBurn
    BurnModule->>BankModule: Burn 100 XPLA burnAmount
    Note over BurnModule: 0 XPLA
    Note over BankModule: 900 XPLA (supply)
{{< /mermaid >}}

## Queries

The burn module provides query endpoints to retrieve information about burn operations and proposals:

### QueryOngoingProposal

This query allows users to retrieve information about ongoing burn proposals by proposal ID.

```go
// QueryOngoingProposalRequest is the request type for the Query/OngoingProposal RPC method.
type QueryOngoingProposalRequest struct {
    ProposalId uint64 `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty"`
}

// QueryOngoingProposalResponse is the response type for the Query/OngoingProposal RPC method.
type QueryOngoingProposalResponse struct {
    Proposal *BurnProposal `protobuf:"bytes,1,opt,name=proposal,proto3" json:"proposal,omitempty"`
}
```

### Available Queries

- **QueryOngoingProposal**: Retrieve details of a specific ongoing burn proposal by proposal ID
- **QueryOngoingProposals**: List all ongoing burn proposals