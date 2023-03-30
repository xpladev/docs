---
title: Interacting with the Contract
weight: 50
---

{{< alert >}}
**Note**

You can also follow these steps in the official web wallet for XPLA Chain, [XPLA Vault](https://vault.xpla.io).
{{< /alert >}}

## Requirements

You should also have the latest version of `xplad` by building the latest version of XPLA Chain core. You will configure `xplad` to use it against your isolated testnet environment.

In a separate terminal, make sure to set up the following mnemonic:

```sh
xplad keys add test1 --recover
```

Using the mnemonic:

```
satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn
```

## Uploading Code

Make sure that the **optimized build** of `my_first_contract.wasm` that you created in the last section is in your current working directory.

```sh
xplad tx wasm store artifacts/my_first_contract.wasm --from test1 --chain-id=cube_47-5 --gas=auto --gas-prices 850000000000axpla --gas-adjustment 1.2 --broadcast-mode=block
```
Or, if you are on an arm64 machine:

```sh
xplad tx wasm store artifacts/my_first_contract-aarch64.wasm --from test1 --chain-id=cube_47-5 --gas=auto --gas-prices 850000000000axpla --gas-adjustment 1.2 --broadcast-mode=block
```

You should see output similar to the following:

```sh
code: 0
codespace: ""
data: 0A250A1E2F636F736D7761736D2E7761736D2E76312E4D736753746F7265436F6465120308C702
events:
- attributes:
  - index: true
    key: c3BlbmRlcg==
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  - index: true
    key: YW1vdW50
    value: MTAwMjI1MjAwMDAwMDAwMDAwMGF4cGxh
  type: coin_spent
- attributes:
  - index: true
    key: cmVjZWl2ZXI=
    value: eHBsYTE3eHBmdmFrbTJhbWc5NjJ5bHM2Zjg0ejNrZWxsOGM1bHc3bXVxdw==
  - index: true
    key: YW1vdW50
    value: MTAwMjI1MjAwMDAwMDAwMDAwMGF4cGxh
  type: coin_received
- attributes:
  - index: true
    key: cmVjaXBpZW50
    value: eHBsYTE3eHBmdmFrbTJhbWc5NjJ5bHM2Zjg0ejNrZWxsOGM1bHc3bXVxdw==
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  - index: true
    key: YW1vdW50
    value: MTAwMjI1MjAwMDAwMDAwMDAwMGF4cGxh
  type: transfer
- attributes:
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: message
- attributes:
  - index: true
    key: ZmVl
    value: MTAwMjI1MjAwMDAwMDAwMDAwMGF4cGxh
  - index: true
    key: ZmVlX3BheWVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: tx
- attributes:
  - index: true
    key: YWNjX3NlcQ==
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYy80
  type: tx
- attributes:
  - index: true
    key: c2lnbmF0dXJl
    value: VzNTZUpMbExvYUlGU21mQkJXdWl2N3dPRFd0NHdKUUhZWXRaZ0VYR1pSTktYeEplbG9jVjBmMkttOEY3VzBITktZUHVQNEk5WXlOT251R3VxdUJGdndBPQ==
  type: tx
- attributes:
  - index: true
    key: YWN0aW9u
    value: L2Nvc213YXNtLndhc20udjEuTXNnU3RvcmVDb2Rl
  type: message
- attributes:
  - index: true
    key: bW9kdWxl
    value: d2FzbQ==
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: message
- attributes:
  - index: true
    key: Y29kZV9pZA==
    value: MzI3
  type: store_code
gas_used: "997846"
gas_wanted: "1179120"
height: "2580633"
info: ""
logs:
- events:
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgStoreCode
    - key: module
      value: wasm
    - key: sender
      value: xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc
    type: message
  - attributes:
    - key: code_id
      value: "327"
    type: store_code
  log: ""
  msg_index: 0
raw_log: '[{"events":[{"type":"message","attributes":[{"key":"action","value":"/cosmwasm.wasm.v1.MsgStoreCode"},{"key":"module","value":"wasm"},{"key":"sender","value":"xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc"}]},{"type":"store_code","attributes":[{"key":"code_id","value":"327"}]}]}]'
timestamp: ""
tx: null
txhash: FB6E917629BA304ABB6316584FA24CAEB55F772C289757D7D77E500BE364A8B9
```

As you can see, your contract was successfully instantiated with Code ID #1.

You can check it out:

```sh
xplad query wasm code-info 327

code_id: "327"
creator: xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc
data_hash: 1B7CD6D73B86DFEF2A06A8B67253E0039DE8C6916DB1B9A93604B8DDFBDFD677
instantiate_permission:
  address: ""
  addresses: []
  permission: Everybody
```

## Creating the Contract

You have now uploaded the code for your contract, but still don't have a contract. Create it with the following InitMsg:

```json
{
  "count": 0
}
```

You can compress the JSON into 1 line with [this online tool](https://goonlinetools.com/json-minifier/).

```sh
xplad tx wasm instantiate 327 '{"count":0}' --from test1 --chain-id=cube_47-5 --label counter --no-admin --gas auto --gas-prices 850000000000axpla --gas-adjustment 1.2 --broadcast-mode=block
```

You should get a response like the following:

```sh
code: 0
codespace: ""
data: 0A6D0A282F636F736D7761736D2E7761736D2E76312E4D7367496E7374616E7469617465436F6E747261637412410A3F78706C6131346D3067613874793466326171746568386B397A39346A6470796D687436616B6A33646D36616E6561663734736E7664326C30717135396D6C34
events:
- attributes:
  - index: true
    key: c3BlbmRlcg==
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  - index: true
    key: YW1vdW50
    value: MTY4MjU5MjAwMDAwMDAwMDAwYXhwbGE=
  type: coin_spent
- attributes:
  - index: true
    key: cmVjZWl2ZXI=
    value: eHBsYTE3eHBmdmFrbTJhbWc5NjJ5bHM2Zjg0ejNrZWxsOGM1bHc3bXVxdw==
  - index: true
    key: YW1vdW50
    value: MTY4MjU5MjAwMDAwMDAwMDAwYXhwbGE=
  type: coin_received
- attributes:
  - index: true
    key: cmVjaXBpZW50
    value: eHBsYTE3eHBmdmFrbTJhbWc5NjJ5bHM2Zjg0ejNrZWxsOGM1bHc3bXVxdw==
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  - index: true
    key: YW1vdW50
    value: MTY4MjU5MjAwMDAwMDAwMDAwYXhwbGE=
  type: transfer
- attributes:
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: message
- attributes:
  - index: true
    key: ZmVl
    value: MTY4MjU5MjAwMDAwMDAwMDAwYXhwbGE=
  - index: true
    key: ZmVlX3BheWVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: tx
- attributes:
  - index: true
    key: YWNjX3NlcQ==
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYy81
  type: tx
- attributes:
  - index: true
    key: c2lnbmF0dXJl
    value: T2FnSjZWNyswemVkczNlYXgwbU50dmVFZmpEemF4NUwzZVNRRFlBS3h0VmgveGxMTEZ6R3R6VGJxL0V0bVRrUDhLb2RHL3NvTytjZW54Tm4yc1ljandFPQ==
  type: tx
- attributes:
  - index: true
    key: YWN0aW9u
    value: L2Nvc213YXNtLndhc20udjEuTXNnSW5zdGFudGlhdGVDb250cmFjdA==
  type: message
- attributes:
  - index: true
    key: bW9kdWxl
    value: d2FzbQ==
  - index: true
    key: c2VuZGVy
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  type: message
- attributes:
  - index: true
    key: X2NvbnRyYWN0X2FkZHJlc3M=
    value: eHBsYTE0bTBnYTh0eTRmMmFxdGVoOGs5ejk0amRweW1odDZha2ozZG02YW5lYWY3NHNudmQybDBxcTU5bWw0
  - index: true
    key: Y29kZV9pZA==
    value: MzI3
  type: instantiate
- attributes:
  - index: true
    key: X2NvbnRyYWN0X2FkZHJlc3M=
    value: eHBsYTE0bTBnYTh0eTRmMmFxdGVoOGs5ejk0amRweW1odDZha2ozZG02YW5lYWY3NHNudmQybDBxcTU5bWw0
  - index: true
    key: bWV0aG9k
    value: aW5zdGFudGlhdGU=
  - index: true
    key: b3duZXI=
    value: eHBsYTF6Mms4NW40OHlkZnZ6c2xydWd3emw0ajJ1N3Z0ZHlmM3h2dWNtYw==
  - index: true
    key: Y291bnQ=
    value: MA==
  type: wasm
gas_used: "180168"
gas_wanted: "197952"
height: "2580676"
info: ""
logs:
- events:
  - attributes:
    - key: _contract_address
      value: xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4
    - key: code_id
      value: "327"
    type: instantiate
  - attributes:
    - key: action
      value: /cosmwasm.wasm.v1.MsgInstantiateContract
    - key: module
      value: wasm
    - key: sender
      value: xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc
    type: message
  - attributes:
    - key: _contract_address
      value: xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4
    - key: method
      value: instantiate
    - key: owner
      value: xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc
    - key: count
      value: "0"
    type: wasm
  log: ""
  msg_index: 0
raw_log: '[{"events":[{"type":"instantiate","attributes":[{"key":"_contract_address","value":"xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4"},{"key":"code_id","value":"327"}]},{"type":"message","attributes":[{"key":"action","value":"/cosmwasm.wasm.v1.MsgInstantiateContract"},{"key":"module","value":"wasm"},{"key":"sender","value":"xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc"}]},{"type":"wasm","attributes":[{"key":"_contract_address","value":"xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4"},{"key":"method","value":"instantiate"},{"key":"owner","value":"xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc"},{"key":"count","value":"0"}]}]}]'
timestamp: ""
tx: null
txhash: 8847389C9236BD91A88C090EDE77A9B4311785B7B7546E87262463C7CBF443EE
```

From the output, you can see that your contract was created above at: `xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4`. Take note of this contract address, as you will need it for the next section.

Check out your contract information:

```sh
xplad query wasm contract xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4
contract_info:
  admin: ""
  code_id: "327"
  created: null
  creator: xpla1z2k85n48ydfvzslrugwzl4j2u7vtdyf3xvucmc
  extension: null
  ibc_port_id: ""
  label: counter
```

You can use the following to decode the Base64 InitMsg:

```sh
echo eyJjb3VudCI6MH0= | base64 --decode
```

This will produce the message you used when initializing the contract:

```json
{ "count": 0 }
```

## Executing the Contract

Let's do the following:

1. Reset count to 5
2. Increment twice

If done properly, you should get a count of 7.

#### Reset

First, to burn:

```json
{
  "reset": {
    "count": 5
  }
}
```

```sh
xplad tx wasm execute xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4 '{"reset":{"count":5}}' --from test1 --chain-id=cube_47-5 --gas auto --gas-prices 850000000000axpla --gas-adjustment 1.2 --broadcast-mode=block
```

#### Incrementing

```json
{
  "increment": {}
}
```

```sh
xplad tx wasm execute xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4 '{"increment":{}}' --from test1 --chain-id=cube_47-5 --gas auto --gas-prices 850000000000axpla --gas-adjustment 1.2 --broadcast-mode=block
```

#### Querying count

Check the result of your executions!

```json
{
  "get_count": {}
}
```

```sh
xplad query wasm contract-state smart xpla14m0ga8ty4f2aqteh8k9z94jdpymht6akj3dm6aneaf74snvd2l0qq59ml4 '{"get_count":{}}'
```

Expected output:

```
query_result:
  count: 7
```

Excellent! Congratulations, you've created your first smart contract, and now know how to get developing with the XPLA Chain dApp Platform.

## What's Next?

So far you've walked through a simple example of a smart contract, that modifies a simple balance within its internal state. Although this is enough to make a simple dApp, you can power more interesting applications by **emitting messages**, which will enable you to interact with other contracts as well as the rest of the blockchain's module.
