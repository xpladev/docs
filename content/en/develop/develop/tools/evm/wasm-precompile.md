---
title: Wasm Precompile
weight: 50
type: docs
---

# Wasm Precompile Example

This example demonstrates how to use the Wasm precompile contract to interact with CosmWasm smart contracts using viem and `@xpla/evm`.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)
- A deployed CosmWasm contract (or use the example contract address)

## Setup

Install the required dependencies:

```bash
pnpm add @xpla/evm viem
```

Or with npm: `npm install @xpla/evm viem`

## Example Code

```typescript
// examples/wasm-precompile.ts
import { wasm, bech32 } from '@xpla/evm/precompiles';
import { conxTestnet } from '@xpla/evm';
import { getContract, createPublicClient, createWalletClient, http, bytesToHex } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';

async function wasmPrecompileExample() {
  console.log('=== Wasm Precompile Example ===\n');

  const transport = http();
  const publicClient = createPublicClient({ chain: conxTestnet, transport });
  const account = privateKeyToAccount(
    (process.env.PRIVATE_KEY as `0x${string}`) || '0x0000000000000000000000000000000000000000000000000000000000000001'
  );
  const walletClient = createWalletClient({ chain: conxTestnet, transport, account });

  const wasmContractRead = getContract({ ...wasm, client: publicClient });
  const wasmContractWrite = getContract({ ...wasm, client: walletClient });
  const bech32Contract = getContract({ ...bech32, client: publicClient });

  const userAddress = account.address;
  console.log(`User Address: ${userAddress}\n`);

  try {
    // Example contract address (Bech32)
    const contractAddressBech32 = 'xpla1qw97zu0xazljpckxzf7wc5g3hevp7weefn40fw8z09ejzm2wz6ms7qverx';

    // Convert Bech32 to EVM address (last 20 bytes of decoded data)
    const decoded = await bech32Contract.read.decode([contractAddressBech32]);
    const contractAddressHexString = bytesToHex(decoded.slice(-20)) as `0x${string}`;

    console.log(`Contract Bech32: ${contractAddressBech32}`);
    console.log(`Contract EVM: ${contractAddressHexString}\n`);

    // Query contract state
    console.log('Querying contract state...');
    const queryData = new TextEncoder().encode('{"get_count": {}}');
    const queryResponse = await wasmContractRead.read.smartContractState([
      contractAddressHexString,
      queryData,
    ]);

    const responseHex = queryResponse.startsWith('0x') ? queryResponse.slice(2) : queryResponse;
    const responseBytes = new Uint8Array(Buffer.from(responseHex, 'hex'));
    const responseText = new TextDecoder().decode(responseBytes);
    const responseData = JSON.parse(responseText);
    console.log(`Current Count: ${responseData.count}\n`);

    // Execute contract function
    console.log('Executing increment function...');
    const executeMsg = new TextEncoder().encode('{"increment": {}}');
    const hash = await wasmContractWrite.write.executeContract([
      userAddress,
      contractAddressHexString,
      executeMsg,
      [],
    ]);
    console.log(`Transaction Hash: ${hash}`);

    const receipt = await publicClient.waitForTransactionReceipt({ hash });
    console.log(`Transaction confirmed in block: ${receipt.blockNumber}\n`);

    // Query updated state
    console.log('Querying updated contract state...');
    const updatedQueryResponse = await wasmContractRead.read.smartContractState([
      contractAddressHexString,
      queryData,
    ]);
    const updatedResponseHex = updatedQueryResponse.startsWith('0x') ? updatedQueryResponse.slice(2) : updatedQueryResponse;
    const updatedResponseBytes = new Uint8Array(Buffer.from(updatedResponseHex, 'hex'));
    const updatedResponseData = JSON.parse(new TextDecoder().decode(updatedResponseBytes));
    console.log(`Updated Count: ${updatedResponseData.count}`);
    console.log('✅ Wasm contract interaction completed successfully!');
  } catch (error) {
    console.error('❌ Wasm operation failed:', error);
  }
}

wasmPrecompileExample().catch(console.error);
```

## Running the Example

Set `PRIVATE_KEY` if needed, then run:

```bash
npx tsx examples/wasm-precompile.ts
```

## Expected Output

```
=== Wasm Precompile Example ===

User Address: 0x123...

Contract Bech32: xpla1qw97zu0xazljpckxzf7wc5g3hevp7weefn40fw8z09ejzm2wz6ms7qverx
Contract EVM: 0xEC5111BE581F3B394CEAF4B8E27973216D4E16B7

Querying contract state...
Current Count: 5

Executing increment function...
Transaction Hash: 0xabc...
Transaction confirmed in block: 12347

Querying updated contract state...
Updated Count: 6
✅ Wasm contract interaction completed successfully!
```

## Key Features

- **Address Conversion**: Use the bech32 precompile `decode` to convert Bech32 contract address to EVM (last 20 bytes)
- **State Queries**: Query CosmWasm state with `smartContractState(contractAddress, queryData)`
- **Contract Execution**: Execute with `executeContract(sender, contractAddress, msg, funds)`
- **viem + @xpla/evm**: Uses `wasm` and `bech32` from `@xpla/evm/precompiles` with viem clients

## Message Format

CosmWasm contracts expect JSON messages:

- **Query**: `{"get_count": {}}`, `{"get_balance": {"address": "..."}}`
- **Execute**: `{"increment": {}}`, `{"transfer": {"to": "...", "amount": "..."}}`

## Related Documentation

- [About @xpla/evm](/develop/develop/tools/evm/about-evm/)
- [Address Conversion](/develop/develop/tools/evm/address-conversion/)
- [Wasm Precompile Reference](/develop/develop/smart-contract-guide/precompile/wasm/)
- [Wasm Module Documentation](/develop/develop/core-modules/wasm/)
- [CosmWasm Documentation](https://docs.cosmwasm.com/)
