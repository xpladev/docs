---
title: Decimal
weight: 100
type: docs
---

xplajs includes `Decimal` and `Integer`, which represent decimal numbers and integer numbers compatible with the Cosmos-SDK.

```ts
import { Decimal, Uint64 } from "@interchainjs/math"

const d = Decimal.fromAtomics("12311", 2)
const d2 = Decimal.fromAtomics("300", 2)
const i = Uint64.fromNumber(2)

d.plus(d2).minus(d2).multiply(i)
```

When working with different decimal places, you need to ensure the fractional digits match. If they don't match, you should convert them to the same precision

**Important**: Always ensure fractional digits match before performing mathematical operations to avoid precision errors.


