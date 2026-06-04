---
title: Get started with @xpla/contracts
weight: 10
type: docs
---

# About @xpla/contracts

`@xpla/contracts` is a collection of smart contracts for the [CONX Chain](../../../smart-contract-guide/evm/use-precompile-contract). The published package includes precompile interface sources (`.sol`) and ABIs (`.json`), providing type-safe interfaces for precompile contracts that allow access to Cosmos SDK module functionality within the EVM environment.

This documentation is based on **@xpla/contracts**.

## Overview

CONX Chain is a Cosmos SDK-based blockchain that supports Solidity smart contracts through the EVM module. Precompile contracts are pre-deployed at specific addresses and expose Cosmos SDK module functionality to the EVM. The `@xpla/contracts` package provides standardized Solidity interfaces and ABIs for interacting with these precompile contracts.

## Package structure

After installation, use the following paths:

| Path | Description |
|------|-------------|
| `@xpla/contracts/precompiles/` | Solidity interface sources (`.sol`) |
| `@xpla/contracts/abi/precompiles/` | ABI as JSON (`.json`) or typed ESM (`.js` + `.d.ts`) |

## Available precompiles

The package includes the following precompile interfaces:

- **IAuth** (`precompiles/auth/IAuth.sol`) – Authentication and account management
  - Account address conversion between EVM and Cosmos formats
  - Module account access and Bech32 prefix management
  - Address format validation and conversion utilities

- **IBank** (`precompiles/bank/IBank.sol`) – Banking and token operations
  - Token transfers between accounts
  - Balance and supply queries for any denomination
  - Multi-denomination support through Coin arrays

- **IWasm** (`precompiles/wasm/IWasm.sol`) – CosmWasm contract support
  - Contract instantiation and execution
  - Cross-contract communication between EVM and Wasm
  - Contract state queries and migration

## Other precompiles (cosmos-evm-contracts)

For additional Cosmos SDK module precompiles (staking, distribution, governance, slashing, Bech32, etc.), use the [cosmos-evm-contracts](https://www.npmjs.com/package/cosmos-evm-contracts) package. CONX Chain’s EVM precompiles are compatible with these interfaces, so you can import and use them in the same project.

**Included in cosmos-evm-contracts:** `bank`, `bech32`, `callbacks`, `common`, `distribution`, `erc20`, `gov`, `ics02`, `ics20`, `slashing`, `staking`, `werc20`.

Install and import in your contract as follows:

```sh
npm install cosmos-evm-contracts
```

```solidity
import "cosmos-evm-contracts/precompiles/staking/StakingI.sol";
import "cosmos-evm-contracts/precompiles/distribution/DistributionI.sol";
import "cosmos-evm-contracts/precompiles/gov/IGov.sol";
import "cosmos-evm-contracts/precompiles/slashing/ISlashing.sol";
import "cosmos-evm-contracts/precompiles/bech32/Bech32I.sol";
// Common types (structs) if needed:
// import "cosmos-evm-contracts/precompiles/common/Types.sol";
```

Use **@xpla/contracts** for CONX-specific precompiles (auth, bank, wasm) and **cosmos-evm-contracts** for the rest.

## Key features

- **Type safety**: Type-safe interfaces for precompile contract functions
- **Dual output**: Solidity sources for compilation and ABIs for frontend/SDK use
- **Typed ABI**: ESM + TypeScript definitions for viem/ethers with inferred types
- **Easy integration**: Works with Hardhat and Foundry

# Getting started

This guide walks through setting up a project with `@xpla/contracts` and creating a smart contract that interacts with CONX Chain precompiles.

## About this tutorial

You will:

1. [Set up a Hardhat project](#1-set-up-your-hardhat-project)
2. [Install @xpla/contracts](#2-install-xplacontracts)
3. [Create a smart contract](#3-create-a-smart-contract)

## Prerequisites

- [Node.js v22 or later](https://nodejs.org/)
- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm), [pnpm](https://pnpm.io/), or [yarn](https://yarnpkg.com/)
- Basic knowledge of Solidity and smart contract development

## 1. Set up your Hardhat project

1. Create a new directory and enter it:

   ```sh
   mkdir my-xpla-contracts-project
   cd my-xpla-contracts-project
   ```

2. Initialize a Hardhat project:

   ```sh
   npx hardhat init
   ```

   Accept the default options to get a project with the Node.js test runner and viem. This will initialize the project and install dependencies.

## 2. Install @xpla/contracts

Install the package:

```sh
npm install @xpla/contracts
# or: npm install @xpla/contracts@1.9.0-rc3
```

With pnpm:

```sh
pnpm add @xpla/contracts
```

With yarn:

```sh
yarn add @xpla/contracts
```

## 3. Create a smart contract

Create `contracts/XplaExample.sol` and import the precompile interfaces from `precompiles/`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@xpla/contracts/precompiles/bank/IBank.sol";
import "@xpla/contracts/precompiles/auth/IAuth.sol";

contract XplaExample {
    IBank public bankContract;
    IAuth public authContract;

    constructor() {
        bankContract = IBank(0x1000000000000000000000000000000000000001);
        authContract = IAuth(0x1000000000000000000000000000000000000005);
    }

    function checkBalance(address account, string memory denom) public view returns (uint256) {
        return bankContract.balance(account, denom);
    }

    function convertAddress(address evmAddress) public view returns (string memory) {
        return authContract.addressBytesToString(evmAddress);
    }

    function getTokenSupply(string memory denom) public view returns (uint256) {
        return bankContract.supplyOf(denom);
    }

    function getBech32Prefix() public view returns (string memory) {
        return authContract.bech32Prefix();
    }
}
```