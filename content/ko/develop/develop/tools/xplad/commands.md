---
title: Commands
weight: 40
type: docs
---

This section describes the commands available from `xplad`, the command line interface that connects a running `xplad` process.

## `add-genesis-account`

Adds a genesis account to `genesis.json`.

**Syntax**
```bash
xplad add-genesis-account <address-or-key-name> '<amount><coin-denominator>,<amount><coin-denominator>'
```

**Example**
```bash
xplad add-genesis-account acc1 '200000000axpla'
```

## `collect-gentxs`

Collects genesis transactions and outputs them to `genesis.json`.

**Syntax**
```bash
xplad collect-gentxs
```

## `debug`

Helps debug the application. For a list of syntax and subcommands, see the [debug subcommands]({{< ref "subcommands#debug-addr" >}}).

## `export`

Exports the state to JSON.

**Syntax**
```bash
xplad export
```

## `gentx`

Adds a genesis transaction to `genesis.json`.

**Syntax**
```bash
xplad gentx <key-name> <amount><coin-denominator>
```

**Example**
```bash
xplad gentx myKey 1000000axpla --home=/path/to/home/dir --keyring-backend=os --chain-id=test-chain-1 \
    --moniker="myValidator" \
    --commission-max-change-rate=0.01 \
    --commission-max-rate=1.0 \
    --commission-rate=0.07 \
    --details="..." \
    --security-contact="..." \
    --website="..."
```

## `help`

Shows help information.

**Syntax**
```bash
xplad help
```

## `init`

Initializes the configuration files for a validator and a node.

**Syntax**
```bash
xplad init <moniker>
```

**Example**
```bash
xplad init myNode
```

## `keys`

Manages Keyring commands. For a list of syntax and subcommands, see the [keys subcommands]({{< ref "subcommands#keys-add" >}}).


## `migrate`
Migrates the source genesis into the target version and prints to STDOUT.

**Syntax**
```bash
xplad migrate <path-to-genesis-file>
```

**Example**
```bash
xplad migrate /genesis.json --chain-id=testnet --genesis-time=2020-04-19T17:00:00Z --initial-height=4000
```

## `query`

Manages queries. For a list of syntax and subcommands, see the [query subcommands]({{< ref "subcommands#query-authz-grants" >}}).

## `rosetta`

Creates a Rosetta server.

**Syntax**
```bash
xplad rosetta
```

## `start`

Runs the full node application with Tendermint in or out of process. By default, the application runs with Tendermint in process.

**Syntax**
```bash
xplad start
```

## `status`

Displays the status of a remote node.

**Syntax**
```bash
xplad status
```

## `tendermint`

Manages the Tendermint protocol.

## `testnet`

Creates a testnet with the specified number of directories and populates each directory with the necessary files.

**Syntax**
```bash
xplad testnet
```

**Example**
```bash
xplad testnet --v 6 --output-dir ./output --starting-ip-address 192.168.10.2
```

## `tx`

Retrieves a transaction by its hash, account sequence, or signature. For a list of full syntax and subcommands, see the [tx subcommands]({{< ref "subcommands#tx-authz-exec" >}}).

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

## `txs`

Retrieves transactions that match the specified events where results are paginated.

**Syntax**
```bash
xplad query txs --events '<event>' --page <page-number> --limit <number-of-results>
```

**Example**
```bash
xplad query txs --events 'message.sender=cosmos1...&message.action=withdraw_delegator_reward' --page 1 --limit 30
```

## `unsafe-reset-all`

Resets the blockchain database, removes address book files, and resets `data/priv_validator_state.json` to the genesis state.

**Syntax**
```bash
xplad tendermint unsafe-reset-all
```

## `validate-genesis`

Validates the genesis file at the default location or at the location specified.

**Syntax**
```bash
xplad validate-genesis </path-to-file>
```

**Example**
```bash
xplad validate-genesis </genesis.json>
```

## `version`

Returns the version of xplad you're running.

**Syntax**
```bash
xplad version
```
