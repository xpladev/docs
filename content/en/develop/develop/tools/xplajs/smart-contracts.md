---
title: Smart Contracts
weight: 130
type: docs
---

This document explains how to perform tasks related to working with smart contracts with xplajs.

## Upload Code

You will first need a compiled WASM smart contract's binary to upload. The following example demonstrates how to upload a smart contract to the XPLA blockchain using the `MsgStoreCode` message type.

The process involves creating a DirectSigner, reading the compiled WASM binary file, constructing a store code message, and then signing and broadcasting the transaction. After the transaction is processed, you can extract the code ID from the transaction response, which is needed for contract instantiation.

**Note**: The `counter.wasm` file in this example is compiled from the [CosmWasm cw-template](https://github.com/CosmWasm/cw-template), which provides a quickstart template for building smart contracts in Rust.

```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { Network } from "@xpla/xpla/defaults"
import { MsgStoreCode, MsgInstantiateContract, MsgExecuteContract } from "@xpla/xplajs/cosmwasm/wasm/v1/tx";
import { MessageComposer } from "@xpla/xplajs/cosmwasm/wasm/v1/tx.registry";
import { toEncoders } from "@interchainjs/cosmos/utils"
import fs from "fs"

const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const [auth] = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
const signer = new DirectSigner(auth, toEncoders(MsgStoreCode, MsgInstantiateContract, MsgExecuteContract), Network.Testnet.rpc);

const msgStoreCode = MsgStoreCode.fromPartial({
    sender: await signer.getAddress(),
    wasmByteCode: Buffer.from(fs.readFileSync("./counter.wasm"))
})
const { storeCode } = await MessageComposer.fromPartial
const msg = await storeCode(msgStoreCode)
const txResponse = await signer.signAndBroadcast({messages: [msg]})

console.log("Waiting for transaction to be included in block...")
await new Promise(resolve => setTimeout(resolve, 10000))

const client = await createRPCQueryClient({rpcEndpoint: Network.Testnet.rpc})
const res = await client.cosmos.tx.v1beta1.getTx({hash: txResponse.hash || ""})
const codeId = res.txResponse?.events.find(e => e.type === "store_code")?.attributes.find(a => a.key === "code_id")?.value
console.log(codeId)
```

## Create a Contract

For XPLA Chain smart contracts, there is a distinction between uploading contract code and creating a contract. This allows multiple contracts to share the same code if there are only minor variations in their logic which can be configured at contract creation. This configuration is passed in an **InitMsg**, and provides the initial state for the contract.

To create or instantiate a smart contract, you must first know the code ID of an uploaded code. You will reference it in a `MsgInstantiateContract` alongside the InitMsg to create the contract. Upon successful creation, your contract will be located at an address that you specify.

```ts
const codeId = 1761n
const sender = await signer.getAddress()
const msgInstantiateContract = MsgInstantiateContract.fromPartial({
    sender,
    codeId,
    label: "counter",
    msg: new Uint8Array(Buffer.from(`{"count": 0}`)),
    admin: sender 
})
const { instantiateContract } = await MessageComposer.fromPartial
const msg = await instantiateContract(msgInstantiateContract)
const txResponse = await signer.signAndBroadcast({messages: [msg]})

console.log("Waiting for transaction to be included in block...")
await new Promise(resolve => setTimeout(resolve, 10000))

const client = await createRPCQueryClient({rpcEndpoint: Network.Testnet.rpc})
const res = await client.cosmos.tx.v1beta1.getTx({hash: txResponse.hash || ""})
const contractAddress = res.txResponse?.events.find(e => e.type === "instantiate")?.attributes.find(a => a.key === "_contract_address")?.value
console.log(contractAddress)
```

## Execute a Contract

Smart contracts respond to JSON messages called **HandleMsg** which can exist as different types. The smart contract writer should provide any end-users of the smart contract with the expected format of all the varieties of HandleMsg the contract is supposed to understand, in the form of a JSON schema. The schema thus provides an analog to Ethereum contracts' ABI.

```ts
const contractAddress = "xpla1cspdyqa2722k5e5wfhhuf9tehy4yh5ngy83yd0gg48x7emjvjjasmg76fz"
const sender = await signer.getAddress()
const msgExecuteContract = MsgExecuteContract.fromPartial({
    sender,
    contract: contractAddress,
    msg: new Uint8Array(Buffer.from(`{"increment": {}}`))
})
const { executeContract } = await MessageComposer.fromPartial
const msg = await executeContract(msgExecuteContract)
const txResponse = await signer.signAndBroadcast({messages: [msg]})
console.log(txResponse)
```

## Query Data from a Contract

A contract can define a query handler, which understands requests for data specified in a JSON message called a QueryMsg. Unlike the message handler, the query handler cannot modify the contract's or blockchain's state -- it is a readonly operation. Therefore, a querying data from a contract does not use a message and transaction, but works directly through the `RPC Client` API.

```ts
const contractAddress = "xpla1cspdyqa2722k5e5wfhhuf9tehy4yh5ngy83yd0gg48x7emjvjjasmg76fz"
const res = await client.cosmwasm.wasm.v1.smartContractState({
    address: contractAddress, 
    queryData: new Uint8Array(Buffer.from(`{"get_count": {}}`))
})

const { count } = JSON.parse(new TextDecoder().decode(res.data));
console.log(count)
```
