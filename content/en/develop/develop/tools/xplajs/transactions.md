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

### Create a Signer

You will first want to create a signer which you can use to sign transactions.

```ts
const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgSend), Network.Testnet.rpc);
```

### Create Messages
```ts
import { MessageComposer } from "@xpla/xplajs/cosmos/bank/v1beta1/tx.registry";
import { MsgSend } from "@xpla/xplajs/cosmos/bank/v1beta1/tx";

const msgSend = MsgSend.fromPartial({
    fromAddress: await signer.getAddress(),
    toAddress: "xpla1888g76xr3phk7qkfknnn8hvxyzfj0e2vuh4jmw",
    amount: [{denom: "axpla", amount: "1000000000000000000"}]
})
const { send } = MessageComposer.fromPartial;
const msg = await send(msgSend);
```

### Create and Sign Transaction

```ts
const {tx} = await signer.sign({messages: [msg]})
```

### Broadcast Transaction

```ts
const res = await signer.broadcast(tx)
```

The default broadcast mode is `block`, which waits until the transaction has been included in a block. This will give you the most information about the transaction, including events and errors while processing.

You can also use `sync` or `async` broadcast modes.

#### Sync
```ts
const res = await signer.broadcast(tx, {checkTx: true})
```

#### async
```ts
const res = await signer.broadcast(tx, {checkTx: false})
```

#### block
```ts
const res = await signer.broadcast(tx, {checkTx: true, deliverTx: true})
```

### Check Events

If you broadcasted the transaction with `block`, you can get the events emitted by your transaction.

```ts
const res = await signer.broadcast(tx, {checkTx: true, deliverTx: true})
console.log(res.events.find(e => e.type === "transfer"))
```
