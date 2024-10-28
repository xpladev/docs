---
title: Troubleshoot Validator Problems
weight: 60
type: docs
---

Use this guide to solve the most common validator problems.

## Validator Has 0 Voting Power

If your validator has 0 voting power, your validator has become auto-unbonded. On the mainnet, validators unbond when they do not vote on `9500` of the last `10000` blocks (`50` of the last `100` blocks on the testnet). Because blocks are proposed every ~5 seconds, a validator that is unresponsive for ~13 hours (~4 minutes on testnet) become unbonded. This problem usually happens when your `xplad` process crashes.

To return the voting power back to your validator:

1. If `xplad` is not running, restart it:

   ```bash
   xplad start
   ```

1. Wait for your full node to reach the latest block, and run:

   ```bash
   xplad tx slashing unjail <xpla> --chain-id=<chain_id> --from=<from>
   ```

   - `<xpla>` is the address of your validator account.
   - `<name>` is the name of the validator account. To find this information, run `xplad keys list`.

   {{< alert context="warning" >}}
   **Warning**

   If you don't wait for `xplad` to sync before running `unjail`, an error message will inform you that your validator is still jailed.
   {{< /alert >}}

1. Check your validator again to see if your voting power is back:

   ```bash
   xplad status
   ```

   If your voting power is less than it was previously, you may have been slashed for downtime.

## `xplad` Crashes Because of Too Many Open Files

The default number of files Linux can open per process is `1024`. `xplad` is known to open more than this amount, causing the process to crash.

1. Increase the number of open files allowed by running `ulimit -n 4096`.

2. Restart the process with `xplad start`.

   If you are using `systemd` or another process manager to launch `xplad`, you might need to configure them. The following  sample `systemd` file fixes the problem:

   ```systemd
   # /etc/systemd/system/xplad.service
   [Unit]
   Description=XPLA Chain Node Daemon
   After=network.target

   [Service]
   Type=simple
   User=ubuntu
   WorkingDirectory=/home/ubuntu
   ExecStart=/home/ubuntu/go/bin/xplad start
   Restart=on-failure
   RestartSec=3
   LimitNOFILE=65536

   [Install]
   WantedBy=multi-user.target
   ```

## `xplad` Crashes Because of Memory Fragmentation

Huge memory allocation can cause memory fragmentation issue. Temporal solution is just using small wasm cache size like 50~100MB.

```toml
contract-memory-cache-size = 100
```

## `xplad` Crashes Because of Exceeding Message Max Size

Default message max size is too small for peer service.

```
xplad[123793]: 1:47AM ERR Stopping peer for error err="message exceeds max size (1406 > 1034)" module=p2p peer={"Data":{},"Logger":{}}
```

Increase `max_packet_msg_payload_size` in `config.toml` to `2048`

```toml
max_packet_msg_payload_size = 2048
```

## The Validator is Not Active

- The validator is jailed. To solve this problem, `unjail` the validator by running:

    `xplad tx slashing unjail <xpla> --chain-id=<chain_id> --from=<from>`

- The validator is not in the [active validator set]({{< ref "glossary#active-set" >}}). Only the top 40 validators are in this set. To fix this problem, increase your total stake to be larger than the 8th validator.
