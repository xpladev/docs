---
title: staking
weight: 60
type: docs
---

## Overview

The Staking precompile provides a comprehensive interface to the Cosmos SDK's `x/staking` module, enabling smart contracts to engage in staking activities. This includes creating and managing validators, delegating and undelegating tokens, and querying a wide range of staking-related information, such as validator details, delegation records, and staking pool status.

#### Precompile Address: `0x0000000000000000000000000000000000000800`
#### Related Module: [x/staking](/develop/develop/core-modules/staking/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on the complexity of the staking operation and chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Transaction Methods

### `createValidator`
Creates a new validator.

### `editValidator`
Edits an existing validator's parameters.

### `delegate`
Delegates tokens to a validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the delegate function
const precompileAbi = [
  "function delegate(address delegatorAddress, string memory validatorAddress, uint256 amount) payable"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000800";
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

// Delegation parameters
const delegatorAddress = await signer.getAddress();
const validatorAddress = "xplavaloper1..."; // Validator operator address
const amount = ethers.parseEther("10.0"); // Amount to delegate in native token

async function delegateTokens() {
  try {
    const tx = await contract.delegate(
      delegatorAddress,
      validatorAddress,
      amount,
      { value: amount } // Must send the amount being delegated
    );

    console.log("Delegation transaction:", tx.hash);
    await tx.wait();
    console.log("Delegation confirmed");
  } catch (error) {
    console.error("Error delegating tokens:", error);
  }
}

// delegateTokens();
```


### `undelegate`
Undelegates tokens from a validator.

### `redelegate`
Redelegates tokens from one validator to another.

### `cancelUnbondingDelegation`
Cancels an unbonding delegation.

## Query Methods

### `validator`
Queries information about a specific validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function validator(address validatorAddress) view returns (tuple(string operatorAddress, string consensusPubkey, bool jailed, uint32 status, uint256 tokens, uint256 delegatorShares, string description, int64 unbondingHeight, uint256 unbondingTime, uint256 commission,  uint256 minSelfDelegation) validator)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x98a68f6861bF20C749D0780644e0225cb5f1b114";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const validatorAddress = "0x000"; // Placeholder

async function getValidator() {
  try {
    const validator = await contract.validator(validatorAddress);
    console.log("Validator Info:", validator);
  } catch (error) {
    console.error("Error fetching validator:", error);
  }
}

getValidator();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev and the placeholder validator address with your actual data.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000800",
            "data": "0x223b3b7a00000000000000000000000098a68f6861bf20c749d0780644e0225cb5f1b114"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `validators`
Queries validators with optional status filtering and pagination.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function validators(string memory status, tuple(bytes key, uint64 offset, uint64 limit, bool countTotal, bool reverse) pageRequest) view returns (tuple(string operatorAddress, string consensusPubkey, bool jailed, uint32 status, uint256 tokens, uint256 delegatorShares, string description, int64 unbondingHeight, uint256 unbondingTime, uint256 commission,  uint256 minSelfDelegation)[] validators, tuple(bytes nextKey, uint64 total) pageResponse)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000800";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const status = "BOND_STATUS_BONDED";
const pagination = {
  key: "0x",
  offset: 0,
  limit: 10,
  countTotal: true,
  reverse: false,
};

async function getValidators() {
  try {
    const result = await contract.validators(status, pagination);
    console.log("Validators:", result.validators);
    console.log("Pagination Response:", result.pageResponse);
  } catch (error) {
    console.error("Error fetching validators:", error);
  }
}

getValidators();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual RPC endpoint.
# This example queries for the first 10 bonded validators.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000800",
            "data": "0x186b2167000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000000000000000000000000000000000012424f4e445f5354415455535f424f4e444544000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `delegation`
Queries the delegation amount between a delegator and a validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function delegation(address delegatorAddress, string memory validatorAddress) view returns (uint256)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000800";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const delegatorAddress = "0x0200741eCaA5Dd338667367CF6b19c1E065174e0"; // Placeholder
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getDelegation() {
  try {
    const delegationAmount = await contract.delegation(delegatorAddress, validatorAddress);
    console.log("Delegation Amount:", delegationAmount.toString());
  } catch (error) {
    console.error("Error fetching delegation:", error);
  }
}

getDelegation();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev and the placeholder addresses with your actual data.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000800",
            "data": "0x241774e60000000000000000000000000200741ecaa5dd338667367cf6b19c1e065174e00000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `unbondingDelegation`
Queries unbonding delegation information.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function unbondingDelegation(address delegatorAddress, string memory validatorAddress) view returns (tuple(address delegatorAddress, string validatorAddress, tuple(uint256 creationHeight, uint256 completionTime, string initialBalance, string balance)[] entries) unbondingDelegation)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000800";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const delegatorAddress = "0x0200741eCaA5Dd338667367CF6b19c1E065174e0"; // Placeholder
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getUnbondingDelegation() {
  try {
    const unbonding = await contract.unbondingDelegation(delegatorAddress, validatorAddress);
    console.log("Unbonding Delegation:", JSON.stringify(unbonding, null, 2));
  } catch (error) {
    console.error("Error fetching unbonding delegation:", error);
  }
}

getUnbondingDelegation();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev and the placeholder addresses with your actual data.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000800",
            "data": "0xa03ffee10000000000000000000000000200741ecaa5dd338667367cf6b19c1e065174e00000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `redelegation`
Queries a specific redelegation.

### `redelegations`
Queries redelegations with optional filters.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function redelegations(address delegatorAddress, string memory srcValidatorAddress, string memory dstValidatorAddress, tuple(bytes key, uint64 offset, uint64 limit, bool countTotal, bool reverse) pageRequest) view returns (tuple(tuple(address delegatorAddress, string validatorSrcAddress, string validatorDstAddress, tuple(uint256 creationHeight, uint256 completionTime, string initialBalance, string sharesDst)[] entries) redelegation, tuple(tuple(uint256 creationHeight, uint256 completionTime, string initialBalance, string sharesDst) redelegationEntry, string balance)[] entries)[] redelegations, tuple(bytes nextKey, uint64 total) pageResponse)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000800";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const delegatorAddress = "0x0200741eCaA5Dd338667367CF6b19c1E065174e0"; // Placeholder
const srcValidatorAddress = ""; // Empty string for all
const dstValidatorAddress = ""; // Empty string for all
const pagination = {
  key: "0x",
  offset: 0,
  limit: 10,
  countTotal: true,
  reverse: false,
};

async function getRedelegations() {
  try {
    const result = await contract.redelegations(delegatorAddress, srcValidatorAddress, dstValidatorAddress, pagination);
    console.log("Redelegations:", JSON.stringify(result.redelegations, null, 2));
    console.log("Pagination Response:", result.pageResponse);
  } catch (error) {
    console.error("Error fetching redelegations:", error);
  }
}

getRedelegations();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev and addresses with your actual data.
# This example queries for the first 10 redelegations.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000800",
            "data": "0x10a2851c0000000000000000000000000200741ecaa5dd338667367cf6b19c1e065174e0000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


**Note**: The `pool()` and `params()` functions are not currently available in this staking precompile implementation. The available query methods are limited to delegation-specific functions shown above.

## Full Solidity Interface & ABI

```solidity title="Staking Solidity Interface" lines expandable
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The StakingI contract's address.
address constant STAKING_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000800;

/// @dev The StakingI contract's instance.
StakingI constant STAKING_CONTRACT = StakingI(STAKING_PRECOMPILE_ADDRESS);

// BondStatus is the status of a validator.
enum BondStatus {
    Unspecified,
    Unbonded,
    Unbonding,
    Bonded
}

// Description contains a validator's description.
struct Description {
    string moniker;
    string identity;
    string website;
    string securityContact;
    string details;
}

// CommissionRates defines the initial commission rates to be used for a validator
struct CommissionRates {
    string rate;
    string maxRate;
    string maxChangeRate;
}

// Commission defines a commission parameters for a given validator.
struct Commission {
    CommissionRates commissionRates;
    uint256 updateTime;
}

// Validator defines a validator, an account that can participate in consensus.
struct Validator {
    string operatorAddress;
    string consensusPubkey;
    bool jailed;
    uint32 status;
    uint256 tokens;
    string delegatorShares;
    Description description;
    int64 unbondingHeight;
    uint256 unbondingTime;
    Commission commission;
    uint256 minSelfDelegation;
}

// Delegation represents the bond with tokens held by an account. It is
// owned by one delegator, and is associated with the voting power of one
// validator.
struct Delegation {
    address delegatorAddress;
    string validatorAddress;
    string shares;
}

// UnbondingDelegation stores all of a single delegator's unbonding bonds
// for a single validator in an array.
struct UnbondingDelegation {
    address delegatorAddress;
    string validatorAddress;
    UnbondingDelegationEntry[] entries;
}

// UnbondingDelegationEntry defines an unbonding object with relevant metadata.
struct UnbondingDelegationEntry {
    uint256 creationHeight;
    uint256 completionTime;
    string initialBalance;
    string balance;
}

// RedelegationEntry defines a redelegation object with relevant metadata.
struct RedelegationEntry {
    uint256 creationHeight;
    uint256 completionTime;
    string initialBalance;
    string sharesDst;
}

// Redelegation contains the list of a particular delegator's redelegating bonds
// from a particular source validator to a particular destination validator.
struct Redelegation {
    address delegatorAddress;
    string validatorSrcAddress;
    string validatorDstAddress;
    RedelegationEntry[] entries;
}

// DelegationResponse is equivalent to Delegation except that it contains a
// balance in addition to shares which is more suitable for client responses.
struct DelegationResponse {
    Delegation delegation;
    Coin balance;
}

// RedelegationEntryResponse is equivalent to a RedelegationEntry except that it
// contains a balance in addition to shares which is more suitable for client
// responses.
struct RedelegationEntryResponse {
    RedelegationEntry redelegationEntry;
    string balance;
}

// RedelegationResponse is equivalent to a Redelegation except that its entries
// contain a balance in addition to shares which is more suitable for client
// responses.
struct RedelegationResponse {
    Redelegation redelegation;
    RedelegationEntryResponse[] entries;
}

// Pool is used for tracking bonded and not-bonded token supply of the bond denomination.
struct Pool {
    string notBondedTokens;
    string bondedTokens;
}

// StakingParams defines the parameters for the staking module.
struct Params {
    uint256 unbondingTime;
    uint256 maxValidators;
    uint256 maxEntries;
    uint256 historicalEntries;
    string bondDenom;
    string minCommissionRate;
}

/// @author The Evmos Core Team
/// @title Staking Precompile Contract
/// @dev The interface through which solidity contracts will interact with Staking
/// @custom:address 0x0000000000000000000000000000000000000800
interface StakingI {
    event CreateValidator(string indexed validatorAddress, uint256 value);
    event EditValidator(string indexed validatorAddress);
    event Delegate(address indexed delegatorAddress, string indexed validatorAddress, uint256 amount);
    event Unbond(address indexed delegatorAddress, string indexed validatorAddress, uint256 amount, uint256 completionTime);
    event Redelegate(address indexed delegatorAddress, address indexed validatorSrcAddress, address indexed validatorDstAddress, uint256 amount, uint256 completionTime);
    event CancelUnbondingDelegation(address indexed delegatorAddress, address indexed validatorAddress, uint256 amount, uint256 creationHeight);

    // Transactions
    function createValidator(
        Description calldata description,
        CommissionRates calldata commission,
        uint256 minSelfDelegation,
        string calldata validatorAddress,
        string calldata pubkey,
        address value
    ) external payable returns (bool);

    function editValidator(
        Description calldata description,
        string calldata validatorAddress,
        uint256 commissionRate,
        uint256 minSelfDelegation
    ) external returns (bool);

    function delegate(
        address delegatorAddress,
        string calldata validatorAddress,
        uint256 amount
    ) external payable returns (bool);

    function undelegate(
        address delegatorAddress,
        string calldata validatorAddress,
        uint256 amount
    ) external returns (bool);

    function redelegate(
        address delegatorAddress,
        string calldata validatorSrcAddress,
        string calldata validatorDstAddress,
        uint256 amount
    ) external returns (bool);

    function cancelUnbondingDelegation(
        address delegatorAddress,
        string calldata validatorAddress,
        uint256 amount,
        uint256 creationHeight
    ) external returns (bool);

    // Queries
    function validator(
        string calldata validatorAddress
    ) external view returns (Validator memory);

    function validators(
        string calldata status,
        PageRequest calldata pageRequest
    ) external view returns (Validator[] memory, PageResponse memory);

    function delegation(
        address delegatorAddress,
        string calldata validatorAddress
    ) external view returns (uint256);

    function unbondingDelegation(
        address delegatorAddress,
        string calldata validatorAddress
    ) external view returns (UnbondingDelegation memory);

    function redelegation(
        address delegatorAddress,
        string calldata srcValidatorAddress,
        string calldata dstValidatorAddress
    ) external view returns (Redelegation memory);

    function redelegations(
        address delegatorAddress,
        string calldata srcValidatorAddress,
        string calldata dstValidatorAddress,
        PageRequest calldata pageRequest
    ) external view returns (RedelegationResponse[] memory, PageResponse memory);
}
```

```json title="Staking ABI" lines expandable
{
  "_format": "hh-sol-artifact-1",
  "contractName": "StakingI",
  "sourceName": "solidity/precompiles/staking/StakingI.sol",
  "abi": [
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": true, "internalType": "address", "name": "validatorAddress", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "creationHeight", "type": "uint256" } ], "name": "CancelUnbondingDelegation", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "string", "name": "validatorAddress", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "value", "type": "uint256" } ], "name": "CreateValidator", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": true, "internalType": "string", "name": "validatorAddress", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "Delegate", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "EditValidator", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": true, "internalType": "address", "name": "validatorSrcAddress", "type": "address" }, { "indexed": true, "internalType": "address", "name": "validatorDstAddress", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "completionTime", "type": "uint256" } ], "name": "Redelegate", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": true, "internalType": "string", "name": "validatorAddress", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" }, { "indexed": false, "internalType": "uint256", "name": "completionTime", "type": "uint256" } ], "name": "Unbond", "type": "event" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint256", "name": "creationHeight", "type": "uint256" } ], "name": "cancelUnbondingDelegation", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "components": [ { "internalType": "string", "name": "moniker", "type": "string" }, { "internalType": "string", "name": "identity", "type": "string" }, { "internalType": "string", "name": "website", "type": "string" }, { "internalType": "string", "name": "securityContact", "type": "string" }, { "internalType": "string", "name": "details", "type": "string" } ], "internalType": "struct Description", "name": "description", "type": "tuple" }, { "components": [ { "internalType": "string", "name": "rate", "type": "string" }, { "internalType": "string", "name": "maxRate", "type": "string" }, { "internalType": "string", "name": "maxChangeRate", "type": "string" } ], "internalType": "struct CommissionRates", "name": "commission", "type": "tuple" }, { "internalType": "uint256", "name": "minSelfDelegation", "type": "uint256" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "internalType": "string", "name": "pubkey", "type": "string" }, { "internalType": "address", "name": "value", "type": "address" } ], "name": "createValidator", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "payable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "delegate", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "payable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "delegation", "outputs": [ { "internalType": "uint256", "name": "", "type": "uint256" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "components": [ { "internalType": "string", "name": "moniker", "type": "string" }, { "internalType": "string", "name": "identity", "type": "string" }, { "internalType": "string", "name": "website", "type": "string" }, { "internalType": "string", "name": "securityContact", "type": "string" }, { "internalType": "string", "name": "details", "type": "string" } ], "internalType": "struct Description", "name": "description", "type": "tuple" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "internalType": "uint256", "name": "commissionRate", "type": "uint256" }, { "internalType": "uint256", "name": "minSelfDelegation", "type": "uint256" } ], "name": "editValidator", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorSrcAddress", "type": "string" }, { "internalType": "string", "name": "validatorDstAddress", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "redelegate", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "srcValidatorAddress", "type": "string" }, { "internalType": "string", "name": "dstValidatorAddress", "type": "string" } ], "name": "redelegation", "outputs": [ { "components": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorSrcAddress", "type": "string" }, { "internalType": "string", "name": "validatorDstAddress", "type": "string" }, { "components": [ { "internalType": "uint256", "name": "creationHeight", "type": "uint256" }, { "internalType": "uint256", "name": "completionTime", "type": "uint256" }, { "internalType": "string", "name": "initialBalance", "type": "string" }, { "internalType": "string", "name": "sharesDst", "type": "string" } ], "internalType": "struct RedelegationEntry[]", "name": "entries", "type": "tuple[]" } ], "internalType": "struct Redelegation", "name": "", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "srcValidatorAddress", "type": "string" }, { "internalType": "string", "name": "dstValidatorAddress", "type": "string" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pageRequest", "type": "tuple" } ], "name": "redelegations", "outputs": [ { "components": [ { "components": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorSrcAddress", "type": "string" }, { "internalType": "string", "name": "validatorDstAddress", "type": "string" }, { "components": [ { "internalType": "uint256", "name": "creationHeight", "type": "uint256" }, { "internalType": "uint256", "name": "completionTime", "type": "uint256" }, { "internalType": "string", "name": "initialBalance", "type": "string" }, { "internalType": "string", "name": "sharesDst", "type": "string" } ], "internalType": "struct RedelegationEntry[]", "name": "entries", "type": "tuple[]" } ], "internalType": "struct Redelegation", "name": "redelegation", "type": "tuple" }, { "components": [ { "components": [ { "internalType": "uint256", "name": "creationHeight", "type": "uint256" }, { "internalType": "uint256", "name": "completionTime", "type": "uint256" }, { "internalType": "string", "name": "initialBalance", "type": "string" }, { "internalType": "string", "name": "sharesDst", "type": "string" } ], "internalType": "struct RedelegationEntry", "name": "redelegationEntry", "type": "tuple" }, { "internalType": "string", "name": "balance", "type": "string" } ], "internalType": "struct RedelegationEntryResponse[]", "name": "entries", "type": "tuple[]" } ], "internalType": "struct RedelegationResponse[]", "name": "", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "string", "name": "validatorAddress", "type": "string" } ], "name": "unbondingDelegation", "outputs": [ { "components": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "components": [ { "internalType": "uint256", "name": "creationHeight", "type": "uint256" }, { "internalType": "uint256", "name": "completionTime", "type": "uint256" }, { "internalType": "string", "name": "initialBalance", "type": "string" }, { "internalType": "string", "name": "balance", "type": "string" } ], "internalType": "struct UnbondingDelegationEntry[]", "name": "entries", "type": "tuple[]" } ], "internalType": "struct UnbondingDelegation", "name": "", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "string", "name": "validatorAddress", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "undelegate", "outputs": [ { "internalType": "bool", "name": "", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "validator", "outputs": [ { "components": [ { "internalType": "string", "name": "operatorAddress", "type": "string" }, { "internalType": "string", "name": "consensusPubkey", "type": "string" }, { "internalType": "bool", "name": "jailed", "type": "bool" }, { "internalType": "uint32", "name": "status", "type": "uint32" }, { "internalType": "uint256", "name": "tokens", "type": "uint256" }, { "internalType": "string", "name": "delegatorShares", "type": "string" }, { "components": [ { "internalType": "string", "name": "moniker", "type": "string" }, { "internalType": "string", "name": "identity", "type": "string" }, { "internalType": "string", "name": "website", "type": "string" }, { "internalType": "string", "name": "securityContact", "type": "string" }, { "internalType": "string", "name": "details", "type": "string" } ], "internalType": "struct Description", "name": "description", "type": "tuple" }, { "internalType": "int64", "name": "unbondingHeight", "type": "int64" }, { "internalType": "uint256", "name": "unbondingTime", "type": "uint256" }, { "components": [ { "components": [ { "internalType": "string", "name": "rate", "type": "string" }, { "internalType": "string", "name": "maxRate", "type": "string" }, { "internalType": "string", "name": "maxChangeRate", "type": "string" } ], "internalType": "struct CommissionRates", "name": "commissionRates", "type": "tuple" }, { "internalType": "uint256", "name": "updateTime", "type": "uint256" } ], "internalType": "struct Commission", "name": "commission", "type": "tuple" }, { "internalType": "uint256", "name": "minSelfDelegation", "type": "uint256" } ], "internalType": "struct Validator", "name": "", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "status", "type": "string" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pageRequest", "type": "tuple" } ], "name": "validators", "outputs": [ { "components": [ { "internalType": "string", "name": "operatorAddress", "type": "string" }, { "internalType": "string", "name": "consensusPubkey", "type": "string" }, { "internalType": "bool", "name": "jailed", "type": "bool" }, { "internalType": "uint32", "name": "status", "type": "uint32" }, { "internalType": "uint256", "name": "tokens", "type": "uint256" }, { "internalType": "string", "name": "delegatorShares", "type": "string" }, { "components": [ { "internalType": "string", "name": "moniker", "type": "string" }, { "internalType": "string", "name": "identity", "type": "string" }, { "internalType": "string", "name": "website", "type": "string" }, { "internalType": "string", "name": "securityContact", "type": "string" }, { "internalType": "string", "name": "details", "type": "string" } ], "internalType": "struct Description", "name": "description", "type": "tuple" }, { "internalType": "int64", "name": "unbondingHeight", "type": "int64" }, { "internalType": "uint256", "name": "unbondingTime", "type": "uint256" }, { "components": [ { "components": [ { "internalType": "string", "name": "rate", "type": "string" }, { "internalType": "string", "name": "maxRate", "type": "string" }, { "internalType": "string", "name": "maxChangeRate", "type": "string" } ], "internalType": "struct CommissionRates", "name": "commissionRates", "type": "tuple" }, { "internalType": "uint256", "name": "updateTime", "type": "uint256" } ], "internalType": "struct Commission", "name": "commission", "type": "tuple" }, { "internalType": "uint256", "name": "minSelfDelegation", "type": "uint256" } ], "internalType": "struct Validator[]", "name": "", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "", "type": "tuple" } ], "stateMutability": "view", "type": "function" }
  ]
}
```