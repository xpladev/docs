---
title: Coin and Coins
weight: 40
type: docs
---

A `Coin` represents a single coin, which is a pair consisting of a denomination and an amount. `Coins` represents a collection of `Coin` objects, that many operators use to group tokens in one construct.

```ts
import { Coin, Coins } from "@xpla/xpla.js";

const c = new Coin("axpla", 1500000000000000000); // 1.5 XPLA
const c2 = new Coin("axpla", 3000000000000000000); // 3 XPLA
c.add(c2); // 4.5 XPLA

const cs = new Coins([c, c2]);
cs.map((x) => console.log(`${x.denom}: ${x.amount}`));
```

Coin / Coins input with decimal input will automatically be converted to a decimal Coin.

```ts
const c = new Coin("axpla", 123.3); // a DecCoin
const d = new Coin("axpla", "123.3"); // a DecCoin
```

Although it is convenient to represent the numbers through JavaScript's native `Number` format, you should refrain from doing so.
