---
title: Numeric
weight: 100
---

xpla.js includes `Dec` and `Int`, which represent decimal numbers and integer numbers compatible with the Cosmos-SDK.

```ts
import { Dec, Int } from '@xpla/xpla.js';

// conversion into dec
const d = new Dec(123.11);

// addition
d.add(3).sub(5).div(3).mod(2);
