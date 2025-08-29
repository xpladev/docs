---
title: bech32
weight: 60
type: docs
---


**Address**: `0x0000000000000000000000000000000000000400`

**Related Module**: Address conversion utilities

The Bech32 precompile provides address format conversion functionality between Ethereum hex addresses and Cosmos bech32 addresses.

## Overview

The Bech32 precompile exposes simple conversion helpers so Solidity contracts can:
1. Convert an Ethereum `address` to a Bech32 string with a desired `prefix` (e.g. `cosmos`).
2. Convert any Bech32 address string back into an EVM `address`.

It is a thin wrapper around the Cosmos address‐conversion library and lives at **`0x0000000000000000000000000000000000000400`**.

## Gas Costs

Both methods use a configurable base gas amount that is set during chain initialization. The gas cost is fixed regardless of string length within reasonable bounds.

## Primary Methods

### `hexToBech32`

**Signature**: `hexToBech32(address addr, string memory prefix) → string memory`

**Description**: Converts an Ethereum hex address to Cosmos bech32 format using the specified prefix.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// Connect to the network
const provider = new ethers.JsonRpcProvider("<RPC_URL>");

// Precompile address and ABI
const precompileAddress = "0x0000000000000000000000000000000000000400";
const precompileAbi = [
  "function hexToBech32(address addr, string memory prefix) returns (string memory)"
];

// Create a contract instance
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const ethAddress = "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045"; // Placeholder
const prefix = "xpla";

async function convertToBech32() {
  try {
    const bech32Address = await contract.hexToBech32.staticCall(ethAddress, prefix);
    console.log("Bech32 Address:", bech32Address);
  } catch (error) {
    console.error("Error converting to Bech32:", error);
  }
}

convertToBech32();
```
```bash cURL expandable lines
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000400",
            "data": "0xf958a98c000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa960450000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000478706c6100000000000000000000000000000000000000000000000000000000"
        },
        "latest"
    ],
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev

cast to-ascii 0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000002b78706c61316d72647868756e66766a6865366c68646e63703732647134366461326a637a396e7a6774777a000000000000000000000000000000000000000000
 +xpla1mrdxhunfvjhe6lhdncp72dq46da2jcz9nzgtwz
```

**Parameters**:
- `addr` (address): The Ethereum hex address to convert
- `prefix` (string): The bech32 prefix to use (e.g., "cosmos")

**Returns**: String containing the bech32 formatted address

### `bech32ToHex`

**Signature**: `bech32ToHex(string memory bech32Address) → address`

**Description**: Converts a Cosmos bech32 address to Ethereum hex format.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

// Connect to the network
const provider = new ethers.JsonRpcProvider("<RPC_URL>");

// Precompile address and ABI
const precompileAddress = "0x0000000000000000000000000000000000000400";
const precompileAbi = [
  "function bech32ToHex(string memory bech32Address) returns (address)"
];

// Create a contract instance
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const bech32Address = "xpla1mrdxhunfvjhe6lhdncp72dq46da2jcz9nzgtwz"; // Placeholder

async function convertToHex() {
  try {
    const hexAddress = await contract.bech32ToHex.staticCall(bech32Address);
    console.log("Hex Address:", hexAddress);
  } catch (error) {
    console.error("Error converting to Hex:", error);
  }
}

convertToHex();
```
```bash cURL expandable lines
# Replace <RPC_URL> with your actual RPC endpoint.
# The address is a placeholder.
curl -X POST --data '{    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
        {
            "to": "0x0000000000000000000000000000000000000400",
            "data": "0xe6df461e0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000002b78706c61316d72647868756e66766a6865366c68646e63703732647134366461326a637a396e7a6774777a000000000000000000000000000000000000000000"
        },
        "latest"
    ],                                                              
    "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev

{"jsonrpc":"2.0","id":1,"result":"0x000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045"}
```

**Parameters**:
- `bech32Address` (string): The bech32 formatted address to convert

**Returns**: Ethereum address in hex format

## Full Interface & ABI

```solidity title="Bech32 Solidity Interface" lines
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

/// @dev The Bech32I contract's address.
address constant Bech32_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000400;

/// @dev The Bech32I contract's instance.
Bech32I constant BECH32_CONTRACT = Bech32I(Bech32_PRECOMPILE_ADDRESS);

interface Bech32I {
    function hexToBech32(address addr,string memory prefix) external returns (string memory bech32Address);
    function bech32ToHex(string memory bech32Address) external returns (address addr);
}
```

```json title="Bech32 ABI" lines expandable
{
  "_format": "hh-sol-artifact-1",
  "contractName": "Bech32I",
  "sourceName": "solidity/precompiles/bech32/Bech32I.sol",
  "abi": [
    {
      "inputs": [
        { "internalType": "string", "name": "bech32Address", "type": "string" }
      ],
      "name": "bech32ToHex",
      "outputs": [
        { "internalType": "address", "name": "addr", "type": "address" }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    },
    {
      "inputs": [
        { "internalType": "address", "name": "addr", "type": "address" },
        { "internalType": "string", "name": "prefix", "type": "string" }
      ],
      "name": "hexToBech32",
      "outputs": [
        { "internalType": "string", "name": "bech32Address", "type": "string" }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    }
  ],
  "bytecode": "0x",
  "deployedBytecode": "0x",
  "linkReferences": {},
  "deployedLinkReferences": {}
}
```

## Implementation Details

### Address Validation

Both methods perform validation on the address format:
- **hexToBech32**: Validates that the hex address is exactly 20 bytes and the prefix is non-empty
- **bech32ToHex**: Validates that the bech32 address contains the separator character "1" and decodes to 20 bytes

### Prefix Handling

For `hexToBech32`:
- The prefix parameter determines the human-readable part of the bech32 address
- Common prefixes include account addresses, validator addresses, and consensus addresses
- Empty or whitespace-only prefixes result in an error with suggested valid prefixes

For `bech32ToHex`:
- The prefix is automatically extracted from the bech32 address
- No prefix parameter is required as it's embedded in the address

### State Mutability

While both methods are marked as `nonpayable` in the ABI, they function as read-only operations and do not modify blockchain state.
