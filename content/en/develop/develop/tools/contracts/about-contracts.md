---
title: Get start @xpla/contracts
weight: 10
type: docs
---

# About @xpladev/contracts

`@xpladev/contracts` is a Solidity interface package for interacting with [CONX Chain's precompile contracts](../../../smart-contract-guide/evm/use-precompile-contract). This package provides type-safe interfaces for precompile contracts that allow access to various Cosmos SDK modules (bank, staking, governance, distribution, etc.) within the EVM environment.

## Overview

CONX Chain is a Cosmos SDK-based blockchain that supports Solidity smart contracts through the EVM module. Precompile contracts are pre-compiled contracts deployed at specific addresses that provide direct access to Cosmos SDK module functionality within the EVM environment.

The `@xpladev/contracts` package provides standardized Solidity interfaces for interacting with these precompile contracts.

## Available Contracts

### Core Interfaces

The package provides the following core precompile contract interfaces:

- **IAuth** - Authentication and account management
  - Account address conversion between EVM and Cosmos formats
  - Module account access and Bech32 prefix management
  - Address format validation and conversion utilities

- **IBank** - Banking and token operations
  - Token transfers between accounts
  - Balance and supply queries for any denomination
  - Multi-denomination support through Coin arrays

- **IWasm** - CosmWasm contract support
  - Contract instantiation and execution
  - Cross-contract communication between EVM and Wasm
  - Contract state queries and migration

- **StakingI** - Staking operations
  - Validator creation and management
  - Delegation, undelegation, and redelegation
  - Validator and delegation queries with pagination

- **DistributionI** - Distribution and rewards
  - Staking reward claims and withdrawals
  - Validator commission management
  - Community pool and validator rewards pool funding

- **IGov** - Governance operations
  - Proposal submission and management
  - Voting and deposit functionality
  - Governance parameter queries

- **ISlashing** - Slashing operations
  - Validator unjailing functionality
  - Signing information queries
  - Slashing parameter management

### Utilities

- **Bech32I** - Bech32 address encoding/decoding
  - Hex to Bech32 address conversion
  - Bech32 to hex address conversion
  - Cross-chain address format support

## Type Definitions

The package includes common type definitions used across contracts:

- **common/Types.sol** - Shared types used across contracts
  - Coin and DecCoin structures
  - PageRequest and PageResponse for pagination
  - Common enums and structs

- **util/Types.sol** - Utility-specific types
  - Specialized data structures for specific modules
  - Custom type definitions for complex operations

## Key Features

- **Type Safety**: Type-safe interfaces for all precompile contract functions
- **Standardized Interfaces**: Standard interfaces compatible with CONX Chain's precompile contracts
- **Easy Integration**: Easy integration with existing Solidity projects
- **Comprehensive Coverage**: Support for all major Cosmos SDK module functionalities
- **Modular Design**: Organized structure with core interfaces and utilities
- **Shared Types**: Common type definitions for consistent data handling

# Getting Started

This guide will walk you through setting up a Hardhat project with `@xpla/contracts` and creating your first smart contract that interacts with CONX Chain's precompile contracts.

## About This Tutorial

In this tutorial, you'll learn how to:

1. [Set up a Hardhat project](#1-set-up-your-hardhat-project)
2. [Install @xpla/contracts](#2-install-xplacontracts)
3. [Create a smart contract](#3-create-a-smart-contract)

By the end of this guide, you'll be able to create and deploy smart contracts that interact with CONX Chain's precompile contracts.

## Prerequisites

- [Node.js v22 or later](https://nodejs.org/)
- [npm or pnpm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- Basic knowledge of Solidity and smart contract development

## 1. Set up Your Hardhat Project

1. Create a new directory for your project:

   ```sh
   mkdir my-xpla-contracts-project
   cd my-xpla-contracts-project
   ```

2. Initialize your Hardhat project:

   ```sh
   npx hardhat --init
   ```

   This command will prompt you with configuration options. You can accept the default answers to quickly scaffold a working setup.

   Using the defaults will:
   - Initialize the project in the current directory
   - Use the sample project that includes the Node.js test runner and viem
   - Automatically install all the required dependencies

## 2. Install @xpla/contracts

Install the `@xpla/contracts` package to access CONX Chain's precompile contract interfaces:

```sh
npm install @xpla/contracts
```

Or if you're using pnpm:

```sh
pnpm add @xpla/contracts
```

## 3. Create a Smart Contract

Now let's create a simple smart contract that demonstrates how to use the precompile contracts. Create a new file `contracts/XplaExample.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "@xpla/contracts/interfaces/IBank.sol";
import "@xpla/contracts/interfaces/IAuth.sol";
import "@xpla/contracts/util/Types.sol";

contract XplaExample {
    IBank public bankContract;
    IAuth public authContract;
    
    constructor() {
        // Initialize precompile contract interfaces
        bankContract = IBank(0x1000000000000000000000000000000000000001);
        authContract = IAuth(0x1000000000000000000000000000000000000005);
    }
    
    // Function to check balance using Bank precompile
    function checkBalance(address account, string memory denom) public view returns (uint256) {
        uint256 balance = bankContract.balance(account, denom);
        return balance;
    }
    
    // Function to convert EVM address to Cosmos address
    function convertAddress(address evmAddress) public view returns (string memory) {
        string memory cosmosAddress = authContract.addressBytesToString(evmAddress);
        return cosmosAddress;
    }
    
    // Function to get total supply of a token
    function getTokenSupply(string memory denom) public view returns (uint256) {
        return bankContract.supplyOf(denom);
    }
    
    // Function to get Bech32 prefix
    function getBech32Prefix() public view returns (string memory) {
        return authContract.bech32Prefix();
    }
}
```