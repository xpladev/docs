---
title: slashing
weight: 90
type: docs
---

## Overview

The Slashing precompile provides an interface to the Cosmos SDK's `x/slashing` module, which is responsible for penalizing validators for misbehavior, such as downtime or double-signing. This precompile allows smart contracts to unjail validators and query slashing-related information, including validator signing info and module parameters.

#### Precompile Address : `0x0000000000000000000000000000000000000806`
#### Related Module : [x/slashing](/develop/develop/core-modules/slashing/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Transaction Methods

### `unjail`
Allows a validator to unjail themselves after being jailed for downtime.

## Query Methods

### `getSigningInfo`
Returns the signing information for a specific validator.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function getSigningInfo(address consAddress) view returns (tuple(address validatorAddress, int64 startHeight, int64 indexOffset, int64 jailedUntil, bool tombstoned, int64 missedBlocksCounter) signingInfo)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000806";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input: The consensus address of the validator
const consAddress = "0x0BA98508F32B74CB8580AFB0C05499A2D9672F8A"; // Placeholder

async function getSigningInfo() {
  try {
    const signingInfo = await contract.getSigningInfo(consAddress);
    console.log("Signing Info:", signingInfo);
  } catch (error) {
    console.error("Error fetching signing info:", error);
  }
}

getSigningInfo();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev and the placeholder consensus address with your actual data.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000806",
            "data": "0x69e1f9df0000000000000000000000000ba98508f32b74cb8580afb0c05499a2d9672f8a"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `getSigningInfos`
Returns the signing information for all validators, with pagination support.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function getSigningInfos(tuple(bytes key, uint64 offset, uint64 limit, bool countTotal, bool reverse) pagination) view returns (tuple(address validatorAddress, int64 startHeight, int64 indexOffset, int64 jailedUntil, bool tombstoned, int64 missedBlocksCounter)[] signingInfos, tuple(bytes nextKey, uint64 total) pageResponse)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000806";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input for pagination
const pagination = {
  key: "0x",
  offset: 0,
  limit: 10,
  countTotal: true,
  reverse: false,
};

async function getSigningInfos() {
  try {
    const result = await contract.getSigningInfos(pagination);
    console.log("Signing Infos:", result.signingInfos);
    console.log("Pagination Response:", result.pageResponse);
  } catch (error) {
    console.error("Error fetching signing infos:", error);
  }
}

getSigningInfos();
```
```bash cURL expandable lines
# This example queries for the first 10 validators' signing info.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000806",
            "data": "0xc6919f55000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


### `getParams`
Returns the current parameters for the slashing module.


```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// ABI definition for the function
const precompileAbi = [
  "function getParams() view returns (tuple(int64 signedBlocksWindow, tuple(uint256 value, uint8 precision) minSignedPerWindow, int64 downtimeJailDuration, tuple(uint256 value, uint8 precision) slashFractionDoubleSign, tuple(uint256 value, uint8 precision) slashFractionDowntime) params)"
];

// Provider and contract setup
const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x0000000000000000000000000000000000000806";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

async function getParams() {
  try {
    const params = await contract.getParams();
    console.log("Slashing Parameters:", JSON.stringify(params, null, 2));
  } catch (error) {
    console.error("Error fetching slashing parameters:", error);
  }
}

getParams();
```

**Note**: The slashing `getParams()` function may return empty data if slashing parameters are not configured on your network. The cURL method will still demonstrate the correct function call format.

```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual RPC endpoint.
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000806",
            "data": "0x5e615a6b"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```


## Full Solidity Interface & ABI

```solidity title="Slashing Solidity Interface" lines expandable
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The ISlashing contract's address.
address constant SLASHING_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000806;

/// @dev The ISlashing contract's instance.
ISlashing constant SLASHING_CONTRACT = ISlashing(SLASHING_PRECOMPILE_ADDRESS);

/// @dev SigningInfo defines a validator's signing info for monitoring their
/// liveness activity.
struct SigningInfo {
    /// @dev Address of the validator
    address validatorAddress;
    /// @dev Height at which validator was first a candidate OR was unjailed
    int64 startHeight;
    /// @dev Index offset into signed block bit array
    int64 indexOffset;
    /// @dev Timestamp until which validator is jailed due to liveness downtime
    int64 jailedUntil;
    /// @dev Whether or not a validator has been tombstoned (killed out of validator set)
    bool tombstoned;
    /// @dev Missed blocks counter (to avoid scanning the array every time)
    int64 missedBlocksCounter;
}

/// @dev Params defines the parameters for the slashing module.
struct Params {
    /// @dev SignedBlocksWindow defines how many blocks the validator should have signed
    int64 signedBlocksWindow;
    /// @dev MinSignedPerWindow defines the minimum blocks signed per window to avoid slashing
    Dec minSignedPerWindow;
    /// @dev DowntimeJailDuration defines how long the validator will be jailed for downtime
    int64 downtimeJailDuration;
    /// @dev SlashFractionDoubleSign defines the percentage of slash for double sign
    Dec slashFractionDoubleSign;
    /// @dev SlashFractionDowntime defines the percentage of slash for downtime
    Dec slashFractionDowntime;
}

/// @author Evmos Team
/// @title Slashing Precompiled Contract
/// @dev The interface through which solidity contracts will interact with slashing.
/// We follow this same interface including four-byte function selectors, in the precompile that
/// wraps the pallet.
/// @custom:address 0x0000000000000000000000000000000000000806
interface ISlashing {
    /// @dev Emitted when a validator is unjailed
    /// @param validator The address of the validator
    event ValidatorUnjailed(address indexed validator);

    /// @dev GetSigningInfo returns the signing info for a specific validator.
    /// @param consAddress The validator consensus address
    /// @return signingInfo The validator signing info
    function getSigningInfo(
        address consAddress
    ) external view returns (SigningInfo memory signingInfo);

    /// @dev GetSigningInfos returns the signing info for all validators.
    /// @param pagination Pagination configuration for the query
    /// @return signingInfos The list of validator signing info
    /// @return pageResponse Pagination information for the response
    function getSigningInfos(
        PageRequest calldata pagination
    ) external view returns (SigningInfo[] memory signingInfos, PageResponse memory pageResponse);

    /// @dev Unjail allows validators to unjail themselves after being jailed for downtime
    /// @param validatorAddress The validator operator address to unjail
    /// @return success true if the unjail operation was successful
    function unjail(address validatorAddress) external returns (bool success);

    /// @dev GetParams returns the slashing module parameters
    /// @return params The slashing module parameters
    function getParams() external view returns (Params memory params);
}
```

```json title="Slashing ABI" lines expandable
{
  "_format": "hh-sol-artifact-1",
  "contractName": "ISlashing",
  "sourceName": "solidity/precompiles/slashing/ISlashing.sol",
  "abi": [
    { "anonymous": false, "inputs": [ { "indexed": true, "internalType": "address", "name": "validator", "type": "address" } ], "name": "ValidatorUnjailed", "type": "event" },
    { "inputs": [], "name": "getParams", "outputs": [ { "components": [ { "internalType": "int64", "name": "signedBlocksWindow", "type": "int64" }, { "components": [ { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct Dec", "name": "minSignedPerWindow", "type": "tuple" }, { "internalType": "int64", "name": "downtimeJailDuration", "type": "int64" }, { "components": [ { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct Dec", "name": "slashFractionDoubleSign", "type": "tuple" }, { "components": [ { "internalType": "uint256", "name": "value", "type": "uint256" }, { "internalType": "uint8", "name": "precision", "type": "uint8" } ], "internalType": "struct Dec", "name": "slashFractionDowntime", "type": "tuple" } ], "internalType": "struct Params", "name": "params", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "consAddress", "type": "address" } ], "name": "getSigningInfo", "outputs": [ { "components": [ { "internalType": "address", "name": "validatorAddress", "type": "address" }, { "internalType": "int64", "name": "startHeight", "type": "int64" }, { "internalType": "int64", "name": "indexOffset", "type": "int64" }, { "internalType": "int64", "name": "jailedUntil", "type": "int64" }, { "internalType": "bool", "name": "tombstoned", "type": "bool" }, { "internalType": "int64", "name": "missedBlocksCounter", "type": "int64" } ], "internalType": "struct SigningInfo", "name": "signingInfo", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "components": [ { "internalType": "bytes", "name": "key", "type": "bytes" }, { "internalType": "uint64", "name": "offset", "type": "uint64" }, { "internalType": "uint64", "name": "limit", "type": "uint64" }, { "internalType": "bool", "name": "countTotal", "type": "bool" }, { "internalType": "bool", "name": "reverse", "type": "bool" } ], "internalType": "struct PageRequest", "name": "pagination", "type": "tuple" } ], "name": "getSigningInfos", "outputs": [ { "components": [ { "internalType": "address", "name": "validatorAddress", "type": "address" }, { "internalType": "int64", "name": "startHeight", "type": "int64" }, { "internalType": "int64", "name": "indexOffset", "type": "int64" }, { "internalType": "int64", "name": "jailedUntil", "type": "int64" }, { "internalType": "bool", "name": "tombstoned", "type": "bool" }, { "internalType": "int64", "name": "missedBlocksCounter", "type": "int64" } ], "internalType": "struct SigningInfo[]", "name": "signingInfos", "type": "tuple[]" }, { "components": [ { "internalType": "bytes", "name": "nextKey", "type": "bytes" }, { "internalType": "uint64", "name": "total", "type": "uint64" } ], "internalType": "struct PageResponse", "name": "pageResponse", "type": "tuple" } ], "stateMutability": "view", "type": "function" },
    { "inputs": [ { "internalType": "address", "name": "validatorAddress", "type": "address" } ], "name": "unjail", "outputs": [ { "internalType": "bool", "name": "success", "type": "bool" } ], "stateMutability": "nonpayable", "type": "function" }
  ]
}
```
