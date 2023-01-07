---
title: Fees
weight: 30
---

On XPLA Chain, there is only one fee that applies to all transactions: the gas fee.

## Gas
[Gas]({{< ref "glossary#fees" >}}) is a small computational fee that covers the cost of processing a transaction. Gas is estimated and added to every transaction in XPLA Vault. Any transaction that does not contain enough gas will not process.
Gas on XPLA Chain works differently than it works on other decentralized platforms:

- Validators can set their own minimum gas fees.
- Most transactions will estimate more than the minimum amount of gas, ensuring the transaction gets completed.
- Unused gas is not refunded.
- Transactions are not queued based on gas amounts, but in the order received.

For an in-depth explanation of how gas fees are calculated, visit the [xplad reference]({{< ref "using-xplad#fees" >}}) page.

To view current gas rates in your browser, visit the [gas rates](https://dimension-fcd.xpla.dev/v1/txs/gas_prices) page.

Every block, gas fees are sent to the reward pool and dispersed to validators who distribute them to delegators in the form of staking rewards.
