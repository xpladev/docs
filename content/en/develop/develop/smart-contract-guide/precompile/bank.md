---
title: bank
weight: 30
type: docs
---

## Overview

The Bank precompile provides access to the Cosmos SDK `x/bank` module, enabling smart contracts to send tokens, query balances, and check token supply information.

#### Precompile Address: `0x1000000000000000000000000000000000000001`

#### Related Module: [x/bank](/develop/develop/core-modules/bank/)

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
string constant MSG_SEND = "/cosmos.bank.v1beta1.MsgSend";
```

## Transaction Methods

### send
Sends tokens from one address to another.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function send(address fromAddress, address toAddress, tuple(string denom, uint256 amount)[] amount) returns (bool success)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000001";
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

// Example inputs
const fromAddress = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder
const toAddress = "0x1234567890123456789012345678901234567890"; // Placeholder
const amount = [
  {
    denom: "axpla",
    amount: ethers.parseUnits("1", 18) // 1 XPLA
  }
];

async function sendTokens() {
  try {
    const tx = await contract.send(fromAddress, toAddress, amount);
    console.log("Transaction sent:", tx);
    await tx.wait();
    console.log("Transaction confirmed");
  } catch (error) {
    console.error("Error sending tokens:", error);
  }
}

sendTokens();
```

## Query Methods

### balance
Queries the balance of a specific denomination for an address.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function balance(address addr, string denom) view returns (uint256 balance)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000001";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Inputs
const address = "0x242Db81884b6ACBEbCBe1227740C5552a6480BA0"; // Placeholder
const denom = "axpla"; // XPLA token denomination

async function getBalance() {
  try {
    const balance = await contract.balance(address, denom);
    console.log("Balance:", ethers.formatUnits(balance, 18), "XPLA");
  } catch (error) {
    console.error("Error fetching balance:", error);
  }
}

getBalance();
```
```bash cURL expandable lines
# Note: Replace with your actual data.
# Data is ABI-encoded: function selector + padded address + encoded string offset + string length + string data
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000001",
      "data":"0x16cadeab000000000000000000000000242db81884b6acbebcbe1227740c5552a6480ba0000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000056178706c61000000000000000000000000000000000000000000000000000000"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

### supplyOf
Queries the total supply of a specific denomination.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function supplyOf(string denom) view returns (uint256 supply)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000001";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

// Input
const denom = "axpla"; // XPLA token denomination

async function getTotalSupply() {
  try {
    const supply = await contract.supplyOf(denom);
    console.log("Total Supply:", ethers.formatUnits(supply, 18), "XPLA");
  } catch (error) {
    console.error("Error fetching supply:", error);
  }
}

getTotalSupply();
```
```bash cURL expandable lines
# Note: Replace with your actual data.
# Data is ABI-encoded: function selector + encoded string offset + string length + string data
curl -X POST --data '{
  "jsonrpc":"2.0",
  "method":"eth_call",
  "params":[
    {
      "to":"0x1000000000000000000000000000000000000001",
      "data":"0x3cda0103000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000056178706c61000000000000000000000000000000000000000000000000000000"
    },
    "latest"
  ],
  "id":1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

## Full Solidity Interface & ABI

```solidity title="Bank Solidity Interface" lines expandable
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Coin} from "../util/Types.sol";

address constant BANK_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000001;

IBank constant BANK_CONTRACT = IBank(
    BANK_PRECOMPILE_ADDRESS
);

interface IBank {
    /**
     * @dev Send defines an event emitted when coins are sended
     * @param from the address of the sender
     * @param to the address of the receiver
     * @param amount the amount of sended coin
     */
    event Send(
        address indexed from,
        address indexed to,
        Coin[] amount
    );

    // Transactions
    function send(
        address fromAddress,
        address toAddress,
        Coin[] memory amount
    ) external returns (bool success);

    // Queries
    function balance(
        address addr,
        string memory denom
    ) external view returns (uint256 balance);

    function supplyOf(
        string memory denom
    ) external view returns (uint256 supply);
}
```

```json title="Bank ABI" lines expandable
[
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "from",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "to",
        "type": "address"
      },
      {
        "components": [
          {
            "internalType": "string",
            "name": "denom",
            "type": "string"
          },
          {
            "internalType": "uint256",
            "name": "amount",
            "type": "uint256"
          }
        ],
        "indexed": false,
        "internalType": "struct Coin[]",
        "name": "amount",
        "type": "tuple[]"
      }
    ],
    "name": "Send",
    "type": "event"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "addr",
        "type": "address"
      },
      {
        "internalType": "string",
        "name": "denom",
        "type": "string"
      }
    ],
    "name": "balance",
    "outputs": [
      {
        "internalType": "uint256",
        "name": "balance",
        "type": "uint256"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "fromAddress",
        "type": "address"
      },
      {
        "internalType": "address",
        "name": "toAddress",
        "type": "address"
      },
      {
        "components": [
          {
            "internalType": "string",
            "name": "denom",
            "type": "string"
          },
          {
            "internalType": "uint256",
            "name": "amount",
            "type": "uint256"
          }
        ],
        "internalType": "struct Coin[]",
        "name": "amount",
        "type": "tuple[]"
      }
    ],
    "name": "send",
    "outputs": [
      {
        "internalType": "bool",
        "name": "success",
        "type": "bool"
      }
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "string",
        "name": "denom",
        "type": "string"
      }
    ],
    "name": "supplyOf",
    "outputs": [
      {
        "internalType": "uint256",
        "name": "supply",
        "type": "uint256"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  }
]
```