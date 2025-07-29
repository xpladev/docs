---
title: Signer
weight: 150
type: docs
---

## Create a Singer

Use `EthSecp256k1Auth` to create a `signer` from a `Key`.

```ts
import { Random } from "@interchainjs/crypto/random";
import { Key } from "@interchainjs/utils";
import { Secp256k1 } from "@interchainjs/crypto/secp256k1"
import { EthAccount } from "@xpla/xpla/accounts/eth-account";

const key = new Key(Random.getBytes(32))
const signer = new EthSecp256k1Auth(key, HDPath.eth().toString())
```

Use `EthSecp256k1Auth` to create a `signer` from a `Mnemonic`.

```ts
import { Bip39, Random } from '@interchainjs/crypto';
import { EthAccount } from "@xpla/xpla/accounts/eth-account";
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"

const mnemonic = Bip39.encode(Random.getBytes(32)).toString();
const signer = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
```

## Usage

### Getting Account Number and Sequence

A wallet is connected to the XPLA Chain and can poll the values of an account's account number and sequence directly:

```ts
const address = await signer.getAddress()
const accountNumber = await signer.queryClient.getAccountNumber(address)
const accountSequence = await signer.queryClient.getSequence(address)
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