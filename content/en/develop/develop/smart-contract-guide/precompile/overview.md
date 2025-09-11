---
title: Overview
weight: 10
type: docs
---

# Precompiled Contracts

Precompiled contracts provide direct, gas-efficient access to native Cosmos SDK functionality from within the EVM. Build powerful hybrid applications that leverage the best of both worlds. This page provides an overview of the available precompiled contracts, each with detailed documentation on its usage, Solidity interface, and ABI.

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