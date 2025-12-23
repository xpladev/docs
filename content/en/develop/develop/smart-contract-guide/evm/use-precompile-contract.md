---
weight: 50
title: Use Precompile Contracts
type: docs
---

# Precompile Contracts

CONX Chain is a Cosmos SDK-based blockchain that supports Solidity smart contracts through the EVM module. Precompile contracts are special contracts that allow you to access functionality from other Cosmos SDK modules (bank, staking, governance, etc.) within the EVM environment.

## Overview

Precompile contracts are pre-compiled contracts deployed at specific addresses that provide access to Cosmos SDK module functionality within the EVM environment. This allows Solidity smart contracts to directly call functions for token transfers, staking, governance, and other Cosmos SDK features.

## Available Precompile Contracts

The following precompile contracts are available on CONX Chain:

| Module | Contract Address | Owner |
|--------|------------------|-------|
| auth | `0x1000000000000000000000000000000000000005` | xpladev |
| bank | `0x1000000000000000000000000000000000000001` | xpladev |
| wasm | `0x1000000000000000000000000000000000000004` | xpladev |
| p256 | `0x0000000000000000000000000000000000000100` | - |
| bech32 | `0x0000000000000000000000000000000000000400` | cosmos |
| staking | `0x0000000000000000000000000000000000000800` | cosmos |
| distribution | `0x0000000000000000000000000000000000000801` | cosmos |
| gov | `0x0000000000000000000000000000000000000805` | cosmos |
| slashing | `0x0000000000000000000000000000000000000806` | cosmos |

## Auth Module Precompile

The Auth module precompile provides account and address conversion functionality.

### Interface

Based on the [IAuth.sol interface](https://github.com/xpladev/xpla/blob/release/v1.8.x/precompile/auth/IAuth.sol)
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

address constant AUTH_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000005;

IAuth constant AUTH_CONTRACT = IAuth(AUTH_PRECOMPILE_ADDRESS);

interface IAuth {
    function account(address evmAddress) external view returns (string calldata stringAddress);
    function moduleAccountByName(string calldata name) external view returns (string calldata stringAddress);
    function bech32Prefix() external view returns (string calldata prefix);
    function addressBytesToString(address evmAddress) external view returns (string calldata stringAddress);
    function addressStringToBytes(string calldata stringAddress) external view returns (address byteAddress);
}
```

### Available Functions

The Auth module precompile provides the following key functionalities:

- **Account Address Conversion**: Convert EVM addresses to Cosmos Bech32 format using `addressBytesToString`
- **Address String to Bytes**: Convert Cosmos Bech32 addresses to EVM addresses using `addressStringToBytes`
- **Account Information**: Get account information for EVM addresses using `account`
- **Module Account Access**: Retrieve addresses for system module accounts using `moduleAccountByName`
- **Bech32 Prefix Management**: Get the chain's Bech32 address prefix using `bech32Prefix`



## Bank Module Precompile

The Bank module precompile allows you to query token balances, supplies, and perform transfers.

### Interface

Based on the [IBank.sol interface](https://github.com/xpladev/xpla/blob/release/v1.8.x/precompile/bank/IBank.sol):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Coin} from "../util/Types.sol";

address constant BANK_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000001;

IBank constant BANK_CONTRACT = IBank(
    BANK_PRECOMPILE_ADDRESS
);

interface IBank {
    /**
     * @dev Send defines an event emitted when coins are sended
     * @param from the address of the sender
     * @param to the address of the receiver
     * @param amount the amount of sended coin
     */
    event Send(
        address indexed from,
        address indexed to,
        Coin[] amount
    );

    // Transactions
    function send(
        address fromAddress,
        address toAddress,
        Coin[] memory amount
    ) external returns (bool success);

    // Queries
    function balance(
        address addr,
        string memory denom
    ) external view returns (uint256 balance);

    function supplyOf(
        string memory denom
    ) external view returns (uint256 supply);
}
```

### Available Functions

The Bank module precompile provides the following key functionalities:

- **Token Transfers**: Send tokens between accounts using `send` function with Coin array
- **Balance Queries**: Check token balances for any account and denomination using `balance`
- **Token Supply Queries**: Get the total supply of any token denomination using `supplyOf`
- **Multi-Denomination Support**: Handle various token types (axpla, ibc tokens, etc.) through Coin arrays


## Wasm Module Precompile

The Wasm module precompile allows interaction with CosmWasm smart contracts.

{{< alert context="warning" >}}
**Important: Contract Address Format**

CosmWasm contract addresses are 32 bytes long, but the Wasm precompile expects only the last 20 bytes. When using a CosmWasm contract address, you must extract the last 20 bytes from the 32-byte address.

**Example:**
```bash
$ xplad keys parse xpla14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s525s0h
bytes: ADE4A5F5803A439835C636395A8D648DEE57B2FC90D98DC17FA887159B69638B
human: xpla
```

For the Wasm precompile, use only the last 20 bytes: `5A8D648DEE57B2FC90D98DC17FA887159B69638B`
{{< /alert >}}

### Interface

Based on the [IWasm.sol interface](https://github.com/xpladev/xpla/blob/release/v1.8.x/precompile/wasm/IWasm.sol):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {Coin} from "../util/Types.sol";

address constant WASM_PRECOMPILE_ADDRESS = 0x1000000000000000000000000000000000000004;

IWasm constant WASM_CONTRACT = IWasm(
    WASM_PRECOMPILE_ADDRESS
);

interface IWasm {
    // Events
    /**
     * @dev InstantiateContract defines an event emitted when a wasm contract is successfully
     * instantiated via instantiateContract or instantiateContract2
     * @param sender the address of the sender
     * @param contractAddress the address of the instantiated contract
     * @param codeId the id of contract
     * @param admin the address of the contract admin
     * @param label the optional metadata to be stored with a contract instance
     * @param msg the message to be passed to the contract on instantiation
     * @param funds the coins that are transferred to the contract on instantiation
     * @param data the bytes to returned from the contract
     */
    event InstantiateContract(
        address indexed sender,
        address indexed contractAddress,
        uint256 indexed codeId,
        address admin,
        string label,
        bytes msg,
        Coin[] funds,
        bytes data
    );

    /**
     * @dev ExecuteContract defines an event emitted when a wasm contract is successfully
     * executed via executeContract
     * @param sender the address of the sender
     * @param contractAddress the address of executed contract
     * @param msg the message to be passed to the contract
     * @param funds the coins that are transferred to the contract on execution
     * @param data the bytes to returned from the contract
     */
    event ExecuteContract(
        address indexed sender,
        address indexed contractAddress,
        bytes msg,
        Coin[] funds,
        bytes data
    );

    /**
     * @dev MigrateContract defines an event emitted when a wasm contract is successfully
     * migrated via migrateContract
     * @param sender the address of the sender
     * @param contractAddress the address of migrated contract
     * @param codeId changed code id
     * @param msg the message to be passed to the contract on migration
     * @param data the bytes to returned from the contract
     */
    event MigrateContract(
        address indexed sender,
        address indexed contractAddress,
        uint256 indexed codeId,
        bytes msg,
        bytes data
    );

    // Transactions
    function instantiateContract(
        address sender,
        address admin,
        uint256 codeId,
        string calldata label,
        bytes calldata msg,
        Coin[] memory funds
    ) external returns (address contractAddress, bytes calldata data);
    function instantiateContract2(
        address sender,
        address admin,
        uint256 codeId,
        string calldata label,
        bytes calldata msg,
        Coin[] memory funds,
        bytes calldata salt,
        bool fixMsg
    ) external returns (address contractAddress, bytes calldata data);
    function executeContract(
        address sender,
        address contractAddress,
        bytes calldata msg,
        Coin[] memory funds
    ) external returns (bytes calldata data);
    function migrateContract(
        address sender,
        address contractAddress,
        uint256 codeId,
        bytes calldata msg
    ) external returns (bytes calldata data);
    
    // Queries
    function smartContractState(
        address contractAddress,
        bytes calldata queryData
    ) external view returns (bytes calldata data);
}
```

### Available Functions

The Wasm module precompile provides the following key functionalities:

- **Contract Instantiation**: Deploy new CosmWasm contracts using `instantiateContract` or `instantiateContract2` (with salt and fixMsg option)
- **Contract Execution**: Execute functions on existing CosmWasm contracts using `executeContract` (using last 20 bytes of contract address)
- **Contract Migration**: Upgrade existing contracts to new code versions using `migrateContract`
- **Contract State Queries**: Query the current state of CosmWasm contracts using `smartContractState`
- **Cross-Contract Communication**: Enable EVM contracts to interact with Wasm contracts through sender and admin parameters


## Bech32 Module Precompile

The Bech32 module precompile provides address format conversion utilities.

### Interface

Based on the [Bech32I.sol interface](https://github.com/cosmos/evm/blob/v0.3.0/precompiles/bech32/Bech32I.sol):

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

/// @dev The Bech32I contract's address.
address constant Bech32_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000400;

/// @dev The Bech32I contract's instance.
Bech32I constant BECH32_CONTRACT = Bech32I(Bech32_PRECOMPILE_ADDRESS);

/// @author Evmos Team
/// @title Bech32 Precompiled Contract
/// @dev The interface through which solidity contracts can convert addresses from
/// hex to bech32 and vice versa.
/// @custom:address 0x0000000000000000000000000000000000000400
interface Bech32I {
    /// @dev Defines a method for converting a hex formatted address to bech32.
    /// @param addr The hex address to be converted.
    /// @param prefix The human readable prefix (HRP) of the bech32 address.
    /// @return bech32Address The address in bech32 format.
    function hexToBech32(
        address addr,
        string memory prefix
    ) external returns (string memory bech32Address);

    /// @dev Defines a method for converting a bech32 formatted address to hex.
    /// @param bech32Address The bech32 address to be converted.
    /// @return addr The address in hex format.
    function bech32ToHex(
        string memory bech32Address
    ) external returns (address addr);
}
```

### Available Functions

The Bech32 module precompile provides the following key functionalities:

- **Address Encoding**: Convert hex addresses to Bech32 format using `hexToBech32` with custom prefixes
- **Address Decoding**: Parse Bech32 addresses to extract hex format using `bech32ToHex`
- **Format Validation**: Verify Bech32 address format and checksums during conversion
- **Cross-Chain Compatibility**: Support for various Cosmos chain address formats through prefix parameter



## Staking Module Precompile

The Staking module precompile provides validator delegation functionality.

### Interface

Based on the [StakingI.sol interface](https://github.com/cosmos/evm/blob/v0.3.0/precompiles/staking/StakingI.sol):

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The StakingI contract's address.
address constant STAKING_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000800;

/// @dev The StakingI contract's instance.
StakingI constant STAKING_CONTRACT = StakingI(STAKING_PRECOMPILE_ADDRESS);

/// @dev Define all the available staking methods.
string constant MSG_CREATE_VALIDATOR = "/cosmos.staking.v1beta1.MsgCreateValidator";
string constant MSG_EDIT_VALIDATOR = "/cosmos.staking.v1beta1.MsgEditValidator";
string constant MSG_DELEGATE = "/cosmos.staking.v1beta1.MsgDelegate";
string constant MSG_UNDELEGATE = "/cosmos.staking.v1beta1.MsgUndelegate";
string constant MSG_REDELEGATE = "/cosmos.staking.v1beta1.MsgBeginRedelegate";
string constant MSG_CANCEL_UNDELEGATION = "/cosmos.staking.v1beta1.MsgCancelUnbondingDelegation";

/// @dev Constant used in flags to indicate that commission rate field should not be updated
int256 constant DO_NOT_MODIFY_COMMISSION_RATE = -1;

/// @dev Constant used in flags to indicate that min self delegation field should not be updated
int256 constant DO_NOT_MODIFY_MIN_SELF_DELEGATION = -1;

/// @dev Defines the initial description to be used for creating
/// a validator.
struct Description {
    string moniker;
    string identity;
    string website;
    string securityContact;
    string details;
}

/// @dev Defines the initial commission rates to be used for creating
/// a validator.
struct CommissionRates {
    uint256 rate;
    uint256 maxRate;
    uint256 maxChangeRate;
}

/// @dev Defines commission parameters for a given validator.
struct Commission {
    CommissionRates commissionRates;
    uint256 updateTime;
}

/// @dev Represents a validator in the staking module.
struct Validator {
    string operatorAddress;
    string consensusPubkey;
    bool jailed;
    BondStatus status;
    uint256 tokens;
    uint256 delegatorShares; // TODO: decimal
    string description;
    int64 unbondingHeight;
    int64 unbondingTime;
    uint256 commission;
    uint256 minSelfDelegation;
}

/// @dev Represents the output of a Redelegations query.
struct RedelegationResponse {
    Redelegation redelegation;
    RedelegationEntryResponse[] entries;
}

/// @dev Represents a redelegation between a delegator and a validator.
struct Redelegation {
    string delegatorAddress;
    string validatorSrcAddress;
    string validatorDstAddress;
    RedelegationEntry[] entries;
}

/// @dev Represents a RedelegationEntryResponse for the Redelegations query.
struct RedelegationEntryResponse {
    RedelegationEntry redelegationEntry;
    uint256 balance;
}

/// @dev Represents a single Redelegation entry.
struct RedelegationEntry {
    int64 creationHeight;
    int64 completionTime;
    uint256 initialBalance;
    uint256 sharesDst; // TODO: decimal
}

/// @dev Represents the output of the Redelegation query.
struct RedelegationOutput {
    string delegatorAddress;
    string validatorSrcAddress;
    string validatorDstAddress;
    RedelegationEntry[] entries;
}

/// @dev Represents a single entry of an unbonding delegation.
struct UnbondingDelegationEntry {
    int64 creationHeight;
    int64 completionTime;
    uint256 initialBalance;
    uint256 balance;
    uint64 unbondingId;
    int64 unbondingOnHoldRefCount;
}

/// @dev Represents the output of the UnbondingDelegation query.
struct UnbondingDelegationOutput {
    string delegatorAddress;
    string validatorAddress;
    UnbondingDelegationEntry[] entries;
}

/// @dev The status of the validator.
enum BondStatus {
    Unspecified,
    Unbonded,
    Unbonding,
    Bonded
}

/// @author Evmos Team
/// @title Staking Precompiled Contract
/// @dev The interface through which solidity contracts will interact with staking.
/// We follow this same interface including four-byte function selectors, in the precompile that
/// wraps the pallet.
/// @custom:address 0x0000000000000000000000000000000000000800
interface StakingI {
    /// @dev Defines a method for creating a new validator.
    /// @param description The initial description
    /// @param commissionRates The initial commissionRates
    /// @param minSelfDelegation The validator's self declared minimum self delegation
    /// @param validatorAddress The validator address
    /// @param pubkey The consensus public key of the validator
    /// @param value The amount of the coin to be self delegated to the validator
    /// @return success Whether or not the create validator was successful
    function createValidator(
        Description calldata description,
        CommissionRates calldata commissionRates,
        uint256 minSelfDelegation,
        address validatorAddress,
        string memory pubkey,
        uint256 value
    ) external returns (bool success);

    /// @dev Defines a method for edit a validator.
    /// @param description Description parameter to be updated. Use the string "[do-not-modify]"
    /// as the value of fields that should not be updated.
    /// @param commissionRate CommissionRate parameter to be updated.
    /// Use commissionRate = -1 to keep the current value and not update it.
    /// @param minSelfDelegation MinSelfDelegation parameter to be updated.
    /// Use minSelfDelegation = -1 to keep the current value and not update it.
    /// @return success Whether or not edit validator was successful.
    function editValidator(
        Description calldata description,
        address validatorAddress,
        int256 commissionRate,
        int256 minSelfDelegation
    ) external returns (bool success);

    /// @dev Defines a method for performing a delegation of coins from a delegator to a validator.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of the bond denomination to be delegated to the validator.
    /// This amount should use the bond denomination precision stored in the bank metadata.
    /// @return success Whether or not the delegate was successful
    function delegate(
        address delegatorAddress,
        string memory validatorAddress,
        uint256 amount
    ) external returns (bool success);

    /// @dev Defines a method for performing an undelegation from a delegate and a validator.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of the bond denomination to be undelegated from the validator.
    /// This amount should use the bond denomination precision stored in the bank metadata.
    /// @return completionTime The time when the undelegation is completed
    function undelegate(
        address delegatorAddress,
        string memory validatorAddress,
        uint256 amount
    ) external returns (int64 completionTime);

    /// @dev Defines a method for performing a redelegation
    /// of coins from a delegator and source validator to a destination validator.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorSrcAddress The validator from which the redelegation is initiated
    /// @param validatorDstAddress The validator to which the redelegation is destined
    /// @param amount The amount of the bond denomination to be redelegated to the validator
    /// This amount should use the bond denomination precision stored in the bank metadata.
    /// @return completionTime The time when the redelegation is completed
    function redelegate(
        address delegatorAddress,
        string memory validatorSrcAddress,
        string memory validatorDstAddress,
        uint256 amount
    ) external returns (int64 completionTime);

    /// @dev Allows delegators to cancel the unbondingDelegation entry
    /// and to delegate back to a previous validator.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of the bond denomination
    /// This amount should use the bond denomination precision stored in the bank metadata.
    /// @param creationHeight The height at which the unbonding took place
    /// @return success Whether or not the unbonding delegation was cancelled
    function cancelUnbondingDelegation(
        address delegatorAddress,
        string memory validatorAddress,
        uint256 amount,
        uint256 creationHeight
    ) external returns (bool success);

    /// @dev Queries the given amount of the bond denomination to a validator.
    /// @param delegatorAddress The address of the delegator.
    /// @param validatorAddress The address of the validator.
    /// @return shares The amount of shares, that the delegator has received.
    /// @return balance The amount in Coin, that the delegator has delegated to the given validator.
    /// This returned balance uses the bond denomination precision stored in the bank metadata.
    function delegation(
        address delegatorAddress,
        string memory validatorAddress
    ) external view returns (uint256 shares, Coin calldata balance);

    /// @dev Returns the delegation shares and coins, that are currently
    /// unbonding for a given delegator and validator pair.
    /// @param delegatorAddress The address of the delegator.
    /// @param validatorAddress The address of the validator.
    /// @return unbondingDelegation The delegations that are currently unbonding.
    function unbondingDelegation(
        address delegatorAddress,
        string memory validatorAddress
    )
        external
        view
        returns (UnbondingDelegationOutput calldata unbondingDelegation);

    /// @dev Queries validator info for a given validator address.
    /// @param validatorAddress The address of the validator.
    /// @return validator The validator info for the given validator address.
    function validator(
        address validatorAddress
    ) external view returns (Validator calldata validator);

    /// @dev Queries all validators that match the given status.
    /// @param status Enables to query for validators matching a given status.
    /// @param pageRequest Defines an optional pagination for the request.
    function validators(
        string memory status,
        PageRequest calldata pageRequest
    )
        external
        view
        returns (
            Validator[] calldata validators,
            PageResponse calldata pageResponse
        );

    /// @dev Queries all redelegations from a source to a destination validator for a given delegator.
    /// @param delegatorAddress The address of the delegator.
    /// @param srcValidatorAddress Defines the validator address to redelegate from.
    /// @param dstValidatorAddress Defines the validator address to redelegate to.
    /// @return redelegation The active redelegations for the given delegator, source and destination
    /// validator combination.
    function redelegation(
        address delegatorAddress,
        string memory srcValidatorAddress,
        string memory dstValidatorAddress
    ) external view returns (RedelegationOutput calldata redelegation);

    /// @dev Queries all redelegations based on the specified criteria:
    /// for a given delegator and/or origin validator address
    /// and/or destination validator address
    /// in a specified pagination manner.
    /// @param delegatorAddress The address of the delegator as string (can be a zero address).
    /// @param srcValidatorAddress Defines the validator address to redelegate from (can be empty string).
    /// @param dstValidatorAddress Defines the validator address to redelegate to (can be empty string).
    /// @param pageRequest Defines an optional pagination for the request.
    /// @return response Holds the redelegations for the given delegator, source and destination validator combination.
    function redelegations(
        address delegatorAddress,
        string memory srcValidatorAddress,
        string memory dstValidatorAddress,
        PageRequest calldata pageRequest
    )
        external
        view
        returns (
            RedelegationResponse[] calldata response,
            PageResponse calldata pageResponse
        );

    /// @dev CreateValidator defines an Event emitted when a create a new validator.
    /// @param validatorAddress The address of the validator
    /// @param value The amount of coin being self delegated
    event CreateValidator(address indexed validatorAddress, uint256 value);

    /// @dev EditValidator defines an Event emitted when edit a validator.
    /// @param validatorAddress The address of the validator.
    /// @param commissionRate The commission rate.
    /// @param minSelfDelegation The min self delegation.
    event EditValidator(
        address indexed validatorAddress,
        int256 commissionRate,
        int256 minSelfDelegation
    );

    /// @dev Delegate defines an Event emitted when a given amount of tokens are delegated from the
    /// delegator address to the validator address.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of bond denomination being delegated
    /// This amount has the bond denomination precision stored in the bank metadata.
    /// @param newShares The new delegation shares being held
    event Delegate(
        address indexed delegatorAddress,
        address indexed validatorAddress,
        uint256 amount,
        uint256 newShares
    );

    /// @dev Unbond defines an Event emitted when a given amount of tokens are unbonded from the
    /// validator address to the delegator address.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of bond denomination being unbonded
    /// This amount has the bond denomination precision stored in the bank metadata.
    /// @param completionTime The time at which the unbonding is completed
    event Unbond(
        address indexed delegatorAddress,
        address indexed validatorAddress,
        uint256 amount,
        uint256 completionTime
    );

    /// @dev Redelegate defines an Event emitted when a given amount of tokens are redelegated from
    /// the source validator address to the destination validator address.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorSrcAddress The address of the validator from which the delegation is retracted
    /// @param validatorDstAddress The address of the validator to which the delegation is directed
    /// @param amount The amount of bond denomination being redelegated
    /// This amount has the bond denomination precision stored in the bank metadata.
    /// @param completionTime The time at which the redelegation is completed
    event Redelegate(
        address indexed delegatorAddress,
        address indexed validatorSrcAddress,
        address indexed validatorDstAddress,
        uint256 amount,
        uint256 completionTime
    );

    /// @dev CancelUnbondingDelegation defines an Event emitted when a given amount of tokens
    /// that are in the process of unbonding from the validator address are bonded again.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of bond denomination that was in the unbonding process which is to be canceled
    /// This amount has the bond denomination precision stored in the bank metadata.
    /// @param creationHeight The block height at which the unbonding of a delegation was initiated
    event CancelUnbondingDelegation(
        address indexed delegatorAddress,
        address indexed validatorAddress,
        uint256 amount,
        uint256 creationHeight
    );
}
```

### Available Functions

The Staking module precompile provides the following key functionalities:

- **Validator Creation**: Create new validators using `createValidator` with description and commission rates
- **Validator Editing**: Edit existing validators using `editValidator` with optional parameter updates
- **Delegation Management**: Delegate, undelegate, and redelegate tokens using `delegate`, `undelegate`, and `redelegate`
- **Unbonding Management**: Cancel unbonding delegations using `cancelUnbondingDelegation`
- **Validator Queries**: Query validator information using `validator` and `validators` with pagination
- **Delegation Queries**: Check delegation amounts and status using `delegation`, `unbondingDelegation`, and `redelegation`

## Distribution Module Precompile

The Distribution module precompile handles staking rewards and commission distribution.

### Interface

Based on the [DistributionI.sol interface](https://github.com/cosmos/evm/blob/v0.3.0/precompiles/distribution/DistributionI.sol):

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The DistributionI contract's address.
address constant DISTRIBUTION_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000801;

/// @dev Define all the available distribution methods.
string constant MSG_SET_WITHDRAWER_ADDRESS = "/cosmos.distribution.v1beta1.MsgSetWithdrawAddress";
string constant MSG_WITHDRAW_DELEGATOR_REWARD = "/cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward";
string constant MSG_WITHDRAW_VALIDATOR_COMMISSION = "/cosmos.distribution.v1beta1.MsgWithdrawValidatorCommission";

/// @dev The DistributionI contract's instance.
DistributionI constant DISTRIBUTION_CONTRACT = DistributionI(
    DISTRIBUTION_PRECOMPILE_ADDRESS
);

struct ValidatorSlashEvent {
    uint64 validatorPeriod;
    Dec fraction;
}

struct ValidatorDistributionInfo {
    string operatorAddress;
    DecCoin[] selfBondRewards;
    DecCoin[] commission;
}

struct DelegationDelegatorReward {
    string validatorAddress;
    DecCoin[] reward;
}

/// @author Evmos Team
/// @title Distribution Precompile Contract
/// @dev The interface through which solidity contracts will interact with Distribution
/// @custom:address 0x0000000000000000000000000000000000000801
interface DistributionI {
    /// @dev ClaimRewards defines an Event emitted when rewards are claimed
    /// @param delegatorAddress the address of the delegator
    /// @param amount the amount being claimed
    event ClaimRewards(address indexed delegatorAddress, uint256 amount);

    /// @dev SetWithdrawerAddress defines an Event emitted when a new withdrawer address is being set
    /// @param caller the caller of the transaction
    /// @param withdrawerAddress the newly set withdrawer address
    event SetWithdrawerAddress(
        address indexed caller,
        string withdrawerAddress
    );

    /// @dev WithdrawDelegatorReward defines an Event emitted when rewards from a delegation are withdrawn
    /// @param delegatorAddress the address of the delegator
    /// @param validatorAddress the address of the validator
    /// @param amount the amount being withdrawn from the delegation
    event WithdrawDelegatorReward(
        address indexed delegatorAddress,
        address indexed validatorAddress,
        uint256 amount
    );

    /// @dev WithdrawValidatorCommission defines an Event emitted when validator commissions are being withdrawn
    /// @param validatorAddress is the address of the validator
    /// @param commission is the total commission earned by the validator
    event WithdrawValidatorCommission(
        string indexed validatorAddress,
        uint256 commission
    );

    /// @dev FundCommunityPool defines an Event emitted when an account
    /// fund the community pool
    /// @param depositor the address funding the community pool
    /// @param denom the denomination of the coin being sent to the community pool
    /// @param amount the amount being sent to the community pool
    event FundCommunityPool(address indexed depositor, string denom, uint256 amount);

    /// @dev DepositValidatorRewardsPool defines an Event emitted when an account
    /// deposits the validator rewards pool
    /// @param depositor the address funding the validator rewards pool
    /// @param validatorAddress the address of the validator
    /// @param denom the denomination of the coin being sent to the validator rewards pool
    /// @param amount the amount of the coin being sent to the validator rewards pool
    event DepositValidatorRewardsPool(
        address indexed depositor,
        address indexed validatorAddress,
        string denom,
        uint256 amount
    );

    /// TRANSACTIONS

    /// @dev Claims all rewards from a select set of validators or all of them for a delegator.
    /// @param delegatorAddress The address of the delegator
    /// @param maxRetrieve The maximum number of validators to claim rewards from
    /// @return success Whether the transaction was successful or not
    function claimRewards(
        address delegatorAddress,
        uint32 maxRetrieve
    ) external returns (bool success);

    /// @dev Change the address, that can withdraw the rewards of a delegator.
    /// Note that this address cannot be a module account.
    /// @param delegatorAddress The address of the delegator
    /// @param withdrawerAddress The address that will be capable of withdrawing rewards for
    /// the given delegator address
    function setWithdrawAddress(
        address delegatorAddress,
        string memory withdrawerAddress
    ) external returns (bool success);

    /// @dev Withdraw the rewards of a delegator from a validator
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @return amount The amount of Coin withdrawn
    function withdrawDelegatorRewards(
        address delegatorAddress,
        string memory validatorAddress
    ) external returns (Coin[] calldata amount);

    /// @dev Withdraws the rewards commission of a validator.
    /// @param validatorAddress The address of the validator
    /// @return amount The amount of Coin withdrawn
    function withdrawValidatorCommission(
        string memory validatorAddress
    ) external returns (Coin[] calldata amount);

    /// @dev fundCommunityPool defines a method to allow an account to directly
    /// fund the community pool.
    /// @param depositor The address of the depositor
    /// @param amount The amount of coin sent to the community pool
    /// @return success Whether the transaction was successful or not
    function fundCommunityPool(
        address depositor,
        Coin[] memory amount
    ) external returns (bool success);

    /// @dev depositValidatorRewardsPool defines a method to allow an account to directly
    /// fund the validator rewards pool.
    /// @param depositor The address of the depositor
    /// @param validatorAddress The address of the validator
    /// @param amount The amount of coin sent to the validator rewards pool
    /// @return success Whether the transaction was successful or not
    function depositValidatorRewardsPool(
        address depositor,
        string memory validatorAddress,
        Coin[] memory amount
    ) external returns (bool success);

    /// QUERIES
    /// @dev Queries validator commission and self-delegation rewards for validator.
    /// @param validatorAddress The address of the validator
    /// @return distributionInfo The validator's distribution info
    function validatorDistributionInfo(
        string memory validatorAddress
    )
        external
        view
        returns (ValidatorDistributionInfo calldata distributionInfo);

    /// @dev Queries the outstanding rewards of a validator address.
    /// @param validatorAddress The address of the validator
    /// @return rewards The validator's outstanding rewards
    function validatorOutstandingRewards(
        string memory validatorAddress
    ) external view returns (DecCoin[] calldata rewards);

    /// @dev Queries the accumulated commission for a validator.
    /// @param validatorAddress The address of the validator
    /// @return commission The validator's commission
    function validatorCommission(
        string memory validatorAddress
    ) external view returns (DecCoin[] calldata commission);

    /// @dev Queries the slashing events for a validator in a given height interval
    /// defined by the starting and ending height.
    /// @param validatorAddress The address of the validator
    /// @param startingHeight The starting height
    /// @param endingHeight The ending height
    /// @param pageRequest Defines a pagination for the request.
    /// @return slashes The validator's slash events
    /// @return pageResponse The pagination response for the query
    function validatorSlashes(
        string memory validatorAddress,
        uint64 startingHeight,
        uint64 endingHeight,
        PageRequest calldata pageRequest
    )
        external
        view
        returns (
            ValidatorSlashEvent[] calldata slashes,
            PageResponse calldata pageResponse
        );

    /// @dev Queries the total rewards accrued by a delegation from a specific address to a given validator.
    /// @param delegatorAddress The address of the delegator
    /// @param validatorAddress The address of the validator
    /// @return rewards The total rewards accrued by a delegation.
    function delegationRewards(
        address delegatorAddress,
        string memory validatorAddress
    ) external view returns (DecCoin[] calldata rewards);

    /// @dev Queries the total rewards accrued by each validator, that a given
    /// address has delegated to.
    /// @param delegatorAddress The address of the delegator
    /// @return rewards The total rewards accrued by each validator for a delegator.
    /// @return total The total rewards accrued by a delegator.
    function delegationTotalRewards(
        address delegatorAddress
    )
        external
        view
        returns (
            DelegationDelegatorReward[] calldata rewards,
            DecCoin[] calldata total
        );

    /// @dev Queries all validators, that a given address has delegated to.
    /// @param delegatorAddress The address of the delegator
    /// @return validators The addresses of all validators, that were delegated to by the given address.
    function delegatorValidators(
        address delegatorAddress
    ) external view returns (string[] calldata validators);

    /// @dev Queries the address capable of withdrawing rewards for a given delegator.
    /// @param delegatorAddress The address of the delegator
    /// @return withdrawAddress The address capable of withdrawing rewards for the delegator.
    function delegatorWithdrawAddress(
        address delegatorAddress
    ) external view returns (string memory withdrawAddress);

    /// @dev Queries the coins in the community pool.
    /// @return coins The coins in the community pool
    function communityPool() external view returns (DecCoin[] calldata coins);
}
```

### Available Functions

The Distribution module precompile provides the following key functionalities:

- **Reward Claims**: Claim all rewards from validators using `claimRewards` with maxRetrieve parameter
- **Withdraw Address Management**: Set withdraw addresses using `setWithdrawAddress`
- **Reward Withdrawal**: Withdraw delegator rewards using `withdrawDelegatorRewards`
- **Commission Withdrawal**: Withdraw validator commission using `withdrawValidatorCommission`
- **Community Pool Funding**: Fund community pool using `fundCommunityPool`
- **Validator Rewards Pool**: Deposit to validator rewards pool using `depositValidatorRewardsPool`
- **Distribution Queries**: Query validator distribution info, outstanding rewards, and commission using respective functions


## Governance Module Precompile

The Governance module precompile provides proposal and voting functionality.

### Interface

Based on the [IGov.sol interface](https://github.com/cosmos/evm/blob/v0.3.0/precompiles/gov/IGov.sol):

```solidity
/// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The IGov contract's address.
address constant GOV_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000805;

/// @dev The IGov contract's instance.
IGov constant GOV_CONTRACT = IGov(GOV_PRECOMPILE_ADDRESS);

/**
 * @dev VoteOption enumerates the valid vote options for a given governance proposal.
 */
enum VoteOption {
    // Unspecified defines a no-op vote option.
    Unspecified,
    // Yes defines a yes vote option.
    Yes,
    // Abstain defines an abstain vote option.
    Abstain,
    // No defines a no vote option.
    No,
    // NoWithWeto defines a no with veto vote option.
    NoWithWeto
}
/// @dev WeightedVote represents a vote on a governance proposal
struct WeightedVote {
    uint64 proposalId;
    address voter;
    WeightedVoteOption[] options;
    string metadata;
}

/// @dev WeightedVoteOption represents a weighted vote option
struct WeightedVoteOption {
    VoteOption option;
    string weight;
}

/// @dev DepositData represents information about a deposit on a proposal
struct DepositData {
    uint64 proposalId;
    address depositor;
    Coin[] amount;
}

/// @dev TallyResultData represents the tally result of a proposal
struct TallyResultData {
    string yes;
    string abstain;
    string no;
    string noWithVeto;
}

/// @dev ProposalData represents a governance proposal
struct ProposalData {
    uint64 id;
    string[] messages;
    uint32 status;
    TallyResultData finalTallyResult;
    uint64 submitTime;
    uint64 depositEndTime;
    Coin[] totalDeposit;
    uint64 votingStartTime;
    uint64 votingEndTime;
    string metadata;
    string title;
    string summary;
    address proposer;
}

/// @dev Params defines the governance parameters
struct Params {
    int64 votingPeriod;
    Coin[] minDeposit;
    int64 maxDepositPeriod;
    string quorum;
    string threshold;
    string vetoThreshold;
    string minInitialDepositRatio;
    string proposalCancelRatio;
    string proposalCancelDest;
    int64 expeditedVotingPeriod;
    string expeditedThreshold;
    Coin[] expeditedMinDeposit;
    bool burnVoteQuorum;
    bool burnProposalDepositPrevote;
    bool burnVoteVeto;
    string minDepositRatio;
}

/// @author The Evmos Core Team
/// @title Gov Precompile Contract
/// @dev The interface through which solidity contracts will interact with Gov
interface IGov {
    /// @dev SubmitProposal defines an Event emitted when a proposal is submitted.
    /// @param proposer the address of the proposer
    /// @param proposalId the proposal of id
    event SubmitProposal(address indexed proposer, uint64 proposalId);

    /// @dev CancelProposal defines an Event emitted when a proposal is canceled.
    /// @param proposer the address of the proposer
    /// @param proposalId the proposal of id
    event CancelProposal(address indexed proposer, uint64 proposalId);

    /// @dev Deposit defines an Event emitted when a deposit is made.
    /// @param depositor the address of the depositor
    /// @param proposalId the proposal of id
    /// @param amount the amount of the deposit
    event Deposit(address indexed depositor, uint64 proposalId, Coin[] amount);

    /// @dev Vote defines an Event emitted when a proposal voted.
    /// @param voter the address of the voter
    /// @param proposalId the proposal of id
    /// @param option the option for voter
    event Vote(address indexed voter, uint64 proposalId, uint8 option);

    /// @dev VoteWeighted defines an Event emitted when a proposal voted.
    /// @param voter the address of the voter
    /// @param proposalId the proposal of id
    /// @param options the options for voter
    event VoteWeighted(
        address indexed voter,
        uint64 proposalId,
        WeightedVoteOption[] options
    );

    /// TRANSACTIONS

    /// @notice submitProposal creates a new proposal from a protoJSON document.
    /// @dev submitProposal defines a method to submit a proposal.
    /// @param jsonProposal The JSON proposal
    /// @param deposit The deposit for the proposal
    /// @return proposalId The proposal id
    function submitProposal(
        address proposer,
        bytes calldata jsonProposal,
        Coin[] calldata deposit
    ) external returns (uint64 proposalId);

    /// @dev cancelProposal defines a method to cancel a proposal.
    /// @param proposalId The proposal id
    /// @return success Whether the transaction was successful or not
    function cancelProposal(
        address proposer,
        uint64 proposalId
    ) external returns (bool success);

    /// @dev deposit defines a method to add a deposit to a proposal.
    /// @param proposalId The proposal id
    /// @param amount The amount to deposit
    function deposit(
        address depositor,
        uint64 proposalId,
        Coin[] calldata amount
    ) external returns (bool success);


    /// @dev vote defines a method to add a vote on a specific proposal.
    /// @param voter The address of the voter
    /// @param proposalId the proposal of id
    /// @param option the option for voter
    /// @param metadata the metadata for voter send
    /// @return success Whether the transaction was successful or not
    function vote(
        address voter,
        uint64 proposalId,
        VoteOption option,
        string memory metadata
    ) external returns (bool success);

    /// @dev voteWeighted defines a method to add a vote on a specific proposal.
    /// @param voter The address of the voter
    /// @param proposalId The proposal id
    /// @param options The options for voter
    /// @param metadata The metadata for voter send
    /// @return success Whether the transaction was successful or not
    function voteWeighted(
        address voter,
        uint64 proposalId,
        WeightedVoteOption[] calldata options,
        string memory metadata
    ) external returns (bool success);

    /// QUERIES

    /// @dev getVote returns the vote of a single voter for a
    /// given proposalId.
    /// @param proposalId The proposal id
    /// @param voter The voter on the proposal
    /// @return vote Voter's vote for the proposal
    function getVote(
        uint64 proposalId,
        address voter
    ) external view returns (WeightedVote memory vote);

    /// @dev getVotes Returns the votes for a specific proposal.
    /// @param proposalId The proposal id
    /// @param pagination The pagination options
    /// @return votes The votes for the proposal
    /// @return pageResponse The pagination information
    function getVotes(
        uint64 proposalId,
        PageRequest calldata pagination
    )
        external
        view
        returns (WeightedVote[] memory votes, PageResponse memory pageResponse);

    /// @dev getDeposit returns the deposit of a single depositor for a given proposalId.
    /// @param proposalId The proposal id
    /// @param depositor The address of the depositor
    /// @return deposit The deposit information
    function getDeposit(
        uint64 proposalId,
        address depositor
    ) external view returns (DepositData memory deposit);

    /// @dev getDeposits returns all deposits for a specific proposal.
    /// @param proposalId The proposal id
    /// @param pagination The pagination options
    /// @return deposits The deposits for the proposal
    /// @return pageResponse The pagination information
    function getDeposits(
        uint64 proposalId,
        PageRequest calldata pagination
    )
        external
        view
        returns (
            DepositData[] memory deposits,
            PageResponse memory pageResponse
        );

    /// @dev getTallyResult returns the tally result of a proposal.
    /// @param proposalId The proposal id
    /// @return tallyResult The tally result of the proposal
    function getTallyResult(
        uint64 proposalId
    ) external view returns (TallyResultData memory tallyResult);

    /// @dev getProposal returns the proposal details based on proposal id.
    /// @param proposalId The proposal id
    /// @return proposal The proposal data
    function getProposal(
        uint64 proposalId
    ) external view returns (ProposalData memory proposal);

    /// @dev getProposals returns proposals with matching status.
    /// @param proposalStatus The proposal status to filter by
    /// @param voter The voter address to filter by, if any
    /// @param depositor The depositor address to filter by, if any
    /// @param pagination The pagination config
    /// @return proposals The proposals matching the filter criteria
    /// @return pageResponse The pagination information
    function getProposals(
        uint32 proposalStatus,
        address voter,
        address depositor,
        PageRequest calldata pagination
    )
        external
        view
        returns (
            ProposalData[] memory proposals,
            PageResponse memory pageResponse
        );

    /// @dev getParams returns the current governance parameters.
    /// @return params The governance parameters
    function getParams() external view returns (Params memory params);

    /// @dev getConstitution returns the current constitution.
    /// @return constitution The current constitution
    function getConstitution() external view returns (string memory constitution);
}
```

### Available Functions

The Governance module precompile provides the following key functionalities:

- **Proposal Management**: Submit and cancel proposals using `submitProposal` and `cancelProposal` with JSON proposals
- **Deposit Management**: Add deposits to proposals using `deposit`
- **Voting**: Cast votes using `vote` with VoteOption enum or weighted votes using `voteWeighted`
- **Proposal Queries**: Get proposal details using `getProposal`, `getProposals` with filtering and pagination
- **Vote Queries**: Query votes using `getVote` and `getVotes` with pagination
- **Deposit Queries**: Query deposits using `getDeposit` and `getDeposits` with pagination
- **Tally Results**: Get proposal tally results using `getTallyResult`
- **Parameter Management**: Query governance parameters using `getParams` and `getConstitution`


## Slashing Module Precompile

The Slashing module precompile handles validator slashing and unjailing.

### Interface

Based on the [ISlashing.sol interface](https://github.com/cosmos/evm/blob/v0.3.0/precompiles/slashing/ISlashing.sol):

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity >=0.8.17;

import "../common/Types.sol";

/// @dev The ISlashing contract's address.
address constant SLASHING_PRECOMPILE_ADDRESS = 0x0000000000000000000000000000000000000806;

/// @dev The ISlashing contract's instance.
ISlashing constant SLASHING_CONTRACT = ISlashing(SLASHING_PRECOMPILE_ADDRESS);

/// @dev SigningInfo defines a validator's signing info for monitoring their
/// liveness activity.
struct SigningInfo {
    /// @dev Address of the validator
    address validatorAddress;
    /// @dev Height at which validator was first a candidate OR was unjailed
    int64 startHeight;
    /// @dev Index offset into signed block bit array
    int64 indexOffset;
    /// @dev Timestamp until which validator is jailed due to liveness downtime
    int64 jailedUntil;
    /// @dev Whether or not a validator has been tombstoned (killed out of validator set)
    bool tombstoned;
    /// @dev Missed blocks counter (to avoid scanning the array every time)
    int64 missedBlocksCounter;
}

/// @dev Params defines the parameters for the slashing module.
struct Params {
    /// @dev SignedBlocksWindow defines how many blocks the validator should have signed
    int64 signedBlocksWindow;
    /// @dev MinSignedPerWindow defines the minimum blocks signed per window to avoid slashing
    Dec minSignedPerWindow;
    /// @dev DowntimeJailDuration defines how long the validator will be jailed for downtime
    int64 downtimeJailDuration;
    /// @dev SlashFractionDoubleSign defines the percentage of slash for double sign
    Dec slashFractionDoubleSign;
    /// @dev SlashFractionDowntime defines the percentage of slash for downtime
    Dec slashFractionDowntime;
}

/// @author Evmos Team
/// @title Slashing Precompiled Contract
/// @dev The interface through which solidity contracts will interact with slashing.
/// We follow this same interface including four-byte function selectors, in the precompile that
/// wraps the pallet.
/// @custom:address 0x0000000000000000000000000000000000000806
interface ISlashing {
    /// @dev Emitted when a validator is unjailed
    /// @param validator The address of the validator
    event ValidatorUnjailed(address indexed validator);

    /// @dev GetSigningInfo returns the signing info for a specific validator.
    /// @param consAddress The validator consensus address
    /// @return signingInfo The validator signing info
    function getSigningInfo(
        address consAddress
    ) external view returns (SigningInfo memory signingInfo);

    /// @dev GetSigningInfos returns the signing info for all validators.
    /// @param pagination Pagination configuration for the query
    /// @return signingInfos The list of validator signing info
    /// @return pageResponse Pagination information for the response
    function getSigningInfos(
        PageRequest calldata pagination
    ) external view returns (SigningInfo[] memory signingInfos, PageResponse memory pageResponse);

    /// @dev Unjail allows validators to unjail themselves after being jailed for downtime
    /// @param validatorAddress The validator operator address to unjail
    /// @return success true if the unjail operation was successful
    function unjail(address validatorAddress) external returns (bool success);

    /// @dev GetParams returns the slashing module parameters
    /// @return params The slashing module parameters
    function getParams() external view returns (Params memory params);
}
```

### Available Functions

The Slashing module precompile provides the following key functionalities:

- **Validator Unjailing**: Release validators from jail using `unjail` when conditions are met
- **Signing Information**: Query validator signing performance and status using `getSigningInfo`
- **Bulk Signing Info**: Get signing information for all validators using `getSigningInfos` with pagination
- **Parameter Queries**: Query slashing module parameters using `getParams`
- **Jail Status**: Check if validators are currently jailed through SigningInfo struct
- **Missed Blocks**: Track validator missed block counts through SigningInfo struct
- **Tombstone Status**: Check if validators are permanently removed through SigningInfo struct


## Best Practices

1. **Gas Optimization**: Precompile contract calls may have lower gas costs compared to regular contract calls.

2. **Error Handling**: Implement proper error handling for precompile contract calls.

3. **Address Validation**: Verify that precompile contract addresses are correct.

4. **Permission Checks**: Some functions may require specific permissions.

5. **Input Validation**: Validate inputs before calling precompile functions.

## Limitations

- Precompile contracts support both read-only and state-changing functions.
- Some complex queries may not be supported.
- Gas costs may vary depending on network conditions.
- Address format conversions are handled automatically by the precompile contracts.

## Troubleshooting

### Common Issues

1. **Invalid Address**: Ensure precompile contract addresses are correct.
2. **Invalid Input**: Verify parameter formats when calling functions.
3. **Insufficient Permissions**: Some functions require specific permissions.
4. **Network Issues**: Check network connectivity and gas limits.

### Debugging Tips

- Use `eth_call` for testing read-only functions via RPC.
- Check detailed error messages when transactions fail.
- Verify network status and gas limits.
- Test with small amounts before large transactions.

