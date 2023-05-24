---
title: Zero Reward
weight: 210
---

The zero reward module is to support validators who want to participate in proposing blocks without profit.

## Concepts

The zero reward module allows validators whose voting power does not enter the active set to propose blocks. These validators can be registered or unregistered through proposals. Commissions earned by these validators are sent to the community pool, and they do not accept any delegation or redelegation.

## State

### Zero Reward Validator

Zero reward validator objects are stored and accessed by a validator address.
- Zero Reward Validators : `0x11 | ValidatorAddrLen (1byte) | ValidatorAddr -> ProtocolBuffer(ZeroRewardValidator)`

```go
// ZeroRewardValidator required for validator set update logic.
type ZeroRewardValidator struct {
	// address is the address of the validator.
	Address string `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	// power defines the power of the validator.
	Power int64 `protobuf:"varint,2,opt,name=power,proto3" json:"power,omitempty"`
	// used when unregistering, reserved for delete
	IsDeleting bool `protobuf:"varint,3,opt,name=is_deleting,json=isDeleting,proto3" json:"is_deleting,omitempty"`
}
```

#### Address

- type : `string`

`Address` is the validator address with a prefix of `xplavaloper1`.

#### Power

- type : `int64`

`Power` is the validator's voting power.

#### IsDeleting

- type : `bool`

`IsDeleting` is a reserved flag for unregistration.

## Proposals

### RegisterZeroRewardValidatorProposal
```go
type RegisterZeroRewardValidatorProposal struct {
	Title                string            `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description          string            `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	ValidatorDescription types.Description `protobuf:"bytes,3,opt,name=validator_description,json=validatorDescription,proto3" json:"validator_description"`
	DelegatorAddress     string            `protobuf:"bytes,4,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress     string            `protobuf:"bytes,5,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	Pubkey               *types1.Any       `protobuf:"bytes,6,opt,name=pubkey,proto3" json:"pubkey,omitempty"`
	Amount               types2.Coin       `protobuf:"bytes,7,opt,name=amount,proto3" json:"amount"`
}
```

Register Zero Reward Proposal is a special type of proposal which, once passed, will automatically go into effect by allowing Zero Reward Validator to propose a block.


{{< alert context="warning" >}}
**Note**

Zero Reward Validator should be created by proposal. In other words, a validator that has already been created cannot be designated as a Zero Reward Validator.
{{< /alert >}}


### UnregisterZeroRewardValidatorProposal
```go
type UnregisterZeroRewardValidatorProposal struct {
	Title            string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description      string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	ValidatorAddress string `protobuf:"bytes,3,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty"`
}
```

Unregister Zero Reward Validator Proposal is a special type of proposal which, once passed, will automatically go into effect by Zero Reward Validator unregister. This means that the validator can no longer propose a block.

## Transitions

### InitGenesis

`InitGenesis` initializes the Zero Reward module genesis state by setting the Zero Reward Validator state to the store.

### ExportGenesis

`ExportGenesis` ABCI function exports the genesis state of the Zero Reward module. In particular, it retrieves all the Zero Reward Validators' information.

### Begin Block
For each abci begin block call, all commissions of `ZeroRewardValidators` are claimed and sent to the community pool.

### End Block
For each abci end block call, the operations to update Zero Reward Validator set changes are specified to execute.

#### Validator Set Changes
The validator's active set is determined by the staking module. When Zero Reward Validators are not part of the active set, process for adding and deleting Tendermint validators.

- In the active set
    - They all follow the operation of the staking module.
- Not in the active set
    - Change the validator's status to bonded and apply voting power to tendermint.
    - When Zero Reward Validator goes to be jailed
        - change the validator to unbonding status
        - change the tendermint voting power to 0
        - update Zero Reward Validator state
    - When a Zero Reward Validator unregistration is reserved
        - change validator to unbonding status
        - tendermint voting power to 0
        - delete Zero Reward Validator state

### Hooks

#### Modify Delegation

Used to sync with the state of the staking module when delegation changes.
