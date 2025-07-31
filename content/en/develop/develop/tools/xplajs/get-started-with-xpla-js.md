---
title: Get Started with xplajs
weight: 10
type: docs
---

xplajs seeks to provide a compatible way to work with the XPLA Chain within JavaScript runtimes, such as Node.js and the browser. xplajs enables the following functions:

- Deserializing blockchain data into JavaScript objects with native data types and methods
- Serializing objects back into a blockchain-compatible format
- Providing access to the XPLA Chain RPC from a JavaScript-based interface
- Providing additional utilities, such as hash functions and key-signing algorithms

## About This Tutorial

This is an in-depth guide on how to use the `xplajs` SDK.

In this tutorial, you'll learn how to:

1. [Set up your project](#1-set-up-your-project)
2. [Initialize the RPC](#2-initialize-the-rpc)
3. [Create and connect a wallet](#3-create-a-cube-testnet-wallet)
4. [Find a contract address](#4-find-a-contract-address)
5. [Query a contract](#5-query-a-contract-and-set-up-the-transaction)
6. [Broadcast the Transaction](#6-broadcast-the-transaction)

By the end of this guide, you'll be able to execute a token swap from your application using xplajs.

## Prerequisites

- [yarn and node.js](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- XPLA extension vault

## 1. Set up Your Project

1. Create a new directory for your project:

   ```sh
   mkdir my-xplajs-project
   ```

2. Enter your new project directory:

   ```sh
   cd my-xplajs-project
   ```

3. Next, initialize yarn, install the `xplajs` package, and create an `index.js` file to house the code:

   ```sh
   corepack enable
   yarn init -y
   yarn add @xpla/xplajs @xpla/xpla
   touch index.ts
   ```

   The `@xpla/xplajs` package provides the core functionality for interacting with the XPLA Chain, while `@xpla/xpla` contains the signing and transaction utilities.

4. Open the `package.json` file in a code editor and add `"type": "module",`.

   ```json
   {
     // ...
     "type": "module",
     // ...
   }
   ```

## 2. Initialize the RPC

XPLA Chain's RPC allows users to connect to the blockchain, make queries, create wallets, and submit transactions. It's the main workhorse behind `xplajs`.

1. Open your `index.js` file in a code editor and input the following to initialize the RPC:

   ```ts
   import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";

   const rpc = await createRPCQueryClient({ 
     rpcEndpoint: "https://cube-rpc.xpla.io" // Use "https://dimension-rpc.xpla.io" for prod
   });
   ```

   The `createRPCQueryClient` function establishes a connection to the XPLA Chain RPC endpoint, allowing you to query blockchain data and submit transactions. This replaces the older LCD (Light Client Daemon) approach with a more direct RPC connection.

   {{< alert >}}
   **Note**

   The previous code block shows how to connect to the cube testnet. To connect to the dimension_37-1 mainnet for production, use "`https://dimension-rpc.xpla.io`".
   {{< /alert >}}

## 3. Create a Cube Testnet Wallet

1. You'll need a wallet to sign and submit transactions. [Create a new wallet]({{< ref "/learn/learn/xpla-vault/vaults/extension-vault" >}}) using the XPLA extension vault. Be sure to save your mnemonic key!

2. After creating your wallet, you'll need to set it to use the testnet. Click the gear icon in the extension and change the network from `mainnet` to `testnet`.

3. Add the following code to your `index.ts` file and input your mnemonic key:

   ```ts
   import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1";
   import { HDPath } from "@interchainjs/types";
   import { DirectSigner } from "@xpla/xpla/signers/direct";
   import { toEncoders } from "@interchainjs/cosmos/utils";
   import { MsgExecuteContract } from "@xpla/xplajs/cosmwasm/wasm/v1/tx";
   import { Network } from "@xpla/xpla/defaults";

   const mnemonic = "-> Input your 24-word mnemonic key here <-";
   const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
   
   const signer = new DirectSigner(auth, toEncoders(MsgExecuteContract), Network.Testnet.rpc);
   
   const address = await signer.getAddress();
   ```

   This code creates a wallet instance using the new xplajs architecture. The `EthSecp256k1Auth` handles authentication using your mnemonic phrase, while `DirectSigner` provides the signing capabilities. 

   **Important**: The `toEncoders(MsgExecuteContract)` function is crucial for transaction signing. It registers the `MsgExecuteContract` message type with the signer, enabling it to properly encode and sign this specific type of transaction. Without this registration, the signer won't know how to handle the message format, and your transactions will fail. If you need to use other message types (like `MsgSend`, `MsgDelegate`, etc.), you must include them in the `toEncoders` function as well: `toEncoders(MsgExecuteContract, MsgSend, MsgDelegate)`.

   The `Network.Testnet.rpc` specifies the testnet configuration for the signer.

   For AMINO encoding, refer to the [XPLA Network Configuration Guide](https://github.com/xpladev/xplajs/blob/main/networks/xpla/README.md).

   {{< alert context="warning" >}}
   **Warning**

   Although this tutorial has you input your mnemonic directly, this practice should be avoided in production.
   For security reasons, it's better to store your mnemonic key data in your environment by using `process.env.SECRET_MNEMONIC` or `process.env.SECRET_PRIV_KEY`. This practice is more secure than a hard-coded string.
   {{< /alert >}}

4. Request testnet funds for your wallet by navigating to the [XPLA faucet](https://faucet.xpla.io) and inputting your wallet address. You'll need these funds to perform swaps and pay for gas fees. Once the funds are in your wallet, you're ready to move on to the next step.

## 4. Find a Contract Address

To find the contract address for a specific Dezswap pair [Dezswap Pair API](https://dimension-api.dezswap.io/v1/pairs)

## 5. Query a Dezswap Contract and Set up the Transaction

Before you can perform a swap, you'll need a belief price. You can calculate the belief price of one token by querying simulation to the pool. The belief price +/- the `max_spread` is the range of possible acceptable prices for this swap.

1. Add the following code to your `index.ts` file. Make sure the contract address is correct.

   ```ts
   import { Decimal } from 'decimal.js';

   const contract = "<POOL_CONTRACT_ADDRESS>"; // A Dezswap pair contract address
   const offerAmount = 1000000000000000000;
   
   // 5.1 simulate the tx
   const pairResponse = await rpc.cosmwasm.wasm.v1.smartContractState({
     address: contract,
     queryData: new Uint8Array(Buffer.from(`{"pair": {}}`))
   });
   const { asset_decimals: assetDecimals } = JSON.parse(new TextDecoder().decode(pairResponse.data));
   
   const res = await rpc.cosmwasm.wasm.v1.smartContractState({
     address: contract,
     queryData: new Uint8Array(Buffer.from(`{
       "simulation": {
         "offer_asset": {
           "info" : {
             "native_token": {
               "denom": "axpla"
             }
           },
           "amount": "${offerAmount}"
         }
       }
     }`))
   });
   const { return_amount: returnAmount } = JSON.parse(new TextDecoder().decode(res.data));

   const Decimal18 = Decimal.set({ precision: 18, rounding: Decimal.ROUND_DOWN });
   const beliefPrice = new Decimal18(offerAmount).dividedBy(10 ** assetDecimals[0]).dividedBy(new Decimal18(returnAmount).dividedBy(10 ** assetDecimals[1]));
   ```

   This code performs two key operations: first, it queries the pool contract to get asset decimals information, then it simulates the swap to calculate the expected return amount. The belief price is calculated using precise decimal arithmetic to ensure accurate swap execution. The `Decimal.js` library is used for high-precision calculations to avoid floating-point errors.

2. Next, generate a message to broadcast to the network:

   ```ts
   import { MessageComposer } from "@xpla/xplajs/cosmwasm/wasm/v1/tx.registry";
   import { Coin } from "@xpla/xplajs/cosmos/base/v1beta1/coin";

   const executeContractMsg = await MsgExecuteContract.fromPartial({
     sender: await signer.getAddress(),
     contract: contract,
     msg: new Uint8Array(Buffer.from(`{
       "swap": {
         "belief_price": "${beliefPrice.toString()}",
         "max_spread": "0.005",
         "offer_asset": {
           "info": {
             "native_token": {
               "denom": "axpla"
             }
           },
           "amount": "${offerAmount}"
         }
       }
     }`)),
     funds: [Coin.fromPartial({denom: "axpla", amount: offerAmount.toString()})]
   });
   ```

   This code creates the transaction message for the swap operation. The `MsgExecuteContract.fromPartial` method constructs the message with the calculated belief price, maximum spread tolerance, and the tokens to be swapped. The `max_spread` parameter (0.5%) ensures the transaction will only execute if the actual price doesn't deviate too much from the expected price.

## 6. Broadcast the Transaction

1. Add the following code to `index.ts` to create, sign, and broadcast the transaction:

   ```ts
   import { StdFee } from "@xpla/xplajs/types";

   const { executeContract } = MessageComposer.fromPartial;
   const msg = await executeContract(executeContractMsg);
   
   const fee: StdFee = await signer.estimateFee({messages: [msg]});
   
   const tx = await signer.signAndBroadcast({messages: [msg], fee});
   ```

   This final step completes the transaction process. The `MessageComposer.fromPartial` creates the properly formatted message, `estimateFee` calculates the appropriate gas fees, and `signAndBroadcast` signs the transaction with your private key and submits it to the network. The transaction will be processed by the XPLA Chain validators and the swap will be executed if all conditions are met.

2. Run the code in your terminal:

   ```sh
   node index.js
   ```

If successful, you'll see a log of the successful transaction and some new tokens in your wallet.

And that's it! You can find other pool addresses [here](https://app.dezswap.io/) to call other swaps. Be sure to use the correct testnet or mainnet contract address.
