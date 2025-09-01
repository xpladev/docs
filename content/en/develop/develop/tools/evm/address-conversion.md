---
title: Address Conversion
weight: 20
type: docs
---

# Address Conversion Example

This example demonstrates how to convert between Cosmos Bech32 addresses and EVM hex addresses.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to XPLA Chain testnet (Cube)

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example Code

```javascript
// examples/address-conversion.js
import { fromBech32, toBech32 } from '@interchainjs/encoding';
import { getAddress, hexlify } from 'ethers';
import { EthSecp256k1HDWallet } from '@xpla/xpla/wallets/ethSecp256k1hd';
import * as bip39 from 'bip39';

async function addressConversionExample() {
  console.log('=== Address Conversion Example ===\n');

  // Generate a new wallet
  const mnemonic = bip39.generateMnemonic();
  const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, [{
      prefix: 'xpla',
      hdPath: "m/44'/60'/0'/0/0",
    }]
  );

  const accounts = await wallet.getAccounts();
  const cosmosAddress = accounts[0].address;
  
  // Convert to EVM address
  const { data: hexByteAddress } = fromBech32(cosmosAddress);
  const evmAddress = getAddress(hexlify(hexByteAddress));
  
  console.log(`Cosmos Address: ${cosmosAddress}`);
  console.log(`EVM Address: ${evmAddress}`);
  
  // Convert back to Cosmos address
  const convertedCosmosAddress = toBech32('xpla', hexByteAddress);
  console.log(`Converted Back: ${convertedCosmosAddress}`);
  
  // Verify conversion
  console.log(`Conversion Valid: ${cosmosAddress === convertedCosmosAddress}`);
}

addressConversionExample().catch(console.error);
```

## Running the Example

```bash
node examples/address-conversion.js
```

## Expected Output

```
=== Address Conversion Example ===

Cosmos Address: xpla1abc...
EVM Address: 0x123...
Converted Back: xpla1abc...
Conversion Valid: true
```

## Key Concepts

- **Bech32 to EVM**: Use `fromBech32()` to extract hex bytes, then convert to EVM address
- **EVM to Bech32**: Use `toBech32()` with the appropriate prefix ('xpla' for XPLA Chain)
- **Address Validation**: Both addresses represent the same account on different interfaces
