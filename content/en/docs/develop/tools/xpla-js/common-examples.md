---
title: Common Examples
weight: 20
---

Use the following common examples to learn how to use xpla.js. If this is your first time using xpla.js, use the [xpla.js installation guide]({{< ref "get-started-with-xpla-js" >}}).

{{< alert >}}
**Tip**

If you are new to XPLA Chain and don't know where to start, visit the [getting started guide]({{< ref "get-started" >}}).
{{< /alert >}}

## Configuring LCDClient

The following code example shows how to initialize the LCDClient. The rest of the examples assume you initialized it by using this example or similar code.

```ts
import fetch from "isomorphic-fetch";
import { MsgSend, MnemonicKey, Coins, LCDClient } from "@xpla/xpla.js";

// Fetch gas prices and convert to `Coin` format.
const gasPrices = await (
  await fetch("https://cube-fcd.xpla.dev/v1/txs/gas_prices", { redirect: 'follow' })
).json();
const gasPricesCoins = new Coins(gasPrices);

const lcd = new LCDClient({
  URL: "https://cube-lcd.xpla.dev/",
  chainID: "cube_47-5",
  gasPrices: gasPricesCoins,
  gasAdjustment: "1.5",
  gas: 10000000,
});
```

## Get Wallet Balance (native tokens)

```js
// Replace with address to check.
const address = "xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
const [balance] = await lcd.bank.balance(address);
console.log(balance.toData());
```

Example response:

```js
[{ denom: "axpla", amount: "5030884" }];
```

## Get Wallet Balance (CW20 tokens)

```js
// TEST on cube_47-5
const tokenAddress = "xpla1747mad58h0w4y589y3sk84r5efqdev9q4r02pc";
const walletAddress = "xpla1f44ddca9awepv2rnudztguq5rmrran2m20zzd6";
const response = await lcd.wasm.contractQuery(tokenAddress, {
  balance: { address: walletAddress },
});

console.log(response);
```

Example response:

```js
{
  balance: "70258667";
}
```

## Get Transaction Status

```js
// Replace with TX hash to lookup.
const hash = "CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B";
const txInfo = await lcd.tx.txInfo(hash);
console.log(txInfo);
```

Example response (modified for readability):

```js
TxInfo {
  height: 8276372,
  txhash: 'CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B',
  raw_log: '[]',
  logs: [
    TxLog {
      msg_index: 0,
      log: '',
      events: [Array],
      eventsByType: [Object]
    }
  ],
  gas_wanted: 177808,
  gas_used: 128827,
  tx: Tx {},
  timestamp: '2022-03-17T18:34:06Z',
  code: 0,
  codespace: ''
}
```

## Get Link to Transaction

```js
const getTransactionLink = (hash, chainID) =>
  `https://explorer.xpla.io/${chainID}/tx/${hash}`;
const hash = "CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B";

console.log(getTransactionLink(hash, "cube_47-5"));
```

Example response:

```
https://explorer.xpla.io/cube_47-5/tx/CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B
```

## Get Link to Wallet Address

```js
const getWalletLink = (address, chainID) =>
  `https://explorer.xpla.io/${chainID}/address/${address}`;
const address = "xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
console.log(getWalletLink(address, "cube_47-5"));
```

Example response:

```
https://explorer.xpla.io/cube_47-5/address/xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Sending Native Tokens

The following code example shows how to send native tokens:

```ts
import { LCDClient, MnemonicKey, MsgSend } from "@xpla/xpla.js";

// const lcd = new LCDClient(...);

const mk = new MnemonicKey({
  mnemonic:
    "satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn",
});

const wallet = lcd.wallet(mk);

// Transfer 1 XPLA.
const send = new MsgSend(
  wallet.key.accAddress,
  "xpla1dcegyrekltswvyy0xy69ydgxn9x8x32zdtapd8",
  { axpla: "1000000000000000000" }
);

const tx = await wallet.createAndSignTx({ msgs: [send] });
const result = await lcd.tx.broadcast(tx);

console.log(result);
```

## Sending CW20 tokens

The following code example shows how to send CW20 tokens:

```ts
import {
  LCDClient,
  MnemonicKey,
  MsgExecuteContract,
} from "@xpla/xpla.js";

// const lcd = new LCDClient(...);

const mk = new MnemonicKey({
  mnemonic:
    "satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn",
});

const wallet = lcd.wallet(mk);

// TEST on cube_47-5
const tokenAddress = "xpla1747mad58h0w4y589y3sk84r5efqdev9q4r02pc";

// Transfer 1 TEST.
const cw20Send = new MsgExecuteContract(wallet.key.accAddress, tokenAddress, {
  transfer: {
    amount: "1000000",
    recipient: wallet.key.accAddress,
  },
});

const tx = await wallet.createAndSignTx({ msgs: [cw20Send] });
const result = await lcd.tx.broadcast(tx);

console.log(result);
```

## Swapping a Native XPLA Chain Asset for a CW20 Token Using Dezswap

The following code example shows how to swap a native asset for CW20 using Dezswap.

Run this example on mainnet.

```ts
import {
  MsgExecuteContract,
  MnemonicKey,
  Coins,
  LCDClient,
} from "@xpla/xpla.js";

// const lcd = new LCDClient(...);

const mk = new MnemonicKey({
  mnemonic:
    "satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn",
});

const wallet = lcd.wallet(mk);

// XPLA <> CTXT
const pool = "xpla1sdzaas0068n42xk8ndm6959gpu6n09tajmeuq7vak8t9qt5jrp6szltsnk";
const xplaAmount = 1000000000000000000

// Fetch the decimal of each asset in the pool and simulation result with `xplaAmount`.
const { asset_decimals: assetDecimals } = await lcd.wasm.contractQuery(pool, { "pair": {} });
const { return_amount: returnAmount } = await lcd.wasm.contractQuery( // Query
  pool,
  {
    "simulation": {
      "offer_asset": {
        "info" : {
          "native_token": {
            "denom": "axpla"
          }
        },
        "amount": `${xplaAmount}`
      }
    }
  });

// Calculate belief price using pool balances.
const beliefPrice = ((xplaAmount / Math.pow(10, assetDecimals[0])) / (returnAmount / Math.pow(10, assetDecimals[1]))).toFixed(assetDecimals[0]);

// Swap 1 XPLA to CTXT with 1% slippage tolerance.
const swapMsg = new MsgExecuteContract(
  wallet.key.accAddress,
  pool,
  {
    swap: {
      max_spread: "0.01",
      offer_asset: {
        info: {
          native_token: {
            denom: "axpla",
          },
        },
        amount: `${xplaAmount}`,
      },
      belief_price: beliefPrice,
    },
  },
  new Coins({ axpla: `${xplaAmount}` })
);

const tx = await wallet.createAndSignTx({ msgs: [swapMsg] });
const result = await lcd.tx.broadcast(tx);

console.log(result);
```

## Decoding Protobuf-encoded Messages

The following code example shows how to decode messages that have been encoded using Protobuf:

```ts
import { LCDClient, Tx } from "@xpla/xpla.js";

// const lcd = new LCDClient(...);

const blockData = await lcd.tendermint.blockInfo(5923213);

const txInfos = blockData.block.data.txs.map((tx) =>
  Tx.unpackAny({ value: Buffer.from(tx, "base64") })
);

// Find messages where a contract was initialized.
const initMessages = txInfos
  .map((tx) => tx.body.messages)
  .flat()
  .find((i) => i.constructor.name === "MsgInstantiateContract");

console.log(initMessages);
```

## Validate a XPLA Chain Address

The following code example shows how to do a basic verification on a XPLA Chain address.

This is a basic version of the verification, it does not require external libraries as it performs a simple comparison with a regex string. It could give false positives since it doesn't verify the checksum of the address.

```ts
// basic address validation (no library required)
function isValid(address) {
  // check the string format:
  // - starts with "xpla1"
  // - length == 44 ("xpla1" + 38)
  // - contains only numbers and lower case letters
  return /(xpla1[a-z0-9]{38})/g.test(address);
}

console.log(isValid("xpla1dcegyrekltswvyy0xy69ydgxn9x8x32zdtapd8")); // true
console.log(isValid("xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx")); // true (even if this doesn't have a valid checksum)
console.log(isValid("cosmos1zz22dfpvw3zqpeyhvhmx944a588fgcalw744ts")); // false
console.log(isValid("random string")); // false
```

This is a more advanced verification that requires the bech32 library which is used to verify the checksum.

```ts
import { bech32 } from "bech32";

// advanced address validation, it verify also the bech32 checksum
function isValid(address) {
  try {
    const { prefix: decodedPrefix } = bech32.decode(address); // throw error if checksum is invalid
    // verify address prefix
    return decodedPrefix === "xpla";
  } catch {
    // invalid checksum
    return false;
  }
}

console.log(isValid("xpla1dcegyrekltswvyy0xy69ydgxn9x8x32zdtapd8")); // true
console.log(isValid("xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx")); // false
console.log(isValid("cosmos1zz22dfpvw3zqpeyhvhmx944a588fgcalw744ts")); // false
console.log(isValid("random string")); // false
```

## Avoid Status 500: Timed Out Waiting for TX to be Included in a Block

Occasionally the broadcast function of xpla.js throws the error `Status 500: timed out waiting for tx to be included in a block`, even if transaction will be confirmed onchain after a few seconds.

This happens because the libraries use by default the `broadcast-mode = block`, with this mode the LCD to which you are broadcasting the transaction sends an http response to your request only when the transaction has been included in a block, but if the chain is overloaded the confirmation may take too long and trigger a timeout in the LCD.

To solve this problem it is recommended to use the `broadcast-mode = sync` and then iterate a request to the LCD with the txhash to understand when it has been included in a block.

This is an example to do it in JavaScript:

```ts
// sign the tx
wallet
  .createAndSignTx(YOUR_TX_HERE)
  // use broadcastSync() instead of broadcast()
  .then((tx) => xpla.tx.broadcastSync(tx))
  .then(async (result) => {
    // TODO: use a for or add a timeout to prevent infinite loops
    while (true) {
      // query txhash
      const data = await xpla.tx.txInfo(result.txhash).catch(() => {});
      // if hash is onchain return data
      if (data) return data;
      // else wait 250ms and then repeat
      await new Promise((resolve) => setTimeout(resolve, 250));
    }
  })
  .then((result) => {
    // this will be executed when the tx has been included into a block
    console.log(result);
  });
```
