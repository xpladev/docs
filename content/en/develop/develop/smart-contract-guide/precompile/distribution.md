---
title: distribution
weight: 70
type: docs
---


## Overview

The Distribution precompile provides access to the Cosmos SDK `x/distribution` module, enabling smart contracts to manage staking rewards, interact with the community pool, and handle validator commission operations.

**Precompile Address**: `0x0000000000000000000000000000000000000801`

**Related Module**: [x/distribution](/develop/develop/core-modules/distribution/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on call complexity and chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Message Type Constants

The precompile defines the following constants for the various Cosmos SDK message types:

```solidity
// Transaction message type URLs
string constant MSG_CLAIM_REWARDS = "/cosmos.distribution.v1beta1.MsgClaimRewards";
string constant MSG_SET_WITHDRAW_ADDRESS = "/cosmos.distribution.v1beta1.MsgSetWithdrawAddress";
string constant MSG_WITHDRAW_DELEGATOR_REWARD = "/cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward";
```

## Transaction Methods

### claimRewards
Claims staking rewards from multiple validators.

<Info>
The `maxRetrieve` parameter limits the number of validators from which to claim rewards in a single transaction. This prevents excessive gas consumption when a delegator has rewards from many validators.
</Info>

### withdrawDelegatorRewards
Withdraws staking rewards from a single, specific validator.

### setWithdrawAddress
Sets or changes the withdrawal address for receiving staking rewards.

### withdrawValidatorCommission
Withdraws a validator's accumulated commission rewards.

### fundCommunityPool
Sends tokens directly to the community pool.

### depositValidatorRewardsPool
Deposits tokens into a specific validator's rewards pool.

## Query Methods

### delegationTotalRewards
Retrieves comprehensive reward information for all of a delegator's positions.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the relevant parts of the precompile
const precompileAbi = [
  "function delegationTotalRewards(address delegatorAddress) view returns (tuple(string validatorAddress, tuple(string denom, uint256 amount, uint8 precision)[] reward)[] rewards, tuple(string denom, uint256 amount, uint8 precision)[] total)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input: The address of the delegator to query
const delegatorAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder

async function getTotalRewards() {
  try {
    const result = await contract.delegationTotalRewards(delegatorAddress);
    console.log("Total Rewards:", JSON.stringify({
      rewards: result.rewards,
      total: result.total
    }, null, 2));
  } catch (error) {
    console.error("Error fetching total rewards:", error);
  }
}

getTotalRewards();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + padded delegator address
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x54be1a28000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### delegationRewards
Queries pending rewards for a specific delegation.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function delegationRewards(address delegatorAddress, string memory validatorAddress) view returns (tuple(string denom, uint256 amount, uint8 precision)[] rewards)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const delegatorAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getDelegationRewards() {
  try {
    const rewards = await contract.delegationRewards(delegatorAddress, validatorAddress);
    console.log("Delegation Rewards:", JSON.stringify(rewards, null, 2));
  } catch (error) {
    console.error("Error fetching delegation rewards:", error);
  }
}

getDelegationRewards();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder addresses with your actual data.
# Data is ABI-encoded: function selector + padded delegator address + encoded validator address string.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x9ad563b4000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba00000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### delegatorValidators
Retrieves a list of all validators from which a delegator has rewards.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function delegatorValidators(address delegatorAddress) view returns (string[] validators)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const delegatorAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder

async function getDelegatorValidators() {
  try {
    const validators = await contract.delegatorValidators(delegatorAddress);
    console.log("Validators:", validators);
  } catch (error) {
    console.error("Error fetching delegator validators:", error);
  }
}

getDelegatorValidators();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + padded delegator address.
curl -X POST --data '{                                    
    "jsonrpc": "2.0",    
    "method": "eth_call",           
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0xa66cb605000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0"        
        },        
        "latest"    
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### delegatorWithdrawAddress
Queries the address that can withdraw rewards for a given delegator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function delegatorWithdrawAddress(address delegatorAddress) view returns (string withdrawAddress)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const delegatorAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder

async function getWithdrawAddress() {
  try {
    const withdrawAddress = await contract.delegatorWithdrawAddress(delegatorAddress);
    console.log("Withdraw Address:", withdrawAddress);
  } catch (error) {
    console.error("Error fetching withdraw address:", error);
  }
}

getWithdrawAddress();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + padded delegator address.
curl -X POST --data '{    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x5431f450000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### communityPool
Queries the current balance of the community pool.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function communityPool() view returns (tuple(string denom, uint256 amount, uint8 precision)[] coins)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

async function getCommunityPoolBalance() {
  try {
    const balance = await contract.communityPool();
    console.log("Community Pool Balance:", JSON.stringify(balance, null, 2));
  } catch (error) {
    console.error("Error fetching community pool balance:", error);
  }
}

getCommunityPoolBalance();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> with your actual RPC endpoint.
# Data is ABI-encoded: just the function selector.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x14d140b0"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" <RPC_URL>
```


### validatorCommission
Queries the accumulated commission for a specific validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function validatorCommission(string memory validatorAddress) view returns (tuple(string denom, uint256 amount, uint8 precision)[] commission)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getValidatorCommission() {
  try {
    const commission = await contract.validatorCommission(validatorAddress);
    console.log("Validator Commission:", JSON.stringify(commission, null, 2));
  } catch (error) {
    console.error("Error fetching validator commission:", error);
  }
}

getValidatorCommission();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + encoded validator address string.
curl -X POST --data '{    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x3dd40f780000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],                                                              
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### validatorDistributionInfo
Queries a validator's commission and self-delegation rewards.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function validatorDistributionInfo(string memory validatorAddress) view returns (tuple(string operatorAddress, tuple(string denom, uint256 amount, uint8 precision)[] selfBondRewards, tuple(string denom, uint256 amount, uint8 precision)[] commission) distributionInfo)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getValidatorDistInfo() {
  try {
    const info = await contract.validatorDistributionInfo(validatorAddress);
    console.log("Validator Distribution Info:", JSON.stringify(info, null, 2));
  } catch (error) {
    console.error("Error fetching validator distribution info:", error);
  }
}

getValidatorDistInfo();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + encoded validator address string.
curl -X POST --data '{    "jsonrpc": "2.0",   
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x54212a890000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],                                                              
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### validatorOutstandingRewards
Queries the outstanding rewards of a validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function validatorOutstandingRewards(string memory validatorAddress) view returns (tuple(string denom, uint256 amount, uint8 precision)[] rewards)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("<RPC_URL>");
const precompileAddress = "0x0000000000000000000000000000000000000801";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const validatorAddress = "xplavaloper1nzng76rphusvwjws0qryfcpztj6lrvg5mlkh2c"; // Placeholder

async function getOutstandingRewards() {
  try {
    const rewards = await contract.validatorOutstandingRewards(validatorAddress);
    console.log("Validator Outstanding Rewards:", JSON.stringify(rewards, null, 2));
  } catch (error) {
    console.error("Error fetching outstanding rewards:", error);
  }
}

getOutstandingRewards();
```
```bash cURL expandable lines
# Note: Replace <RPC_URL> and the placeholder address with your actual data.
# Data is ABI-encoded: function selector + encoded validator address string.
curl -X POST --data '{    "jsonrpc": "2.0",     
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000801",
            "data": "0x85b2d2da0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003278706c6176616c6f706572316e7a6e673736727068757376776a7773307172796663707a746a366c727667356d6c6b6832630000000000000000000000000000"
        },
        "latest"
    ],                                                              
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


## Full Solidity Interface & ABI

```solidity title="Distribution Solidity Interface" lines expandable
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The DistributionI contract's address.
address constant DISTRIBUTION_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000801;

/// @dev The DistributionI contract's instance.
DistributionI constant DISTRIBUTION_CONTRACT = DistributionI(
    DISTRIBUTION_PRECOMPILE_ADDRESS
);

struct ValidatorSlashEvent {
    uint64 validatorPeriod;
    Dec fraction;
}

struct ValidatorDistributionInfo {
    string operatorAddress;
    DecCoin[] selfBondRewards;
    DecCoin[] commission;
}

struct DelegationDelegatorReward {
    string validatorAddress;
    DecCoin[] reward;
}

/// @author Evmos Team
/// @title Distribution Precompile Contract
/// @dev The interface through which solidity contracts will interact with Distribution
/// @custom:address 0x0000000000000000000000000000000000000801
interface DistributionI {
    event ClaimRewards(address indexed delegatorAddress, uint256 amount);
    event SetWithdrawerAddress(address indexed caller, string withdrawerAddress);
    event WithdrawDelegatorReward(address indexed delegatorAddress, address indexed validatorAddress, uint256 amount);
    event WithdrawValidatorCommission(string indexed validatorAddress, uint256 commission);
    event FundCommunityPool(address indexed depositor, string denom, uint256 amount);
    event DepositValidatorRewardsPool(address indexed depositor, address indexed validatorAddress, string denom, uint256 amount);

    function claimRewards(address delegatorAddress, uint32 maxRetrieve) external returns (bool success);
    function setWithdrawAddress(address delegatorAddress, string memory withdrawerAddress) external returns (bool success);
    function withdrawDelegatorRewards(address delegatorAddress, string memory validatorAddress) external returns (Coin[] calldata amount);
    function withdrawValidatorCommission(string memory validatorAddress) external returns (Coin[] calldata amount);
    function fundCommunityPool(address depositor, Coin[] memory amount) external returns (bool success);
    function depositValidatorRewardsPool(address depositor, string memory validatorAddress, Coin[] memory amount) external returns (bool success);

    function validatorDistributionInfo(string memory validatorAddress) external view returns (ValidatorDistributionInfo calldata distributionInfo);
    function validatorOutstandingRewards(string memory validatorAddress) external view returns (DecCoin[] calldata rewards);
    function validatorCommission(string memory validatorAddress) external view returns (DecCoin[] calldata commission);
    function validatorSlashes(string memory validatorAddress, uint64 startingHeight, uint64 endingHeight, PageRequest calldata pageRequest) external view returns (ValidatorSlashEvent[] calldata slashes, PageResponse calldata pageResponse);
    function delegationRewards(address delegatorAddress, string memory validatorAddress) external view returns (DecCoin[] calldata rewards);
    function delegationTotalRewards(address delegatorAddress) external view returns (DelegationDelegatorReward[] calldata rewards, DecCoin[] calldata total);
    function delegatorValidators(address delegatorAddress) external view returns (string[] calldata validators);
    function delegatorWithdrawAddress(address delegatorAddress) external view returns (string memory withdrawAddress);
    function communityPool() external view returns (DecCoin[] calldata coins);
}
```

```json title="Distribution ABI" lines expandable
{
  "_format": "hh-sol-artifact-1",
  "contractName": "DistributionI",
  "sourceName": "solidity/precompiles/distribution/DistributionI.sol",
  "abi": [
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "ClaimRewards", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "depositor", "type": "address" }, { "indexed": true, "internalType": "address", "name": "validatorAddress", "type": "address" }, { "indexed": false, "internalType": "string", "name": "denom", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "DepositValidatorRewardsPool", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "depositor", "type": "address" }, { "indexed": false, "internalType": "string", "name": "denom", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "FundCommunityPool", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "caller", "type": "address" }, { "indexed": false, "internalType": "string", "name": "withdrawerAddress", "type": "string" } ], "name": "SetWithdrawerAddress", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "indexed": true, "internalType": "address", "name": "validatorAddress", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "amount", "type": "uint256" } ], "name": "WithdrawDelegatorReward", "type": "event" },
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "string", "name": "validatorAddress", "type": "string" }, { "indexed": false, "internalType": "uint256", "name": "commission", "type": "uint256" } ], "name": "WithdrawValidatorCommission", "type": "event" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "uint32", "name": "maxRetrieve", "type": "uint32" } ], "name": "claimRewards", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [], "name": "communityPool", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "coins", "type": "tuple[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "delegationRewards", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "rewards", "type": "tuple[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" } ], "name": "delegationTotalRewards", "outputs": [ { "components": [ { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "reward", "type": "tuple[]" } ], "internalType": "struct DelegationDelegatorReward[]", "name": "rewards", "type": "tuple[]" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "total", "type": "tuple[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" } ], "name": "delegatorValidators", "outputs": [ { "internalType": "string[]", "name": "validators", "type": "string[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" } ], "name": "delegatorWithdrawAddress", "outputs": [ { "internalType": "string", "name": "withdrawAddress", "type": "string" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "depositor", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "name": "depositValidatorRewardsPool", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "depositor", "type": "address" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "name": "fundCommunityPool", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "withdrawerAddress", "type": "string" } ], "name": "setWithdrawAddress", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "validatorCommission", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "commission", "type": "tuple[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "validatorDistributionInfo", "outputs": [ { "components": [ { "internalType": "string", "name": "operatorAddress", "type": "string" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "selfBondRewards", "type": "tuple[]" }, { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "commission", "type": "tuple[]" } ], "internalType": "struct ValidatorDistributionInfo", "name": "distributionInfo", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "validatorOutstandingRewards", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct DecCoin[]", "name": "rewards", "type": "tuple[]" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" }, { "internalType": "uint64", "name": "startingHeight", "type": "uint64" }, { "internalType": "uint64", "name": "endingHeight", "type": "uint64" }, { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pageRequest", "type": "tuple" } ], "name": "validatorSlashes", "outputs": [ { "components": [ { "internalType": "uint64", "name": "validatorPeriod", "type": "uint64" }, { "components": [ { "internalType": "uint256", "name": "amount", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct Dec", "name": "fraction", "type": "tuple" } ], "internalType": "struct ValidatorSlashEvent[]", "name": "slashes", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "delegatorAddress", "type": "address" }, { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "withdrawDelegatorRewards", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "stateMutability": "nonpayable", "type": "function" },
    { "inputs": [ { "internalType": "string", "name": "validatorAddress", "type": "string" } ], "name": "withdrawValidatorCommission", "outputs": [ { "components": [ { "internalType": "string", "name": "denom", "type": "string" }, { "internalType": "uint256", "name": "amount", "type": "uint256" } ], "internalType": "struct Coin[]", "name": "amount", "type": "tuple[]" } ], "stateMutability": "nonpayable", "type": "function" }
  ]
}
```