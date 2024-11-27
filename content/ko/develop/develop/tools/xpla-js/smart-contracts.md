---
title: Smart Contracts
weight: 130
type: docs
---

This document explains how to perform tasks related to working with smart contracts with xpla.js.

## Upload Code

You will first need a compiled WASM smart contract's binary to upload.

```ts
import { LCDClient, MsgStoreCode, MnemonicKey, isTxError } from '@xpla/xpla.js';
import * as fs from 'fs';

const mk = new MnemonicKey({
  mnemonic: 'notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius'
})

const xpla = new LCDClient({
  URL: 'https://cube-lcd.xpla.dev',
  chainID: 'cube_47-5'
});

const wallet = xpla.wallet(mk);

const storeCode = new MsgStoreCode(
  wallet.key.accAddress,
  fs.readFileSync('contract.wasm').toString('base64')
);
const storeCodeTx = await wallet.createAndSignTx({
  msgs: [storeCode],
});
const storeCodeTxResult = await xpla.tx.broadcast(storeCodeTx);

console.log(storeCodeTxResult);

if (isTxError(storeCodeTxResult)) {
  throw new Error(
    `store code failed. code: ${storeCodeTxResult.code}, codespace: ${storeCodeTxResult.codespace}, raw_log: ${storeCodeTxResult.raw_log}`
  );
}

const {
  store_code: { code_id },
} = storeCodeTxResult.logs[0].eventsByType;
```

## Create a Contract

For XPLA Chain smart contracts, there is a distinction between uploading contract code and creating a contract. This allows multiple contracts to share the same code if there are only minor variations in their logic which can be configured at contract creation. This configuration is passed in an **InitMsg**, and provides the initial state for the contract.

To create or instantiate a smart contract, you must first know the code ID of an uploaded code. You will reference it in a `MsgInstantiateContract` alongside the InitMsg to create the contract. Upon successful creation, your contract will be located at an address that you specify.

```ts
import { MsgInstantiateContract } from '@xpla/xpla.js';


const instantiate = new MsgInstantiateContract(
  wallet.key.accAddress,
  +code_id[0], // code ID
  {
    count: 0,
  }, // InitMsg
  { axpla: 10000000 }, // init coins
  false // migratable
);

const instantiateTx = await wallet.createAndSignTx({
  msgs: [instantiate],
});
const instantiateTxResult = await xpla.tx.broadcast(instantiateTx);

console.log(instantiateTxResult);

if (isTxError(instantiateTxResult)) {
  throw new Error(
    `instantiate failed. code: ${instantiateTxResult.code}, codespace: ${instantiateTxResult.codespace}, raw_log: ${instantiateTxResult.raw_log}`
  );
}

const {
  instantiate_contract: { contract_address },
} = instantiateTxResult.logs[0].eventsByType;
```

## Execute a Contract

Smart contracts respond to JSON messages called **HandleMsg** which can exist as different types. The smart contract writer should provide any end-users of the smart contract with the expected format of all the varieties of HandleMsg the contract is supposed to understand, in the form of a JSON schema. The schema thus provides an analog to Ethereum contracts' ABI.

```ts
import { MsgExecuteContract } from '@xpla/xpla.js';

const execute = new MsgExecuteContract(
  wallet.key.accAddress, // sender
  contract_address[0], // contract account address
  { ...executeMsg }, // handle msg
  { axpla: 100000 } // coins
);

const executeTx = await wallet.createAndSignTx({
  msgs: [execute]
});

const executeTxResult = await xpla.tx.broadcast(executeTx);
```

## Query Data from a Contract

A contract can define a query handler, which understands requests for data specified in a JSON message called a QueryMsg. Unlike the message handler, the query handler cannot modify the contract's or blockchain's state -- it is a readonly operation. Therefore, a querying data from a contract does not use a message and transaction, but works directly through the `LCDClient` API.

```ts
const result = await xpla.wasm.contractQuery(
  contract_address[0],
  { query: { queryMsgArguments } } // query msg
);
```
