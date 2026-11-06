---
title: Public and Private Endpoints
weight: 30
type: docs
---

## Ecosystem Endpoints

- TBD

## Public Endpoints

{{< alert context="warning" >}}
**Public endpoint warning**

Public RPC endpoints should only be used in development. For live dApps, use a private RPC endpoint from the list above or [set up your own endpoint]({{< ref "/full-node/full-node/run-a-full-node/system-configuration" >}}).
{{< /alert >}}

### Public LCD

| Endpoint                    | Network           |
| --------------------------- | ----------------- |
| https://dimension-lcd.xpla.dev | `dimension_37-1` mainnet |
| https://cube-lcd.xpla.dev | `cube_47-5` testnet   |

### Public RPC

| Endpoint | Network            |
| -------- | ------------------ |
| https://dimension-rpc.xpla.dev | `dimension_37-1` mainnet |
| https://cube-rpc.xpla.dev | `cube_47-5` testnet    |

To compare the live latency of the public mainnet RPC endpoints, including community-run ones, see the [OpenChainBench CONX (XPLA) RPC benchmark](https://openchainbench.com/benchmarks/xpla-rpc), which probes each one every 60 seconds from US-East, EU-West and Singapore, and publishes p50 / p90 / p99 latency and success rate.
