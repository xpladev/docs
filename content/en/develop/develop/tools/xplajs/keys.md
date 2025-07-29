---
title: Keys
weight: 70
type: docs
---

To perform actions using an account and private-public key pairs with xplajs, you need an implementation of the [ *Key* ](https://github.com/xpladev/xpla.js/blob/main/src/key/Key.ts) class, which provides an abstraction around the signing functions of an account.


## EthSecp256k1Auth from Key

The most basic implementation of a `Key`, which is created using a plain private key. `Key`  wraps the 32 bytes of a private key and supplies a corresponding public key:

```ts
import { Random } from "@interchainjs/crypto/random";
import { Key } from "@interchainjs/utils";
import { Secp256k1 } from "@interchainjs/crypto/secp256k1"
import { EthAccount } from "@xpla/xpla/accounts/eth-account";

const key = new Key(Random.getBytes(32))
const keyPair = new EthSecp256k1Auth(key, HDPath.eth().toString())
const publicKey = keyPair.getPublicKey()
console.log(publicKey)  // Key { value: Uint8Array(33) [2, 249,  28,  32, 171, 207, 122, 153, 132, 198, 250, 177, 126, 136,  13,  66, 113, 170,  65,  10, 205,  50, 169, 229, 183,  11, 188, 210,  29, 210,  75, 252,  31]}
const ethAccount = new EthAccount("xpla", keyPair)
console.log(ethAccount.getAddressByPubKey())    // xpla1tw7n4xyl7myx7mzn5sdh5prz3wswk83qr9ejp7
```

## EthSecp256k1Auth from Mnemonic


A `Mnemonic` derives itself from a 24-word  [ BIP-39 ](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) mnemonic as opposed to the bytes of a  private key.
A `MnemonicKey` has various levels of definition:
- Supply no arguments for the mnemonic to be randomly generated ( effectively generating a random key ).
- Supply only a 24-word BIP-39 mnemonic to generate a corresponding key.
- Supply a full HD path (using either a  random or particular mnenomic).

```ts
import { Bip39, Random } from '@interchainjs/crypto';
import { EthAccount } from "@xpla/xpla/accounts/eth-account";
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"

const mnemonic = Bip39.encode(Random.getBytes(32)).toString();
console.log(mnemonic) // grant hip garden wedding bicycle earth insect hobby assault mosquito cross make slam badge grace version cattle survey age mobile toe add reopen transfer

const keyPair = EthSecp256k1Auth.fromMnemonic(mnemonic, [HDPath.eth().toString()]);
console.log(keyPair[0].getPublicKey())  // Key { value: Uint8Array(33) [ 3, 212, 135,  21, 224,   2, 162,  63, 94,  74,  72, 144, 171, 122, 103,  21, 116,  14, 158, 227,  60, 128, 219,  40, 106, 233,  83,   6, 153,  36, 208, 241, 235 ] }
const ethAccount = new EthAccount("xpla", keyPair[0])
console.log(ethAccount.getAddressByPubKey()) // xpla1azetrlgvwhzdva9pax3jxw0dvfrqvyvjse3ry9

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
const mne_key = EthSecp256k1Auth.fromMnemonic(
    mnemonic, 
    [HDPath.cosmos().toString()]    // Cosmos(118)' coin type ( XPLA Chain had inherited initially )
)
```

- [ BIP-39 Mnemonics ](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

- Coin Types Numbers `60` and `118` above refer to "coin-types" for `Cosmos` and `Ethereum` blockchains accordingly. These numbers are defined according to the [ BIP044 ](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) standard. You can find more information [ here ](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).