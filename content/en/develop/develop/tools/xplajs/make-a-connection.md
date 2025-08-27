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
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { DirectSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";

const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");
const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {derivations: [{
    prefix: "xpla",
    hdPath: HDPath.eth().toString()
}]});

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

```

### AminoSigner

The `AminoSigner` uses the legacy Amino-based signing method, which is maintained for compatibility with older wallets and applications:

```ts
import { EthSecp256k1HDWallet } from "@xpla/xpla"
import { HDPath } from "@interchainjs/types"
import { AminoSigner } from "@interchainjs/cosmos"
import { createCosmosQueryClient } from "@interchainjs/cosmos"
import { DEFAULT_COSMOS_EVM_SIGNER_CONFIG } from "@xpla/xpla/signers/config";

const queryClient = await createCosmosQueryClient("https://cube-rpc.xpla.io");
const mnemonic = "abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about";
const wallet = await EthSecp256k1HDWallet.fromMnemonic(mnemonic, {derivations: [{
    prefix: "xpla",
    hdPath: HDPath.eth().toString()
}]});

const baseSignConfig = {
    queryClient: queryClient,
    chainId: "cube_47-5",
    addressPrefix: "xpla",
}
const signerConfig = {
    ...DEFAULT_COSMOS_EVM_SIGNER_CONFIG,
    ...baseSignConfig
}

const signer = new AminoSigner(wallet, signerConfig);
```