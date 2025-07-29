---
title: Fees
weight: 50
type: docs
---

```ts
const fee: StdFee = {
    amount: [Coin.fromPartial({denom: "axpla", amount: "28000000000000000"})],
    gas: "100000",
}
const tx = await signer.signAndBroadcast({messages: [msg], fee: fee})
console.log(tx)
```

## Automatic Fee Estimation

If you don't specify a fee when you create your transaction, it will automatically be estimated by simulating it within the node.

```ts
const tx = await signer.signAndBroadcast({messages: [msg], fee: fee})
```

### How to calculate accurate Fee
The following example demonstrates how to automatically estimate transaction fees using the `estimateFee` method. This approach is more accurate than manually setting fees as it simulates the transaction on the network to determine the exact gas requirements:

```ts
const fee: StdFee = await signer.estimateFee({messages: [msg], options: {multiplier: 1.3}})
const tx = await signer.signAndBroadcast({messages: [msg], fee: fee})
```
