---
title: gov
weight: 80
type: docs
---

## Overview

The Governance precompile provides comprehensive access to the Cosmos SDK's `x/gov` module, enabling smart contracts to participate in on-chain governance. It allows for submitting and canceling proposals, depositing funds, and casting votes. Additionally, it offers extensive query capabilities to retrieve information about proposals, votes, deposits, and overall governance parameters.

**Precompile Address**: `0x0000000000000000000000000000000000000805`
**Related Module**: [x/gov](/develop/develop/core-modules/governance/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on proposal complexity and chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Transaction Methods

### `submitProposal`
Submits a new governance proposal.

### `vote`
Casts a single vote on an active proposal.

### `voteWeighted`
Casts a weighted vote, splitting voting power across multiple options.

### `deposit`
Adds funds to a proposal's deposit during the deposit period.

### `cancelProposal`
Cancels a proposal that is still in its deposit period.

## Query Methods

### `getProposal`
Retrieves detailed information about a specific proposal.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile, including the complex return struct
const precompileAbi = [
  "function getProposal(uint64 proposalId) view returns (tuple(uint64 id, address proposer, string metadata, uint64 submit_time, uint64 voting_start_time, uint64 voting_end_time, uint8 status, tuple(string yes_count, string abstain_count, string no_count, string no_with_veto_count) final_tally_result, tuple(string denom, uint256 amount)[] total_deposit, string[] messages) proposal)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input: The ID of the proposal to query
const proposalId = 180;

async function getProposalDetails() {
  try {
    const proposal = await contract.getProposal(proposalId);
    console.log(`Proposal ${proposalId} Details:`, JSON.stringify(proposal, null, 2));
  } catch (error) {
    console.error(`Error fetching proposal ${proposalId}:`, error);
  }
}

getProposalDetails();

```

```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual data.
# This example queries for proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID
curl -X POST --data '{                            
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0xf1610a2800000000000000000000000000000000000000000000000000000000000000b4"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `getProposals`
Retrieves a filtered and paginated list of proposals.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile, including the complex return struct
const precompileAbi = [
  "function getProposals(uint32 proposal_status, address voter, address depositor, tuple(bytes key, uint64 offset, uint64 limit, bool count_total, bool reverse) pagination) view returns (tuple(uint64 id, address proposer, string metadata, uint64 submit_time, uint64 voting_start_time, uint64 voting_end_time, uint8 status, tuple(string yes_count, string abstain_count, string no_count, string no_with_veto_count) final_tally_result, tuple(string denom, uint256 amount)[] total_deposit, string[] messages)[] proposals, tuple(bytes next_key, uint64 total) page_response)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs for filtering and pagination
const pagination = {
  offset: 0,
  key: "0x", // Start from the beginning
  limit: 10,
  count_total: true,
  reverse: false,
};
const proposalStatus = 0; // 0 for Unspecified, 1 for Deposit, 2 for Voting, etc.
const voterAddress = ethers.ZeroAddress; // Filter by voter, or ZeroAddress for none
const depositorAddress = ethers.ZeroAddress; // Filter by depositor, or ZeroAddress for none

async function getProposalsList() {
  try {
    const result = await contract.getProposals(proposalStatus, voterAddress, depositorAddress, pagination);
    // The result object contains 'proposals' and 'page_response'
    console.log("Proposals:", JSON.stringify(result.proposals, null, 2));
    console.log("Pagination Response:", result.page_response);
  } catch (error) {
    console.error("Error fetching proposals:", error);
  }
}

getProposalsList();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# This example queries for the first 10 proposals with any status.
# Data is ABI-encoded: function selector + complex input struct.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x62564c48000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getTallyResult`
Retrieves the current or final vote tally for a proposal.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getTallyResult(uint64 proposalId) view returns (tuple(string yes_count, string abstain_count, string no_count, string no_with_veto_count) tally)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input: The ID of the proposal to get the tally for
const proposalId = 1;

async function getTally() {
  try {
    const tally = await contract.getTallyResult(proposalId);
    console.log(`Tally for Proposal ${proposalId}:`, {
      yes: tally.yes_count,
      abstain: tally.abstain_count,
      no: tally.no_count,
      noWithVeto: tally.no_with_veto_count,
    });
  } catch (error) {
    console.error(`Error fetching tally for proposal ${proposalId}:`, error);
  }
}

getTally();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual data.
# This example queries the tally for proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0xba66a6480000000000000000000000000000000000000000000000000000000000000001"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getVote`
Retrieves the vote cast by a specific address on a proposal.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getVote(uint64 proposalId, address voter) view returns (tuple(uint64 proposal_id, address voter, tuple(uint8 option, string weight)[] options, string metadata) vote)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const proposalId = 1;
const voterAddress = "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"; // Placeholder

async function getVoteDetails() {
  try {
    const vote = await contract.getVote(proposalId, voterAddress);
    console.log("Vote Details:", JSON.stringify(vote, null, 2));
  } catch (error) {
    console.error("Error fetching vote:", error);
  }
}

getVoteDetails();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# This example queries the vote of a specific address on proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID + padded voter address
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x54f6530f0000000000000000000000000000000000000000000000000000000000000001000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getVotes`
Retrieves all votes cast on a proposal, with pagination.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getVotes(uint64 proposalId, tuple(uint64 offset, bytes key, uint64 limit, bool count_total, bool reverse) pagination) view returns (tuple(uint64 proposal_id, address voter, tuple(uint8 option, string weight)[] options, string metadata)[] votes, tuple(bytes next_key, uint64 total) page_response)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const proposalId = 1;
const pagination = {
  offset: 0,
  key: "0x",
  limit: 10,
  count_total: true,
  reverse: false,
};

async function getVotesList() {
  try {
    const result = await contract.getVotes(proposalId, pagination);
    console.log(`Votes for proposal ${proposalId}:`, JSON.stringify(result.votes, null, 2));
    console.log("Pagination Response:", result.page_response);
  } catch (error) {
    console.error("Error fetching votes:", error);
  }
}

getVotesList();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# This example queries for the first 10 votes on proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID + pagination struct.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x6f74a8f300000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getDeposit`
Retrieves deposit information for a specific depositor on a proposal.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getDeposit(uint64 proposalId, address depositor) view returns (tuple(address depositor, tuple(string denom, uint256 amount)[] amount) deposit)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const proposalId = 1;
const depositorAddress = "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"; // Placeholder

async function getDepositDetails() {
  try {
    const deposit = await contract.getDeposit(proposalId, depositorAddress);
    console.log("Deposit Details:", JSON.stringify(deposit, null, 2));
  } catch (error) {
    console.error("Error fetching deposit:", error);
  }
}

getDepositDetails();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# This example queries the deposit of a specific address on proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID + padded depositor address
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x582138a30000000000000000000000000000000000000000000000000000000000000001000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getDeposits`
Retrieves all deposits made on a proposal, with pagination.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getDeposits(uint64 proposalId, tuple(uint64 offset, bytes key, uint64 limit, bool count_total, bool reverse) pagination) view returns (tuple(address depositor, tuple(string denom, uint256 amount)[] amount)[] deposits, tuple(bytes next_key, uint64 total) page_response)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const proposalId = 1;
const pagination = {
  offset: 0,
  key: "0x",
  limit: 10,
  count_total: true,
  reverse: false,
};

async function getDepositsList() {
  try {
    const result = await contract.getDeposits(proposalId, pagination);
    console.log(`Deposits for proposal ${proposalId}:`, JSON.stringify(result.deposits, null, 2));
    console.log("Pagination Response:", result.page_response);
  } catch (error) {
    console.error("Error fetching deposits:", error);
  }
}

getDepositsList();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# This example queries for the first 10 deposits on proposal with ID 1.
# Data is ABI-encoded: function selector + padded proposal ID + pagination struct.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x3677fd5500000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getParams`
Retrieves current governance parameters.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getParams() view returns (tuple(string[] min_deposit, string max_deposit_period, string voting_period, string yes_quorum, string veto_threshold, string min_initial_deposit_ratio, string proposal_cancel_ratio, string proposal_cancel_dest, string min_deposit_ratio) params)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

async function getGovParams() {
  try {
    const params = await contract.getParams();
    console.log("Governance Parameters:", JSON.stringify(params, null, 2));
  } catch (error) {
    console.error("Error fetching governance parameters:", error);
  }
}

getGovParams();
```

**Note**: The `getParams()` function returns a complex nested structure that may require manual ABI decoding in some ethers.js versions. The cURL method is more reliable for this specific function.

```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# Data is ABI-encoded: just the function selector.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x5e615a6b"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### `getConstitution`
Retrieves the current governance constitution.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI for the precompile
const precompileAbi = [
  "function getConstitution() view returns (string constitution)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000805";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

async function getGovConstitution() {
  try {
    const constitution = await contract.getConstitution();
    console.log("Governance Constitution:", constitution);
  } catch (error) {
    console.error("Error fetching governance constitution:", error);
  }
}

getGovConstitution();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# Data is ABI-encoded: just the function selector.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000805",
            "data": "0x04b12c7f"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


## Full Solidity Interface & ABI

```solidity title="Governance Solidity Interface" lines expandable
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The IGov contract's address.
address constant GOV_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000805;

/// @dev The IGov contract's instance.
IGov constant GOV_CONTRACT = IGov(GOV_PRECOMPILE_ADDRESS);

/**
 * @dev VoteOption enumerates the valid vote options for a given governance proposal.
 */
enum VoteOption {
    // Unspecified defines a no-op vote option.
    Unspecified,
    // Yes defines a yes vote option.
    Yes,
    // Abstain defines an abstain vote option.
    Abstain,
    // No defines a no vote option.
    No,
    // NoWithWeto defines a no with veto vote option.
    NoWithWeto
}

/// @dev WeightedVote represents a vote on a governance proposal
struct WeightedVote {
    uint64 proposalId;
    address voter;
    WeightedVoteOption[] options;
    string metadata;
}

/// @dev WeightedVoteOption represents a weighted vote option
struct WeightedVoteOption {
    VoteOption option;
    string weight;
}

/// @dev DepositData represents information about a deposit on a proposal
struct DepositData {
    uint64 proposalId;
    address depositor;
    Coin[] amount;
}

/// @dev TallyResultData represents the tally result of a proposal
struct TallyResultData {
    string yes;
    string abstain;
    string no;
    string noWithVeto;
}

/// @dev ProposalData represents a governance proposal
struct ProposalData {
    uint64 id;
    string[] messages;
    uint32 status;
    TallyResultData finalTallyResult;
    uint64 submitTime;
    uint64 depositEndTime;
    Coin[] totalDeposit;
    uint64 votingStartTime;
    uint64 votingEndTime;
    string metadata;
    string title;
    string summary;
    address proposer;
}

/// @dev Params defines the governance parameters
struct Params {
    int64 votingPeriod;
    Coin[] minDeposit;
    int64 maxDepositPeriod;
    string quorum;
    string threshold;
    string vetoThreshold;
    string minInitialDepositRatio;
    string proposalCancelRatio;
    string proposalCancelDest;
    int64 expeditedVotingPeriod;
    string expeditedThreshold;
    Coin[] expeditedMinDeposit;
    bool burnVoteQuorum;
    bool burnProposalDepositPrevote;
    bool burnVoteVeto;
    string minDepositRatio;
}

/// @author The Evmos Core Team
/// @title Gov Precompile Contract
/// @dev The interface through which solidity contracts will interact with Gov
interface IGov {
    /// @dev SubmitProposal defines an Event emitted when a proposal is submitted.
    /// @param proposer the address of the proposer
    /// @param proposalId the proposal of id
    event SubmitProposal(address indexed proposer, uint64 proposalId);

    /// @dev CancelProposal defines an Event emitted when a proposal is canceled.
    /// @param proposer the address of the proposer
    /// @param proposalId the proposal of id
    event CancelProposal(address indexed proposer, uint64 proposalId);

    /// @dev Deposit defines an Event emitted when a deposit is made.
    /// @param depositor the address of the depositor
    /// @param proposalId the proposal of id
    /// @param amount the amount of the deposit
    event Deposit(address indexed depositor, uint64 proposalId, Coin[] amount);

    /// @dev Vote defines an Event emitted when a proposal voted.
    /// @param voter the address of the voter
    /// @param proposalId the proposal of id
    /// @param option the option for voter
    event Vote(address indexed voter, uint64 proposalId, uint8 option);

    /// @dev VoteWeighted defines an Event emitted when a proposal voted.
    /// @param voter the address of the voter
    /// @param proposalId the proposal of id
    /// @param options the options for voter
    event VoteWeighted(
        address indexed voter,
        uint64 proposalId,
        WeightedVoteOption[] options
    );

    /// TRANSACTIONS

    /// @notice submitProposal creates a new proposal from a protoJSON document.
    /// @dev submitProposal defines a method to submit a proposal.
    /// @param jsonProposal The JSON proposal
    /// @param deposit The deposit for the proposal
    /// @return proposalId The proposal id
    function submitProposal(
        address proposer,
        bytes calldata jsonProposal,
        Coin[] calldata deposit
    ) external returns (uint64 proposalId);

    /// @dev cancelProposal defines a method to cancel a proposal.
    /// @param proposalId The proposal id
    /// @return success Whether the transaction was successful or not
    function cancelProposal(
        address proposer,
        uint64 proposalId
    ) external returns (bool success);

    /// @dev deposit defines a method to add a deposit to a proposal.
    /// @param proposalId The proposal id
    /// @param amount The amount to deposit
    function deposit(
        address depositor,
        uint64 proposalId,
        Coin[] calldata amount
    ) external returns (bool success);

    /// @dev vote defines a method to add a vote on a specific proposal.
    /// @param voter The address of the voter
    /// @param proposalId the proposal of id
    /// @param option the option for voter
    /// @param metadata the metadata for voter send
    /// @return success Whether the transaction was successful or not
    function vote(
        address voter,
        uint64 proposalId,
        VoteOption option,
        string memory metadata
    ) external returns (bool success);

    /// @dev voteWeighted defines a method to add a vote on a specific proposal.
    /// @param voter The address of the voter
    /// @param proposalId The proposal id
    /// @param options The options for voter
    /// @param metadata The metadata for voter send
    /// @return success Whether the transaction was successful or not
    function voteWeighted(
        address voter,
        uint64 proposalId,
        WeightedVoteOption[] calldata options,
        string memory metadata
    ) external returns (bool success);

    /// QUERIES

    /// @dev getVote returns the vote of a single voter for a
    /// given proposalId.
    /// @param proposalId The proposal id
    /// @param voter The voter on the proposal
    /// @return vote Voter's vote for the proposal
    function getVote(
        uint64 proposalId,
        address voter
    ) external view returns (WeightedVote memory vote);

    /// @dev getVotes Returns the votes for a specific proposal.
    /// @param proposalId The proposal id
    /// @param pagination The pagination options
    /// @return votes The votes for the proposal
    /// @return pageResponse The pagination information
    function getVotes(
        uint64 proposalId,
        PageRequest calldata pagination
    )
        external
        view
        returns (WeightedVote[] memory votes, PageResponse memory pageResponse);

    /// @dev getDeposit returns the deposit of a single depositor for a given proposalId.
    /// @param proposalId The proposal id
    /// @param depositor The address of the depositor
    /// @return deposit The deposit information
    function getDeposit(
        uint64 proposalId,
        address depositor
    ) external view returns (DepositData memory deposit);

    /// @dev getDeposits returns all deposits for a specific proposal.
    /// @param proposalId The proposal id
    /// @param pagination The pagination options
    /// @return deposits The deposits for the proposal
    /// @return pageResponse The pagination information
    function getDeposits(
        uint64 proposalId,
        PageRequest calldata pagination
    )
        external
        view
        returns (
            DepositData[] memory deposits,
            PageResponse memory pageResponse
        );

    /// @dev getTallyResult returns the tally result of a proposal.
    /// @param proposalId The proposal id
    /// @return tallyResult The tally result of the proposal
    function getTallyResult(
        uint64 proposalId
    ) external view returns (TallyResultData memory tallyResult);

    /// @dev getProposal returns the proposal details based on proposal id.
    /// @param proposalId The proposal id
    /// @return proposal The proposal data
    function getProposal(
        uint64 proposalId
    ) external view returns (ProposalData memory proposal);

    /// @dev getProposals returns proposals with matching status.
    /// @param proposalStatus The proposal status to filter by
    /// @param voter The voter address to filter by, if any
    /// @param depositor The depositor address to filter by, if any
    /// @param pagination The pagination config
    /// @return proposals The proposals matching the filter criteria
    /// @return pageResponse The pagination information
    function getProposals(
        uint32 proposalStatus,
        address voter,
        address depositor,
        PageRequest calldata pagination
    )
        external
        view
        returns (
            ProposalData[] memory proposals,
            PageResponse memory pageResponse
        );

    /// @dev getParams returns the current governance parameters.
    /// @return params The governance parameters
    function getParams() external view returns (Params memory params);

    /// @dev getConstitution returns the current constitution.
    /// @return constitution The current constitution
    function getConstitution() external view returns (string memory constitution);
}
```

```json title="Governance ABI" lines expandable
{
  "_format": "hh-sol-artifact-1",
  "contractName": "IGov",
  "sourceName": "solidity/precompiles/gov/IGov.sol",
  "abi": [
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "proposer", "type": "address" }, { "indexed": true, "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "name": "CancelProposal", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "depositor", "type": "address" }, { "indexed": true, "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "indexed": false, "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "name": "Deposit", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "proposer", "type": "address" }, { "indexed": true, "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "name": "SubmitProposal", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "voter", "type": "address" }, { "indexed": true, "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "indexed": false, "internalType": "uint8", "name": "option", "type": "uint8" } ], "name": "Vote", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "voter", "type": "address" }, { "indexed": true, "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "indexed": false, "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" } ], "name": "VoteWeighted", "type": "event" },
    { "inputs": [ { "internalType": "address", "name": "proposer", "type": "address" }, { "internalType": "bytes", "name": "jsonProposal", "type": "bytes" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "deposit", "type": "tuple[]" } ], "name": "submitProposal", "outputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "name": "vote", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "name": "voteWeighted", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "proposer", "type": "address" }, { "internalType": "bytes", "name": "jsonProposal", "type": "bytes" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "deposit", "type": "tuple[]" } ], "name": "submitProposal", "outputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getVotes", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "voter", "type": "address" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "internalType": "struct WeightedVote[]", "name": "votes", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" } ], "name": "getDeposit", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "internalType": "struct DepositData", "name": "deposit", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getDeposits", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "internalType": "struct DepositData[]", "name": "deposits", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [], "name": "getConstitution", "outputs": [ { "internalType": "string", "name": "constitution", "type": "string" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint32", "name": "proposalStatus", "type": "uint32" }, { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getProposals", "outputs": [ { "components": [ { "internalType": "uint64", "name": "id", "type": "uint64" }, { "internalType": "string[]", "name": "messages", "type": "string[]" }, { "internalType": "uint32", "name": "status", "type": "uint32" }, { "components": [ { "internalType": "string", "name": "yes", "type": "string" }, { "internalType": "string", "name": "abstain", "type": "string" }, { "internalType": "string", "name": "no", "type": "string" }, { "internalType": "string", "name": "noWithVeto", "type": "string" } ], "internalType": "struct TallyResultData", "name": "finalTallyResult", "type": "tuple" }, { "internalType": "uint64", "name": "submitTime", "type": "uint64" }, { "internalType": "uint64", "name": "depositEndTime", "type": "uint64" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "totalDeposit", "type": "tuple[]" }, { "internalType": "uint64", "name": "votingStartTime", "type": "uint64" }, { "internalType": "uint64", "name": "votingEndTime", "type": "uint64" }, { "internalType": "string", "name": "metadata", "type": "string" }, { "internalType": "string", "name": "title", "type": "string" }, { "internalType": "string", "name": "summary", "type": "string" }, { "internalType": "address", "name": "proposer", "type": "address" } ], "internalType": "struct ProposalData[]", "name": "proposals", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "name": "getTallyResult", "outputs": [ { "components": [ { "internalType": "string", "name": "yes", "type": "string" }, { "internalType": "string", "name": "abstain", "type": "string" }, { "internalType": "string", "name": "no", "type": "string" }, { "internalType": "string", "name": "noWithVeto", "type": "string" } ], "internalType": "struct TallyResultData", "name": "tallyResult", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "voter", "type": "address" } ], "name": "getVote", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "voter", "type": "address" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "internalType": "struct WeightedVote", "name": "vote", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getVotes", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "voter", "type": "address" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "internalType": "struct WeightedVote[]", "name": "votes", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "proposer", "type": "address" }, { "internalType": "bytes", "name": "jsonProposal", "type": "bytes" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "deposit", "type": "tuple[]" } ], "name": "submitProposal", "outputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "name": "vote", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "enum VoteOption", "name": "option", "type": "uint8" }, { "internalType": "string", "name": "weight", "type": "string" } ], "internalType": "struct WeightedVoteOption[]", "name": "options", "type": "tuple[]" }, { "internalType": "string", "name": "metadata", "type": "string" } ], "name": "voteWeighted", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" } ], "name": "getDeposit", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "internalType": "struct DepositData", "name": "deposit", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getDeposits", "outputs": [ { "components": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "internalType": "struct DepositData[]", "name": "deposits", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [], "name": "getParams", "outputs": [ { "components": [ { "internalType": "int64", "name": "votingPeriod", "type": "int64" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "minDeposit", "type": "tuple[]" }, { "internalType": "int64", "name": "maxDepositPeriod", "type": "int64" }, { "internalType": "string", "name": "quorum", "type": "string" }, { "internalType": "string", "name": "threshold", "type": "string" }, { "internalType": "string", "name": "vetoThreshold", "type": "string" }, { "internalType": "string", "name": "minInitialDepositRatio", "type": "string" }, { "internalType": "string", "name": "proposalCancelRatio", "type": "string" }, { "internalType": "string", "name": "proposalCancelDest", "type": "string" }, { "internalType": "int64", "name": "expeditedVotingPeriod", "type": "int64" }, { "internalType": "string", "name": "expeditedThreshold", "type": "string" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "expeditedMinDeposit", "type": "tuple[]" }, { "internalType": "bool", "name": "burnVoteQuorum", "type": "bool" }, { "internalType": "bool", "name": "burnProposalDepositPrevote", "type": "bool" }, { "internalType": "bool", "name": "burnVoteVeto", "type": "bool" }, { "internalType": "string", "name": "minDepositRatio", "type": "string" } ], "internalType": "struct Params", "name": "params", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint64", "name": "proposalId", "type": "uint64" } ], "name": "getProposal", "outputs": [ { "components": [ { "internalType": "uint64", "name": "id", "type": "uint64" }, { "internalType": "string[]", "name": "messages", "type": "string[]" }, { "internalType": "uint32", "name": "status", "type": "uint32" }, { "components": [ { "internalType": "string", "name": "yes", "type": "string" }, { "internalType": "string", "name": "abstain", "type": "string" }, { "internalType": "string", "name": "no", "type": "string" }, { "internalType": "string", "name": "noWithVeto", "type": "string" } ], "internalType": "struct TallyResultData", "name": "finalTallyResult", "type": "tuple" }, { "internalType": "uint64", "name": "submitTime", "type": "uint64" }, { "internalType": "uint64", "name": "depositEndTime", "type": "uint64" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "totalDeposit", "type": "tuple[]" }, { "internalType": "uint64", "name": "votingStartTime", "type": "uint64" }, { "internalType": "uint64", "name": "votingEndTime", "type": "uint64" }, { "internalType": "string", "name": "metadata", "type": "string" }, { "internalType": "string", "name": "title", "type": "string" }, { "internalType": "string", "name": "summary", "type": "string" }, { "internalType": "address", "name": "proposer", "type": "address" } ], "internalType": "struct ProposalData", "name": "proposal", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "uint32", "name": "proposalStatus", "type": "uint32" }, { "internalType": "address", "name": "voter", "type": "address" }, { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getProposals", "outputs": [ { "components": [ { "internalType": "uint64", "name": "id", "type": "uint64" }, { "internalType": "string[]", "name": "messages", "type": "string[]" }, { "internalType": "uint32", "name": "status", "type": "uint32" }, { "components": [ { "internalType": "string", "name": "yes", "type": "string" }, { "internalType": "string", "name": "abstain", "type": "string" }, { "internalType": "string", "name": "no", "type": "string" }, { "internalType": "string", "name": "noWithVeto", "type": "string" } ], "internalType": "struct TallyResultData", "name": "finalTallyResult", "type": "tuple" }, { "internalType": "uint64", "name": "submitTime", "type": "uint64" }, { "internalType": "uint64", "name": "depositEndTime", "type": "uint64" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "totalDeposit", "type": "tuple[]" }, { "internalType": "uint64", "name": "votingStartTime", "type": "uint64" }, { "internalType": "uint64", "name": "votingEndTime", "type": "uint64" }, { "internalType": "string", "name": "metadata", "type": "string" }, { "internalType": "string", "name": "title", "type": "string" }, { "internalType": "string", "name": "summary", "type": "string" }, { "internalType": "address", "name": "proposer", "type": "address" } ], "internalType": "struct ProposalData[]", "name": "proposals", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" }
  ]
}