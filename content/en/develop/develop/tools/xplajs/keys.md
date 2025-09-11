---
title: Keys
weight: 70
type: docs
---

To perform actions using an account and private-public key pairs with xplajs, you need an implementation of the [ *Key* ](https://github.com/xpladev/xpla.js/blob/main/src/key/Key.ts) class, which provides an abstraction around the signing functions of an account.


## EthSecp256k1HDWallet from PrivateKey

The most basic implementation of a `PrivateKey`, which is created using a plain private key. `PrivateKey`  wraps the 32 bytes of a private key and supplies a corresponding public key:

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

const account = (await wallet.getAccounts())[0]
console.log(account.address)    // xpla1tw7n4xyl7myx7mzn5sdh5prz3wswk83qr9ejp7
```

## EthSecp256k1HDWallet from Mnemonic


A `Mnemonic` derives itself from a 24-word  [ BIP-39 ](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic as opposed to the bytes of a  private key.
A `MnemonicKey` has various levels of definition:
- Supply no arguments for the mnemonic to be randomly generated ( effectively generating a random key ).
- Supply only a 24-word BIP-39 mnemonic to generate a corresponding key.
- Supply a full HD path (using either a  random or particular mnenomic).

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { Bip39, Random } from "@interchainjs/crypto";

const mnemonic = Bip39.encode(Random.getBytes(32)).toString();
console.log(mnemonic) // grant hip garden wedding bicycle earth insect hobby assault mosquito cross make slam badge grace version cattle survey age mobile toe add reopen transfer

const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {
    derivations: [{
        prefix: "xpla",
        hdPath: HDPath.eth().toString()
    }]
})

const account = (await wallet.getAccounts())[0]
console.log(account.address) // xpla1azetrlgvwhzdva9pax3jxw0dvfrqvyvjse3ry9

```


### Specifying an HD path

`Mnemonic` can used to recover a wallet with a particular BIP44 HD path: `m/44'/${coinType}'/${account}'/0/${index}`.

{{< alert >}}
**Tip**

As per [ *Cosmos HD Key Derivation* ](https://github.com/confio/cosmos-hd-key-derivation-spec):

Cosmos blockchains support hierarchical deterministic key generation (HD keys) for deriving multiple cryptographic keypairs from a single secret value. This allows the user to use different keypairs for different accounts on one blockchain and create accounts on multiple blockchains without having to manage multiple secrets.
{{< /alert >}}

For example, to recover a mnemonic with the old XPLA Vault HD path using coin type for ATOM (118):

```ts
const mne_key = EthSecp256k1HDWallet.fromMnemonic(
    mnemonic, 
    {
        derivations: [{
            prefix: "cosmos",
            hdPath: HDPath.cosmos().toString()
        }]    // Cosmos(118)' coin type ( XPLA Chain had inherited initially )
    }
)
```

- [ BIP-39 Mnemonics ](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

- Coin Types Numbers `60` and `118` above refer to "coin-types" for `Cosmos` and `Ethereum` blockchains accordingly. These numbers are defined according to the [ BIP-44 ](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) standard. You can find more information [ here ](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).