---
title: auth
weight: 20
type: docs
---

## Overview

The Auth precompile exposes selected features of the Cosmos SDK `x/auth` module to EVM smart contracts, focusing on account address conversions and module account lookups.

#### Precompile Address: `0x1000000000000000000000000000000000000005`

#### Related Module: [x/auth](/develop/develop/core-modules/auth/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on call complexity and chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Methods

### account
Returns the account Bech32 string address for a given EVM `address`.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function account(address evmAddress) view returns (string stringAddress)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000005";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

const evmAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder

async function getAccountBech32() {
  const bech32 = await contract.account(evmAddress);
  console.log("Bech32:", bech32);
}

getAccountBech32();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual data.
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000005",
      "data":"0x73b9aa91000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

### addressBytesToString
Converts an EVM `address` to its Bech32 string representation.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function addressBytesToString(address evmAddress) view returns (string stringAddress)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000005";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

const evmAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder

async function toBech32() {
  const bech32 = await contract.addressBytesToString(evmAddress);
  console.log("Bech32:", bech32);
}

toBech32();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual data.
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000005",
      "data":"0xf9f90741000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

### addressStringToBytes
Converts a Bech32 string account address to an EVM `address`.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function addressStringToBytes(string stringAddress) view returns (address byteAddress)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000005";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

const bech32 = "xpla1yskmsxyyk6kta097zgnhgrz422nyszaqcyeytp"; // Placeholder

async function toBytes() {
  const addr = await contract.addressStringToBytes(bech32);
  console.log("EVM Address:", addr);
}

toBytes();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual data.
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000005",
      "data":"0xaed310590000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000002b78706c613179736b6d737879796b366b74613039377a676e6867727a3432326e79737a6171637965797470000000000000000000000000000000000000000000"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

### bech32Prefix
Returns the current Bech32 prefix used by the chain (e.g., `xpla`).

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function bech32Prefix() view returns (string prefix)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000005";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

async function getPrefix() {
  const prefix = await contract.bech32Prefix();
  console.log("Bech32 Prefix:", prefix);
}

getPrefix();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual data.
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000005",
      "data":"0xd6aa6674"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

### moduleAccountByName
Returns the Bech32 address of a module account by its name.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function moduleAccountByName(string name) view returns (string stringAddress)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000005";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

const moduleName = "distribution"; // Example placeholder

async function getModuleAccount() {
  const addr = await contract.moduleAccountByName(moduleName);
  console.log("Module Account:", addr);
}

getModuleAccount();
```
```bash cURL expandable lines
# Note: Replace https://cube-evm-rpc.xpla.dev with your actual data.
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000005",
      "data":"0x61dd1dd40000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000c646973747269627574696f6e0000000000000000000000000000000000000000"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

## Full Solidity Interface & ABI

```solidity title="Auth Solidity Interface" lines expandable
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

address constant AUTH_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000005;

IAuth constant AUTH_CONTRACT = IAuth(
    AUTH_PRECOMPILE_ADDRESS
);

interface IAuth {
    // function accountAddressByID(uint accountId) external view returns (string calldata stringAddress);
    // function accounts(address evmAddress) external view returns (string[] calldata stringAddress);
    function account(address evmAddress) external view returns (string calldata stringAddress);
    // function params() external view returns (...);
    // function moduleAccounts() external view returns (string[] calldata stringAddresses);
    function moduleAccountByName(string calldata name) external view returns (string calldata stringAddress);
    function bech32Prefix() external view returns (string calldata prefix);
    function addressBytesToString(address evmAddress) external view returns (string calldata stringAddress);
    function addressStringToBytes(string calldata stringAddress) external view returns (address byteAddress);
    // function accountInfo(address evmAddress) external view returns (...);
}
```

```json title="Auth ABI" lines expandable
[
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "evmAddress",
        "type": "address"
      }
    ],
    "name": "account",
    "outputs": [
      {
        "internalType": "string",
        "name": "stringAddress",
        "type": "string"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "evmAddress",
        "type": "address"
      }
    ],
    "name": "addressBytesToString",
    "outputs": [
      {
        "internalType": "string",
        "name": "stringAddress",
        "type": "string"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "string",
        "name": "stringAddress",
        "type": "string"
      }
    ],
    "name": "addressStringToBytes",
    "outputs": [
      {
        "internalType": "address",
        "name": "byteAddress",
        "type": "address"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "bech32Prefix",
    "outputs": [
      {
        "internalType": "string",
        "name": "prefix",
        "type": "string"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "string",
        "name": "name",
        "type": "string"
      }
    ],
    "name": "moduleAccountByName",
    "outputs": [
      {
        "internalType": "string",
        "name": "stringAddress",
        "type": "string"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  }
]
```
