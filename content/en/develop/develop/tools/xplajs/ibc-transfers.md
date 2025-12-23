---
weight: 60
title: IBC Transfers
type: docs
---

CONX Chain has full IBC transfer capabilities through xplajs. Although IBC functionality is not readily exposed through XPLA Vault’s front-end, it can be fully incorporated into any dApp. It is up to a dApp’s front end to initiate IBC transfers.

## MsgTransfer

xpla.js exports a `MsgTransfer` class that can be used to construct IBC transfers.

```js
const msgTransfer = MsgTransfer.fromPartial({
    sourcePort: "transfer", // IBC port
    sourceChannel: "channel-0", // Outbound channel (Axelar)
    token: {denom: "axpla", amount: "1000000000000000000"}, // 1 XPLA
    sender: "xpla1npvwllfr9dqr8erajqqr6s0vxnk2ak55hh2h5f", // Source Address on CONX Chain
    receiver: "axelar1npvwllfr9dqr8erajqqr6s0vxnk2ak55d7yr5m", // Destination address on Axelar network
    memo: "",
    timeoutTimestamp: BigInt(Date.now() + 60 * 60 * 10 ** 3) * (10n ** 6n) // Timeout timestamp (in nanoseconds) relative to the current block timestamp.
})
```

## Supported Channels

Channels are defined when a relayer is set up between CONX Chain and an external chain. For each new connected chain the channel ID is incremented.

You can use [Mintscan](https://www.mintscan.io/xpla/relayers) to find the available channels and their IDs.

## Derive Cosmos Chain Addresses from a CONX Chain Address

Cosmos SDK based blockchains use bech32 to encode the public key for display. Assuming the same private key is used on multiple Cosmos SDK chains it is possible to decode a CONX Chain address and generate the corresponding public key on another chain.

Here's a quick example using the [@interchainjs/encoding](https://github.com/hyperweb-io/interchainjs/blob/main/packages/encoding/README.md) JavaScript library:

```js
import { fromBech32, toBech32 } from "@interchainjs/encoding/bech32";

const xplaAddress = 'xpla1npvwllfr9dqr8erajqqr6s0vxnk2ak55hh2h5f';
const {prefix, data: hex} = fromBech32(address)
const axelarAddress = toBech32("axelar", hex)
```

## Complete Example

The following example demonstrates how to send 1 XPLA from CONX Chain to the Axelar network.

```JS

import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG, encodeCosmosEvmPublicKey } from "@xpla/xpla/signers/config";
import { MsgTransfer } from "@xpla/xplajs";
import { fromBech32, toBech32 } from "@interchainjs/encoding/bech32";
import { transfer } from "@xpla/xplajs"

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

const {prefix, data: hex} = fromBech32(signerAddress)
const receiver = toBech32("axelar", hex)

const tx = await transfer(
    signer, 
    signerAddress,
    MsgTransfer.fromPartial({
        sourcePort: "transfer", // IBC port
        sourceChannel: "channel-0", // Outbound channel (Axelar)
        token: {denom: "axpla", amount: "1000000000000000000"},
        sender: signerAddress, // Source Address on CONX Chain
        receiver: receiver, // Destination address on Axelar network
        memo: "",
        timeoutTimestamp: BigInt(Date.now() + 60 * 60 * 10 ** 3) * (10n ** 6n) // Timeout timestamp (in nanoseconds) relative to the current block timestamp.
    }),
    {
        amount: [{denom: "axpla", amount: "56000000000000000"}],
        gas: "200000",
    },
    ""
)

const res = await tx.wait()
console.log(res)
```
