---
title: Send Message
weight: 120
type: docs
---

## Sending transactions

To send transactions on the XPLA blockchain, you need to create a signer and construct the appropriate message. The following example demonstrates how to send XPLA tokens from one account to another using the `MsgSend` message type.

The process involves creating a DirectSigner with your mnemonic, constructing a send message with the recipient address and amount, and then signing and broadcasting the transaction to the network.

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