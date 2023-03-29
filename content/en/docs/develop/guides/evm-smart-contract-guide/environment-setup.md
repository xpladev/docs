---
title: Environment Setup
weight: 40
---

As a smart contract developer, you will need to write, compile, upload, and test your contracts before deploying them on the Dimension mainnet. The first step is to set up a specialized environment to streamline development.

## Install XPLA Chain Core Locally

Visit [build XPLA Chain core]({{< ref "build-xpla-core" >}}) to install the latest version of XPLA Chain Core to obtain a working version of `xplad`. You will need this to connect to your local XPLA Chain test network to work with smart contracts.

## Install Node.js

While EVM smart contracts can theoretically be written in Solidity or Vyper language, Node.js is also used for the contract development environment, especially [Truffle](https://trufflesuite.com/) and [Hardhat](https://hardhat.org/)

### Ubuntu

```sh
sudo apt update
sudo apt install curl git
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### MacOS

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
nvm install 18
nvm use 18
nvm alias default 18
npm install npm --global # Upgrade npm to the latest version
```

## Solidity compiler & tool suite

### Ubuntu

```sh
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

### MacOS

```sh
brew update
brew upgrade
brew tap ethereum/ethereum
brew install solidity
```

## Solidity development environment (Optional)

Different from Rust, Solidity language has no convenient feature like unit test, deployment, mock EVM, etc. There are some development environment tools for complementing these features. Node.js is essential to use this feature.

- [Truffle](https://trufflesuite.com/)
- [Hardhat](https://hardhat.org/)

## Install libraries

```sh
# yarn
yarn add @openzeppelin/contracts

# npm
npm install @openzeppelin/contracts
```
