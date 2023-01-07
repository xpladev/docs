---
title: Build XPLA Chain Core
weight: 20
---

XPLA Chain core is the official Golang reference implementation of the XPLA Chain node software. Use this guide to install XPLA Chain core and `xplad`, the command-line interface and daemon that connects to XPLA Chain and enables you to interact with the XPLA Chain.

## Get the XPLA Chain Core Source Code

1. Use `git` to retrieve [XPLA Chain core](https://github.com/xpladev/xpla/), and check out the `main` branch, which contains the latest stable release. You can find the latest tag on the [tags page](https://github.com/xpladev/xpla/tags) or via autocomplete in your terminal: type `git checkout v` and press `<TAB>`.

   ```sh
   git clone https://github.com/xpladev/xpla
   cd xpla
   git checkout [latest version]
   ```

2. Build XPLA Chain core. This will install the `xplad` executable to your [ `GOPATH` ](https://go.dev/doc/gopath_code) environment variable.

   ```bash
   make install
   ```

3. Verify that XPLA Chain core is installed correctly.

   ```bash
   xplad version --long
   ```

   **Example**:

   ```bash
   name: xpla
   server_name: xplad
   version: v2.0.0
   commit: ea682c41e7e71ba0b182c9e7f989855fb9595885
   build_tags: netgo,ledger
   go: go version go1.18.2 darwin/amd64
   # ...followed by a lot of dependencies
   ```

{{< hint info >}}
**Note**
If the `xplad: command not found` error message is returned, confirm that the Go binary path is correctly configured by running the following command:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```
{{< /hint >}}
