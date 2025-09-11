---
title: Query Data
weight: 110
type: docs
---

## Querying data

After you’re connected to the blockchain via an RPCQueryClient instance, you can query data from it. Data access is organized into various module APIs, which are accessible from within the RPCQueryClient instance. Because they make HTTP requests in the background, they are Promises that can be awaited in order to not block during network IO.

Each module has its own set of querying functions.


```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";

const client = await createRPCQueryClient({rpcEndpoint:  "https://cube-rpc.xpla.io"})
const res = await client.cosmos.bank.v1beta1.balance({address: "xpla1npvwllfr9dqr8erajqqr6s0vxnk2ak55hh2h5f", denom: "axpla"})
console.log(res)
// { balance: { denom: 'axpla', amount: '74305119519999999998' } }
```