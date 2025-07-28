---
title: Coin and Coins
weight: 40
type: docs
---

A `Coin` represents a single coin, which is a pair consisting of a denomination and an amount. `Coins` represents a collection of `Coin` objects, that many operators use to group tokens in one construct.

```ts
import {addCoins, coin, coins} from "@interchainjs/amino/coins"

const c = coin("1500000000000000000", "axpla") // 1.5 XPLA
const c2 = coin("3000000000000000000", "axpla") // 3 XPLA

console.log(addCoins(c, c2)) // 4.5 XPLA

const cs = coins("1500000000000000000", "axpla")
cs.push(coin("3000000000000000000", "axpla"))
cs.map((c) => {
    console.log(`${c.denom}: ${c.amount}`)
})
```

Although it is convenient to represent the numbers through JavaScript's native `Number` format, you must use bigint, and if you need decimal operations you have to use Decimal.
