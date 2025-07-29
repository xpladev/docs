---
weight: 60
title: IBC Transfers
type: docs
---

XPLA Chain has full IBC transfer capabilities through xplajs. Although IBC functionality is not readily exposed through XPLA Vault’s front-end, it can be fully incorporated into any dApp. It is up to a dApp’s front end to initiate IBC transfers.

## MsgTransfer

xpla.js exports a `MsgTransfer` class that can be used to construct IBC transfers.

```js
const msgTransfer = MsgTransfer.fromPartial({
    sourcePort: "transfer", // IBC port
    sourceChannel: "channel-0", // Outbound channel (Axelar)
    token: {denom: "axpla", amount: "1000000000000000000"}, // 1 XPLA
    sender: "xpla1npvwllfr9dqr8erajqqr6s0vxnk2ak55hh2h5f", // Source Address on XPLA Chain
    receiver: "axelar1npvwllfr9dqr8erajqqr6s0vxnk2ak55d7yr5m", // Destination address on Axelar network
    memo: "",
    timeoutTimestamp: BigInt(Date.now() + 60 * 60 * 10 ** 3) * (10n ** 6n) // Timeout timestamp (in nanoseconds) relative to the current block timestamp.
})
```

## Supported Channels

Channels are defined when a relayer is set up between XPLA Chain and an external chain. For each new connected chain the channel ID is incremented.

You can use [Mintscan](https://www.mintscan.io/xpla/relayers) to find the available channels and their IDs.

## Derive Cosmos Chain Addresses from a XPLA Chain Address

Cosmos SDK based blockchains use bech32 to encode the public key for display. Assuming the same private key is used on multiple Cosmos SDK chains it is possible to decode a XPLA Chain address and generate the corresponding public key on another chain.

Here's a quick example using the [@interchainjs/encoding](https://github.com/hyperweb-io/interchainjs/blob/main/packages/encoding/README.md) JavaScript library:

```js
import { fromBech32, toBech32 } from "@interchainjs/encoding/bech32";

const xplaAddress = 'xpla1npvwllfr9dqr8erajqqr6s0vxnk2ak55hh2h5f';
const {prefix, data: hex} = fromBech32(address)
const axelarAddress = toBech32("axelar", hex)
```

## Complete Example

The following example demonstrates how to send 1 XPLA from XPLA Chain to the Axelar network.

```JS

import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { toEncoders } from "@interchainjs/cosmos/utils"
import { MsgTransfer } from "@xpla/xplajs/ibc/applications/transfer/v1/tx";
import { Network } from "@xpla/xpla/defaults"
import { Coin } from "@xpla/xplajs/cosmos/base/v1beta1/coin";
import { MessageComposer } from "@xpla/xplajs/ibc/applications/transfer/v1/tx.registry";
import { StdFee } from "@xpla/xplajs/types";
import { fromBech32, toBech32 } from "@interchainjs/encoding/bech32";

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgTransfer), Network.Testnet.rpc);

const address = await signer.getAddress()
const {prefix, data: hex} = fromBech32(address)
const receiver = toBech32("axelar", hex)


const msgTransfer = MsgTransfer.fromPartial({
    sourcePort: "transfer", // IBC port
    sourceChannel: "channel-0", // Outbound channel (Axelar)
    token: {denom: "axpla", amount: "1000000000000000000"},
    sender: address, // Source Address on XPLA Chain
    receiver: receiver, // Destination address on Axelar network
    memo: "",
    timeoutTimestamp: BigInt(Date.now() + 60 * 60 * 10 ** 3) * (10n ** 6n) // Timeout timestamp (in nanoseconds) relative to the current block timestamp.
})

const msg = MessageComposer.fromPartial.transfer(msgTransfer);
const tx = await signer.signAndBroadcast({messages: [msg]})
console.log(tx)
```
