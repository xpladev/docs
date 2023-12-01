---
title: System Configuration
weight: 10
type: docs
---

{{< alert context="warning" >}}
**Recommended operating systems**

This guide has been tested against Linux distributions only. To ensure a successful production environment setup, consider using a Linux system.
{{< /alert >}}

Running a full XPLA Chain node is a resource-intensive process that requires a persistent server. If you want to use XPLA Chain without downloading the entire blockchain, use [XPLA web vault](https://vault.xpla.io/).

## Hardware Requirements

The minimum requirements for running a XPLA Chain full node are:

| Network                                                                | CPU cores      | RAM   | Disk                       | BANDWIDTH |
|------------------------------------------------------------------------|----------------|-------|----------------------------|-----------|
| [`dimension_37-1`](../join-a-network#join-a-public-network)            | 4 (+4 threads) | 32 GB | 2 TB (SSD 2000 MB/s R/W)   | 300 Mbps  |
| [`cube_47-5`](../join-a-network#join-a-public-network)                 | 2 (+2 threads) | 16 GB | 500 GB (SSD 1000 MB/s R/W) | 150 Mbps  |
| [`private-network`](../join-a-network#start-your-private-xpla-network) | 1              | 2 GB  | 20 GB (SSD 500 MB/s R/W)   | N/A       |

{{< alert context="warning" >}}
**Storage requirements**

As the network grows, the minimum hardware requirements will also grow. It is recommended that you monitor the system so you can prevent it from running out of resources.
{{< /alert >}}

## Prerequisites

- [Golang v1.18+ linux/amd64](https://go.dev/dl/)

  {{< details "Installing Go for macOS & Linux" >}}
  Go releases can be found here: [ https://go.dev/dl/ ](https://go.dev/dl/)

  In your browser, you can right-click the correct release (V1.18) and `Copy link`.

  ```bash
  # 1. Download the archive

  wget https://go.dev/dl/go1.18.2.linux-amd64.tar.gz

  # Optional: remove previous /go files:

  sudo rm -rf /usr/local/go

  # 2. Unpack:

  sudo tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz

  # 3. Add the path to the go-binary to your system path:
  # (for this to persist, add this line to your ~/.profile or ~/.bashrc or  ~/.zshrc)

  export PATH=$PATH:/usr/local/go/bin

  # 4. Verify your installation:

  go version

  # go version go1.18.2 linux/amd64

  ```
  {{< /details >}}

- Linux users: `sudo apt-get install -y build-essential`

## Commonly Used Ports

`xplad` uses the following TCP ports. Toggle their settings to match your environment.

Most validators will only need to open the following port:

- `26656`: The default port for the P2P protocol. This port is used to communicate with other nodes and must be open to join a network. However, it does not have to be open to the public. For validator nodes, [configuring `persistent_peers`]({{< ref "updates-and-additional-settings#additional-settings" >}}) and closing this port to the public are recommended.

Additional ports:

- `1317`: The default port for the [Light Client Daemon]({{< ref "start-the-Light-Client-Daemon" >}}) (LCD), which can be executed by `xplad rest-server`. The LCD provides an HTTP RESTful API layer to allow applications and services to interact with your `xplad` instance through RPC. For usage examples, see [XPLA Chain REST API](https://dimension-lcd.xpla.dev/swagger/). You don't need to open this port unless you have use for it.

- `26660`: The default port for interacting with the [Prometheus](https://prometheus.io) database, which can be used to monitor the environment. In the default configuration, this port is not open.

- `26657`: The default port for the RPC protocol. Because this port is used for querying and sending transactions, it must be open for serving queries from `xplad`.

{{< alert context="warning" >}}
**Warning**

Do not open port `26657` to the public unless you plan to run a public node.
{{< /alert >}}
