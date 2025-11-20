---
title: Signer
weight: 150
type: docs
---

## Create a Singer

Use `EthSecp256k1HDWallet` to create a `signer` from a `PrivateKey`.

```ts
import { createCosmosEvmConfig, EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { Random } from "@interchainjs/crypto/random";
import { PrivateKey } from "@interchainjs/auth";
import { BaseCryptoBytes } from "@interchainjs/utils"

const cryptoBytes = BaseCryptoBytes.from(Random.getBytes(32))
const key = PrivateKey.fromBytes(cryptoBytes, createCosmosEvmConfig().privateKeyConfig)
const wallet = new EthSecp256k1HDWallet([key], {
    derivations: [{
        prefix: "xpla",
        hdPath: HDPath.eth().toString()
    }]
})
```

Use `EthSecp256k1HDWallet` to create a `signer` from a `Mnemonic`.

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { Bip39, Random } from "@interchainjs/crypto";

const mnemonic = Bip39.encode(Random.getBytes(32)).toString();
const signer = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {
    derivations: [{
        prefix: "xpla",
        hdPath: HDPath.eth().toString()
    }]
})
```

## Usage

### Getting Account Number and Sequence

A wallet is connected to the CONX Chain and can poll the values of an account's account number and sequence directly:

```ts
const baseSignConfig = {
    queryClient: queryClient,
    chainId: "cube_47-5",
    addressPrefix: "xpla",
}
const signerConfig = {
    ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
    ...baseSignConfig
}

const signer = new DirectSigner(wallet, signerConfig);
const address = (await signer.getAddresses())[0]
const accountNumber = await signer.getAccountNumber(address)
const sequence = await signer.getSequence(address)
```

### Creating Transactions

A wallet makes it easy to create a transaction by automatically fetching the account number and sequence from the blockchain. The fee parameter is optional -- if you don't include it, xplajs will automatically estimation settings to simulate the transaction within the node and include the resultant fee in your transaction.

```ts
const msgs = [ ... ]; // list of messages
const fee: StdFee = {
    amount: [Coin.fromPartial({denom: "axpla", amount: "28000000000000000"})],
    gas: "100000",
}
const res = await signer.signAndBroadcast({messages: msgs, fee: fee, memo: "this is optional"})
```