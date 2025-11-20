---
title: Common Examples
weight: 20
type: docs
---

Use the following common examples to learn how to use xplajs. If this is your first time using xplajs, use the [xplajs installation guide]({{< ref "/develop/develop/tools/xplajs/get-started-with-xpla-js" >}}).

{{< alert >}}
**Tip**

If you are new to CONX Chain and don't know where to start, visit the [getting started guide]({{< ref "get-started" >}}).
{{< /alert >}}

## Configuring LCDClient

The following code example shows how to initialize the LCDClient. The rest of the examples assume you initialized it by using this example or similar code.

```ts
import { createLCDClient } from "@xpla/xplajs/xpla/lcd";

const lcd = await createLCDClient({restEndpoint: "https://cube-lcd.xpla.io"})
```

## Configuring RPCClient

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
    queryData: new TextEncoder().encode(`{"balance": {"address": "${walletAddress}"}}`)
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
const getTransactionLink = (hash, network) =>
  `https://explorer.xpla.io/${network}/tx/${hash}`;
const hash = "CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B";

console.log(getTransactionLink(hash, "testnet"));
```

Example response:

```
https://explorer.xpla.io/testnet/tx/CAB264B3D92FF3DFE209DADE791A866876DE5DD2A320C1200F9C5EC5F0E7B14B
```

## Get Link to Wallet Address

```js
const getWalletLink = (address, network) =>
  `https://explorer.xpla.io/${network}/address/${address}`;
const address = "xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
console.log(getWalletLink(address, "testnet"));
```

Example response:

```
https://explorer.xpla.io/testnet/address/xpla1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Sending Native Tokens

The following code example shows how to send native tokens:

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";
import { send } from "@xpla/xplajs";

const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");
const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {derivations: [{
    prefix: "xpla",
    hdPath: HDPath.eth().toString()
}]});

const baseSignConfig = {
    queryClient: queryClient,
    chainId: "cube_47-5",
    addressPrefix: "xpla",
}
const signerConfig = {
    ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
    ...baseSignConfig
}

const signer = new DirectSigner(wallet, signerConfig);
const signerAddress = (await signer.getAddresses())[0]

const tx = await send(
    signer,
    signerAddress,
    {
        fromAddress: signerAddress,
        toAddress: "xpla1888g76xr3phk7qkfknnn8hvxyzfj0e2vuh4jmw",
        amount: [{denom: "axpla", amount: "1000000000000000000"}]
    },
    {
        amount: [{denom: "axpla", amount: "56000000000000000"}],
        gas: "200000"
    },
    ""

)

try {
    await tx.wait()
} catch (error) {
    console.log(error)
}
console.log(tx)
```

## Sending CW20 tokens

The following code example shows how to send CW20 tokens:

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";
import { executeContract } from "@xpla/xplajs";

const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");
const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {derivations: [{
    prefix: "xpla",
    hdPath: HDPath.eth().toString()
}]});

const baseSignConfig = {
    queryClient: queryClient,
    chainId: "cube_47-5",
    addressPrefix: "xpla",
}
const signerConfig = {
    ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
    ...baseSignConfig
}

const signer = new DirectSigner(wallet, signerConfig);
const signerAddress = (await signer.getAddresses())[0]

const tokenAddress = "xpla19w8vmg7tmh07ztr3v7lq8sdny6jjkj6pk03a7fk52gpgepfxnlgq8g7r50";

const tx = await executeContract(
    signer,
    signerAddress,
    {
        sender: signerAddress,
        contract: tokenAddress,
        msg: new TextEncoder().encode(`{"transfer": {"recipient": "xpla1888g76xr3phk7qkfknnn8hvxyzfj0e2vuh4jmw", "amount": "1"}}`),
        funds: []
    },
    {
        amount: [{denom: "axpla", amount: "56000000000000000"}],
        gas: "200000"
    },
    ""
)

try {
    await tx.wait()
} catch (error) {
    console.log(error)
}
console.log(tx)
```

## Swapping a Native Token(denominator, IBC, ERC20) Using Dezswap

The following code example shows how to swap a native token for CW20 using Dezswap.

{{< alert context="warning">}}
**Native Token**

Native tokens on CONX Chain include XPLA (the main token), IBC tokens, and ERC20 tokens. For more information about Dezswap, visit [Dezswap docs](https://deploy-preview-16--docs-dezswap.netlify.app/docs/introduction/asset/).
{{< /alert >}}

#### Example: Swap XPLA to CW20 Token
```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { Decimal } from 'decimal.js'
import { Coin } from "@xpla/xplajs/cosmos/base/v1beta1/coin";
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";
import { executeContract } from "@xpla/xplajs";

const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {derivations: [{
    prefix: "xpla",
    hdPath: HDPath.eth().toString()
}]});

const baseSignConfig = {
    queryClient: queryClient,
    chainId: "cube_47-5",
    addressPrefix: "xpla",
}
const signerConfig = {
    ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
    ...baseSignConfig
}

const signer = new DirectSigner(wallet, signerConfig);
const signerAddress = (await signer.getAddresses())[0]

// XPLA <> CTXT
const pool = "xpla1vgay526xzh725vpur7drsyxhlvg4fxfvu5dczcctz35ct23q4vpqxqdemw";
const xplaAmount = 1000000000000000000

const rpcClient = await createRPCQueryClient({rpcEndpoint: "https://cube-rpc.xpla.io"})

// Fetch the decimal of each asset in the pool and simulation result with `xplaAmount`.
const res = await rpcClient.cosmwasm.wasm.v1.smartContractState({
    address: pool,
    queryData: new TextEncoder().encode(`{"pair": {}}`)
})
const { asset_decimals: assetDecimals } = JSON.parse(new TextDecoder().decode(res.data))

const simulateResponse = await rpcClient.cosmwasm.wasm.v1.smartContractState({
    address: pool,
    queryData: new TextEncoder().encode(`{
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
}`)
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

const tx = await executeContract(
    signer,
    signerAddress,
    {
        sender: signerAddress,
        contract: pool,
        msg: new TextEncoder().encode(JSON.stringify(swapMsg)),
        funds: [Coin.fromPartial({denom: "axpla", amount: xplaAmount.toString()})]
    },
    {
        amount: [{denom: "axpla", amount: "112000000000000000"}],
        gas: "400000"
    },
    ""

)

try {
    await tx.wait()
} catch (error) {
    console.log(error)
}
console.log(tx)
```

#### Example: Swap ERC20 to XPLA
```ts
// TKN (ERC20) <> XPLA
const pool = "xpla1d4hn6l43sqtklmcypzea0c9xhags9f33g67vp6lntetmzgdagvys3mzdyy";
const erc20Amount = 1000000;

const rpcClient = await createRPCQueryClient({rpcEndpoint: "https://cube-rpc.xpla.io"})

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
                "denom": "xerc20:69FD386467E3659F81e58b6EC7a12C64b32FB1E2"
            }
            },
            "amount": "${erc20Amount}"
        }
    }
}`))
})
const {return_amount: returnAmount} = JSON.parse(new TextDecoder().decode(simulateResponse.data))

// Calculate belief price using pool balances.
const Decimal18 = Decimal.set({ precision: 18, rounding: Decimal.ROUND_DOWN });
const beliefPrice = new Decimal18(erc20Amount).dividedBy(10 ** assetDecimals[0]).dividedBy(new Decimal18(returnAmount).dividedBy(10 ** assetDecimals[1]))

// Swap 1 XPLA to CTXT with 1% slippage tolerance.
const swapMsg = {
    swap: {
        max_spread: "0.01",
        offer_asset: {
            info: {
                native_token: {
                    denom: "xerc20:69FD386467E3659F81e58b6EC7a12C64b32FB1E2"
                }
            },
            amount: erc20Amount.toString()
        },
        belief_price: beliefPrice.toString()
    }
};

const tx = await executeContract(
    signer,
    signerAddress,
    {
        sender: signerAddress,
        contract: pool,
        msg: new Uint8Array(Buffer.from(JSON.stringify(swapMsg))),
        funds: [Coin.fromPartial({denom: "xerc20:69FD386467E3659F81e58b6EC7a12C64b32FB1E2", amount: erc20Amount.toString()})]
    },
    {
        amount: [{denom: "axpla", amount: "28000000000000000000"}],
        gas: "100000000"
    },
    ""

)

try {
    await tx.wait()
} catch (error) {
    console.log(error)
}
console.log(tx)
```

## Validate a CONX Chain Address

The following code example shows how to do a basic verification on a CONX Chain address.

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

console.log(isValid("xpla1dcegyrekltswvyy0xy69ydgxn9x8x32z4glay5")); // true
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
