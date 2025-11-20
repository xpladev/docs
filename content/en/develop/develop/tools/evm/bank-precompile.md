---
title: Bank Precompile
weight: 30
type: docs
---

# Bank Precompile Example

This example demonstrates how to use the Bank precompile contract for token transfers.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)
- Test tokens in your wallet

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example Code

```javascript
// examples/bank-precompile.js
import { JsonRpcProvider, Wallet } from 'ethers';
import { createPrecompileBank } from '@xpla/evm/precompiles';
import { EthSecp256k1HDWallet } from '@xpla/xpla/wallets/ethSecp256k1hd';
import * as bip39 from 'bip39';

async function bankPrecompileExample() {
  console.log('=== Bank Precompile Example ===\n');

  const RPC_URL = 'https://cube-evm-rpc.xpla.dev';
  const provider = new JsonRpcProvider(RPC_URL);

  // Generate wallets
  const mnemonic1 = bip39.generateMnemonic();
  const mnemonic2 = bip39.generateMnemonic();
  
  const wallet1 = Wallet.fromPhrase(mnemonic1);
  const wallet2 = Wallet.fromPhrase(mnemonic2);
  
  const senderAddress = wallet1.address;
  const receiverAddress = wallet2.address;
  
  console.log(`Sender: ${senderAddress}`);
  console.log(`Receiver: ${receiverAddress}\n`);

  // Create bank contract instance
  const bankContract = createPrecompileBank(provider);
  const bankContractWithSigner = bankContract.connect(wallet1.connect(provider));

  // Check initial balances
  const initialBalance = await provider.getBalance(senderAddress);
  console.log(`Initial Sender Balance: ${initialBalance} wei`);

  const receiverBalance = await provider.getBalance(receiverAddress);
  console.log(`Initial Receiver Balance: ${receiverBalance} wei\n`);

  // Prepare transfer amount
  const transferAmount = {
    denom: 'axpla',
    amount: '1000000000000000000' // 1 XPLA
  };

  try {
    // Execute transfer
    console.log('Executing transfer...');
    const txResponse = await bankContractWithSigner.send(
      senderAddress, 
      receiverAddress, 
      [transferAmount]
    );
    
    console.log(`Transaction Hash: ${txResponse.hash}`);
    
    // Wait for transaction confirmation
    const txReceipt = await txResponse.wait();
    console.log(`Transaction confirmed in block: ${txReceipt.blockNumber}\n`);

    // Check updated balances
    const newSenderBalance = await provider.getBalance(senderAddress);
    const newReceiverBalance = await provider.getBalance(receiverAddress);
    
    console.log(`New Sender Balance: ${newSenderBalance} wei`);
    console.log(`New Receiver Balance: ${newReceiverBalance} wei`);
    
    console.log('✅ Bank transfer completed successfully!');
    
  } catch (error) {
    console.error('❌ Transfer failed:', error);
  }
}

bankPrecompileExample().catch(console.error);
```

## Running the Example

```bash
node examples/bank-precompile.js
```

## Expected Output

```
=== Bank Precompile Example ===

Sender: 0x123...
Receiver: 0x456...

Initial Sender Balance: 1000000000000000000 wei
Initial Receiver Balance: 0 wei

Executing transfer...
Transaction Hash: 0xabc...
Transaction confirmed in block: 12345

New Sender Balance: 999000000000000000 wei
New Receiver Balance: 1000000000000000000 wei
✅ Bank transfer completed successfully!
```

## Key Features

- **Token Transfer**: Transfer native tokens using the Bank precompile
- **Balance Queries**: Check account balances before and after transfers
- **Transaction Handling**: Proper async/await pattern for transaction execution
- **Error Handling**: Graceful handling of transfer failures

## Related Documentation

- [Bank Precompile Reference](/develop/develop/smart-contract-guide/precompile/bank/)
- [Bank Module Documentation](/develop/develop/core-modules/bank/)
