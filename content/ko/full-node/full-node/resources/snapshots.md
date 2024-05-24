---
title: Snapshots
weight: 20
type: docs
---

The snapshot service for both XPLA Mainnet (dimension_37-1) and the Testnet (cube_47-5) offered by Bware Labs are accessible [here](https://bwarelabs.com/snapshots).

If you require assistance with bootstrapping your full node or need a snapshot to facilitate the migration or initiation of your validator, Bware Labs' daily snapshot service can simplify the process for you.

**Please note that the snapshot nodes are configured with the following pruning values, with state-sync and indexer turned off, and are hosted as Docker containers on machines running Ubuntu 22.04.**

```toml
# in app.toml
pruning = "custom"
pruning-keep-recent = "100"
pruning-keep-every = "0"
pruning-interval = "10"
```
