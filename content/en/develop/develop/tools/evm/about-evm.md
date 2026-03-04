---
title: Get start @xpla/evm
weight: 10
type: docs
---

# About @xpla/evm

`@xpla/evm` provides chain definitions and precompile addresses/ABIs for XPLA EVM. Use it with [viem](https://viem.sh/) to interact with CONX Chain's EVM and precompile contracts with full TypeScript type inference.

## Overview

CONX Chain exposes Cosmos SDK module functionality via precompile contracts in the EVM environment. The `@xpla/evm` package exports chain configs and precompile `{ abi, address }` objects for use with viem, so you can call `readContract` and `writeContract` with full type safety.

## Key Features

- **Chain definitions**: `conxMainnet`, `conxTestnet`, `conxLocal` for viem
- **Precompile ABI + address**: Each precompile exported as `{ abi, address }` for viem
- **Type inference**: Full type inference for `readContract` / `writeContract` and `getContract`
- **Precompile support**: auth, bank, wasm, bech32, distribution, gov, slashing, staking

## Prerequisites

- [Node.js v18 or later](https://nodejs.org/)
- [pnpm](https://pnpm.io/), npm, or yarn
- Basic knowledge of TypeScript/JavaScript
- Understanding of [viem](https://viem.sh/) and smart contracts

## Installation

```bash
pnpm add @xpla/evm viem
```

Or with npm:

```bash
npm install @xpla/evm viem
```

Or with yarn:

```bash
yarn add @xpla/evm viem
```

## Chain (viem)

Use the exported chain definitions with viem's `createPublicClient`:

```typescript
import { conxMainnet, conxTestnet, conxLocal } from '@xpla/evm';
import { createPublicClient, http } from 'viem';

const client = createPublicClient({
  chain: conxMainnet,
  transport: http(),
});
```

## Precompile addresses and ABI

Each precompile is exported as `{ abi, address }` from `@xpla/evm/precompiles` for use with viem. You get full type inference for `readContract` and `writeContract`.

### Query example (read)

Example that queries bank balance and staking validators.

```typescript
import { bank, staking } from '@xpla/evm/precompiles';
import { getContract, createPublicClient, http } from 'viem';
import { conxMainnet } from '@xpla/evm';

async function main() {
  const publicClient = createPublicClient({
    chain: conxMainnet,
    transport: http(),
  });

  // Bank: query balance
  const bankContract = getContract({ ...bank, client: publicClient });
  const testAddress = '0x1234567890123456789012345678901234567890' as const;
  const denom = 'axpla';

  try {
    const balance = await bankContract.read.balance([testAddress, denom]);
    console.log('Balance:', balance);
  } catch (error) {
    console.log('Balance query failed (expected for test address)');
  }

  // Staking: query validators
  const stakingContract = getContract({ ...staking, client: publicClient });
  try {
    const validators = await stakingContract.read.validators([
      'BOND_STATUS_BONDED',
      { key: new Uint8Array(), offset: 0n, limit: 10n, countTotal: false, reverse: false },
    ]);
    console.log('Validators count:', validators.validators?.length ?? 0);
  } catch (error) {
    console.log('Validators query failed (expected if none exist)');
  }
}

main();
```

### Write contract example (with signer)

Example using a wallet (signer) to read balance and send transactions via the Bank precompile.

```typescript
import { bank } from '@xpla/evm/precompiles';
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { conxMainnet } from '@xpla/evm';
import { getContract, writeContract } from 'viem';

async function main() {
  const transport = http();
  const account = privateKeyToAccount(
    (process.env.PRIVATE_KEY || '0x0000000000000000000000000000000000000000000000000000000000000001') as `0x${string}`
  );

  const walletClient = createWalletClient({ chain: conxMainnet, transport, account });

  const bankContract = getContract({ ...bank, client: walletClient });

  // Read: query balance for signer address
  const balance = await bankContract.read.balance([account.address, 'axpla']);
  console.log('Signer balance:', balance);

  // Write: call send (use real toAddress, amount, denom for actual transfer)
  await writeContract(walletClient, {
     ...bank,
     functionName: 'send',
     args: [toAddress, amount, denom],
  });
}

main();
```

To load your private key from environment variables, add `PRIVATE_KEY=0x...` to a `.env` file and use `dotenv`:

```bash
npm install dotenv
```

```typescript
import 'dotenv/config';
// ... rest of your code
```

## Available precompiles

The following precompiles are exported from `@xpla/evm/precompiles` (use these instead of legacy constants):

| Export     | Description              |
| ---------- | ------------------------ |
| **auth**   | Authentication           |
| **bank**   | Bank / token operations  |
| **wasm**   | CosmWasm                 |
| **bech32** | Bech32 encoding          |
| **distribution** | Distribution       |
| **gov**    | Governance               |
| **slashing** | Slashing              |
| **staking**  | Staking                |


## ABI sources

- **@xpla/contracts**: Auth, Bank, Wasm
- **cosmos-evm-contracts**: Bech32, Distribution, Gov, Slashing, Staking
