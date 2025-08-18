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

### Ongoing Burn Proposal

The burn module tracks ongoing burn proposals to ensure proper execution when proposals pass.

```go
// OngoingBurnProposal represents a burn proposal that is currently being processed
type OngoingBurnProposal struct {
    // Proposal ID of the burn proposal
    ProposalId uint64 `protobuf:"varint,1,opt,name=proposal_id,json=proposalId,proto3" json:"proposal_id,omitempty"`
    // Amount of tokens to be burned
    Amount github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

## Message Types

### MsgBurn

`MsgBurn` is the message type used to execute token burning operations. This message is typically executed automatically when a burn proposal passes.

```go
// MsgBurn represents a message to burn tokens
type MsgBurn struct {
    // Address that initiated the burn operation
    FromAddress string `protobuf:"bytes,1,opt,name=from_address,json=fromAddress,proto3" json:"from_address,omitempty"`
    // Amount of tokens to burn
    Amount github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,2,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

## Proposals

### BurnProposal

Burn proposals are governance proposals that allow the community to vote on token burning operations. When a burn proposal passes, it automatically executes the burn operation.

```go
// BurnProposal defines a proposal to burn tokens
type BurnProposal struct {
    Title       string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
    Description string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
    // Amount of tokens to burn if the proposal passes
    Amount github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,3,rep,name=amount,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"amount"`
}
```

{{< alert context="warning" >}}
**Note**

Burn proposals follow the same governance process as other proposal types. They require a minimum deposit and must pass the voting period to be executed.
{{< /alert >}}

## Transitions

### InitGenesis

`InitGenesis` initializes the burn module genesis state by setting up the initial burn module state.

### ExportGenesis

`ExportGenesis` exports the genesis state of the burn module, including any ongoing burn proposals.

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

    Note over GovModule: Execute Burn Proposal
    GovModule->>BurnModule: Execute MsgBurn
    BurnModule->>BankModule: Burn 100 XPLA burnAmount
    Note over BurnModule: 0 XPLA
    Note over BankModule: 900 XPLA (supply)
{{< /mermaid >}}

## Queries

The burn module provides query endpoints to retrieve information about burn operations and proposals:

- Query ongoing burn proposals