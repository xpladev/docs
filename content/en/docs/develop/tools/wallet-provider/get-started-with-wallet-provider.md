---
title: Get Started with Wallet Provider
weight: 10
---

[Wallet Provider](https://github.com/xpladev/wallet-provider) makes it easy to build XPLA Vault (browser extension and mobile) functionality into your React application. It contains custom hooks that drastically simplify common tasks like connecting a wallet and triggering transactions.

This guide will cover how to set up a React app, integrate Wallet Provider, check the balance of the connected account, and call a token swap. If you want to integrate XPLA Vault into an existing React app you can skip past the `Project Setup` section.

{{< hint info >}}
**Just want to dive in?**
Check out the getting started section for the premade templates [in GitHub](https://github.com/xpladev/wallet-provider/).
{{< /hint >}}

If you're using a frontend framework other than React you'll need to use [Wallet Controller](https://www.npmjs.com/package/@xpla/wallet-controller) instead. Controller provides the sub-structure of Provider.

## Prerequisites

- [XPLA Chain Chrome extension vault]({{< ref "/docs/learn/xpla-vault/vaults/extension-vault" >}})
- Node.js version 16+
- [NPM](https://www.npmjs.com/)

## 1. Project Setup

1. To get started, you'll need some basic React scaffolding. To generate this, run the following in your terminal:

   ```sh
   npx create-react-app my-xpla-app
   cd my-xpla-app
   ```

2. Then, install the `@xpla/wallet-provider` package:

   ```sh
   npm install @xpla/wallet-provider
   ```

## 2. Wrap Your App in `WalletProvider`

Next, you'll wrap your `App` with `<WalletProvider>` to give all your components access to useful data, hooks, and utilities. You'll also need to pass in information about XPLA Chain, such as the mainnet or chainId, into the provider via `getChainOptions`.

1. Navigate to your `Index.js` in a code editor and replace the code with the following:

   ```js
   import React from 'react';
   import ReactDOM from 'react-dom/client';
   import './index.css';
   import App from './App';
   import reportWebVitals from './reportWebVitals';
   import {
     getChainOptions,
     WalletProvider,
   } from "@xpla/wallet-provider";

   const root = ReactDOM.createRoot(document.getElementById('root'));
   getChainOptions().then((chainOptions) => {
     root.render(
       <WalletProvider {...chainOptions}>
         <App />
       </WalletProvider>
     );
   });

   // If you want to start measuring performance in your app, pass a function
   // to log results (for example: reportWebVitals(console.log))
   // or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
   reportWebVitals();
   ```

2. Start the application to make sure it works:

   ```sh
   npm start
   ```

Your browser should open to `http://localhost:3000/`, and you should see the React logo with a black background and some text.

{{< expand "Getting polyfill errors?" >}}

To solve these errors, can downgrade `react-scripts`: `4.0.3` in your `package.json` and reinstall your dependencies as a quick fix:

1.  Navigate to `my-xpla-app` in your terminal and run the following:

    ```sh
    npm install react-scripts@4.0.3
    ```

2.  Reinstall your dependencies:

    ```sh
    npm install
    ```

3.  Restart your app:

    ```sh
    npm start
    ```

Alternatively, you can configure your webpack to include the necessary fallbacks. Here's an [example](https://github.com/xpladev/wallet-provider/blob/main/templates/create-react-app/config-overrides.js) that uses [react-app-rewired](https://www.npmjs.com/package/react-app-rewired).
{{< /expand >}}

3.  Create a new directory called `components` in the `source` directory. This directory will house components to trigger different actions from our connected wallet.

## 3. Put `useWallet` to Work

Now that `App.js` has inherited the context of `WalletProvider`, you can start putting your imports to work. You'll use the multi-purpose `useWallet` hook to connect your XPLA extension vault to your web browser.

1. Create a new file in the `components` directory called `Connect.js`.

2. Populate the `Connect.js` file with the following:

   ```js
   import { useWallet, WalletStatus } from "@xpla/wallet-provider";
   import React from "react";
   export default function Connect() {
     const {
       status,
       network,
       wallets,
       availableConnectTypes,
       connect,
       disconnect,
     } = useWallet();
     return (
       <>
         {JSON.stringify({ status, network, wallets }, null, 2)}
         {status === WalletStatus.WALLET_NOT_CONNECTED && (
           <>
             {availableConnectTypes.map((connectType) => (
               <button
                 key={"connect-" + connectType}
                 onClick={() => connect(connectType)}
               >
                 Connect {connectType}
               </button>
             ))}
           </>
         )}
         {status === WalletStatus.WALLET_CONNECTED && (
           <button onClick={() => disconnect()}>Disconnect</button>
         )}
       </>
     );
   }
   ```

3. Open `App.js` in your code editor and replace the code with the following:

   ```js
   import "./App.css";
   import Connect from "./components/Connect";

   function App() {
     return (
       <div className="App">
         <header className="App-header">
           <Connect />
         </header>
       </div>
     );
   }

   export default App;
   ```

4. Refresh your browser. There should be some new text and buttons in your browser.

5. Make sure your XPLA extension vault is connected to a wallet. Click **Connect EXTENSION**, and the app will connect to your wallet.

The `status`, `network`, and `wallets` properties in your browser provide useful information about the state of the XPLA Vault. Before connecting, the `status` variable is `WALLET_NOT_CONNECTED`, and upon connection the status becomes `WALLET_CONNECTED`. In addition, the `wallets` array now has one entry with the `connectType` and `xplaAddress` you used to connect.

You should be able to see these changes in real-time.

## 4. Querying a Wallet Balance

It's common for an app to show the connected user's XPLA balance. To achieve this you'll need two hooks. The first is `useLCDClient`. An `LCDClient` is essentially a REST-based adapter for the XPLA Chain. You can use it to query an account balance. The second is `useConnectedWallet`, which tells you if a wallet is connected, and if so, basic information about that wallet such as its address.

Note that if your wallet is empty you won't see any tokens.

1. Create a file in your `Components` folder named `Query.js`.

2. Populate `Query.js` with the following:

   ```js
   import {
     useConnectedWallet,
     useLCDClient,
   } from "@xpla/wallet-provider";
   import React, { useEffect, useState } from "react";

   export default function Query() {
     const lcd = useLCDClient(); // LCD stands for Light Client Daemon
     const connectedWallet = useConnectedWallet();
     const [balance, setBalance] = useState(null);

     useEffect(() => {
       if (connectedWallet) {
         lcd.bank.balance(connectedWallet.walletAddress).then(([coins]) => {
           setBalance(coins.toString());
         });
       } else {
         setBalance(null);
       }
     }, [connectedWallet, lcd]); // useEffect is called when these variables change

     return (
       <div>
         {balance && <p>{balance}</p>}
         {!connectedWallet && <p>Wallet not connected!</p>}
       </div>
     );
   }
   ```

3. Open `App.js` in your code editor and add `import Query from './components/Query'` to line 3, and ` <Query />` to line 11. The whole file should look like the following:

   ```js
   import "./App.css";
   import Connect from "./components/Connect";
   import Query from "./components/Query";

   function App() {
     return (
       <div className="App">
         <header className="App-header">
           <Connect />
           <Query />
         </header>
       </div>
     );
   }

   export default App;
   ```

4. Refresh your browser. Your wallet balance will appear in micro-denominations. Multiply by {{< katex >}}10^{18}{{< /katex >}} for an accurate balance.

## 5. Sending a Transaction

WalletProvider also helps create and send transactions to the XPLA Chain. You'll also need `xpla.js` to help generate the sample transaction:

```sh
npm install @xpla/xpla.js
```

Before broadcasting this example transaction, ensure you're on the XPLA Chain testnet. To change networks click the gear icon in your XPLA Vault and select `testnet`.

You can request testnet funds from the [faucet](https://faucet.xpla.io/).

A XPLA transfer transaction needs a fee and a message containing the sender address, recipient address, and send amount (in this case 1 XPLA). Once the message is constructed, the `post` method on `connectedWallet` broadcasts it to the network.

{{< hint info >}}
**What happens if something goes wrong?**
Wallet provider also supplies useful error types. This example will handle the `UserDenied` error case. You can find other cases to handle on [GitHub](https://github.com/xpladev/wallet-provider/blob/4e601c2dece7bec92c9ce95991d2314220a2c954/packages/src/%40xpladev/wallet-controller/exception/mapExtensionTxError.ts#L23).
{{< /hint >}}

1. Create a file in your `Components` folder named `Tx.js`.

2. Populate `Tx.js` with the following:

   ```js
   import { Fee, MsgSend } from "@xpla/xpla.js";
   import {
     useConnectedWallet,
     UserDenied,
   } from "@xpla/wallet-provider";
   import React, { useCallback, useState } from "react";

   const TEST_TO_ADDRESS = "xpla12hnhh5vtyg5juqnzm43970nh4fw42pt27nw9g9";

   export default function Tx() {
     const [txResult, setTxResult] = useState(null);
     const [txError, setTxError] = useState(null);

     const connectedWallet = useConnectedWallet();

     const testTx = useCallback(async () => {
       if (!connectedWallet) {
         return;
       }
       if (connectedWallet.network.chainID.startsWith("dimension")) {
         alert(`Please only execute this example on Testnet`);
         return;
       }

       try {
         const transactionMsg = {
           fee: new Fee(1000000, "20000axpla"),
           msgs: [
             new MsgSend(connectedWallet.walletAddress, TEST_TO_ADDRESS, {
               axpla: 1000000,
             }),
           ],
         };

         const tx = await connectedWallet.post(transactionMsg);
         setTxResult(tx);
       } catch (error) {
         if (error instanceof UserDenied) {
           setTxError("User Denied");
         } else {
           setTxError(
             "Unknown Error: " +
               (error instanceof Error ? error.message : String(error))
           );
         }
       }
     }, [connectedWallet]);

     return (
       <>
         {connectedWallet?.availablePost && !txResult && !txError && (
           <button onClick={testTx}>Send 1USD to {TEST_TO_ADDRESS}</button>
         )}

         {txResult && <>{JSON.stringify(txResult, null, 2)}</>}
         {txError && <pre>{txError}</pre>}

         {connectedWallet && !connectedWallet.availablePost && (
           <p>This connection does not support post()</p>
         )}
       </>
     );
   }
   ```

{{< hint info >}}
**Note**
Because all coins are denominated in micro-units, you will need to multiply any coins by 10 to the power of 18. For example, 1000000000000000000 axpla = 1 XPLA.
{{< /hint >}}

3. Open `App.js` in your code editor and add `import Tx from './components/Tx'` to line 4, and ` <Tx />` to line 12. The whole file should look like the following:

   ```js
   import "./App.css";
   import Connect from "./components/Connect";
   import Query from "./components/Query";
   import Tx from "./components/Tx";

   function App() {
     return (
       <div className="App">
         <header className="App-header">
           <Connect />
           <Query />
           <Tx />
         </header>
       </div>
     );
   }

   export default App;
   ```

4. Refresh your browser. You'll see a new **Send** button. Click the button to send your transaction. Your extension wallet will ask you to confirm the transaction.
