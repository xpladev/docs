---
title: Environment Setup
weight: 30
type: docs
---

As a smart contract developer, you will need to write, compile, upload, and test your contracts before deploying them on the Dimension mainnet. The first step is to set up a specialized environment to streamline development.

## Install CONX Chain Core Locally

Visit [build CONX Chain core]({{< ref "build-xpla-core" >}}) to install the latest version of CONX Chain Core to obtain a working version of `xplad`. You will need this to connect to your local CONX Chain test network to work with smart contracts.

## Install Rust

While WASM smart contracts can theoretically be written in any programming language, it is currently only recommended to use Rust as it is the only language for which mature libraries and tooling exist for CosmWasm. For this tutorial, you'll need to also install the latest version of Rust by following the instructions [here](https://www.rust-lang.org/tools/install).

Once you'll installed Rust and its toolchain (cargo et al.), you'll need to add the `wasm32-unknown-unknown` compilation target.

```sh
rustup default stable
rustup target add wasm32-unknown-unknown
```

Next, install `cargo-run-script`, which is required to optimize smart contracts.

```sh
cargo install cargo-run-script
```
