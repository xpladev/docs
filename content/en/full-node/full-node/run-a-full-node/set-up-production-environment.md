---
title: Set-up Production Environment
weight: 30
type: docs
---

Use the following information to set up and manage your production-level XPLA Chain full node.

For information about running a validator node, visit the [validator guide]({{< ref "/full-node/full-node/manage-a-validator/overview" >}}).

## Create a Dedicated User

Although `xplad` does not require a super user account, during the setup process you'll need super user permission to create and modify some files. It is strongly recommended using a normal user when running `xplad`.

## Increase the Maximum Files `xplad` Can Open

`xplad` is set to open 1024 files by default. It is recommended that you increase this amount.

Modify `/etc/security/limits.conf`[\*](https://linux.die.net/man/5/limits.conf) to increase the amount, where `nofile` is the number of files `xplad` can open.

```bash
# If you have never changed this system config or your system is fresh, most of this file will be commented
# ...
*                soft    nofile          65535   # Uncomment the following two lines at the bottom
*                hard    nofile          65535   # Change the default values to ~65535
# ...
```

# Run the Server as a Daemon

`xplad` must be running at all times. It is recommended that you register `xplad` as a `systemd` service so that it will be started automatically when the system reboots.

## Register `xplad` as a Service

1. Create a service definition file in `/etc/systemd/system/xplad.service`.

   **Example**:

   ```bash
   [Unit]
   Description=XPLA Chain Daemon
   After=network.target

   [Service]
   Type=simple
   User=<XPLA_USER>
   ExecStart=<PATH_TO_XPLAD>/xplad start
   Restart=on-abort

   [Install]
   WantedBy=multi-user.target

   [Service]
   LimitNOFILE=65535
   ```

2. Modify the `Service` section according to your environment:

   - Enter the user (likely your username, unless you created a user specifically for `xplad`)
   - Enter the path to the `xplad` executable. `<PATH_TO_XPLAD>` is likely `/home/<YOUR_USER>/go/bin/xplad` or `/usr/go/bin`. Confirm this with `whereis xplad`
   - Make sure you made the correct edits to /etc/security/limits.conf

3. Run `systemctl daemon-reload` followed by `systemctl enable xplad`. This will register `xplad` as a system service and turn it on upon startup.

4. Now start the service with `systemctl start xplad`.

### Controlling the Service

Use `systemctl` to start, stop, and restart the service:

```bash
# Check health
systemctl status xplad
# Start
systemctl start xplad
# Stop
systemctl stop xplad
# Restart
systemctl restart xplad
```

### Access Logs

Use `journalctl -t` to access entire logs, entire logs in reverse, and the latest and continuous log.

```bash
# Entire log reversed
journalctl -t xplad -r
# Entire log
journalctl -t xplad
# Latest and continuous
journalctl -t xplad -f
```
