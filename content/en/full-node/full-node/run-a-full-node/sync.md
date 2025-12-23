---
title: Sync
weight: 60
type: docs
---

## Before Sync

Certain files will need to be absent or deleted prior to download. A quicksync replaces blockchain data with a custom snapshot. For most use cases a "pruned" version is adequate. Pruned versions will have certain transactions removed from the archive to improve node performance. If you are running a node for archival purposes, you will want an `archive` or `default` download.

After choosing the appropriate download type, examine your node and ensure that `.xpla/data` is empty.

**Example**:

```bash
6:22PM INF Removed all blockchain history dir=/home/ubuntu/.xpla/data
```

{{< alert context="warning" >}}
**Warning**

If you are a validator, ensure that you do not remove your private key.

Example of a removed private key:

```bash
6:22PM INF Reset private validator file to genesis state keyFile=/home/ubuntu/.xpla/config/priv_validator_key.json stateFile=/home/ubuntu/.xpla/data/priv_validator_state.json
```
{{< /alert >}}

If you have an address book downloaded, you may keep it. Otherwise, you will need to download the [appropriate addressbook]({{< ref "join-a-network#join-a-public-network" >}}) prior to running `xplad start`.

## During Sync

After [Joining a public network]({{< ref "join-a-network#join-a-public-network" >}}), your node will begin to sync.

{{< alert context="warning" >}}
**Sync start times**

Nodes take at least an hour to start syncing. This wait time is normal. Before troubleshooting a sync, please wait an hour for the sync to start.
{{< /alert >}}

## Monitor the Sync

Your node is catching up with the network by replaying all the transactions from genesis and recreating the blockchain state locally. You can verify this process by checking the `latest_block_height` in the `SyncInfo` of the `xplad status` response:

```json
  {
    "SyncInfo": {
        "latest_block_height": "42", <-----
        "catching_up"        : true
    },
  ...
  }
```

Compare this height to the **Latest Blocks** by checking the API for latest block heights on [dimension](https://dimension-lcd.xpla.dev/blocks/latest), or [cube](https://cube-lcd.xpla.dev/blocks/latest) to see your progress.

## State Sync

You can significantly accelerate the synchronization process by providing `xplad` with a recent snapshot of the network state. Snapshots are made publicly available by members of the CONX Chain community, and not maintained as part of this documentation.

## Sync Complete

You can tell that your node is in sync with the network when `SyncInfo.catching_up` in the `xplad status` response returns `false` and the `latest_block_height` corresponds to the public network blockheight found on the API for either [dimension](https://dimension-lcd.xpla.dev/blocks/latest), or [cube](https://cube-lcd.xpla.dev/blocks/latest).

```bash
xplad status
```

**Example**:

```json
  {
    "SyncInfo": {
        "latest_block_height": "7356350",
        "catching_up"        : false
    },
  ...
  }
```

Validators can view the status of the network using [XPLA Explorer](https://explorer.xpla.io).

## Sync faster during testing

Sometimes you may want to sync faster by foregoing checks.

{{< alert context="warning" >}}
**Warning**

The following command should only be used by advanced users in non-production environments:
```bash
xplad start --x-crisis-skip-assert-invariants
```
{{< /alert >}}

## Congratulations!

You've successfully joined a network as a full node operator. If you are a validator, continue to [manage a CONX Chain validator]({{< ref "/full-node/full-node/manage-a-validator/overview" >}}) for the next steps.
