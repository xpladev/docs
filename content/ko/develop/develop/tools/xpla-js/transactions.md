---
title: Transactions
weight: 140
type: docs
---

This document explains how to influence the blockchain's state by broadcasting transactions.

Transactions include:

- a list of messages
- an optional memo
- a fee
- a signature from a key

The messages included in a transaction contain the information that will be routed to a proper message handler in the node, which in turn parses the inputs and determines the next state of the blockchain.

## Create Transactions

### Create a Wallet

You will first want to create a wallet which you can use to sign transactions.

```ts
import { MnemonicKey, LCDClient } from "@xpla/xpla.js";

const mk = new MnemonicKey();
const xpla = new LCDClient({
  URL: "https://cube-lcd.xpla.dev",
  chainID: "cube_47-5",
});
const wallet = xpla.wallet(mk);
```

### Create Messages

```ts
import { MsgSend } from "@xpla/xpla.js";

const send = new MsgSend(wallet.key.accAddress, "<random-xpla-address>", {
  axpla: 1000,
});
```

### Create and Sign Transaction

```ts
const tx = await wallet.createAndSignTx({
  msgs: [send],
  memo: "Hello",
});
```

### Broadcast Transaction

```ts
const txResult = await xpla.tx.broadcast(tx);
```

The default broadcast mode is `block`, which waits until the transaction has been included in a block. This will give you the most information about the transaction, including events and errors while processing.

You can also use `sync` or `async` broadcast modes.

```ts
// const syncTxResult = await xpla.tx.broadcastSync(tx);
// const asyncTxResult = await xpla.tx.broadcastAsync(tx);
```

### Check Events

If you broadcasted the transaction with `block`, you can get the events emitted by your transaction.

```ts
import { isTxError } from "@xpla/xpla.js";

const txResult = xpla.tx.broadcast(tx);

if (isTxError(txResult)) {
  throw new Error(
    `encountered an error while running the transaction: ${txResult.code} ${txResult.codespace}`
  );
}

// check for events from the first message
txResult.logs[0].eventsByType.store_code;
```
