---
title: Wallets
weight: 150
type: docs
---

## Create a Wallet

Use `LCDClient.wallet()` to create a `Wallet` from a `Key`.

```ts
import { LCDClient, MnemonicKey } from "@xpla/xpla.js";

const xpla = new LCDClient({
  URL: "https://dimension-lcd.xpla.dev",
  chainId: "dimension_37-1",
});

const mk = new MnemonicKey();
const wallet = xpla.wallet(mk);
```

In the above example, a `MnemonicKey` was specified for the wallet, but any type of `Key` implementation can be used instead.

## Usage

### Getting Account Number and Sequence

A wallet is connected to the XPLA Chain and can poll the values of an account's account number and sequence directly:

```ts
console.log(await wallet.accountNumber());
console.log(await wallet.sequence());
```

### Creating Transactions

A wallet makes it easy to create a transaction by automatically fetching the account number and sequence from the blockchain. The fee parameter is optional -- if you don't include it, xpla.js will automatically use your LCD's fee estimation settings to simulate the transaction within the node and include the resultant fee in your transaction.

```ts
const msgs = [ ... ]; // list of messages
const fee = Fee(...); // optional fee

const unsignedTx = await wallet.createTx({
  msgs,
  // fee, (optional)
  memo: 'this is optional'
});
```

You can then sign the transaction with the wallet's key, which will create a `StdTx` which you can later broadcast:

```ts
const tx = wallet.key.signTx(unsignedTx);
```

You can also use the convenience function `Wallet.createAndSignTx()`, which automatically generates a signed transaction to be broadcast:

```ts
const tx = await wallet.createAndSignTx({
  msgs,
  fee,
  memo: "this is optional",
});
```
