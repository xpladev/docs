---
title: Bank Precompile
weight: 30
type: docs
---

# Bank Precompile Example

This example demonstrates how to use the Bank precompile contract for token transfers with viem and `@xpla/evm`.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)
- Test tokens in your wallet

## Setup

Install the required dependencies:

```bash
pnpm add @xpla/evm viem
```

Or with npm: `npm install @xpla/evm viem`

## Example Code

```typescript
// examples/bank-precompile.ts
import { bank } from '@xpla/evm/precompiles';
import { conxTestnet } from '@xpla/evm';
import { createPublicClient, createWalletClient, getContract, http, parseEther } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';

async function bankPrecompileExample() {
  console.log('=== Bank Precompile Example ===\n');

  const transport = http();

  const publicClient = createPublicClient({ chain: conxTestnet, transport });
  const account = privateKeyToAccount(
    (process.env.PRIVATE_KEY as `0x${string}`) || '0x0000000000000000000000000000000000000000000000000000000000000001'
  );
  const walletClient = createWalletClient({ chain: conxTestnet, transport, account });

  const senderAddress = account.address;
  const receiverAddress = (process.env.RECEIVER_ADDRESS as `0x${string}`) || '0x0000000000000000000000000000000000000002';

  console.log(`Sender: ${senderAddress}`);
  console.log(`Receiver: ${receiverAddress}\n`);

  const bankContractRead = getContract({ ...bank, client: publicClient });
  const bankContractWrite = getContract({ ...bank, client: walletClient });

  // Check initial balances (native / EVM balance)
  const initialSenderBalance = await publicClient.getBalance({ address: senderAddress });
  const initialReceiverBalance = await publicClient.getBalance({ address: receiverAddress });
  console.log(`Initial Sender Balance: ${initialSenderBalance} wei`);
  console.log(`Initial Receiver Balance: ${initialReceiverBalance} wei\n`);

  // Bank module balance (axpla) via precompile
  try {
    const senderAxpla = await bankContractRead.read.balance([senderAddress, 'axpla']);
    console.log(`Sender axpla balance: ${senderAxpla}\n`);
  } catch (e) {
    console.log('Balance query skipped or failed\n');
  }

  // Prepare transfer: send(sender, receiver, coins)
  const transferAmount = parseEther('1'); // 1 XPLA
  const coins = [{ denom: 'axpla', amount: transferAmount.toString() }];

  try {
    console.log('Executing transfer...');
    const hash = await bankContractWrite.write.send([senderAddress, receiverAddress, coins]);
    console.log(`Transaction Hash: ${hash}`);

    const receipt = await publicClient.waitForTransactionReceipt({ hash });
    console.log(`Transaction confirmed in block: ${receipt.blockNumber}\n`);

    const newSenderBalance = await publicClient.getBalance({ address: senderAddress });
    const newReceiverBalance = await publicClient.getBalance({ address: receiverAddress });
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

Set `PRIVATE_KEY` (and optionally `RECEIVER_ADDRESS`), then run:

```bash
npx tsx examples/bank-precompile.ts
```

Or with ts-node:

```bash
npx ts-node examples/bank-precompile.ts
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

- **Token Transfer**: Transfer native tokens using the Bank precompile `send`
- **Balance Queries**: Check EVM balance and Bank (axpla) balance via precompile
- **viem + @xpla/evm**: Uses `getContract` with `bank` from `@xpla/evm/precompiles` and viem clients

## Related Documentation

- [About @xpla/evm](/develop/develop/tools/evm/about-evm/)
- [Bank Precompile Reference](/develop/develop/smart-contract-guide/precompile/bank/)
- [Bank Module Documentation](/develop/develop/core-modules/bank/)
