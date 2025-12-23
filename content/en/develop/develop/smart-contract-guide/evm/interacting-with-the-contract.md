---
title: Interacting with the Contract
weight: 50
type: docs
---

## Requirements

### Wallet credential

You should also have the latest version of `xplad` by building the latest version of CONX Chain core. You will configure `xplad` to use it against your isolated testnet environment.

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

    await token.deployed();

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

## Querying & executing on the contract

Like other EVM-based blockchain network, you may use `web3.js` to interact with EVM contracts on CONX Chain. To query and send transactions to a smart contract on the Ethereum blockchain using web3.js, you will need to do the following:

- Create an instance of the web3 object and connect to an Ethereum network
- Load the contract ABI (Application Binary Interface) and instantiate the contract object
- Query the smart contract for information
- Send a transaction to the smart contract

### Querying

CONX Chain opens `8545` port for EVM based interaction. Here's an example of how to connect to the CONX Chain:

```javascript
const Web3 = require('web3');
const web3 = new Web3('http://localhost:8545');
```

Then, you need to load the contract ABI and instantiate the contract object.

The ABI (Application Binary Interface) is a JSON file that describes the functions and variables of a smart contract. You will need to load the ABI and instantiate the contract object in order to interact with the smart contract.

In your Hardhat project folder, you may find the ABI from `artifacts/contracts/Token.sol/Token.json`. You may take the file into your frontend project.

Here's an example of how to load the ABI and instantiate the contract object:

```javascript
const contractABI = require('./Token.json');
const contractAddress = '0xB1ED1d08A66067A0DA102aA50ad7e92f1AA0f5f4';
const myContract = new web3.eth.Contract(contractABI, contractAddress);

myContract.methods.balanceOf("0xabcderfgh........").call()
  .then(result => console.log(result))
  .catch(error => console.log(error));
```

### Executing

To send a transaction to the smart contract, you can use the send method on the contract object. The send method creates a transaction on the blockchain and modifies the state of the smart contract.

Here's an example of how to send a transaction to a smart contract:

```javascript
const privateKey = 'XPLA_PRIVATE_KEY';
const account = web3.eth.accounts.privateKeyToAccount(privateKey);

myContract.methods.transfer(param1, param2).send({
  from: account.address,
  gas: 500000,
  gasPrice: '20000000000'
})
.then(receipt => console.log(receipt))
.catch(error => console.log(error));
```

Please fill `XPLA_PRIVATE_KEY` from the value of [Wallet credential]({{< ref "#wallet-credential" >}})
