---
title: Get Started with xpla.js
weight: 10
type: docs
---

xpla.js seeks to provide a compatible way to work with the XPLA Chain within JavaScript runtimes, such as Node.js and the browser. xpla.js enables the following functions:

- Deserializing blockchain data into JavaScript objects with native data types and methods
- Serializing objects back into a blockchain-compatible format
- Providing access to the `xplad` node API (LCD) from a JavaScript-based interface
- Providing additional utilities, such as hash functions and key-signing algorithms

## About This Tutorial

This is an in-depth guide on how to use the `xpla.js` SDK.

In this tutorial, you'll learn how to:

1. [Set up your project](#1-set-up-your-project)
2. [Set up a XPLA Chain LCD (light client daemon)](#2-initialize-the-lcd)
3. [Create and connect a wallet](#3-create-a-cube-testnet-wallet)
4. [Find a contract address](#4-find-a-contract-address)
5. [Query a contract](#5-query-a-contract-and-set-up-the-transaction)
6. [Create, sign, and broadcast a transaction](#6-broadcast-the-transaction)

By the end of this guide, you'll be able to execute a token swap from your application using xpla.js.

## Prerequisites

- [npm and node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- XPLA extension vault

## 1. Set up Your Project

1. Create a new directory for your project:

   ```sh
   mkdir my-xpla-js-project
   ```

2. Enter your new project directory:

   ```sh
   cd my-xpla-js-project
   ```

3. Next, initialize npm, install the `xpla.js` package, and create an `index.js` file to house the code:

   ```sh
   npm init -y
   npm install @xpla/xpla.js
   touch index.js
   ```

4. Open the `package.json` file in a code editor and add `"type": "module",`.

   ```json
   {
     // ...
     "type": "module",
     // ...
   }
   ```

## 2. Initialize the LCD

XPLA Chain’s LCD or Light Client Daemon allows users to connect to the blockchain, make queries, create wallets, and submit transactions. It's the main workhorse behind `xpla.js`.

1. Install a fetch library to make HTTP requests and dynamically pull recommended gas prices. You can use the one wallet referenced below or choose your favorite.

   ```sh
   npm install --save isomorphic-fetch
   ```

2. Open your `index.js` file in a code editor and input the following to initialize the LCD:

   ```ts
   import fetch from "isomorphic-fetch";
   import { Coins, LCDClient } from "@xpla/xpla.js";
   const gasPrices = await fetch(
     "https://cube-fcd.xpla.dev/v1/txs/gas_prices", { redirect: 'follow' }
   );
   const gasPricesJson = await gasPrices.json();
   const gasPricesCoins = new Coins(gasPricesJson);
   const lcd = new LCDClient({
     URL: "https://cube-lcd.xpla.dev", // Use "https://dimension-lcd.xpla.dev" for prod
     chainID: "cube_47-5", // Use "dimension_37-1" for production
     gasPrices: gasPricesCoins,
     gasAdjustment: "1.5", // Increase gas price slightly so transactions go through smoothly.
   });
   ```

   {{< alert >}}
   **Note**

   The previous code block shows how to connect to the cube testnet. To connect to the dimension_37-1 mainnet for production, use “`https://dimension-lcd.xpla.dev`”.
   You will also need to change the `chainID` from `"cube_47-5"` to `"dimension_37-1"`.
   {{< /alert >}}

## 3. Create a Cube Testnet Wallet

1. You'll need a wallet to sign and submit transactions. [Create a new wallet]({{< ref "/learn/learn/xpla-vault/vaults/extension-vault" >}}) using the XPLA extension vault. Be sure to save your mnemonic key!

2. After creating your wallet, you’ll need to set it to use the testnet. Click the gear icon in the extension and change the network from `mainnet` to `testnet`.

3. Add the following code to your `index.js` file and input your mnemonic key:

   ```ts
   import { MnemonicKey } from "@xpla/xpla.js";
   const mk = new MnemonicKey({
     mnemonic: "-> Input your 24-word mnemonic key here <-",
   });
   const wallet = lcd.wallet(mk);
   ```

   {{< alert context="warning" >}}
   **Warning**

   Although this tutorial has you input your mnemonic directly, this practice should be avoided in production.
   For security reasons, it's better to store your mnemonic key data in your environment by using `process.env.SECRET_MNEMONIC` or `process.env.SECRET_PRIV_KEY`. This practice is more secure than a hard-coded string.
   {{< /alert >}}

4. Request testnet funds for your wallet by navigating to the [XPLA faucet](https://faucet.xpla.io) and inputting your wallet address. You'll need these funds to perform swaps and pay for gas fees. Once the funds are in your wallet, you’re ready to move on to the next step.

## 4. Find a Contract Address

To find the contract address for a specific Dezswap pair, visit [Dezswap](https://app.dezswap.io/)

## 5. Query a Dezswap Contract and Set up the Transaction

Before you can perform a swap, you’ll need a belief price. You can calculate the belief price of one token by querying simulation to the pool. The belief price +/- the `max_spread` is the range of possible acceptable prices for this swap.

1. Add the following code to your `index.js` file. Make sure the contract address is correct.

   ```ts
   const contract = "<POOL_CONTRACT_ADDRESS>"; // A Dezswap pair contract address on cube_47-5(CW20 token <> XPLA)
   const offerAmount = 1000000000000000000
   const { asset_decimals: assetDecimals } = await lcd.wasm.contractQuery(contract, { "pair": {} }); // Query
   const { return_amount: returnAmount } = await lcd.wasm.contractQuery( // Query
     contract,
     {
       "simulation": {
         "offer_asset": {
           "info" : {
             "native_token": {
               "denom": "axpla"
             }
           },
           "amount": `${offerAmount}`
         }
       }
     });
   const beliefPrice = ((offerAmount / Math.pow(10, assetDecimals[1])) / (returnAmount / Math.pow(10, assetDecimals[0]))).toFixed(assetDecimals[1]); // Calculate belief price using proportion of simulated asset amounts
   ```

2. Next, generate a message to broadcast to the network:

   ```ts
   import { MsgExecuteContract } from "@xpla/xpla.js";
   const txMsg = new MsgExecuteContract(
     wallet.key.accAddress,
     contract,
     {
       swap: {
         max_spread: "0.005",
         offer_asset: {
           info: {
             native_token: {
               denom: "axpla",
             },
           },
           amount: `${offerAmount}`,
         },
         belief_price: beliefPrice,
       },
     },
     new Coins({ axpla: `${offerAmount}` })
   );
   ```

## 6. Broadcast the Transaction

1. Add the following code to `index.js` to create, sign, and broadcast the transaction. It's important to specify `axpla` as the fee denomination because XPLA is the only denomination the faucet sends.

   ```ts
   const tx = await wallet.createAndSignTx({
     msgs: [txMsg],
     feeDenoms: ["axpla"],
   });
   const result = await lcd.tx.broadcast(tx);
   console.log(result);
   ```

2. Run the code in your terminal:

   ```sh
   node index.js
   ```

If successful, you'll see a log of the successful transaction and some new tokens in your wallet.

And that's it! You can find other pool addresses [here](https://app.dezswap.io/) to call other swaps. Be sure to use the correct testnet or mainnet contract address.

## More Examples

View the [Common examples]({{< ref "common-examples" >}}) section for more information on using xpla.js.
