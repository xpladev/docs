---
title: Install `xplad`
weight: 20
---

`xplad` is the command-line interface and daemon that connects to XPLA Chain and enables you to interact with the XPLA Chain. XPLA Chain core is the official Golang reference implementation of the XPLA Chain node software.

This guide is for developers who want to install `xplad` and interact with XPLA Chain core without running a full node. If you want to run a full node or join a network, visit [Run a Full Node]({{< ref "run-a-full-node" >}}).

### Prerequisites

- [Golang v1.18 linux/amd64](https://golang.org/doc/install)
- Ensure your `GOPATH` and `GOBIN` environment variables are set up correctly.
- Linux users: install [build-essential](http://linux-command.org/en/build-essential.html).

{{< hint warning >}}
**xplad for Mac**
If you are using a Mac, follow the [`xplad` Mac installation guide]({{< ref "xplad-mac" >}}).
{{< /hint >}}

## From Binary

The easiest way to install `xplad` and XPLA Chain core is by downloading a pre-built binary for your operating system. You can find the latest binaries on the [releases](https://github.com/xpladev/xpla/releases) page. If you have a Mac, follow the [Mac installation instructions]({{< ref "xplad-mac" >}}).

## From Source

### 1. Get the XPLA Chain core source code

Use `git` to retrieve [XPLA Chain core](https://github.com/xpladev/xpla/), and check out the `main` branch, which contains the latest stable release.

```
git clone https://github.com/xpladev/xpla
cd xpla
git checkout [latest version]
```

### 2. Build XPLA Chain core from source

Build XPLA Chain core, and install the `xplad` executable to your `GOPATH` environment variable.

```bash
make install
```

### 3. Verify your XPLA Chain core installation

Verify that XPLA Chain core is installed correctly.

```bash
xplad version --long
```

The following example shows version information when XPLA Chain core is installed correctly:

```bash
name: xpla
server_name: xplad
version: v0.0.5
commit: d947adaefadda0f29c92f18e8b33f769816f3c33
build_tags: netgo,ledger
go: go version go1.18.4 darwin/amd64
```

{{< hint info >}}
**Tip**
If the `xplad: command not found` error message is returned, confirm that the Go binary path is correctly configured by running the following command:
```sh
export PATH=$PATH:$(go env GOPATH)/bin
```
{{< /hint >}}

## Next steps

For more information on `xplad` commands and usage, see [Using xplad]({{< ref "using-xplad" >}}).
