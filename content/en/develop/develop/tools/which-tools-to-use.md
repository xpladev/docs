---
title: Which Tools to Use
weight: 10
type: docs
---

There are a variety of tools in the CONX Chain Development Suite. Use this guide to find the right tool for your project.

If you are new to CONX Chain, start with the [beginner's guide]({{< ref "get-started" >}}).

```
CONX Chain Development Suite
│
├── xplajs: JavaScript SDK.
│
├── xpla.js: JavaScript SDK.
│
├── Wallet Provider: React tooling for frontend integrations.
│
└── Other tools
    │
    ├── xplad: Node daemon and CLI.
    │
    ├── XPLA Explorer: Block explorer.
    │
    └── Faucet: Get testnet funds.
```

## xplajs

xplajs is generated code through hyperweb-io's telescope, which replaces the existing xpla.js library. It is compatible with interchain-kit and interchain-js, providing a more modular and maintainable approach to interacting with the XPLA blockchain.

Use xplajs to create bots, mint NFTs, and build out backend services. Follow the [xplajs tutorial]({{< ref "/develop/develop/tools/xplajs/get-started-with-xpla-js" >}}) to get started.


## xpla.js

Use xpla.js to create bots, mint NFTs, and build out backend services. Follow the [xpla.js tutorial]({{< ref "/develop/develop/tools/xpla-js/get-started-with-xpla-js" >}}) to get started.

## Wallet Provider and Templates

Wallet Provider makes it easy to connect a React-based web app to XPLA Vault. Follow the [Wallet Provider tutorial]({{< ref "get-started-with-wallet-provider" >}}) to get started.

## Other Tools

Use these tools to interact with the CONX Chain.

### xplad

The command line interface and node daemon for interacting with the CONX Chain. Use `xplad` to run a full node or interact with the chain. Follow the [`xplad` install guide]({{< ref "install-xplad" >}}) to get started.

### XPLA Explorer

CONX Chain’s multi-purpose [block explorer](https://explorer.xpla.io/).

### Faucet

Get tokens sent to your testnet address using the [Faucet](https://faucet.xpla.io).

