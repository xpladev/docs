---
title: Common Examples
weight: 20
type: docs
---

Use the following common examples to learn how to use xplajs. If this is your first time using xplajs, use the [xplajs installation guide]({{< ref "/develop/develop/tools/xplajs/get-started-with-xpla-js" >}}).

{{< alert >}}
**Tip**

If you are new to XPLA Chain and don't know where to start, visit the [getting started guide]({{< ref "get-started" >}}).
{{< /alert >}}

## Configuring LCDClient

The following code example shows how to initialize the RPCClient. The rest of the examples assume you initialized it by using this example or similar code.

```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";

const rpcClient = await createRPCQueryClient({ rpcEndpoint: "https://cube-rpc.xpla.io" });
```

## Get Wallet Balance (native tokens)

```js
// Replace with address to check.
const address = "xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
const {balances} = await rpcClient.cosmos.bank.v1beta1.allBalances({ address, resolveDenom: true })
console.log(balances)
```

Example response:

```js
[{ denom: "axpla", amount: "5030884" }];
```

## Get Wallet Balance (CW20 tokens)

```js
// TEST on cube_47-5
const tokenAddress = "xpla19w8vmg7tmh07ztr3v7lq8sdny6jjkj6pk03a7fk52gpgepfxnlgq8g7r50";
const walletAddress = "xpla1dr6hsalnns5tjvd68kg7tfam3qzmt3958dra8x";
const response = await rpcClient.cosmwasm.wasm.v1.smartContractState({
    address: tokenAddress,
    queryData: new Uint8Array(Buffer.from(`{"balance": {"address": "${walletAddress}"}}`))
})

console.log(JSON.parse(new TextDecoder().decode(response.data)))
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
const hash = "56015BCAD8952D1505241881A2F581B697A10F73EF2706D31B8A36F633A20829"
const response = await rpcClient.cosmos.tx.v1beta1.getTx({hash: hash})
console.log(response)
```

Example response (modified for readability):

```js
{
  tx: {
    body: {
      messages: [Array],
      memo: '{"amount":"42507883.877670559450509373 XPLA","percent":0.708464,"baseFee":"0.2 XPLA","averag_ratio":0.999346}',
      timeoutHeight: 0n,
      extensionOptions: [],
      nonCriticalExtensionOptions: []
    },
    authInfo: { signerInfos: [Array], fee: [Object], tip: undefined },
    signatures: [ [Uint8Array] ]
  },
  txResponse: {
    height: 15578109n,
    txhash: '56015BCAD8952D1505241881A2F581B697A10F73EF2706D31B8A36F633A20829',
    codespace: '',
    code: 0,
    data: '12260A242F636F736D6F732E62616E6B2E763162657461312E4D736753656E64526573706F6E7365',
    rawLog: '',
    logs: [],
    info: '',
    gasWanted: 1901178n,
    gasUsed: 96940n,
    tx: { typeUrl: '/cosmos.tx.v1beta1.Tx', value: [Uint8Array] },
    timestamp: '2025-07-28T07:49:06Z',
    events: [
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object]
    ]
  }
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
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { toEncoders } from "@interchainjs/cosmos/utils"
import { MsgSend } from "@xpla/xplajs/cosmos/bank/v1beta1/tx";
import { Network } from "@xpla/xpla/defaults"
import { Coin, DecCoin } from "@xpla/xplajs/cosmos/base/v1beta1/coin";
import { MessageComposer } from "@xpla/xplajs/cosmos/bank/v1beta1/tx.registry";

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgSend), Network.Testnet.rpc);
const msgSend = MsgSend.fromPartial({
    fromAddress: await signer.getAddress(),
    toAddress: "xpla1888g76xr3phk7qkfknnn8hvxyzfj0e2vuh4jmw",
    amount: [Coin.fromPartial({denom: "axpla", amount: "1000000000000000000"})]
})

const { send } = MessageComposer.fromPartial;
const msg = await send(msgSend);
const tx = await signer.signAndBroadcast({messages: [msg]})
console.log(tx)
```

## Sending CW20 tokens

The following code example shows how to send CW20 tokens:

```ts
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { toEncoders } from "@interchainjs/cosmos/utils"
import { MsgExecuteContract } from "@xpla/xplajs/cosmwasm/wasm/v1/tx";
import { Network } from "@xpla/xpla/defaults"
import { MessageComposer } from "@xpla/xplajs/cosmwasm/wasm/v1/tx.registry";

const rpcClient = await createRPCQueryClient({ rpcEndpoint: "https://cube-rpc.xpla.io" });

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgExecuteContract), Network.Testnet.rpc);
const tokenAddress = "xpla19w8vmg7tmh07ztr3v7lq8sdny6jjkj6pk03a7fk52gpgepfxnlgq8g7r50";
const msgExecuteContract = MsgExecuteContract.fromPartial({
    sender: await signer.getAddress(),
    contract: tokenAddress,
    msg: new Uint8Array(Buffer.from(`{"transfer": {"recipient": "xpla1888g76xr3phk7qkfknnn8hvxyzfj0e2vuh4jmw", "amount": "1"}}`))
})

const { executeContract } = MessageComposer.fromPartial;
const msg = await executeContract(msgExecuteContract);
const tx = await signer.signAndBroadcast({messages: [msg]})
console.log(tx)
```

## Swapping a Native XPLA Chain Asset for a CW20 Token Using Dezswap

The following code example shows how to swap a native asset for CW20 using Dezswap.

Run this example on mainnet.

```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { toEncoders } from "@interchainjs/cosmos/utils"
import { MsgExecuteContract } from "@xpla/xplajs/cosmwasm/wasm/v1/tx";
import { Network } from "@xpla/xpla/defaults"
import { MessageComposer } from "@xpla/xplajs/cosmwasm/wasm/v1/tx.registry";
import { Decimal } from 'decimal.js'
import { Coin } from "@xpla/xplajs/cosmos/base/v1beta1/coin";

const rpcClient = await createRPCQueryClient({ rpcEndpoint: "https://cube-rpc.xpla.io" });

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgExecuteContract), Network.Testnet.rpc);

// XPLA <> CTXT
const pool = "xpla1vgay526xzh725vpur7drsyxhlvg4fxfvu5dczcctz35ct23q4vpqxqdemw";
const xplaAmount = 1000000000000000000

// Fetch the decimal of each asset in the pool and simulation result with `xplaAmount`.
const res = await rpcClient.cosmwasm.wasm.v1.smartContractState({
    address: pool,
    queryData: new Uint8Array(Buffer.from(`{"pair": {}}`))
})
const { asset_decimals: assetDecimals } = JSON.parse(new TextDecoder().decode(res.data))

const simulateResponse = await rpcClient.cosmwasm.wasm.v1.smartContractState({
    address: pool,
    queryData: new Uint8Array(Buffer.from(`{
        "simulation": {
        "offer_asset": {
            "info" : {
            "native_token": {
                "denom": "axpla"
            }
            },
            "amount": "${xplaAmount}"
        }
    }
}`))
})
const {return_amount: returnAmount} = JSON.parse(new TextDecoder().decode(simulateResponse.data))

// Calculate belief price using pool balances.
const Decimal18 = Decimal.set({ precision: 18, rounding: Decimal.ROUND_DOWN });
const beliefPrice = new Decimal18(xplaAmount).dividedBy(10 ** assetDecimals[0]).dividedBy(new Decimal18(returnAmount).dividedBy(10 ** assetDecimals[1]))

// Swap 1 XPLA to CTXT with 1% slippage tolerance.
const swapMsg = {
    swap: {
        max_spread: "0.01",
        offer_asset: {
            info: {
                native_token: {
                    denom: "axpla"
                }
            },
            amount: xplaAmount.toString()
        },
        belief_price: beliefPrice.toString()
    }
};

const msgExecuteContract = MsgExecuteContract.fromPartial({
    sender: await signer.getAddress(),
    contract: pool,
    msg: new Uint8Array(Buffer.from(JSON.stringify(swapMsg))),
    funds: [Coin.fromPartial({denom: "axpla", amount: xplaAmount.toString()})]
})

const { executeContract } = MessageComposer.fromPartial;
const msg = await executeContract(msgExecuteContract);
const tx = await signer.signAndBroadcast({messages: [msg]})
console.log(tx)
```

## Decoding Protobuf-encoded Messages

The following code example shows how to decode messages that have been encoded using Protobuf:

```ts
```

## Validate a XPLA Chain Address

The following code example shows how to do a basic verification on a XPLA Chain address.

This is a basic version of the verification, it does not require external libraries as it performs a simple comparison with a regex string. It could give false positives since it doesn't verify the checksum of the address.

```ts
function isValid(address) {
  try {
    const rpcClient = await createRPCQueryClient({ rpcEndpoint: "https://cube-rpc.xpla.io" });
    const response = await rpcClient.cosmos.auth.v1beta1.addressStringToBytes({addressString: address}); // throw 
    return true;
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
signer.signAndBroadcast({messages: [msg]})
  .then(async (tx) => {
    // TODO: use a for or add a timeout to prevent infinite loops
    while (true) {
      // query txhash
      const data = await rpcClient.cosmos.tx.v1beta1.getTx({hash: tx.hash || ""}).catch(() => {});
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
