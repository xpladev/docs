---
title: Contracts
weight: 20
type: docs
---

Smart contracts are an advanced feature of XPLA Vault. If you’re using XPLA Vault for the first time, follow the [XPLA Vault tutorial]({{< ref "xpla-vault" >}}).

## Prerequisites

Compile a contract locally and create a `.wasm` file.

## Upload

Deploy a contract by uploading your `.wasm` file to XPLA Vault.

1. Open XPLA Vault and connect your wallet. Click **Contracts**.

2. Click **Upload**.

3. Upload your `.wasm` file and enter your password.

4. Click **Submit**.

Your contract is now uploaded, and you received a contract code ID.

## Instantiate

Use **Instantiate** to initialize your contract after uploading.

1. Click **Create**.

2. Enter your contract code ID, `InitMsg JSON`, name, and description.

3. Confirm the fee amounts and enter your password. Click **Submit**.

Your contract is now initialized.

## Query

Use **Query** to find out contract values. Querying does not cost anything.

1. Click **Query** located under your contract address.

2. Enter your `Input`. Click **Submit**.

XPLA Vault will show your query result.

## Execute

Use **Execute** to use the contract. Interacting will spend gas.

1. Click **Execute** located under your contract address.

2. Enter your `Msg`.

3. Confirm the fee amounts and click **Submit**.
