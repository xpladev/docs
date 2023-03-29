---
title: Interacting with the Contract
weight: 70
---

## Requirements

### Wallet credential

You should also have the latest version of `xplad` by building the latest version of XPLA Chain core. You will configure `xplad` to use it against your isolated testnet environment.

In a separate terminal, make sure to set up the following mnemonic:

```sh
xplad keys add test1 --recover
```

Using the mnemonic:

```sh
satisfy adjust timber high purchase tuition stool faith fine install that you unaware feed domain license impose boss human eager hat rent enjoy dawn
```

You need to extract the private key corresponding the above mnemonics. Use the command below:

```sh
xplad keys export test1 --unarmored-hex --unsafe
```

You may see the printed hex value. Please use the hex with the header `0x`.

### Development environment

For deploying and executing the contract, you may compile on your local environment, take a copy of the compiled bytecode, and deploy on [MEW](https://www.myetherwallet.com/wallet/deploy) by manual way. But if the size of the contract goes bigger and need better management, we recommend to use the development environment tool followings.

- [Truffle](https://trufflesuite.com/)
- [Hardhat](https://hardhat.org/)

In this tutorial, we assume that:

- You are using on Hardhat.
- You have already completed [Setting up the environment](https://hardhat.org/tutorial/setting-up-the-environment), [Creating a new Hardhat project](https://hardhat.org/tutorial/creating-a-new-hardhat-project)
- Copy `Token.sol` into `contracts` folder, and try to compile following the step in [Writing and compiling contracts](https://hardhat.org/tutorial/writing-and-compiling-contracts), and confirmed that the contract is compilable.

## Uploading Code

The content is mainly from [Deploying to a live network](https://hardhat.org/tutorial/deploying-to-a-live-network).

### Write deploy.js

You may modify `scripts/deploy.js` by copy & paste the below:

```javascript
const { ethers } = require('hardhat');

async function main() {
    const [deployer] = await ethers.getSigners();
  
    console.log("Deploying contracts with the account:", deployer.address);
  
    console.log("Account balance:", (await deployer.getBalance()).toString());
  
    const Token = await ethers.getContractFactory("Token");
    const token = await Token.deploy();
  
    console.log("Token address:", token.address);
  }
  
  main()
    .then(() => process.exit(0))
    .catch((error) => {
      console.error(error);
      process.exit(1);
    });

```

### Network config

You may modify `hardhat.config.js` and add network entry like the below:

```javascript
require("@nomicfoundation/hardhat-toolbox");

const XPLA_PRIVATE_KEY = "XPLA_PRIVATE_KEY";

module.exports = {
  solidity: {
    version: "0.8.17",
    settings: {
      optimizer: {
        enabled: true,
        runs: 10,
      },
    },
  },
  networks: {
    xpla: {
      url: `http://localhost:8545`,
      accounts: [
        XPLA_PRIVATE_KEY,
        // and more
      ]
    }
  }
};

```

Please fill `XPLA_PRIVATE_KEY` from the value of [Wallet credential]({{< ref "#wallet-credential" >}})

### Deployment

Before executing the command below, please make sure that **`xplad` is running on your local.**

```sh
npx hardhat run scripts/deploy.js --network xpla
```

Then, you may see like below:

```plain
Deploying contracts with the account: 0x12ac7A4ea72352c143e3e21C2FD64Ae798b69131
Account balance: 998000000000000000000
Token address: 0xB1ED1d08A66067A0DA102aA50ad7e92f1AA0f5f4
```

#### Troubleshooting

If there is an error and it says about `JsonRpcProvider`, please:

- Open `package.json`
- Downgrade `ethers` as `"ethers": "^5.7.2",`
- Delete `node_modules`
- Install the dependencies again

And then execute the command above again!

## What's Next?

Congratulations! You've finally succeeded to deploy the first EVM smart contract on your local XPLA chain network. From now, all ways to interact with EVM smart contract on XPLA chain are same as the ways of other EVM based chain. You may interact with `web3.js` and create & send transactions, or use the features from development environment tools.
