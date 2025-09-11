---
title: Wasm Precompile
weight: 50
type: docs
---

# Wasm Precompile Example

This example demonstrates how to use the Wasm precompile contract to interact with CosmWasm smart contracts.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to XPLA Chain testnet (Cube)
- A deployed CosmWasm contract (or use the example contract address)

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example Code

```javascript
// examples/wasm-precompile.js
import { JsonRpcProvider, Wallet } from 'ethers';
import { createPrecompileWasm } from '@xpla/evm/precompiles';
import { fromBech32 } from '@interchainjs/encoding';
import { hexlify } from 'ethers';
import * as fs from 'fs';
import * as path from 'path';

async function wasmPrecompileExample() {
  console.log('=== Wasm Precompile Example ===\n');

  const RPC_URL = 'https://cube-evm-rpc.xpla.dev';
  const provider = new JsonRpcProvider(RPC_URL);

  // Generate wallet
  const mnemonic = 'abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about';
  const wallet = Wallet.fromPhrase(mnemonic);
  const userAddress = wallet.address;
  
  console.log(`User Address: ${userAddress}\n`);

  // Create wasm contract instance
  const wasmContract = createPrecompileWasm(provider);
  const wasmContractWithSigner = wasmContract.connect(wallet.connect(provider));

  try {
    // For this example, we'll assume a contract is already deployed
    // In a real scenario, you would first deploy the contract using CosmWasm
    
    // Example contract address
    const contractAddressBech32 = 'xpla1qw97zu0xazljpckxzf7wc5g3hevp7weefn40fw8z09ejzm2wz6ms7qverx';
    
    // Convert to EVM address format (last 20 bytes)
    const { data: contractAddressHex } = fromBech32(contractAddressBech32);
    const contractAddressHexString = hexlify(contractAddressHex.slice(12));
    
    console.log(`Contract Bech32: ${contractAddressBech32}`);
    console.log(`Contract EVM: ${contractAddressHexString}\n`);

    // Query contract state
    console.log('Querying contract state...');
    const queryData = new Uint8Array(Buffer.from('{"get_count": {}}'));
    
    const queryResponse = await wasmContract.smartContractState(contractAddressHexString, queryData);
    
    // Parse response
    const responseHex = queryResponse.startsWith('0x') ? queryResponse.slice(2) : queryResponse;
    const responseBytes = new Uint8Array(Buffer.from(responseHex, 'hex'));
    const responseText = new TextDecoder().decode(responseBytes);
    const responseData = JSON.parse(responseText);
    
    console.log(`Current Count: ${responseData.count}\n`);

    // Execute contract function
    console.log('Executing increment function...');
    const executeMsg = new Uint8Array(Buffer.from('{"increment": {}}'));
    
    const txResponse = await wasmContractWithSigner.executeContract(
      userAddress, 
      contractAddressHexString, 
      executeMsg, 
      []
    );
    
    console.log(`Transaction Hash: ${txResponse.hash}`);
    
    // Wait for transaction confirmation
    const txReceipt = await txResponse.wait();
    console.log(`Transaction confirmed in block: ${txReceipt.blockNumber}\n`);

    // Query updated state
    console.log('Querying updated contract state...');
    const updatedQueryResponse = await wasmContract.smartContractState(contractAddressHexString, queryData);
    
    const updatedResponseHex = updatedQueryResponse.startsWith('0x') ? updatedQueryResponse.slice(2) : updatedQueryResponse;
    const updatedResponseBytes = new Uint8Array(Buffer.from(updatedResponseHex, 'hex'));
    const updatedResponseText = new TextDecoder().decode(updatedResponseBytes);
    const updatedResponseData = JSON.parse(updatedResponseText);
    
    console.log(`Updated Count: ${updatedResponseData.count}`);
    console.log('✅ Wasm contract interaction completed successfully!');
    
  } catch (error) {
    console.error('❌ Wasm operation failed:', error);
  }
}

wasmPrecompileExample().catch(console.error);
```

## Running the Example

```bash
node examples/wasm-precompile.js
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

- **Contract Address Conversion**: Convert between Bech32 and EVM address formats
- **State Queries**: Query CosmWasm contract state using JSON messages
- **Contract Execution**: Execute contract functions with proper message encoding
- **Response Parsing**: Decode hex responses back to readable JSON

## Common Operations

This example demonstrates:
- Converting CosmWasm contract addresses to EVM format
- Querying contract state with structured messages
- Executing contract functions
- Parsing binary responses from contract calls

## Message Format

CosmWasm contracts expect messages in JSON format:
- **Query Messages**: `{"get_count": {}}`, `{"get_balance": {"address": "..."}}`
- **Execute Messages**: `{"increment": {}}`, `{"transfer": {"to": "...", "amount": "..."}}`

## Related Documentation

- [Wasm Precompile Reference](/develop/develop/smart-contract-guide/precompile/wasm/)
- [Wasm Module Documentation](/develop/develop/core-modules/wasm/)
- [CosmWasm Documentation](https://docs.cosmwasm.com/)
