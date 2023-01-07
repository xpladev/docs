---
title: Subcommands
weight: 50
---

This section describes the subcommands available for `xplad`.

## `debug addr`

Changes an address from hex encoding to bech32.

**Syntax**

```bash
xplad debug addr <address>
```

**Example**

```bash
xplad debug addr xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm
```

## `debug pubkey`

Decodes a pubkey from proto JSON and displays the address.

**Syntax**

```bash
xplad debug pubkey <pubkey>
```

**Example**

```bash
xplad debug pubkey '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AurroA7jvfPd1AadmmOvWM2rJSwipXfRf8yD6pLbA2DJ"}'
```

## `debug raw-bytes`

Changes raw bytes to hex.

**Syntax**

```bash
xplad debug raw-bytes <raw-bytes>
```

**Example**

```bash
xplad debug raw-bytes [72 101 108 108 111 44 32 112 108 97 121 103 114 111 117 110 100]
```

## `keys add`

Generates a public and private key pair for an account so that you can receive funds, send funds, create bonding transactions, and so on.

{{< hint info >}}
**Tip**
For security purposes, run this command on an offline computer.
{{< /hint >}}

**Syntax**

```bash
xplad keys add <your-key-name>
```

where `<your-key-name>` is the name of the account. It references the account number used to derive the key pair from the mnemonic. When you want to send a transaction, you will use this name to identify your account.

To specify the path \(`0`, `1`, `2`, ...\) you want to use to generate your account, you can append the optional `--account` flag. By default, account `0` is generated.

The command generates a 24-word mnemonic and saves the private and public keys for account `0` simultaneously. You are prompted to specify a passphrase that is used to encrypt the private key of account `0` on disk. Each time you want to send a transaction, this password is required. If you lose the password, you can always recover the private key by using the mnemonic phrase.

{{< hint danger >}}
**Danger**
To prevent theft or loss of funds, ensure that you keep multiple copies of your mnemonic and store it in a secure place and that only you know how to access it. If someone is able to gain access to your mnemonic, they are able to gain access to your private keys and control the accounts associated with them.
{{< /hint >}}

{{< hint info >}}
**Tip**
After you have triple-checked your mnemonic and safely stored it, you can delete bash history to ensure no one can retrieve it.
```bash
history -c
rm ~/.bash_history
```
{{< /hint >}}

To generate more accounts from the same mnemonic, run:

```bash
xplad keys add <your-key-name> --recover --account 1
```

You are prompted to specify a passphrase and your mnemonic. To generate a different account, change the account number.

{{< hint danger >}}
**Danger**
Do not use the same passphrase for multiple keys. Do not lose or share your mnemonic with anyone.
{{< /hint >}}

**Example**

```bash
xplad keys add myAccount
```

In some cases, you might need to recover your key. If you have the mnemonic that was used to generate your private key, you can recover it and re-register your key. Issuing the following command will prompt you to enter your 24-word mnemonic.

**Syntax**

```bash
xplad keys add <yourKeyName> --recover
```

For information about generating multisignature accounts and signing transactions, see [Sign with a multisig account]({{< ref "sign-with-a-multisig-account" >}}).

## `keys show`

Retrieves an address for a specified account. The address is prefixed by `xpla-`, for example `xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc`. To receive funds, you must give an account address to the sender.

**Syntax**

```bash
xplad keys show <account-name>
```

**Example**

```bash
xplad keys show myAccount
```

To show a validator's address, append the `--bech=val` flag to the end of the command statement, as shown in the following example:

```bash
xplad keys show accountExample --bech=val
```

To show the validator consensus address that is generated when the node is created by `xplad init` and the Tendermint signing key for the node, use the `tendermint` command, as shown in the following example:

```bash
xplad tendermint show-address
```

## `keys list`

Lists all your keys.

**Syntax**

```bash
xplad keys list
```

## `query authz grants`

Retrieves all existing grants between a granter and a grantee.

**Syntax**

```bash
xplad query authz grants <granter-address> <grantee-address>
```

**Example**

```bash
xplad query authz grants xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm
```

Additionally, the `grants` command can retrieve the specific grant between a granter and a grantee for a message type by appending the message type URL to the end of the command statement, as shown in the following example:

```bash
xplad query authz grants xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm /cosmos.bank.v1beta1.MsgSend
```

## `query bank balances`

Displays your account balance, account number, and sequence number (nonce).

**Syntax**

```bash
xplad query bank balances <account-address>
```

**Example**

```bash
xplad query bank balances xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

{{< hint info >}}
**Note**
When you query an account balance that has zero tokens or you fund an account before your node has fully synced with the chain, this error message is sent:
`No account with address <account-address> was found in the state`.
{{< /hint >}}

## `query bank denom-metadata`

Query the client metadata for coin denominations.

## `query bank total`

Query the total supply of coins of the chain.

## `query distribution rewards`

Checks the all the current outstanding rewards that have not been withdrawn.

**Syntax**

```bash
xplad query distribution rewards
```

Check the current rewards earned by a specific delegator by appending the `<delegator-address>` at the end of the command statement, as shown in the following example:

```bash
xplad query distribution rewards xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm
```

Check the current rewards earned by a delegator and restricted to one validator by appending the `<delegator-address>` followed by the `<validator-address>` at the end of the command statement, as shown in the following example:

```bash
xplad query distribution rewards xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm xpla19t4gde4f8ndwx67qhbnur9yqdc31xznpksajbcy
```

## `query distribution commission`

Checks the current outstanding commission for a validator.

**Syntax**

```bash
xplad query distribution commission <validator_address>
```

**Example**

```bash
xplad query distribution commission xpla19t4gde4f8ndwx67qhbnur9yqdc31xznpksajbcy
```

## `query distribution slashes`

Checks historical slashes for a validator within a range of blocks.

**Syntax**

```bash
xplad query distribution slashes <validator-address> <start-block-height> <end-block-height>
```

**Example**

```bash
xplad query distribution slashes xpla19t4gde4f8ndwx67qhbnur9yqdc31xznpksajbcy 25 300
```

## `query distribution community-pool`

Checks all coins in the community pool.

**Syntax**

```bash
xplad query distribution community-pool
```

## `query distribution params`

Checks the current distribution parameters.

**Syntax**

```bash
xplad query distribution params
```

The parameters are returned in YAML, as shown in the following example:

```yaml
community_tax: "0.000000000000000000"
base_proposer_reward: "0.010000000000000000"
bonus_proposer_reward: "0.040000000000000000"
withdraw_addr_enabled: true
```

## `query gov deposit`

Retrieves information about a single proposal deposit on a proposal by its identifier.

**Syntax**

```bash
xplad query gov deposit <proposal-id> <depositor-address>
```

**Example**

```bash
xplad query gov deposit 4 xpla1skjwj5whet0lpe65qaq4rpq03hjxlwd9nf39lk
```

## `query gov deposits`

Retrieves all deposits submitted to a proposal after it is created.

**Syntax**

```bash
xplad query gov deposits <proposal-id>
```

**Example**

```bash
xplad query gov deposits 5
```

## `query gov proposal`

Retrieves information about one proposal.

`proposal` retrieves information about one proposal.

**Syntax**

```bash
xplad query gov proposal <proposal-id>
```

**Example**

```bash
xplad query gov proposal 3
```

## `query gov proposals`

Retrieves information about all available proposals.

**Syntax**

```bash
xplad query gov proposals
```

Additionally, you can query proposals filtered by details, such as `voter` or `depositor`, by appending the corresponding flag and address at the end of the command statement, as shown in the following example:

```bash
xplad query gov proposals --voter xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb
```

## `query gov vote`

Retrieves information about a single vote by a specific voter.

**Syntax**

```bash
xplad query gov vote <proposal-id> <voter-address>
```

**Example**

```bash
xplad query gov vote 7 xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb
```

## `query gov votes`

Retrieves all the votes submitted to the proposal.

**Syntax**

```bash
xplad query gov votes <proposal-id>
```

**Example**

```bash
xplad query gov votes 9
```

## `query gov tally`

Retrieves the current tally for a specified proposal.

**Syntax**

```bash
xplad query gov tally <proposal-id>
```

**Example**

```bash
xplad query gov tally 4
```

## `query gov param`

Retrieves all the parameters for the specified governance process.

**Syntax**

```bash
xplad query gov param <process-type>
```

**Example**

```bash
xplad query gov param voting
```

## `query gov params`

Retrieves all the parameters for all governance processes.

**Syntax**

```bash
xplad query gov params
```

The parameters are returned in the following format:

```yaml
voting_params:
  voting_period: 0m0s
tally_params:
  quorum: "0.000000000000000000"
  threshold: "0.000000000000000000"
  veto: "0.000000000000000000"
deposit_parmas:
  min_deposit:
    - denom: axpla
      amount: "00000000"
  max_deposit_period: 0h0m0s
```

## `query mint annual-provisions`

Retrieves the value of annual provisions.

**Syntax**

```sh
xplad query mint annual-provisions
```

## `query mint inflation`

Retrieves the current value of inflation.

**Syntax**

```sh
xplad query mint inflation
```

## `query mint params`

Retrieves the mint module's parameters.

**Syntax**

```sh
xplad query mint params
```

Parameters are returned in the following format:

```yaml
mint_denom: axpla
inflation_rate_change: "0.000000000000000000"
inflation_max: "0.000000000000000000"
inflation_min: "0.000000000000000000"
goal_bonded: "0.670000000000000000"
blocks_per_year: 6311520
```

## `query slashing signing-info`

Retrieves a validator's signing info.

**Syntax**

```bash
xplad query slashing signing-info <validator-consensus-public-key>
```

**Example**

```bash
xplad query slashing signing-info xplavalconspub1atjdueldlxwft8d4729pqhdhm3nlss0u4wx7wpeqb1zhjf8yr1tn7cgw2b4q4yv9na
```

## `query slashing signing-infos`

Retrieves signing information of all validators.

**Syntax**

```bash
xplad query slashing signing-infos
```

## `query slashing params`

Retrieves the genesis parameters for the slashing module.

**Syntax**

```bash
xplad query slashing params
```

The parameters are returned in the following format:

```yaml
signed_blocks_window: 100
min_signed_per_window: "0.000000000000000000"
downtime_jail_duration: 10m0s
slash_fraction_double_sign: "0.000000000000000000"
slash_fraction_downtime: "0.000000000000000000"
```

## `query staking delegation`

Retrieves delegation information for a validator.

**Syntax**

```bash
xplad query staking delegation <delegator-address> <validator-address>
```

**Example**

```bash
xplad query staking delegation xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p xplavaloper15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

## `query staking delegations`

Retrieves delegations for a delegator on all validators.

**Syntax**

```bash
xplad query staking delegations <delegator-address>
```

**Example**

```bash
xplad query staking delegations xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p
```

## `query staking delegations-to`

Retrieves all of the delegations on a particular validator.

**Syntax**

```bash
xplad query staking delegations-to <validator-address>
```

**Example**

```bash
xplad query staking delegations-to xplavaloper15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

## `query staking historical-info`

Retrieves all historical information at a specified height.

**Syntax**

```bash
xplad query staking historical-info <height>
```

**Example**

```bash
xplad query staking historical-info 23
```

## `query staking params`

Retrieves all staking parameters.

**Syntax**

```bash
xplad query staking params
```

The parameters are returned in the following format:

```yaml
unbonding_time: 504h0m0s
max_validators: 100
max_entries: 100
historical_entries: 0
bond_denom: axpla
```

## `query staking pool`

Retrieves amounts stored in the staking pool.

**Syntax**

```bash
xplad query staking pool
```

The following information is returned:

- Not-bonded and bonded tokens
- Token supply
- Current annual inflation and the block in which the last inflation was processed
- Last recorded bonded shares

## `query staking redelegation`

Retrieves redelegation information for an individual delegator between a source validator and a destination validator.

**Syntax**

```bash
xplad query staking redelegation <delegator-address> <src-val-addr> <dst-val-addr>
```

**Example**

```bash
xplad query staking redelegation xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p xplavaloper1l2rsakp388kuv9k8qzq6lrm9taddae7fpx59wm xplavaloper1gghjut3ccd8ay0zduzj64hwre2fxs9ldmqhffj
```

## `query staking redelegations`

Retrieves all redelegation information for a delegator.

**Syntax**

```bash
xplad query staking redelegations <delegator-address>
```

**Example**

```bash
xplad query staking redelegations xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p
```

## `query staking redelegations-from`

Retrieves all the delegations that are redelegating from a specified validator:

**Syntax**

```bash
xplad query staking redelegations-from <validator-address>
```

**Example**

```bash
xplad query staking redelegations-from xplavaloper1gghjut3ccd8ay0zduzj64hwre2fxs9ldmqhffj
```

## `query staking unbonding-delegation`

Retrieves information about unbonding delegations for a specified delegator and validator.

**Syntax**

```bash
xplad query staking unbonding-delegation <delegator-address> <validator-address>
```

**Example**

```bash
xplad query staking unbonding-delegation xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p xplavaloper1l2rsakp388kuv9k8qzq6lrm9taddae7fpx59wm
```

## `query staking unbonding-delegations`

Retrieves all your current unbonding delegations for a specified delegator.

**Syntax**

```bash
xplad query staking unbonding-delegations <delegator-address>
```

**Example**

```bash
xplad query staking unbonding-delegations xpla1gghjut3ccd8ay0zduzj64hwre2fxs9ld75ru9p
```

## `query staking unbonding-delegations-from`

Retrieves all the unbonding delegations from a specified validator.

**Syntax**

```bash
xplad query staking unbonding-delegations-from <validator-address>
```

**Example**

```bash
xplad query staking unbonding-delegations-from xplavaloper1l2rsakp388kuv9k8qzq6lrm9taddae7fpx59wm
```

## `query staking validators`

Retrieves the list of all registered validators.

**Syntax**

```bash
xplad query staking validators
```

To retrieve the information of a single validator, append the validator address to the end of the command statement, as shown in the following example:

```bash
xplad query staking validator xplavaloper15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

## `query tx`

Retrieves a transaction by its hash, account sequence, or signature.

**Syntax to query by hash**

```bash
xplad query tx <hash>
```

**Syntax to query by account sequence**

```bash
xplad query tx --type=acc_seq <address>:<sequence>
```

**Syntax to query by signature**

```bash
xplad query tx --type=signature <sig1_base64,sig2_base64...>
```

## `query txs`

Retrieves transactions that match the specified events where results are paginated.

**Syntax**

```bash
xplad query txs --events '<event>' --page <page-number> --limit <number-of-results>
```

**Example**

```bash
xplad query txs --events 'message.sender=cosmos1...&message.action=withdraw_delegator_reward' --page 1 --limit 30
```

## `query wasm bytecode`

Retrieves the contract's WASM bytecode by referencing its ID.

**Syntax**

```sh
xplad query wasm bytecode <code-id>
```

## `query wasm code`

Retrieves information about a piece of uploaded code by referencing its ID.

**Syntax**

```sh
xplad query wasm code <code-id>
```

## `query wasm contract`

Retrieves the metadata information about an instantiated contract.

**Syntax**

```sh
xplad query wasm contract <contract-address>
```

## `query wasm contract-store`

Retrieves data about the contract store of the address and prints the results.

**Syntax**

```sh
xplad query wasm contract-store <contract-address> <query-msg>
```

where `<query-msg>` is a JSON string that encodes the QueryMsg.

**Example**

```sh
xplad query wasm contract-store xpla1plju286nnfj3z54wgcggd4enwaa9fgf5kgrgzl '{"config":{}}'
```

## `query wasm params`

Retrieves the current WASM module's parameters.

**Syntax**

```sh
xplad query wasm params
```

The parameters are returned in the following format:

```yaml
max_contract_size: 512000
max_contract_gas: 100000000
max_contract_msg_size: 1024
```

## `query wasm raw-store`

Retrieves the raw store of a contract and prints the results.

**Syntax**

```sh
xplad query wasm raw-store <contract-address> <key> <subkey>
```

If the data uses a `Singleton`, it has only a key. If the data uses a prefixed data store, such as `PrefixedStorage` or `Bucket`, it can accept a subkey too.

## `tx authz exec`

Runs a transaction for the granter.

**Syntax**

```bash
xplad tx authz exec <msg-tx-json-filename> --from=<grantee-address>
```

**Example**

```bash
xplad tx authz exec tx.json --from=<xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm
```

## `tx authz grant`

Authorizes another address to run transactions for you.

**Syntax**

```bash
xplad tx authz grant <grantee-address> <authorization-type> --from=<your-address>
```

**Example**

```bash
xplad tx authz grant xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm send /cosmos.bank.v1beta1.MsgSend --from=xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

Additionally, you can restrict this authorization to a specified allowance by including the `--spend-limit` flag, as shown in the following example:

```bash
xplad tx authz grant xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm send /cosmos.bank.v1beta1.MsgSend --spend-limit=15000axpla --from=xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc
```

## `tx authz revoke`

Removes authorization from an account.

**Syntax**

```bash
xplad tx authz revoke <grantee-address> <authorization-type> --from=<granter-address>
```

**Example**
xplad tx authz revoke xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm /cosmos.bank.v1beta1.MsgSend --from=xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc

## `tx bank send`

Sends coins from one account to another account.

**Syntax**

```bash
xplad tx bank send \
    <from-key-or-address> \
    <to-address> \
    <coins> \
    --chain-id=<chain-id> \
```

where

- `<from-key-or-address>` is either the key name or the address. If the `--generate-only` flag is used, only addresses are accepted.
- `to-address` is an account address.
- `<coins>` is a comma-separated list of coins specified as `<amount><coin-denominator>`. For example, `1000axpla` or `1000axpla,1000<other coin denom>`.

**Example**

```bash
xplad tx bank send \
    xpla15h6vd5f0wqps26zjlwrc6chah08ryu4hzzdwhc \
    xpla14h2od5f3vahd28uywwvt8sqbi52upnzagshtrm \
    8600axpla \
    --chain-id=testnet \
```

## `tx distribution fund-community-pool`

Funds the community pool with the specified amount.

**Syntax**

```bash
xplad tx distribution fund-community-pool <amount>
```

**Example**
xplad tx distribution fund-community-pool 100axpla

## `tx distribution set-withdraw-addr`

Changes the default withdrawal address for rewards associated with an address.

**Syntax**

```bash
xplad tx distribution set-withdraw-addr <withdrawal-address>
```

**Example**

```bash
xplad tx distribution set-withdraw-addr xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb
```

## `tx distribution withdraw-all-rewards`

Withdraws all rewards.

**Syntax**

```bash
xplad tx distribution withdraw-all-rewards
```

## `tx distribution withdraw-rewards`

Withdraws your rewards against a validator.

**Syntax**

```bash
xplad tx distribution withdraw-rewards <validator-address>
```

**Example**

```bash
xplad tx distribution withdraw-rewards xpla19t4gde4f8ndwx67qhbnur9yqdc31xznpksajbcy
```

## `tx gov deposit`

For a proposal to be sent to the network, the amount deposited must be above a minimum amount specified by `minDeposit` (genesis value is `512000000axpla`). If the proposal you previously created didn't meet this requirement, you can still increase the total amount deposited to activate it. After the minimum deposit is reached, the voting period for the proposal begins.

**Syntax**

```bash
xplad tx gov deposit <proposal-id> "<deposit-amount>" \
    --from=<name> \
    --chain-id=<chain-id>
```

**Example**

```bash
xplad tx gov deposit 15 "10000000laxpla" \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

{{< hint warning >}}
**Warning**
Proposals that don't meet this requirement are deleted after `MaxDepositPeriod` is reached.
{{< /hint >}}

## `tx gov submit-proposal`

Submits proposals and deposits. To create a governance proposal, you must submit an initial deposit along with a title and description of your proposal. Alternatively, you can provide the proposal directly through the `--proposal` flag, which points to a JSON file containing the proposal.

### Text Proposals

**Syntax**

```bash
xplad tx gov submit-proposal \
    --title=<title> \
    --description=<description> \
    --type="Text" \
    --deposit="<amount>" \
    --from=<name-or-address> \
    --chain-id=<chain-id>
```

**Example**

```bash
xplad tx gov submit-proposal \
    --title=Funding for NFT platform \
    --description=Information about the NFT platform \
    --type="Text" \
    --deposit="100000000000000000axpla" \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

### Parameter-change Proposals

When you submit a proposal to change a parameter, it is recommended that you send the proposal as a JSON file.

**Syntax**

```bash
xplad tx gov submit-proposal \
    param-change <path/to/proposal.json> \
    --from=<name> \
    --chain-id=<chain_id>
```

**Example**

```bash
xplad tx gov submit-proposal \
    param-change /proposals/proposal.json \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

where `proposal.json` contains the following information:

```json
{
  "title": "Param Change",
  "description": "Update max validators",
  "changes": [
    {
      "subspace": "staking",
      "key": "MaxValidators",
      "value": 105
    }
  ],
  "deposit": [
    {
      "denom": "axpla",
      "amount": "10000000"
    }
  ]
}
```

{{< hint warning >}}
**Warning**
Because parameter changes are evaluated but not validated, ensure that new value you propose is valid for its parameter. For example, the proposed value for `MaxValidators` must be an integer, not a decimal.
{{< /hint >}}

### Community Pool Spend Proposal

When you submit a community pool spend proposal, it is recommended that you send the proposal as a JSON file.

**Syntax**

```bash
xplad tx gov submit-proposal \
    community-pool-spend <path/to/proposal.json> \
    --from=<name> \
    --chain-id=<chain_id>
```

**Example**

```bash
xplad tx gov submit-proposal \
    community-pool-spend /proposals/proposal.json \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

where `proposal.json` contains the following information:

```json
{
  "title": "Community Pool Spend",
  "description": "Pay me some XPLAs!",
  "recipient": "xpla1s5afhd6gxevu37mkqcvvsj8qeylhn0rzn7cdaq",
  "amount": [
    {
      "denom": "axpla",
      "amount": "10000"
    }
  ],
  "deposit": [
    {
      "denom": "axpla",
      "amount": "10000"
    }
  ]
}
```

### Software Upgrade Proposals

The proposal to upgrade the software follows the syntax of a `text` proposal.

**Syntax**

```bash
xplad tx gov submit-proposal software-upgrade <name> \
    --title=<title> \
    --description=<description> \
    --upgrade-height=<block-height> \
    --upgrade-info=<binary-details> \
    --type="Text" \
    --deposit="<amount>" \
    --from=<name-or-address> \
    --chain-id=<chain_id>
```

**Example**

```bash
xplad tx gov submit-proposal software-upgrade v0.5.0-beta3 \
    --title="Upgrade to v0.6.0-beta3" \
    --description="let's upgrade to v0.6.0-beta3" \
    --upgrade-height=20 \
    --upgrade-info='{"binaries":{"darwin/amd64":"/Workspace/xpla/core/build/xplad?checksum=sha256:2032356fe0899dec0cdd559f1c649bc81e53a9b4063b333059135e3a2aae8728"}}' \
    --type="Text" \
    --deposit="50000000000000000000axpla" \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

To cancel a software upgrade:

**Syntax**

```bash
xplad tx gov submit-proposal cancel-software-upgrade --title "<title>" --description "<description>"
```

**Example**

```bash
xplad tx gov submit-proposal cancel-software-upgrade --title "Upgrade to v0.5.0-beta3" --description "let's upgrade to v0.6.0-beta3"
```

## `tx gov vote`

After a proposal's deposit reaches the `MinDeposit` value, the voting period begins, and bonded XPLA holders can vote.

**Syntax**

```bash
xplad tx gov vote \
    <proposal-id> <vote-type> \
    --from=<name> \
    --chain-id=<chain_id>
```

**Example**

```bash
xplad tx gov vote \
    7 yes \
    --from=xpla13a8ddv3h7kbcn73akcbpe7ueks22vaolewpaxmb \
    --chain-id=dimension_37-1
```

## `tx slashing unjail`

Releases a jailed validator.

**Syntax**

```bash
xplad tx slashing unjail
```

**Example**

```bash
xplad tx slashing unjail
```

## `tx staking create-validator`

Creates a new validator that is initialized with a self-delegation.

**Syntax**

```bash
xplad tx staking create-validator \
    --amount=<axpla-amount> \
    --pubkey=$(xplad tendermint show-validator) \
    --moniker="<moniker>" \
    --website="<validator-website>" \
    --identity="<keybase-identity>" \
    --details="<validator-optional-details" \
    --commission-rate="<commission-rate>" \
    --commission-max-rate="<commission-max-rate>" \
    --commission-max-change-rate="<commission-max-change-rate>" \
    --min-self-delegation="<self-delegation-amount>"
    --chain-id=<chain-ID> \
    --from=<key-name> \
```

## `tx staking delegate`

Delegates an amount of liquid coins from your wallet to a validator.

**Syntax**

```bash
xplad tx staking delegate <validator-address> <amount>
```

**Example**

```bash
xplad tx staking delegate xplavaloper1l2rsakp388kuv9k8qzq6lrm9taddae7fpx59wm 2500stake
```

## `tx staking edit-validator`

Edits an existing validator account.

**Syntax**

```bash
xplad tx staking edit-validator \
    --moniker="<moniker>" \
    --details="<validator-optional-details>" \
    --commission-rate="commission-rate>" \
    --min-self-delegation="<minimum-self-delegation-amount" \
```

## `tx staking redelegate`

Redelegates an amount of illiquid staking tokens from one validator to a different validator.

**Syntax**

```bash
xplad tx staking redelegate <from-validator-address> <to-validator-address> <amount>
```

**Example**

```bash
xplad tx staking redelegate xplavaloper1gghjut3ccd8ay0zduzj64hwre2fxs9ldmqhffj xplavaloper1l2rsakp388kuv9k8qzq6lrm9taddae7fpx59wm 350stake
```

## `tx staking unbond`

Unbonds an amount of bonded shares from a validator.

**Syntax**

```bash
xplad tx staking unbond <validator-address> <stake-amount>
```

**Example**

```bash
xplad tx staking unbond xplavaloper1gghjut3ccd8ay0zduzj64hwre2fxs9ldmqhffj 600stake
```

## `tx wasm clear-admin`

Removes the contract admin so that the contract cannot be migrated.

**Syntax**

```bash
xplad tx wasm clear-admin <contract-address>
```

## `tx wasm execute`

Invokes processing functions on the smart contract.

**Syntax**

```sh
xplad tx wasm execute <contract-address> <handle-msg> <coins>
```

Where `<handle-msg>` is a raw JSON string containing the `HandleMsg` that is parsed and routed to the correct message handling logic in the contract, and `<coins>` is the optional amount of coins specified in a comma-separated list that you want to send with your message, in case the logic requires some fees.

## `tx wasm instantiate`

Creates a new contract by referencing the code ID of a contract that has been uploaded.

**Syntax**

```sh
xplad tx wasm instantiate <code-id> <init-msg> <coins>
```

where `<init-msg>` is a JSON string containing the `InitMsg` to initialize your contract's state, and `<coins>` is the optional amount of coins specified in a comma-separated list that you want to send to the new contract account.

**Example**

```sh
xplad tx wasm instantiate 1 '{"arbiter": "xpla~~"}' "1000000axpla"
```

## `tx wasm migrate`

Updates the code ID of the contract to migrate to a new code ID. This command can be issued only from the key corresponding to the contract's owner.

**Syntax**

```sh
xplad tx wasm migrate <contract-address> <new-code-id> <migrate-msg>
```

**Example**

```sh
xplad tx wasm migrate xpla... 10 '{"verifier": "xpla..."}'
```

## `tx wasm store`

Uploads a new WASM binary or migrates to existing binary to the dimension_37-1 release.

**Syntax for a new WASM binary**

```sh
xplad tx wasm store <path-to-wasm-file>
```

where `<path-to-wasm-file>` is the path of a file that is the compiled binary of the smart contract code that you want to upload.

## `tx wasm update-admin`

Updates a contract owner to a new address. This command can be issued only from the key corresponding to the contract's owner.

**Syntax**

```sh
xplad tx wasm update-admin <contract-address> <new-owner>
```
