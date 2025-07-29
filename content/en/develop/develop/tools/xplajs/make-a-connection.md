---
title: Make a Connection
weight: 80
type: docs
---

Users can interact with the blockchain by using the following modes:

- Querying data
- Broadcasting a transaction

## Querying data

To perform these actions, connect to the blockchain by using an `RPCQueryClient` object, which represents a connection to a node running the rpc client (RPC). The RPC serves as a JSON RPC over HTTP. xplajs abstracts away the details of making raw API calls and provide an interface with which you can work.

```ts
import { createRPCQueryClient } from "@xpla/xplajs/xpla/rpc.query";

const client = await createRPCQueryClient({rpcEndpoint: "https://dimension-lcd.xpla.dev"})
```

## Broadcasting a transaction

To broadcast transactions to the blockchain, you need to create a signer that can sign and broadcast transactions. xplajs provides two types of signers: `DirectSigner` and `AminoSigner`. The signer handles the process of creating, signing, and broadcasting transactions to the network.

### DirectSigner

The `DirectSigner` uses the newer Protobuf-based signing method, which is more efficient and is the recommended approach for most applications:

```ts
import { DirectSigner } from "@xpla/xpla/signers/direct"
import { MsgSend } from "@xpla/xplajs/cosmos/bank/v1beta1/tx"
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types";
import { Network } from "@xpla/xpla/defaults"

const [auth] = EthSecp256k1Auth.fromMnemonic("<MNEMONIC>", [HDPath.eth().toString()])
const signer = new DirectSigner(auth, toEncoders(MsgSend), Network.Testnet.rpc)

```

### AminoSigner

The `AminoSigner` uses the legacy Amino-based signing method, which is maintained for compatibility with older wallets and applications:

```ts
import { AminoSigner } from "@xpla/xpla/signers/amino"
import { MsgSend } from "@xpla/xplajs/cosmos/bank/v1beta1/tx"
import { EthSecp256k1Auth } from "@interchainjs/auth/ethSecp256k1"
import { HDPath } from "@interchainjs/types";
import { Network } from "@xpla/xpla/defaults"

const [auth] = EthSecp256k1Auth.fromMnemonic("<MNEMONIC>", [HDPath.eth().toString()])
const signer = new AminoSigner(auth, toEncoders(MsgSend), toConverters(MsgSend), Network.Testnet.rpc)
```