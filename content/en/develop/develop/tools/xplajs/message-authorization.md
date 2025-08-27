---
title: Message Authorization
weight: 90
type: docs
---

This example demonstrates how to use message authorization in xplajs, allowing one account (grantee) to execute transactions on behalf of another account (granter) within specified limits.

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";
import { Any, Coin } from "@xpla/xplajs";
import { grant, Grant, SendAuthorization, exec, MsgSend } from "@xpla/xplajs";
import { MessageComposer } from "@xpla/xplajs/cosmos/bank/v1beta1/tx.registry";


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
const granterAddress = (await granterSigner.getAddresses())[0]
  const sendAuth = SendAuthorization.fromPartial({
    spendLimit: [spendLimit]
  })

  const authAny = Any.fromPartial({
    typeUrl: "/cosmos.bank.v1beta1.SendAuthorization",
    value: SendAuthorization.encode(sendAuth).finish()
  })

const tx = await grant(
    granterSigner,
    granterAddress,
    {
        granter: granterAddress,
        grantee: granteeAddress,
        grant: Grant.fromPartial({
            authorization: authAny,
            expiration: duration
            }),
        },
        {
        amount: [{denom: "axpla", amount: "56000000000000000"}],
        gas: "200000",
        },
        ""
)
  
  return tx
}

async function sendAuthorized(
  granteeSigner: DirectSigner,
  granterAddress: string,
  toAddress: string,
  amount: Coin
) {
    const granteeAddress = (await granteeSigner.getAddresses())[0]
  const msgSend = MsgSend.fromPartial({
    fromAddress: granterAddress,
    toAddress: toAddress,
    amount: [amount]
  })

  const sendMsgAny = Any.fromPartial({
    typeUrl: "/cosmos.bank.v1beta1.MsgSend",
    value: MsgSend.encode(msgSend).finish()
  })

  const tx = await exec(
    granteeSigner,
    granteeAddress,
    {
        grantee: granteeAddress,
        msgs: [sendMsgAny]
      },
      {
        amount: [{denom: "axpla", amount: "56000000000000000"}],
        gas: "200000",
      },
      ""
  )

  return tx
}

async function main() {
    const prefix = "xpla"
    const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");
    const baseSignConfig = {
        queryClient: queryClient,
        chainId: "cube_47-5",
        addressPrefix: prefix,
    }
    const signerConfig = {
        ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
        ...baseSignConfig
    }

  // Create granter signer (xpla16wx7ye3ce060tjvmmpu8lm0ak5xr7gm2dp0kpt)
  const granterWallet = await EthSecp256k1HDWallet.fromMnemonic(granterMnemonic, {
    derivations: [{
      prefix,
      hdPath: HDPath.eth().toString()
    }]
  })
  const granterSigner = new DirectSigner(granterWallet, signerConfig)
  const granterAddress = (await granterSigner.getAddresses())[0]

  // Create grantee signer (xpla1pe9mc2q72u94sn2gg52ramrt26x5efw6hr5gt4)
  const granteeWallet = await EthSecp256k1HDWallet.fromMnemonic(granteeMnemonic, {
    derivations: [{
      prefix,
      hdPath: HDPath.eth().toString()
    }]
  })
  const granteeSigner = new DirectSigner(granteeWallet, signerConfig)
  const granteeAddress = (await granteeSigner.getAddresses())[0]

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