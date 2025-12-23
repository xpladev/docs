---
title: Crisis
weight: 60
type: docs
---

{{< alert >}}
**Note**

CONX Chain's crisis module inherits from the Cosmos SDK's [`crisis`](https://docs.cosmos.network/v0.45/modules/crisis/) module. This document is a stub and covers mainly important CONX Chain-specific notes about how it is used.
{{< /alert >}}

The crisis module halts the blockchain under the circumstance that a blockchain invariant is broken. Invariants can be registered with the application during the application initialization process.

## Constant Fee

Because the gas cost of verifying an invariant is high, the crisis module utilizes a constant fee instead of the typical gas calculation method.
