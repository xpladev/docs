---
title: WASM
weight: 200
type: docs
---

The WASM module implements the execution environment for WebAssembly smart contracts, powered by [CosmWasm](https://cosmwasm.com).

## Concepts

### Smart Contracts

Smart contracts are autonomous agents that can interact with other entities on the XPLA Chain, such as human-owned accounts, validators, and other smart contracts. Each smart contract has:

- A unique **contract address** with an account that holds funds.
- A **code ID**, where its logic is defined.
- Its own **key-value store**, where it can persist and retrieve data.

#### Contract Address

Upon instantiation, each contract is automatically assigned a XPLA Chain account address, called the contract address. The address is procedurally generated on-chain without an accompanying private and public key pair, and it can be completely determined by the contract's number order of existence. For instance, on two separate XPLA Chain, the first contract will always be assigned the address `xpla14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s525s0h`, and similarly for the second, third, and so on.

#### Code ID

On XPLA Chain, code upload and contract creation are separate events. A smart contract writer first uploads WASM bytecode onto the blockchain to obtain a code ID, which they then can use to initialize an instance of that contract. This scheme promotes efficient storage because most contracts share the same underlying logic and vary only in their initial configuration. Vetted, high-quality contracts for common use cases like fungible tokens and multisig wallets can be easily reused without the need to upload new code.

#### Key-value Store

Each smart contract is given its own dedicated keyspace in LevelDB, prefixed by the contract address. Contract code is safely sandboxed and can only set and delete new keys and values within its assigned keyspace.

### Interaction

You can interact with smart contracts in several ways.

#### Instantiation

You can instantiate a new smart contract by sending a `MsgInstantiateContract`. In it, you can:

- Assign an owner to the contract.
- Specify code will be used for the contract via a code ID.
- Define the initial parameters / configuration through an `InitMsg`.
- Provide the new contract's account with some initial funds.
- Denote whether the contract is migratable (i.e. can change code IDs).

The `InitMsg` is a JSON message whose expected format is defined in the contract's code. Every contract contains a section that defines how to set up the initial state depending on the provided `InitMsg`.

#### Execution

You can execute a smart contract to invoke one of its defined functions by sending a `MsgExecuteContract`. In it, you can:

- Specify which function to call with a `HandleMsg`.
- Send funds to the contract, which may be expected during execution.

The `HandleMsg` is a JSON message that contains function call arguments and gets routed to the appropriate handling logic. From there, the contract executes the function's instructions during which the contract's own state can be modified. The contract can only modify outside state, such as state in other contracts or modules, after its own execution has ended, by returning a list of blockchain messages, such as `MsgSend` and `MsgExecuteContract`. These messages are appended to the same transaction as the `MsgExecuteContract`, and, if any of the messages are invalid, the whole transaction is invalidated.

#### Migration

If a user is the contract's owner, and a contract is instantiated as migratable, they can issue a `MsgMigrateContract` to reset its code ID to a new one. The migration can be parameterized with a `MigrateMsg`, a JSON message.

#### Transfer of Ownership

The current owner of the smart contract can reassign a new owner to the contract with `MsgUpdateAdmin`.

#### Query

Contracts can define query functions, or read-only operations meant for data-retrieval. Doing so allows contracts to expose rich, custom data endpoints with JSON responses instead of raw bytes from the low-level key-value store. Because the blockchain state cannot be changed, the node can directly run the query without a transaction.

Users can specify which query function alongside any arguments with a JSON `QueryMsg`. Even though there is no gas fee, the query function's execution is capped by gas determined by metered execution, which is not charged, as a form of spam protection.

### Wasmer VM

The actual execution of WASM bytecode is performed by [wasmer](https://github.com/wasmerio/wasmer), which provides a lightweight, sandboxed runtime with metered execution to account for the resource cost of computation.

#### Gas Meter

In addition to the regular gas fees incurred from creating the transaction, XPLA Chain also calculates a separate gas when executing smart contract code. This is tracked by the **gas meter**, which is during the execution of every opcode and gets translated back to native XPLA Chain gas via a constant multiplier (currently set to 100).

### Gas Fees

WASM data and event spend gas up to `1 * bytes`. Passing the event and data to another contract also spends gas in reply.

## Data

### CodeInfo

```go
type CodeInfo struct {
	// CodeHash is the unique identifier created by wasmvm
	CodeHash []byte `protobuf:"bytes,1,opt,name=code_hash,json=codeHash,proto3" json:"code_hash,omitempty"`
	// Creator address who initially stored the code
	Creator string `protobuf:"bytes,2,opt,name=creator,proto3" json:"creator,omitempty"`
	// InstantiateConfig access control to apply on contract creation, optional
	InstantiateConfig AccessConfig `protobuf:"bytes,5,opt,name=instantiate_config,json=instantiateConfig,proto3" json:"instantiate_config"`
}
```

### ContractInfo

```go
type ContractInfo struct {
	// CodeID is the reference to the stored Wasm code
	CodeID uint64 `protobuf:"varint,1,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	// Creator address who initially instantiated the contract
	Creator string `protobuf:"bytes,2,opt,name=creator,proto3" json:"creator,omitempty"`
	// Admin is an optional address that can execute migrations
	Admin string `protobuf:"bytes,3,opt,name=admin,proto3" json:"admin,omitempty"`
	// Label is optional metadata to be stored with a contract instance.
	Label string `protobuf:"bytes,4,opt,name=label,proto3" json:"label,omitempty"`
	// Created Tx position when the contract was instantiated.
	// This data should kept internal and not be exposed via query results. Just
	// use for sorting
	Created   *AbsoluteTxPosition `protobuf:"bytes,5,opt,name=created,proto3" json:"created,omitempty"`
	IBCPortID string              `protobuf:"bytes,6,opt,name=ibc_port_id,json=ibcPortId,proto3" json:"ibc_port_id,omitempty"`
	// Extension is an extension point to store custom metadata within the
	// persistence model.
	Extension *types.Any `protobuf:"bytes,7,opt,name=extension,proto3" json:"extension,omitempty"`
}
```

## State

### Last Code ID

- type: `uint64`

A counter for the last uploaded code ID.

### Last Instance ID

- type: `uint64`

A counter for the last instantiated contract number.

### Code

- type: `map[uint64]CodeInfo`

Maps a code ID to `CodeInfo` entry.

### Contract Info

- type: `map[bytes]ContractInfo`

Maps contract address to its corresponding `ContractInfo`.

### Contract Store

- type: `map[bytes]KVStore`

Maps contract address to its dedicated KVStore.

## Message Types

### MsgStoreCode

Uploads new code to the blockchain and results in a new code ID, if successful. `WASMByteCode` is accepted as either uncompressed or gzipped binary data encoded as Base64.

```go
type MsgStoreCode struct {
	// Sender is the actor that signed the messages
	Sender string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	// WASMByteCode can be raw or gzip compressed
	WASMByteCode []byte `protobuf:"bytes,2,opt,name=wasm_byte_code,json=wasmByteCode,proto3" json:"wasm_byte_code,omitempty"`
	// InstantiatePermission access control to apply on contract creation,
	// optional
	InstantiatePermission *AccessConfig `protobuf:"bytes,5,opt,name=instantiate_permission,json=instantiatePermission,proto3" json:"instantiate_permission,omitempty"`
}

type AccessConfig struct {
	Permission AccessType `protobuf:"varint,1,opt,name=permission,proto3,enum=cosmwasm.wasm.v1.AccessType" json:"permission,omitempty" yaml:"permission"`
	Address    string     `protobuf:"bytes,2,opt,name=address,proto3" json:"address,omitempty" yaml:"address"`
}
```

### MsgInstantiateContract

Creates a new instance of a smart contract. Initial configuration is provided in the `InitMsg`, which is a JSON message encoded in Base64. If `Migratable` is set to `true`, the owner of the contract is permitted to reset the contract's code ID to a new one.

```go
type MsgInstantiateContract struct {
	// Sender is the actor that signed the messages
	Sender string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	// Admin is an optional address that can execute migrations
	Admin string `protobuf:"bytes,2,opt,name=admin,proto3" json:"admin,omitempty"`
	// CodeID is the reference to the stored WASM code
	CodeID uint64 `protobuf:"varint,3,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	// Label is optional metadata to be stored with a contract instance.
	Label string `protobuf:"bytes,4,opt,name=label,proto3" json:"label,omitempty"`
	// Msg json encoded message to be passed to the contract on instantiation
	Msg RawContractMessage `protobuf:"bytes,5,opt,name=msg,proto3,casttype=RawContractMessage" json:"msg,omitempty"`
	// Funds coins that are transferred to the contract on instantiation
	Funds github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,6,rep,name=funds,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"funds"`
}
```

### MsgExecuteContract

Invokes a function defined within the smart contract. Function and parameters are encoded in `ExecuteMsg`, which is a JSON message encoded in Base64.

```go
type MsgExecuteContract struct {
	// Sender is the actor that signed the messages
	Sender string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	// Contract is the address of the smart contract
	Contract string `protobuf:"bytes,2,opt,name=contract,proto3" json:"contract,omitempty"`
	// Msg json encoded message to be passed to the contract
	Msg RawContractMessage `protobuf:"bytes,3,opt,name=msg,proto3,casttype=RawContractMessage" json:"msg,omitempty"`
	// Funds coins that are transferred to the contract on execution
	Funds github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,5,rep,name=funds,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"funds"`
}
```

### MsgMigrateContract

Can be issued by the owner of a migratable smart contract to reset its code ID to another one. `MigrateMsg` is a JSON message encoded in Base64.

```go
type MsgMigrateContract struct {
	// Sender is the actor that signed the messages
	Sender string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	// Contract is the address of the smart contract
	Contract string `protobuf:"bytes,2,opt,name=contract,proto3" json:"contract,omitempty"`
	// CodeID references the new WASM code
	CodeID uint64 `protobuf:"varint,3,opt,name=code_id,json=codeId,proto3" json:"code_id,omitempty"`
	// Msg json encoded message to be passed to the contract on migration
	Msg RawContractMessage `protobuf:"bytes,4,opt,name=msg,proto3,casttype=RawContractMessage" json:"msg,omitempty"`
}
```

### MsgUpdateAdmin

Can be issued by the smart contract's owner to transfer ownership.

```go
type MsgUpdateAdmin struct {
	// Sender is the actor that signed the messages
	Sender string `protobuf:"bytes,1,opt,name=sender,proto3" json:"sender,omitempty"`
	// NewAdmin address to be set
	NewAdmin string `protobuf:"bytes,2,opt,name=new_admin,json=newAdmin,proto3" json:"new_admin,omitempty"`
	// Contract is the address of the smart contract
	Contract string `protobuf:"bytes,3,opt,name=contract,proto3" json:"contract,omitempty"`
}
```

## Parameters

The subspace for the WASM module is `wasm`.

```go
type Params struct {
	CodeUploadAccess             AccessConfig `protobuf:"bytes,1,opt,name=code_upload_access,json=codeUploadAccess,proto3" json:"code_upload_access" yaml:"code_upload_access"`
	InstantiateDefaultPermission AccessType   `protobuf:"varint,2,opt,name=instantiate_default_permission,json=instantiateDefaultPermission,proto3,enum=cosmwasm.wasm.v1.AccessType" json:"instantiate_default_permission,omitempty" yaml:"instantiate_default_permission"`
}
```
