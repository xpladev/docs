---
title: Overview
weight: 10
type: docs
---

# Precompiled Contracts

Precompiled contracts provide direct, gas-efficient access to native Cosmos SDK functionality from within the EVM. Build powerful hybrid applications that leverage the best of both worlds. This page provides an overview of the available precompiled contracts, each with detailed documentation on its usage, Solidity interface, and ABI.

<Warning>
  **Critical: Understanding Token Decimals in Precompiles**

  All Cosmos EVM precompile contracts use the **native chain's decimal precision**, not Ethereum's standard 18 decimals (wei).

  **Why this matters:**
  - Cosmos chains typically use 6 decimals, while Ethereum uses 18
  - Passing values while assuming conventional Ethereum exponents could cause significant miscalculations
  - Example: On a 6-decimal chain, passing `1000000000000000000` (1 token in wei) will be interpreted as **1 trillion tokens**

  **Before using any precompile:**
  1. Check your chain's native token decimal precision
  2. Convert amounts to match the native token's format
  3. Never assume 18-decimal precision

  ```solidity
  // WRONG: Assuming Ethereum's 18 decimals
  uint256 amount = 1 ether; // 1000000000000000000
  staking.delegate(validator, amount); // Delegates 1 trillion tokens!

  // CORRECT: Using chain's native 6 decimals
  uint256 amount = 1000000; // 1 token with 6 decimals
  staking.delegate(validator, amount); // Delegates 1 token
  ```
</Warning>

## Available Precompiles

Access native Cosmos SDK features directly from Solidity:

| Module | Contract Address | Owner |
|--------|------------------|-------|
| auth | `0x1000000000000000000000000000000000000005` | xpladev |
| bank | `0x1000000000000000000000000000000000000001` | xpladev |
| wasm | `0x1000000000000000000000000000000000000004` | xpladev |
| p256 | `0x0000000000000000000000000000000000000100` | - |
| bech32 | `0x0000000000000000000000000000000000000400` | cosmos |
| staking | `0x0000000000000000000000000000000000000800` | cosmos |
| distribution | `0x0000000000000000000000000000000000000801` | cosmos |
| gov | `0x0000000000000000000000000000000000000805` | cosmos |
| slashing | `0x0000000000000000000000000000000000000806` | cosmos |