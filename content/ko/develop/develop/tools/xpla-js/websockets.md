---
title: Websockets
weight: 160
type: docs
---

## `WebSocketClient`

xpla.js comes with `WebSocketClient`, which abstracts a subscription to Tendermint RPC's WebSocket endpoint. This requires access to a XPLA Chain node's RPC server, which may require privileged access as it exposes functions that can kill a node's operation.

```ts
import { WebSocketClient } from '@xplad/xpla.js';

const wsclient = new WebSocketClient('ws://localhost:26657/websocket');

const xpla = new LocalXpla();

let count = 0;
wsclient.subscribe('NewBlock', {}, (_) => {
  console.log(count);
  count += 1;

  if (count === 3) {
    wsclient.destroy();
  }
});

// send tracker
wsclient.subscribe('Tx', { 'message.action': '/cosmos.bank.v1beta1.MsgSend' }, data => {
  console.log('Send occured!');
  console.log(data.value);
});

wsclient.start();
```

### Supported Events

You can subscribe to the following recognized Tendermint events:

  - `CompleteProposal`
  - `Evidence`
  - `Lock`
  - `NewBlock`
  - `NewBlockHeader`
  - `NewRound`
  - `NewRoundStep`
  - `Polka`
  - `Relock`
  - `TimeoutPropose`
  - `TimeoutWait`
  - `Tx`
  - `Unlock`
  - `ValidatorSetUpdates`
  - `ValidBlock`
  - `Vote`

### Query

Use the following syntax to specify the Tendermint query:

```ts
type TendermintQueryOperand = string | number | Date;

interface TendermintQuery {
  [k: string]:
    | TendermintQueryOperand
    | ['>', number | Date]
    | ['<', number | Date]
    | ['<=', number | Date]
    | ['>=', number | Date]
    | ['CONTAINS', string]
    | ['EXISTS'];
}
```

The following shows an example of how to construct a `TendermintQuery` and use it for a subscription:

```ts
const tmQuery = {
  "message.action": "/cosmos.bank.v1beta1.MsgSend",
  "tx.timestamp": [">=", new Date()],
  "store_code.abc": ["EXISTS"],
  "abc.xyz": ["CONTAINS", "xpla1..."]
};

wsclient.subscribe('Tx', tmQuery, (data) => {
  // do something with data ...
});
```

The resultant query will be:

`tm.event='Tx' AND message.action='/cosmos.bank.v1beta1.MsgSend' tx.timestamp >= 2020-12-12 AND store_code.abc EXISTS AND abc.xyz CONTAINS 'xpla1...'`

### `subscribeTx`

It is a common use case to subscribe to transactions with a Tendermint query, such as listening for when specific addresses send or receive funds, or when specific events are triggered from within smart contracts. However, it is hard to extract data because the transaction result is encoded in Base64 Amino encoding. If you use `subscribeTx`, xpla.js will automatically inject the `txhash` into the resultant data value so you can more easily look up the transaction to decode it using `LCDClient`.

```ts
// swap tracker
wsclient.subscribeTx({ 'message.action': 'swap' }, async data => {
  console.log('Swap occured!');
  const txInfo = await xpla.tx.txInfo(data.value.TxResult.txhash);
  if (txInfo.logs) {
    console.log(txInfo.logs[0].eventsByType);
  }
});
```
