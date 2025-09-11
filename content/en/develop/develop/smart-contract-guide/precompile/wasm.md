---
title: wasm
weight: 40
type: docs
---

## Overview

The Wasm precompile provides access to the Cosmos SDK `x/wasm` module (CosmWasm) from EVM smart contracts. It enables instantiation, execution, and migration of CosmWasm contracts, as well as querying contract state.

#### Precompile Address: `0x1000000000000000000000000000000000000004`

#### Related Module: [x/wasm](/develop/develop/core-modules/wasm/)

## Gas Costs

<Info>
Gas costs are approximated and may vary based on call complexity and chain settings.
</Info>

| Method | Gas Cost |
|---|---|
| **Transactions** | `2000 + (30 × bytes of input)` |
| **Queries** | `1000 + (3 × bytes of input)` |

## Transaction Methods

### instantiateContract
Instantiates a CosmWasm contract from a given `codeId`.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function instantiateContract(address sender, address admin, uint256 codeId, string label, bytes msg, tuple(string denom, uint256 amount)[] funds) returns (address contractAddress, bytes data)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const precompileAddress = "0x1000000000000000000000000000000000000004";
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

const sender = await signer.getAddress();
const admin = sender; // or another address
const codeId = 1n; // Example code ID
const label = "my-counter";
const msgBytes = ethers.toUtf8Bytes('{"instantiate": {"count": 0}}');
const funds = [{ denom: "axpla", amount: ethers.parseUnits("0", 18) }];

async function instantiate() {
  const tx = await contract.instantiateContract(sender, admin, codeId, label, msgBytes, funds);
  console.log("Tx sent:", tx.hash);
  const receipt = await tx.wait();
  console.log("Confirmed in block:", receipt.blockNumber);
}

instantiate();
```

### instantiateContract2
Instantiates a contract using CREATE2-style salt. When `fixMsg` is true, the `msg` is treated as already fixed for address-dependent fields.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function instantiateContract2(address sender, address admin, uint256 codeId, string label, bytes msg, tuple(string denom, uint256 amount)[] funds, bytes salt, bool fixMsg) returns (address contractAddress, bytes data)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const precompileAddress = "0x1000000000000000000000000000000000000004";
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

const sender = await signer.getAddress();
const admin = sender;
const codeId = 1n;
const label = "my-counter-create2";
const msgBytes = ethers.toUtf8Bytes('{"instantiate": {"count": 0}}');
const funds = [];
const salt = ethers.toUtf8Bytes("my_salt");
const fixMsg = false;

async function instantiate2() {
  const tx = await contract.instantiateContract2(sender, admin, codeId, label, msgBytes, funds, salt, fixMsg);
  console.log("Tx sent:", tx.hash);
  await tx.wait();
}

instantiate2();
```

### executeContract
Executes a message on an existing CosmWasm contract.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function executeContract(address sender, address contractAddress, bytes msg, tuple(string denom, uint256 amount)[] funds) returns (bytes data)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const precompileAddress = "0x1000000000000000000000000000000000000004";
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

const sender = await signer.getAddress();
const contractAddress = "0x0000000000000000000000000000000000000000"; // Placeholder (CosmWasm contract EVM address)
const msgBytes = ethers.toUtf8Bytes('{"increment": {}}');
const funds = [];

async function exec() {
  const tx = await contract.executeContract(sender, contractAddress, msgBytes, funds);
  console.log("Tx sent:", tx.hash);
  const receipt = await tx.wait();
  console.log("Confirmed in block:", receipt.blockNumber);
}

exec();
```

### migrateContract
Migrates an existing contract to a new `codeId` with a migration message.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function migrateContract(address sender, address contractAddress, uint256 codeId, bytes msg) returns (bytes data)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const signer = new ethers.Wallet("<PRIVATE_KEY>", provider);
const precompileAddress = "0x1000000000000000000000000000000000000004";
const contract = new ethers.Contract(precompileAddress, precompileAbi, signer);

const sender = await signer.getAddress();
const contractAddress = "0x0000000000000000000000000000000000000000"; // Placeholder
const codeId = 2n;
const msgBytes = ethers.toUtf8Bytes('{"migrate": {}}');

async function migrate() {
  const tx = await contract.migrateContract(sender, contractAddress, codeId, msgBytes);
  console.log("Tx sent:", tx.hash);
  await tx.wait();
}

migrate();
```

## Query Methods

### smartContractState
Queries a CosmWasm contract's state with a binary JSON query.

```javascript Ethers.js expandable lines
import { ethers } from "ethers";

const precompileAbi = [
  "function smartContractState(address contractAddress, bytes queryData) view returns (bytes data)"
];

const provider = new ethers.JsonRpcProvider("https://cube-evm-rpc.xpla.dev");
const precompileAddress = "0x1000000000000000000000000000000000000004";
const contract = new ethers.Contract(precompileAddress, precompileAbi, provider);

const contractAddress = "0xEC5111BE581F3B394CEAF4B8E27973216D4E16B7"; // Placeholder
const queryBytes = ethers.toUtf8Bytes('{"get_count": {}}');

async function query() {
  const result = await contract.smartContractState(contractAddress, queryBytes);
  // result is hex-encoded bytes; decode to string then JSON if applicable
  const hex = result.startsWith("0x") ? result.slice(2) : result;
  const text = new TextDecoder().decode(Buffer.from(hex, "hex"));
  console.log("Query Response:", text);
}

query();
```
```bash cURL expandable lines
# Note: Replace with real selector and ABI-encoded inputs.
curl -X POST --data '{
  "jsonrpc": "2.0",
  "method": "eth_call",
  "params": [
    {
      "to": "0x1000000000000000000000000000000000000004",
      "data": "0x72a8d499000000000000000000000000ec5111be581f3b394ceaf4b8e27973216d4e16b7000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000117b226765745f636f756e74223a207b7d7d000000000000000000000000000000"
    },
    "latest"
  ],
  "id": 1
}' -H "Content-Type: application/json" https://cube-evm-rpc.xpla.dev
```

## Full Solidity Interface & ABI

```solidity title="Wasm Solidity Interface" lines expandable
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Coin} from "../util/Types.sol";

address constant WASM_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000004;

IWasm constant WASM_CONTRACT = IWasm(
    WASM_PRECOMPILE_ADDRESS
);

interface IWasm {
    // Events
    /**
     * @dev InstantiateContract defines an event emitted when a wasm contract is successfully
     * instantiated via instantiateContract or instantiateContract2
     * @param sender the address of the sender
     * @param contractAddress the address of the instantiated contract
     * @param codeId the id of contract
     * @param admin the address of the contract admin
     * @param label the optional metadata to be stored with a contract instance
     * @param msg the message to be passed to the contract on instantiation
     * @param funds the coins that are transferred to the contract on instantiation
     * @param data the bytes to returned from the contract
     */
    event InstantiateContract(
        address indexed sender,
        address indexed contractAddress,
        uint256 indexed codeId,
        address admin,
        string label,
        bytes msg,
        Coin[] funds,
        bytes data
    );

    /**
     * @dev ExecuteContract defines an event emitted when a wasm contract is successfully
     * executed via executeContract
     * @param sender the address of the sender
     * @param contractAddress the address of executed contract
     * @param msg the message to be passed to the contract
     * @param funds the coins that are transferred to the contract on execution
     * @param data the bytes to returned from the contract
     */
    event ExecuteContract(
        address indexed sender,
        address indexed contractAddress,
        bytes msg,
        Coin[] funds,
        bytes data
    );

    /**
     * @dev MigrateContract defines an event emitted when a wasm contract is successfully
     * migrated via migrateContract
     * @param sender the address of the sender
     * @param contractAddress the address of migrated contract
     * @param codeId changed code id
     * @param msg the message to be passed to the contract on migration
     * @param data the bytes to returned from the contract
     */
    event MigrateContract(
        address indexed sender,
        address indexed contractAddress,
        uint256 indexed codeId,
        bytes msg,
        bytes data
    );

    // Transactions
    function instantiateContract(
        address sender,
        address admin,
        uint256 codeId,
        string calldata label,
        bytes calldata msg,
        Coin[] memory funds
    ) external returns (address contractAddress, bytes calldata data);
    function instantiateContract2(
        address sender,
        address admin,
        uint256 codeId,
        string calldata label,
        bytes calldata msg,
        Coin[] memory funds,
        bytes calldata salt,
        bool fixMsg
    ) external returns (address contractAddress, bytes calldata data);
    function executeContract(
        address sender,
        address contractAddress,
        bytes calldata msg,
        Coin[] memory funds
    ) external returns (bytes calldata data);
    function migrateContract(
        address sender,
        address contractAddress,
        uint256 codeId,
        bytes calldata msg
    ) external returns (bytes calldata data);
    
    // Queries
    function smartContractState(
        address contractAddress,
        bytes calldata queryData
    ) external view returns (bytes calldata data);
}
```

```json title="Wasm ABI" lines expandable
[
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
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
        "name": "funds",
        "type": "tuple[]"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "name": "ExecuteContract",
    "type": "event"
  },
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "uint256",
        "name": "codeId",
        "type": "uint256"
      },
      {
        "indexed": false,
        "internalType": "address",
        "name": "admin",
        "type": "address"
      },
      {
        "indexed": false,
        "internalType": "string",
        "name": "label",
        "type": "string"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
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
        "name": "funds",
        "type": "tuple[]"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "name": "InstantiateContract",
    "type": "event"
  },
  {
    "anonymous": false,
    "inputs": [
      {
        "indexed": true,
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "indexed": true,
        "internalType": "uint256",
        "name": "codeId",
        "type": "uint256"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
      },
      {
        "indexed": false,
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "name": "MigrateContract",
    "type": "event"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
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
        "name": "funds",
        "type": "tuple[]"
      }
    ],
    "name": "executeContract",
    "outputs": [
      {
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "internalType": "address",
        "name": "admin",
        "type": "address"
      },
      {
        "internalType": "uint256",
        "name": "codeId",
        "type": "uint256"
      },
      {
        "internalType": "string",
        "name": "label",
        "type": "string"
      },
      {
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
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
        "name": "funds",
        "type": "tuple[]"
      }
    ],
    "name": "instantiateContract",
    "outputs": [
      {
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "internalType": "address",
        "name": "admin",
        "type": "address"
      },
      {
        "internalType": "uint256",
        "name": "codeId",
        "type": "uint256"
      },
      {
        "internalType": "string",
        "name": "label",
        "type": "string"
      },
      {
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
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
        "name": "funds",
        "type": "tuple[]"
      },
      {
        "internalType": "bytes",
        "name": "salt",
        "type": "bytes"
      },
      {
        "internalType": "bool",
        "name": "fixMsg",
        "type": "bool"
      }
    ],
    "name": "instantiateContract2",
    "outputs": [
      {
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "sender",
        "type": "address"
      },
      {
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "internalType": "uint256",
        "name": "codeId",
        "type": "uint256"
      },
      {
        "internalType": "bytes",
        "name": "msg",
        "type": "bytes"
      }
    ],
    "name": "migrateContract",
    "outputs": [
      {
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "contractAddress",
        "type": "address"
      },
      {
        "internalType": "bytes",
        "name": "queryData",
        "type": "bytes"
      }
    ],
    "name": "smartContractState",
    "outputs": [
      {
        "internalType": "bytes",
        "name": "data",
        "type": "bytes"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  }
]
```
