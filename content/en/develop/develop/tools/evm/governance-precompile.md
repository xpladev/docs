---
title: Governance Precompile
weight: 60
type: docs
---

# Governance Precompile Example

This example demonstrates how to use the Governance precompile contract to query proposals and governance parameters with viem and `@xpla/evm`.

## Prerequisites

Before running this example, make sure you have:

- Node.js v18 or later
- Access to CONX Chain testnet (Cube)

## Setup

Install the required dependencies:

```bash
pnpm add @xpla/evm viem
```

Or with npm: `npm install @xpla/evm viem`

## Example Code

```typescript
// examples/governance-precompile.ts
import { gov } from '@xpla/evm/precompiles';
import { conxTestnet } from '@xpla/evm';
import { getContract, createPublicClient, http } from 'viem';

async function governancePrecompileExample() {
  console.log('=== Governance Precompile Example ===\n');

  const publicClient = createPublicClient({
    chain: conxTestnet,
    transport: http(),
  });

  const govContract = getContract({ ...gov, client: publicClient });

  try {
    // Query governance proposals
    console.log('Querying governance proposals...');
    const proposalsResponse = await govContract.read.getProposals([
      0, // proposal status (0 = all)
      '0x0000000000000000000000000000000000000000' as `0x${string}`, // voter
      '0x0000000000000000000000000000000000000000' as `0x${string}`, // depositor
      {
        key: new Uint8Array(),
        offset: 0n,
        limit: 10n,
        countTotal: false,
        reverse: false,
      },
    ]);

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
    const params = await govContract.read.getParams();
    console.log('Governance Parameters:');
    console.log(`- Voting Period: ${params.votingPeriod} seconds`);
    console.log(`- Min Deposit: ${params.minDeposit.map((d) => `${d.amount} ${d.denom}`).join(', ')}`);
    console.log(`- Quorum: ${params.quorum}`);
    console.log(`- Threshold: ${params.threshold}`);
    console.log(`- Veto Threshold: ${params.vetoThreshold}\n`);

    // Query constitution
    console.log('Querying constitution...');
    const constitution = await govContract.read.getConstitution();
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
npx tsx examples/governance-precompile.ts
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

- **Proposal Queries**: Retrieve proposals via `getProposals` with pagination
- **Parameter Queries**: Get governance parameters with `getParams`
- **Constitution**: Query the chain constitution with `getConstitution`
- **viem + @xpla/evm**: Uses `getContract` with `gov` from `@xpla/evm/precompiles`

## Proposal Statuses

Common proposal statuses:

- `PROPOSAL_STATUS_UNSPECIFIED`
- `PROPOSAL_STATUS_DEPOSIT_PERIOD`
- `PROPOSAL_STATUS_VOTING_PERIOD`
- `PROPOSAL_STATUS_PASSED`
- `PROPOSAL_STATUS_REJECTED`
- `PROPOSAL_STATUS_FAILED`

## Related Documentation

- [About @xpla/evm](/develop/develop/tools/evm/about-evm/)
- [Governance Precompile Reference](/develop/develop/smart-contract-guide/precompile/gov/)
- [Governance Module Documentation](/develop/develop/core-modules/gov/)
