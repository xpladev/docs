---
weight: 60
title: Writing the Contract
---

## Start with a Template

Please check [ERC20 token contract on OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/erc20#constructing-an-erc20-token-contract). Not only ERC20, but there are many templates including ERC721 and others to import in OpenZeppelin.

```javascript
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
    constructor() ERC20("Test", "TST") {
        _mint(msg.sender, 1000 * 10 ** 18);
    }
}
```

Please write this contract, and save as `Token.sol`.

This helps get you started by providing the basic boilerplate and structure for a smart contract. Instantiate will be executed when it is deployed by the constructor.

## Compile & deploy

You may compile by CLI in your local environment, or use [Remix IDE on Web](https://remix.ethereum.org/#lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.18+commit.87f61d96.js) to get the compiled bytecode. But we recommend to development environment tools like Hardhat and Truffle to use better lifecycle management features.
