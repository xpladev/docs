---
title: Reset and Troubleshooting
weight: 80
---

## Complete Reset

Occasionally you may need to perform a complete reset of your node due to data corruption or misconfiguration. Resetting will remove all data in `~/.xpla/data` and the addressbook in `~/.xpla/config/addrbook.json` and reset the node to genesis state.

To perform a complete reset of your `xplad` state, use:

```sh
xplad tendermint unsafe-reset-all
```

Running this command successfully will produce the following log:

```sh
[ INF ] Removed existing address book file=/home/user/.xpla/config/addrbook.json
[ INF ] Removed all blockchain history dir=/home/user/.xpla/data
[ INF ] Reset private validator file to genesis state keyFile=/home/user/.xpla/config/priv_validator_key.json stateFile=/home/user/.xpla/data/priv_validator_state.json
```

{{< hint info >}}
**Note**
After resetting, make sure the addressbook contains peer addresses and is in the correct spot. If not, [download an adressbook](../join-a-network#1-select-a-network) and place it in `~/.xpla/config/`.
{{< /hint >}}

### Change Genesis

To change the genesis version, delete `~/.xpla/config/genesis.json`.

You can recreate a genesis version via the following steps:

```bash
 xplad add-genesis-account $(xplad keys show <account-name> -a) 100000000000000000000axpla
 xplad gentx <account-name> 10000000000000000000axpla --chain-id=<network-name>
 xplad collect-gentxs
```

### Reset Personal Data

{{< hint danger >}}
**Danger**
You may be unable to use your node and its associated accounts after changing your personal data. Do not perform this action unless your node is disposable.
{{< /hint >}}

To change your personal data to a pristine state, delete both `~/.xpla/config/priv_validator_state.json` and `~/.xpla/config/node_key.json`.

### Node health

A healthy node will have the following files in place and populated:

- Addressbook `~/.xpla/config/addrbook.json`
- Genesis file `~/.xpla/config/genesis.json`
- Validator state `~/.xpla/config/priv_validator_state.json`
- Node key `~/.xpla/config/node_key.json`

### Resync

You can proceed to [resync manually]({{< ref "sync" >}}).
