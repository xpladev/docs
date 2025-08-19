---
title: Common example
weight: 20
type: docs
---

# Common Examples

This page provides practical examples of how to use `@xpla/evm` to interact with XPLA Chain's precompile contracts. These examples demonstrate real-world usage patterns for different precompile contract modules.

## Prerequisites

Before running these examples, make sure you have:

- Node.js v18 or later
- Access to XPLA Chain testnet (Cube)
- Test tokens in your wallet

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example 1: Address Conversion

This example demonstrates how to convert between Cosmos Bech32 addresses and EVM hex addresses.

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

## Example 2: Bank Precompile Contract

This example demonstrates how to use the Bank precompile contract for token transfers.

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

## Example 3: Staking Precompile Contract

This example demonstrates how to use the Staking precompile contract for validator delegation.

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

## Example 4: Wasm Precompile Contract

This example demonstrates how to use the Wasm precompile contract to interact with CosmWasm smart contracts.

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

## Example 5: Governance Precompile Contract

This example demonstrates how to use the Governance precompile contract to query proposals.

```javascript
// examples/governance-precompile.js
import { JsonRpcProvider, Wallet } from 'ethers';
import { createPrecompileGov } from '@xpla/evm/precompiles';

async function governancePrecompileExample() {
  console.log('=== Governance Precompile Example ===\n');

  const RPC_URL = 'https://cube-evm-rpc.xpla.dev';
  const provider = new JsonRpcProvider(RPC_URL);

  // Create governance contract instance
  const govContract = createPrecompileGov(provider);

  try {
    // Query governance proposals
    console.log('Querying governance proposals...');
    const proposalsResponse = await govContract.getProposals(
      0, // proposal status (0 = all)
      '0x0000000000000000000000000000000000000000', // voter address
      '0x0000000000000000000000000000000000000000', // depositor address
      {
        key: new Uint8Array(),
        offset: 0n,
        limit: 10n,
        countTotal: false,
        reverse: false
      }
    );
    
    console.log(`Found ${proposalsResponse.proposals.length} proposals\n`);
    
    if (proposalsResponse.proposals.length > 0) {
      const proposal = proposalsResponse.proposals[0];
      console.log('Sample Proposal:');
      console.log(`- ID: ${proposal.id}`);
      console.log(`- Status: ${proposal.status}`);
      console.log(`- Title: ${proposal.title}`);
      console.log(`- Summary: ${proposal.summary}`);
      console.log(`- Proposer: ${proposal.proposer}`);
      console.log(`- Submit Time: ${new Date(Number(proposal.submitTime) * 1000).toLocaleString()}\n`);
    }

    // Query governance parameters
    console.log('Querying governance parameters...');
    const params = await govContract.getParams();
    
    console.log('Governance Parameters:');
    console.log(`- Voting Period: ${params.votingPeriod} seconds`);
    console.log(`- Min Deposit: ${params.minDeposit.map(d => `${d.amount} ${d.denom}`).join(', ')}`);
    console.log(`- Quorum: ${params.quorum}`);
    console.log(`- Threshold: ${params.threshold}`);
    console.log(`- Veto Threshold: ${params.vetoThreshold}\n`);

    // Query constitution
    console.log('Querying constitution...');
    const constitution = await govContract.getConstitution();
    console.log(`Constitution: ${constitution}`);
    
    console.log('✅ Governance queries completed successfully!');
    
  } catch (error) {
    console.error('❌ Governance operation failed:', error);
  }
}

governancePrecompileExample().catch(console.error);
```

## Running the Examples

1. **Ensure you have access to XPLA Chain testnet**:
   ```bash
   # Examples will connect to https://cube-evm-rpc.xpla.dev
   ```

2. **Run individual examples**:
   ```bash
   node examples/address-conversion.js
   node examples/bank-precompile.js
   node examples/staking-precompile.js
   node examples/wasm-precompile.js
   node examples/governance-precompile.js
   ```

## Key Features Demonstrated

- **Address Conversion**: Converting between Cosmos Bech32 and EVM hex addresses
- **Token Transfers**: Using Bank precompile for token transfers
- **Validator Delegation**: Using Staking precompile for delegation
- **Smart Contract Interaction**: Using Wasm precompile for CosmWasm contracts
- **Governance Queries**: Using Governance precompile for proposal queries

## Error Handling

All examples include proper error handling to gracefully handle common issues such as:
- Network connectivity problems
- Insufficient funds
- Invalid contract addresses
- Transaction failures

## Next Steps

After running these examples, you can:
- Modify the examples to work with your specific use cases
- Integrate them into your dApp
- Explore other precompile contract functions
- Check out the [@xpla/contracts](../contracts/common-example) package for Solidity examples

