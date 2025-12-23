---
title: Governance Precompile
weight: 60
type: docs
---

# Governance Precompile Example

This example demonstrates how to use the Governance precompile contract to query proposals and governance parameters.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)

## Setup

Install the required dependencies:

```bash
npm install @xpla/evm @xpla/xpla @interchainjs/cosmos @interchainjs/utils ethers bip39
```

## Example Code

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

## Running the Example

```bash
node examples/governance-precompile.js
```

## Expected Output

```
=== Governance Precompile Example ===

Querying governance proposals...
Found 3 proposals

Sample Proposal:
- ID: 1
- Status: PROPOSAL_STATUS_PASSED
- Title: Parameter Change Proposal
- Summary: Update staking parameters
- Proposer: xpla1abc...
- Submit Time: 2024-01-15 10:30:00

Governance Parameters:
- Voting Period: 604800 seconds
- Min Deposit: 10000000 axpla
- Quorum: 0.334000000000000000
- Threshold: 0.500000000000000000
- Veto Threshold: 0.334000000000000000

Constitution: This chain operates under...
✅ Governance queries completed successfully!
```

## Key Features

- **Proposal Queries**: Retrieve active and historical governance proposals
- **Parameter Queries**: Get current governance parameters like voting periods
- **Constitution Access**: Query the chain's constitution document
- **Filtering Options**: Filter proposals by status, voter, or depositor

## Common Operations

This example demonstrates:
- Querying all governance proposals with pagination
- Extracting proposal details (ID, status, title, summary, proposer)
- Retrieving governance parameters (voting period, deposit requirements, thresholds)
- Accessing the chain constitution

## Proposal Statuses

Common proposal statuses you might encounter:
- `PROPOSAL_STATUS_UNSPECIFIED`
- `PROPOSAL_STATUS_DEPOSIT_PERIOD`
- `PROPOSAL_STATUS_VOTING_PERIOD`
- `PROPOSAL_STATUS_PASSED`
- `PROPOSAL_STATUS_REJECTED`
- `PROPOSAL_STATUS_FAILED`

## Related Documentation

- [Governance Precompile Reference](/develop/develop/smart-contract-guide/precompile/gov/)
- [Governance Module Documentation](/develop/develop/core-modules/gov/)
