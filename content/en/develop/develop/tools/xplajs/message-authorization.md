---
title: Message Authorization
weight: 90
type: docs
---

This example demonstrates how to use message authorization in xplajs, allowing one account (grantee) to execute transactions on behalf of another account (granter) within specified limits.

```ts
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { toEncoders } from "@interchainjs/cosmos/utils"
import { Network } from "@xpla/xpla/defaults"
import { Coin } from "@xpla/xplajs/cosmos/base/v1beta1/coin"
import { MsgSend } from "@xpla/xplajs/cosmos/bank/v1beta1/tx"
import { MsgGrant } from "@xpla/xplajs/cosmos/authz/v1beta1/tx"
import { MsgExec } from "@xpla/xplajs/cosmos/authz/v1beta1/tx"
import { SendAuthorization } from "@xpla/xplajs/cosmos/bank/v1beta1/authz"
import { MessageComposer } from "@xpla/xplajs/cosmos/bank/v1beta1/tx.registry"
import { MessageComposer as AuthzMessageComposer } from "@xpla/xplajs/cosmos/authz/v1beta1/tx.registry"
import { StdFee } from "@xpla/xplajs/types"
import { Duration } from "@xpla/xplajs/google/protobuf/duration"
import { Grant, GenericAuthorization } from "@xpla/xplajs/cosmos/authz/v1beta1/authz"
import { Any } from "@xpla/xplajs/google/protobuf/any"
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query"

// Granter mnemonic
const granterMnemonic = "notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius"

// Grantee mnemonic  
const granteeMnemonic = "quality vacuum heart guard buzz spike sight swarm shove special gym robust assume sudden deposit grid alcohol choice devote leader tilt noodle tide penalty"

async function grantAuthorization(
  granterSigner: DirectSigner,
  granteeAddress: string,
  spendLimit: Coin,
  duration: Date
) {
  const sendAuth = SendAuthorization.fromPartial({
    spendLimit: [spendLimit]
  })

  const authAny = Any.fromPartial({
    typeUrl: "/cosmos.bank.v1beta1.SendAuthorization",
    value: SendAuthorization.encode(sendAuth).finish()
  })

  const grant = Grant.fromPartial({
    authorization: authAny,
    expiration: duration
  })

  const msgGrant = MsgGrant.fromPartial({
    granter: await granterSigner.getAddress(),
    grantee: granteeAddress,
    grant,
  })

  const msg = AuthzMessageComposer.fromPartial.grant(msgGrant)
  
  return await granterSigner.signAndBroadcast({messages: [msg]})
}

async function sendAuthorized(
  granteeSigner: DirectSigner,
  granterAddress: string,
  toAddress: string,
  amount: Coin
) {
  const msgSend = MsgSend.fromPartial({
    fromAddress: granterAddress,
    toAddress: toAddress,
    amount: [amount]
  })

  const sendMsg = MessageComposer.fromPartial.send(msgSend)

  const sendMsgAny = Any.fromPartial({
    typeUrl: "/cosmos.bank.v1beta1.MsgSend",
    value: MsgSend.encode(msgSend).finish()
  })

  const msgExec = MsgExec.fromPartial({
    grantee: await granteeSigner.getAddress(),
    msgs: [sendMsgAny]
  })

  const msg = AuthzMessageComposer.fromPartial.exec(msgExec)
  
  return await granteeSigner.signAndBroadcast({messages: [msg]})
}

async function main() {
  // Create granter signer (xpla16wx7ye3ce060tjvmmpu8lm0ak5xr7gm2dp0kpt)
  const [granterAuth] = EthSecp256k1Auth.fromMnemonic(granterMnemonic, [HDPath.eth().toString()])
  const granterSigner = new DirectSigner(granterAuth, toEncoders(MsgGrant, Any), Network.Testnet.rpc)

  // Create grantee signer (xpla1pe9mc2q72u94sn2gg52ramrt26x5efw6hr5gt4)
  const [granteeAuth] = EthSecp256k1Auth.fromMnemonic(granteeMnemonic, [HDPath.eth().toString()])
  const granteeSigner = new DirectSigner(granteeAuth, toEncoders(MsgExec, Any), Network.Testnet.rpc)

  try {
    // Grant authorization
    console.log("Granting authorization...")
    const grantTx = await grantAuthorization(
      granterSigner,
      granteeAddress,
      Coin.fromPartial({denom: "axpla", amount: "1000000000000000000"}),
      new Date(Date.now() + 24 * 60 * 60 * 1000) // 24 hours from now
    )
    console.log("Authorization granted:", grantTx)

    // Wait for transaction to be processed 
    console.log("Waiting for transaction to be included in block...")
    await new Promise(resolve => setTimeout(resolve, 10000))

    // Execute authorized send
    console.log("Executing authorized send...")
    const execTx = await sendAuthorized(
      granteeSigner,
      granterAddress,
      granterAddress, // Send back to granter for testing
      Coin.fromPartial({denom: "axpla", amount: "1000000000000000000"}) // 1 XPLA
    )
    console.log("Authorized send executed:", execTx)

  } catch (error) {
    console.error("Error:", error)
  }
}

main().catch(console.error)
```