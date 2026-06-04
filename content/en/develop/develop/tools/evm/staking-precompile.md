---
title: Staking Precompile
weight: 40
type: docs
---

# Staking Precompile Example

This example demonstrates how to use the Staking precompile contract for validator queries and delegation with viem and `@xpla/evm`.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)
- Test tokens in your wallet for delegation

## Setup

Install the required dependencies:

```bash
pnpm add @xpla/evm viem
```

Or with npm: `npm install @xpla/evm viem`

## Example Code

```typescript
// examples/staking-precompile.ts
import { staking, bech32 } from '@xpla/evm/precompiles';
import { conxTestnet } from '@xpla/evm';
import { getContract, createPublicClient, createWalletClient, http, hexToBytes } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';

async function stakingPrecompileExample() {
  console.log('=== Staking Precompile Example ===\n');

  const transport = http();
  const publicClient = createPublicClient({ chain: conxTestnet, transport });
  const account = privateKeyToAccount(
    (process.env.PRIVATE_KEY as `0x${string}`) || '0x0000000000000000000000000000000000000000000000000000000000000001'
  );
  const walletClient = createWalletClient({ chain: conxTestnet, transport, account });

  const delegatorAddress = account.address;

  const stakingContractRead = getContract({ ...staking, client: publicClient });
  const stakingContractWrite = getContract({ ...staking, client: walletClient });
  const bech32Contract = getContract({ ...bech32, client: publicClient });

  console.log(`Delegator Address: ${delegatorAddress}\n`);

  try {
    // Query available validators
    console.log('Querying available validators...');
    const pageRequest = {
      key: new Uint8Array(),
      offset: 0n,
      limit: 10n,
      countTotal: false,
      reverse: false,
    };

    const validatorsResponse = await stakingContractRead.read.validators([
      'BOND_STATUS_BONDED',
      pageRequest,
    ]);

    if (!validatorsResponse.validators?.length) {
      console.log('No validators found');
      return;
    }

    const validator = validatorsResponse.validators[0];
    // Convert validator operator address (hex) to Bech32 for delegation calls
    const operatorAddressBech32 = await bech32Contract.read.encode([
      'xplavaloper',
      hexToBytes(validator.operatorAddress as `0x${string}`),
    ]);

    console.log(`Selected Validator: ${operatorAddressBech32}`);
    console.log(`Validator Status: ${validator.status}\n`);

    // Check initial delegation
    console.log('Checking initial delegation...');
    try {
      const initialDelegation = await stakingContractRead.read.delegation([
        delegatorAddress,
        operatorAddressBech32,
      ]);
      console.log(`Initial Delegation: ${initialDelegation.balance.amount} ${initialDelegation.balance.denom}\n`);
    } catch {
      console.log('No initial delegation found\n');
    }

    // Execute delegation
    const delegationAmount = 1000000000000000000n; // 1 XPLA
    console.log(`Delegating ${delegationAmount} wei to validator...`);

    const hash = await stakingContractWrite.write.delegate([
      delegatorAddress,
      operatorAddressBech32,
      delegationAmount,
    ]);
    console.log(`Transaction Hash: ${hash}`);

    const receipt = await publicClient.waitForTransactionReceipt({ hash });
    console.log(`Transaction confirmed in block: ${receipt.blockNumber}\n`);

    const updatedDelegation = await stakingContractRead.read.delegation([
      delegatorAddress,
      operatorAddressBech32,
    ]);
    console.log(`Updated Delegation: ${updatedDelegation.balance.amount} ${updatedDelegation.balance.denom}`);
    console.log(`Delegation Shares: ${updatedDelegation.shares}`);
    console.log('✅ Delegation completed successfully!');
  } catch (error) {
    console.error('❌ Staking operation failed:', error);
  }
}

stakingPrecompileExample().catch(console.error);
```

## Running the Example

Set `PRIVATE_KEY` in your environment, then run:

```bash
npx tsx examples/staking-precompile.ts
```

## Expected Output

```
=== Staking Precompile Example ===

Delegator Address: 0x123...

Querying available validators...
Selected Validator: xplavaloper1abc...
Validator Status: BOND_STATUS_BONDED

Checking initial delegation...
No initial delegation found

Delegating 1000000000000000000 wei to validator...
Transaction Hash: 0xdef...
Transaction confirmed in block: 12346

Updated Delegation: 1000000000000000000 axpla
Delegation Shares: 1000000000000000000
✅ Delegation completed successfully!
```

## Key Features

- **Validator Discovery**: Query bonded validators with `validators`
- **Bech32 Encoding**: Use the bech32 precompile to convert operator address to Bech32 for delegation
- **Delegation**: Delegate with `delegate(delegator, validatorBech32, amount)`
- **Delegation Queries**: Check delegation with `delegation(delegator, validatorBech32)`
- **viem + @xpla/evm**: Uses `staking` and `bech32` from `@xpla/evm/precompiles` with viem clients

## Common Operations

- Query bonded validators and paginate with `pageRequest`
- Convert validator operator address (hex) to Bech32 via the bech32 precompile `encode`
- Delegate and query delegation amounts/shares

## Related Documentation

- [About @xpla/evm](/develop/develop/tools/evm/about-evm/)
- [Address Conversion](/develop/develop/tools/evm/address-conversion/)
- [Staking Precompile Reference](/develop/develop/smart-contract-guide/precompile/staking/)
- [Staking Module Documentation](/develop/develop/core-modules/staking/)
