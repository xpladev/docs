---
title: Send Message
weight: 120
type: docs
---

```js
import {
  LCDClient,
  MnemonicKey,
  MsgMultiSend,
  StdTx,
  Account,
} from "@xpla/xpla.js";

const {
  TESTNET_LCD_URL = "https://cube-lcd.xpla.dev",
  TESTNET_API_GAS_PRICES = "https://cube-fcd.xpla.dev/v1/txs/gas_prices",
  TESTNET_CHAIN_ID = "cube_47-5",
} = process.env;

async function main() {
  const gasPrices = await(
    await fetch(TESTNET_API_GAS_PRICES, {
      redirect: "follow",
    })
  ).json();
  const gasPricesCoins = new Coins(gasPrices);
  const client = new LCDClient({
    URL: TESTNET_LCD_URL,
    chainID: TESTNET_CHAIN_ID,
    gasPrices: gasPricesCoins,
    gasAdjustment: 1.5,
  });

  const keys = {
    test1: new MnemonicKey({
      mnemonic:
        "notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius",
    }),
    test2: new MnemonicKey({
      mnemonic:
        "quality vacuum heart guard buzz spike sight swarm shove special gym robust assume sudden deposit grid alcohol choice devote leader tilt noodle tide penalty",
    }),
    test3: new MnemonicKey({
      mnemonic:
        "symbol force gallery make bulk round subway violin worry mixture penalty kingdom boring survey tool fringe patrol sausage hard admit remember broken alien absorb",
    }),
  };

  const wallet = client.wallet(keys.test1);
  const output = new MsgMultiSend.Output(
    "xpla199vw7724lzkwz6lf2hsx04lrxfkz09tg8dlp6r",
    "1000000axpla"
  );

  const tx = await wallet.createTx({
    msgs: [
      new MsgMultiSend(
        [
          new MsgMultiSend.Input(keys.test1.accAddress, "1000000axpla"),
          new MsgMultiSend.Input(keys.test2.accAddress, "1000000axpla"),
          new MsgMultiSend.Input(keys.test3.accAddress, "1000000axpla"),
        ],
        [output, output, output]
      ),
    ],
  });

  const signatures = await Promise.all(
    ["test1", "test2", "test3"].map(async (keyName) => {
      const key = keys[keyName] as MnemonicKey;
      const acc = (await client.auth.accountInfo(key.accAddress)) as Account;

      tx.account_number = acc.account_number;
      tx.sequence = acc.sequence;

      return key.createSignature(tx);
    })
  );

  const stdTx = new StdTx(tx.msgs, tx.fee, signatures, tx.memo);

  await client.tx
    .broadcastSync(stdTx)
    .then((result) => {
      console.log(result);
    })
    .catch((err) => {
      console.error(err.message);
    });
}

main().catch(console.error);
```
