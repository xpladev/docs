---
title: Message Authorization
weight: 90
type: docs
---

1. `test1` creates MsgGrantAuthorization message to grant MsgSend authorization to grantee `test2`.
2. `test2` creates MsgExecAuthorized message to send `2000000000000000000axpla` from the `test1` account to the `test3` account.

```ts
import {
  LCDClient,
  MnemonicKey,
  Wallet,
  MsgGrantAuthorization,
  SendAuthorization,
  MsgSend,
  Int,
  MsgExecAuthorized,
  Coins,
} from "@xpla/xpla.js";

function grant(
  granter: Wallet,
  grantee: string,
  spendLimit: Coins.Input,
  duration: Int
) {
  const msgs = [
    new MsgGrantAuthorization(
      granter.key.accAddress,
      grantee,
      new SendAuthorization(spendLimit),
      duration
    ),
  ];

  return granter.createAndSignTx({ msgs });
}

function sendAuthorized(
  granter: Wallet,
  grantee: Wallet,
  to: string,
  amount: Coins.Input
) {
  const msgs = [
    new MsgExecAuthorized(grantee.key.accAddress, [
      new MsgSend(
        granter.key.accAddress, // From test1
        to,
        amount
      ),
    ]),
  ];

  return grantee.createAndSignTx({ msgs });
}

async function main() {
  const gasPrices = await(
    await fetch("https://cube-fcd.xpla.dev/v1/txs/gas_prices", {
      redirect: "follow",
    })
  ).json();
  const gasPricesCoins = new Coins(gasPrices);
  const client = new LCDClient({
    URL: "https://cube-lcd.xpla.dev",
    chainID: "cube_47-5",
    gasPrices: gasPricesCoins,
    gasAdjustment: "1.5",
  });

  // Granter (xpla1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v)
  const granter = client.wallet(
    new MnemonicKey({
      mnemonic:
        "notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius",
    })
  );

  // Grantee (xpla17lmam6zguazs5q5u6z5mmx76uj63gldnse2pdp)
  const grantee = client.wallet(
    new MnemonicKey({
      mnemonic:
        "quality vacuum heart guard buzz spike sight swarm shove special gym robust assume sudden deposit grid alcohol choice devote leader tilt noodle tide penalty",
    })
  );

  // MsgGrantAuthorization
  await grant(
    granter,
    grantee.key.accAddress,
    // Set enough spend limit since it will be decreased upon every MsgSend transactions
    "1000000000000axpla",
    // expire after 100 year
    new Int(86400 * 365 * 100 * 1000000000)
  )
    .then((tx) => client.tx.broadcast(tx))
    .then(console.info)
    .catch((err) => {
      if (err.response) {
        console.error(err.response.data);
      } else {
        console.error(err.message);
      }
    });

  // MsgExecAuthorized of MsgSend
  await sendAuthorized(
    granter,
    grantee,
    // Test3
    "xpla1757tkx08n0cqrw7p86ny9lnxsqeth0wgp0em95",
    "2000000000000axpla"
  )
    .then((tx) => client.tx.broadcast(tx))
    .then(console.info)
    .catch((err) => {
      if (err.response) {
        // unauthorized: authorization not found: failed to execute message; message index: 0: failed to simulate tx
        // happenes when there's not enough amount of granted amount of token

        // insufficient funds: insufficient account funds; ...
        // happenes when granter does not have enough amount of token
        console.error(err.response.data);
      } else {
        console.error(err.message);
      }
    });
}

main().catch(console.error);

```
