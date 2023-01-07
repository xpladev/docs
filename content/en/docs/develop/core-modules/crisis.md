---
title: Crisis
weight: 60
---

{{< hint info >}}
**Note**
XPLA Chain's crisis module inherits from the Cosmos SDK's [`crisis`](https://docs.cosmos.network/master/modules/crisis/) module. This document is a stub and covers mainly important XPLA Chain-specific notes about how it is used.
{{< /hint >}}

The crisis module stops block production by halting the blockchain when an invariant is broken. Invariants are set during the initialization process.

## Constant Fee

Because the gas cost of verifying an invariant is high, the crisis module utilizes a constant fee instead of the typical gas calculation method.
