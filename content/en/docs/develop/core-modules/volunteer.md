---
title: Volunteer
weight: 190
---

The volunteer module is to support validators who want to participate in proposing blocks without profit.

## Concepts

The volunteer module allows validators whose voting power does not enter the active set to propose blocks. These validators can be registered or unregistered through proposals. Commissions earned by these validators are sent to the community pool, and they only accept their own delegation or redelegation.

## State

### Volunteer Validator

Volunteer validator objects are stored and accessed by a validator address.
- Volunteer Validators : `0x11 | ValidatorAddrLen (1byte) | ValidatorAddr -> ProtocolBuffer(VolunteerValidator)`

```go
// VolunteerValidator required for validator set update logic.
type VolunteerValidator struct {
	// address is the address of the validator.
	Address string `protobuf:"bytes,1,opt,name=address,proto3" json:"address,omitempty"`
	// power defines the power of the validator.
	Power int64 `protobuf:"varint,2,opt,name=power,proto3" json:"power,omitempty"`
}
```

#### Address

- type : `string`

`Address` is the validator address with a prefix of `xplavaloper1`.

#### Power

- type : `int64`

`Power` is the validator's voting power.

## Proposals

### RegisterVolunteerValidatorProposal
```go
type RegisterVolunteerValidatorProposal struct {
	Title                string            `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description          string            `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	ValidatorDescription types.Description `protobuf:"bytes,3,opt,name=validator_description,json=validatorDescription,proto3" json:"validator_description"`
	DelegatorAddress     string            `protobuf:"bytes,4,opt,name=delegator_address,json=delegatorAddress,proto3" json:"delegator_address,omitempty" yaml:"delegator_address"`
	ValidatorAddress     string            `protobuf:"bytes,5,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty" yaml:"validator_address"`
	Pubkey               *types1.Any       `protobuf:"bytes,6,opt,name=pubkey,proto3" json:"pubkey,omitempty"`
	Amount               types2.Coin       `protobuf:"bytes,7,opt,name=amount,proto3" json:"amount"`
}
```

Register Volunteer Proposal is a special type of proposal which, once passed, will automatically go into effect by allowing Volunteer Validator to propose a block.


{{< alert context="warning" >}}
**Note**

Volunteer Validator should be created by proposal. In other words, a validator that has already been created cannot be designated as a Volunteer Validator.
{{< /alert >}}


### UnregisterVolunteerValidatorProposal
```go
type UnregisterVolunteerValidatorProposal struct {
	Title            string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Description      string `protobuf:"bytes,2,opt,name=description,proto3" json:"description,omitempty"`
	ValidatorAddress string `protobuf:"bytes,3,opt,name=validator_address,json=validatorAddress,proto3" json:"validator_address,omitempty"`
}
```

Unregister Volunteer Validator Proposal is a special type of proposal which, once passed, will automatically go into effect by Volunteer Validator unregister. This means that the validator can no longer propose a block.

## Transitions

### InitGenesis

`InitGenesis` initializes the Volunteer module genesis state by setting the Volunteer Validator state to the store.

### ExportGenesis

`ExportGenesis` ABCI function exports the genesis state of the Volunteer module. In particular, it retrieves all the Volunteer Validators' information.

### Begin Block
For each abci begin block call, all commissions of `VolunteerValidators` are claimed and sent to the community pool.
