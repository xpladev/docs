---
weight: 60
title: IBC Transfers
---

XPLA Chain has full IBC transfer capabilities through both xpla.js and XPLA Vault. Although IBC functionality is not readily exposed through XPLA Vault’s front-end, it can be fully incorporated into any dApp. It is up to a dApp’s front end to initiate IBC transfers.

## MsgTransfer

xpla.js exports a `MsgTransfer` class that can be used to construct IBC transfers.

```js
new MsgTransfer(
  "transfer", // IBC port
  "channel-1", // Outbound channel (Osmosis)
  new Coin("axpla", "1000000000000000000"), // 1 XPLA
  "xpla1cvw8sundusurqajhurpcfk7yvuzlh92cvkpy28", // Source Address on XPLA Chain
  "osmo1cl4qw7u35uf77l4scjtv0qej8ycevu4mrdpvmg", // Destination address on Osmosis
  undefined, // Timeout block height (optional)
  (Date.now() + 60 * 1000) * 1e6 // Timeout timestamp (in nanoseconds) relative to the current block timestamp.
);
```

## Supported Channels

Channels are defined when a relayer is set up between XPLA Chain and an external chain. For each new connected chain the channel ID is incremented.

You can use [Map of Zones](https://mapofzones.com/zone?period=24&source=dimension_37-1&tableOrderBy=success&tableOrderSort=desc&testnet=false) to find the available channels and their IDs.

## Derive Cosmos Chain Addresses from a XPLA Chain Address

Cosmos SDK based blockchains use bech32 to encode the public key for display. Assuming the same private key is used on multiple Cosmos SDK chains it is possible to decode a XPLA Chain address and generate the corresponding public key on another chain.

Here's a quick example using the [bech32](https://github.com/bitcoinjs/bech32) JavaScript library:

```js
import { bech32 } from 'bech32';

const xplaAddress = 'xpla1cvw8sundusurqajhurpcfk7yvuzlh92cvkpy28';
const decodedAddress = bech32.decode(xplaAddress);
const osmosisAddress = bech32.encode('osmo', decodedAddress.words);
```

## Complete Example

The following example demonstrates how to send 1 XPLA from XPLA Chain to the Osmosis blockchain.

```JS
import {
  LCDClient,
  MnemonicKey,
  MsgTransfer,
  Coin,
} from "@xpla/xpla.js";

// const lcd = new LCDClient(...);

const mk = new MnemonicKey({
  mnemonic: 'satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn',
});

const wallet = lcd.wallet(mk);

// Send 1 XPLA to the Osmosis blockchain.
const transfer = new MsgTransfer(
 'transfer',
 'channel-1',
  new Coin('axpla', '1000000000000000000'),
  'xpla1cvw8sundusurqajhurpcfk7yvuzlh92cvkpy28',
  'osmo1cl4qw7u35uf77l4scjtv0qej8ycevu4mrdpvmg',
  undefined,
  (Date.now() + 60 * 1000) * 1e6,
);

const tx = await wallet.createAndSignTx({ msgs: [transfer] });
const result = await lcd.tx.broadcast(tx);

console.log(result);
```

Instructions for initializing LCDClient can be found [here]({{< ref "common-examples#configuring-lcdclient" >}}).
