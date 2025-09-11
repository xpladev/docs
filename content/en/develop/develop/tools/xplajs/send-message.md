---
title: Send Message
weight: 120
type: docs
---

## Sending transactions

To send transactions on the XPLA blockchain, you need to create a signer and construct the appropriate message. The following example demonstrates how to send XPLA tokens from one account to another using the `MsgSend` message type.

The process involves creating a DirectSigner with your mnemonic, constructing a send message with the recipient address and amount, and then signing and broadcasting the transaction to the network.

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