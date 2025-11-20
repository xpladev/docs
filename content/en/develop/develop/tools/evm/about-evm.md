---
title: Get start @xpla/evm
weight: 10
type: docs
---

# About @xpla/evm

`@xpla/evm` is a TypeScript library that provides TypeChain-generated interfaces for CONX Chain's precompile contracts. This package enables developers to easily interact with CONX Chain's precompile contracts using ethers.js with full TypeScript support and type safety.

## Overview

CONX Chain provides various precompile contracts that allow access to Cosmos SDK module functionality within the EVM environment. The `@xpla/evm` package provides pre-generated TypeScript interfaces for these precompile contracts, making it easy to interact with them using familiar ethers.js patterns.

## Key Features

- **TypeScript Interfaces**: Pre-generated TypeChain interfaces for all precompile contracts
- **Ethers.js Integration**: Seamless integration with ethers.js library
- **Type Safety**: Full TypeScript support with complete type definitions
- **Precompile Contract Support**: Easy access to CONX Chain's precompile contracts
- **Factory Classes**: Generated factory classes for contract instantiation
- **Convenience Functions**: Helper functions for creating pre-connected precompile contracts

## Getting Started

This guide will walk you through setting up a project with `@xpla/evm` and interacting with CONX Chain's precompile contracts.

## About This Tutorial

In this tutorial, you'll learn how to:

1. [Set up your project](#1-set-up-your-project)
2. [Install @xpla/evm](#2-install-xpla-evm)
3. [Connect to precompile contracts](#3-connect-to-precompile-contracts)
4. [Use individual precompile contracts](#4-use-individual-precompile-contracts)
5. [Use convenience functions](#5-use-convenience-functions)

By the end of this guide, you'll be able to interact with CONX Chain's precompile contracts using the `@xpla/evm` package.

## Prerequisites

- [Node.js v18 or later](https://nodejs.org/)
- [npm or yarn](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- Basic knowledge of TypeScript/JavaScript
- Understanding of ethers.js and smart contracts

## 1. Set up Your Project

1. Create a new directory for your project:

   ```sh
   mkdir my-xpla-evm-project
   cd my-xpla-evm-project
   ```

2. Initialize your project:

   ```sh
   npm init -y
   ```

3. Create the basic project structure:

   ```sh
   mkdir src
   touch src/index.ts
   ```

## 2. Install @xpla/evm

Install the `@xpla/evm` package and its dependencies:

```sh
npm install @xpla/evm
```

Or if you're using yarn:

```sh
yarn add @xpla/evm
```

## 3. Use Query Functions

Create a script using the convenience functions for easier precompile contract access. Create `src/query-example.ts`:

```typescript
import { ethers } from 'ethers';
import { 
  createPrecompileBank, 
  createPrecompileGov, 
  createPrecompileContracts 
} from '@xpla/evm/precompiles';

async function main() {
  console.log('=== Convenience Functions Example ===\n');

  // Setup provider
  const provider = new ethers.JsonRpcProvider('https://cube-evm-rpc.xpla.dev');

  try {
    // Method 1: Create individual precompile contracts
    console.log('Creating individual precompile contracts...');
    const bankContract = createPrecompileBank(provider);
    const govContract = createPrecompileGov(provider);
    
    console.log('✅ Individual contracts created successfully!\n');

    // Method 2: Create all precompile contracts at once
    console.log('Creating all precompile contracts...');
    const contracts = createPrecompileContracts(provider);
    
    console.log('✅ All contracts created successfully!\n');

    // Example: Using individual contracts
    const testAddress = '0x1234567890123456789012345678901234567890';
    console.log(`Querying balance using individual contract: ${testAddress}`);
    
    try {
      const balance = await bankContract.balance(testAddress, 'axpla');
      console.log(`Balance: ${ethers.formatEther(balance)} XPLA\n`);
    } catch (error) {
      console.log('Balance query failed (expected for test address)\n');
    }

    // Example: Using staking contract from contracts object
    console.log('Querying staking validators using contracts object...');
    try {
      const validators = await contracts.staking.validators('BOND_STATUS_BONDED', {
        key: new Uint8Array(),
        offset: 0n,
        limit: 10n,
        countTotal: false,
        reverse: false
      });
      console.log(`Found ${validators.validators.length} validators\n`);
    } catch (error) {
      console.log('Validators query failed (expected if no validators exist)\n');
    }

  } catch (error) {
    console.error('❌ Convenience functions failed:', error);
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error('Script failed:', error);
    process.exit(1);
  });
```

## Using with Signer

Create a script that demonstrates using precompile contracts with a signer for transactions. Create `src/signer-example.ts`:

```typescript
import { ethers } from 'ethers';
import { IBank__factory } from '@xpla/evm';

async function main() {
  console.log('=== Precompile Contracts with Signer Example ===\n');

  // Setup provider and signer
  const provider = new ethers.JsonRpcProvider('https://cube-evm-rpc.xpla.dev');
  const privateKey = process.env.PRIVATE_KEY || 'your-private-key-here';
  const signer = new ethers.Wallet(privateKey, provider);

  console.log(`Using signer address: ${signer.address}\n`);

  // Precompile contract address
  const BANK_ADDRESS = '0x1000000000000000000000000000000000000001';

  try {
    // Create contract instance with signer
    const bankContract = IBank__factory.connect(BANK_ADDRESS, signer);

    console.log('✅ Bank contract connected with signer!\n');

    // Example: Send transaction (this would require proper parameters)
    console.log('Note: Sending transactions to precompile contracts requires specific parameters');
    console.log('and may require special permissions. This is for demonstration purposes.\n');

    // Example: Query balance using signer
    const balance = await bankContract.balance(signer.address, 'axpla');
    console.log(`Signer balance: ${ethers.formatEther(balance)} XPLA\n`);

  } catch (error) {
    console.error('❌ Signer example failed:', error);
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error('Script failed:', error);
    process.exit(1);
  });
```

## Environment Setup

Create a `.env` file for your private key:

```env
PRIVATE_KEY=your-private-key-here
```

Install dotenv to load environment variables:

```sh
npm install dotenv
```

Update your scripts to use environment variables:

```typescript
import 'dotenv/config';
// ... rest of your code
```

## Running Your Project

1. **Test connection**:
   ```sh
   npx ts-node src/index.ts
   ```

2. **Run query functions example**:
   ```sh
   npx ts-node src/query-example.ts
   ```

4. **Run signer example**:
   ```sh
   npx ts-node src/signer-example.ts
   ```

## Included Interfaces

The `@xpla/evm` package includes TypeScript interfaces for the following precompile contracts:

- **IAuth** - Authentication related contracts
- **IBank** - Bank/Token related contracts  
- **IGov** - Governance related contracts
- **StakingI** - Staking related contracts
- **DistributionI** - Distribution related contracts
- **ISlashing** - Slashing related contracts
- **IWasm** - CosmWasm related contracts
- **Bech32I** - Bech32 encoding related contracts

## Installation

```bash
npm install @xpla/evm
```

Or

```bash
yarn add @xpla/evm
```