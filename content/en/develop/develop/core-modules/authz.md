---
title: AuthZ
weight: 30
type: docs
---

{{< alert >}}
**Note**

CONX Chain's fee grant module inherits from the Cosmos SDK's [`authz`](https://docs.cosmos.network/v0.45/modules/authz/) module. This document is a stub and explains mainly important CONX Chain-specific notes about how it is used.
{{< /alert >}}

The authz (message authorization) module allows users to authorize another account to send messages on their behalf. Certain authorizations, such as the spending of another account's tokens, can be parameterized to constrain the permissions of the grantee, such as setting a spending limit.

## Concepts

### Authorization and Grant

`x/authz` module defines interfaces and messages grant authorizations to perform actions
on behalf of one account to other accounts. The design is defined in the [ADR 030](https://docs.cosmos.network/v0.45/architecture/adr-030-authz-module.html).

Grant is an allowance to execute a Msg by the grantee on behalf of the granter.
Authorization is an interface which must be implemented by a concrete authorization logic to validate and execute grants. They are extensible and can be defined for any Msg service method even outside of the module where the Msg method is defined. See the `SendAuthorization` example in the next section for more details.

```go
// Authorization represents the interface of various Authorization types implemented
// by other modules.
type Authorization interface {
	proto.Message

	// MsgTypeURL returns the fully-qualified Msg service method URL (as described in ADR 031),
	// which will process and accept or reject a request.
	MsgTypeURL() string

	// Accept determines whether this grant permits the provided sdk.Msg to be performed,
	// and if so provides an upgraded authorization instance.
	Accept(ctx sdk.Context, msg sdk.Msg) (AcceptResponse, error)

	// ValidateBasic does a simple validation check that
	// doesn't require access to any other information.
	ValidateBasic() error
}
```

### Built-in Authorizations

Cosmos-SDK `x/authz` module comes with following authorization types

#### SendAuthorization

`SendAuthorization` implements the `Authorization` interface for the `cosmos.bank.v1beta1.MsgSend` Msg. It takes a `SpendLimit` that specifies the maximum amount of tokens the grantee can spend, which is updated as the tokens are spent.

```go
// SendAuthorization allows the grantee to spend up to spend_limit coins from
// the granter's account.
//
// Since: cosmos-sdk 0.43
type SendAuthorization struct {
	SpendLimit github_com_cosmos_cosmos_sdk_types.Coins `protobuf:"bytes,1,rep,name=spend_limit,json=spendLimit,proto3,castrepeated=github.com/cosmos/cosmos-sdk/types.Coins" json:"spend_limit"`
}
```

- `spent_limit` keeps track of how many coins are left in the authorization.

#### GenericAuthorization

`GenericAuthorization` implements the `Authorization` interface, that gives unrestricted permission to execute the provided Msg on behalf of granter's account.

```go
// GenericAuthorization gives the grantee unrestricted permissions to execute
// the provided method on behalf of the granter's account.
type GenericAuthorization struct {
	// Msg, identified by it's type URL, to grant unrestricted permissions to execute
	Msg string `protobuf:"bytes,1,opt,name=msg,proto3" json:"msg,omitempty"`
}
```

- `msg` stores Msg type URL.

### Gas

In order to prevent DoS attacks, granting `StakeAuthorizaiton`s with `x/authz` incur gas. `StakeAuthorizaiton` allows you to authorize another account to delegate, undelegate, or redelegate to validators. The authorizer can define a list of validators they will allow and/or deny delegations to. The SDK will iterate over these lists and charge 10 gas for each validator in both of the lists.


## Message Types

### MsgGrant

```go
// MsgGrant is a request type for Grant method. It declares authorization to the grantee
// on behalf of the granter with the provided expiration time.
type MsgGrant struct {
	Granter string `protobuf:"bytes,1,opt,name=granter,proto3" json:"granter,omitempty"`
	Grantee string `protobuf:"bytes,2,opt,name=grantee,proto3" json:"grantee,omitempty"`
	Grant   Grant  `protobuf:"bytes,3,opt,name=grant,proto3" json:"grant"`
}

// Grant gives permissions to execute
// the provide method with expiration time.
type Grant struct {
	Authorization *types.Any `protobuf:"bytes,1,opt,name=authorization,proto3" json:"authorization,omitempty"`
	Expiration    time.Time  `protobuf:"bytes,2,opt,name=expiration,proto3,stdtime" json:"expiration"`
}
```

### MsgRevoke

```go
// MsgRevoke revokes any authorization with the provided sdk.Msg type on the
// granter's account with that has been granted to the grantee.
type MsgRevoke struct {
	Granter    string `protobuf:"bytes,1,opt,name=granter,proto3" json:"granter,omitempty"`
	Grantee    string `protobuf:"bytes,2,opt,name=grantee,proto3" json:"grantee,omitempty"`
	MsgTypeUrl string `protobuf:"bytes,3,opt,name=msg_type_url,json=msgTypeUrl,proto3" json:"msg_type_url,omitempty"`
}
```

### MsgExecAuthorized

```go
// MsgExec attempts to execute the provided messages using
// authorizations granted to the grantee. Each message should have only
// one signer corresponding to the granter of the authorization.
type MsgExec struct {
	Grantee string `protobuf:"bytes,1,opt,name=grantee,proto3" json:"grantee,omitempty"`
	// Authorization Msg requests to execute. Each msg must implement Authorization interface
	// The x/authz will try to find a grant matching (msg.signers[0], grantee, MsgTypeURL(msg))
	// triple and validate it.
	Msgs []*types.Any `protobuf:"bytes,2,rep,name=msgs,proto3" json:"msgs,omitempty"`
}
```
