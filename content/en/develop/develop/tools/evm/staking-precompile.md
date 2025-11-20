---
title: Staking Precompile
weight: 40
type: docs
---

# Staking Precompile Example

This example demonstrates how to use the Staking precompile contract for validator delegation.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)
- Test tokens in your wallet for delegation

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example Code

```javascript
// examples/staking-precompile.js
import { JsonRpcProvider, Wallet, getBytes } from 'ethers';
import { createPrecompileStaking } from '@xpla/evm/precompiles';
import { toBech32 } from '@interchainjs/encoding';
import * as bip39 from 'bip39';

async function stakingPrecompileExample() {
  console.log('=== Staking Precompile Example ===\n');

  const RPC_URL = 'https://cube-evm-rpc.xpla.dev';
  const provider = new JsonRpcProvider(RPC_URL);

  // Generate wallet
  const mnemonic = bip39.generateMnemonic();
  const wallet = Wallet.fromPhrase(mnemonic);
  const delegatorAddress = wallet.address;
  
  console.log(`Delegator Address: ${delegatorAddress}\n`);

  // Create staking contract instance
  const stakingContract = createPrecompileStaking(provider);
  const stakingContractWithSigner = stakingContract.connect(wallet.connect(provider));

  try {
    // Query available validators
    console.log('Querying available validators...');
    const pageRequest = {
      key: new Uint8Array(),
      offset: 0n,
      limit: 10n,
      countTotal: false,
      reverse: false
    };
    
    const validatorsResponse = await stakingContract.validators('BOND_STATUS_BONDED', pageRequest);
    
    if (validatorsResponse.validators.length === 0) {
      console.log('No validators found');
      return;
    }
    
    const validator = validatorsResponse.validators[0];
    const operatorAddressHex = validator.operatorAddress;
    const operatorAddressBech32 = toBech32('xplavaloper', getBytes(operatorAddressHex));
    
    console.log(`Selected Validator: ${operatorAddressBech32}`);
    console.log(`Validator Status: ${validator.status}\n`);

    // Check initial delegation
    console.log('Checking initial delegation...');
    try {
      const initialDelegation = await stakingContract.delegation(delegatorAddress, operatorAddressBech32);
      console.log(`Initial Delegation: ${initialDelegation.balance.amount} ${initialDelegation.balance.denom}\n`);
    } catch (error) {
      console.log('No initial delegation found\n');
    }

    // Execute delegation
    const delegationAmount = 1000000000000000000n; // 1 XPLA
    console.log(`Delegating ${delegationAmount} wei to validator...`);
    
    const txResponse = await stakingContractWithSigner.delegate(
      delegatorAddress, 
      operatorAddressBech32, 
      delegationAmount
    );
    
    console.log(`Transaction Hash: ${txResponse.hash}`);
    
    // Wait for transaction confirmation
    const txReceipt = await txResponse.wait();
    console.log(`Transaction confirmed in block: ${txReceipt.blockNumber}\n`);

    // Check updated delegation
    const updatedDelegation = await stakingContract.delegation(delegatorAddress, operatorAddressBech32);
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

```bash
node examples/staking-precompile.js
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

- **Validator Discovery**: Query available validators on the network
- **Delegation Management**: Delegate tokens to validators for staking rewards
- **Delegation Queries**: Check current delegation amounts and shares
- **Transaction Handling**: Proper async/await pattern for staking operations

## Common Operations

This example demonstrates:
- Querying bonded validators
- Converting between EVM and Bech32 validator addresses
- Delegating tokens to a validator
- Checking delegation status

## Related Documentation

- [Staking Precompile Reference](/develop/develop/smart-contract-guide/precompile/staking/)
- [Staking Module Documentation](/develop/develop/core-modules/staking/)
