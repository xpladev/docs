---
title: Join a Network
weight: 50
---

It is highly recommended that you set up a private local network before joining a public network. This will help you get familiar with the setup process, and provide an environment for testing. The following sections outline this process. If you want to join a public network without setting up a private network, you can skip to [join a public network ]({{< ref "#join-a-public-network" >}}).

### Create a Single Node

The simplest XPLA Chain you can set up is a local testnet with just a single node. In a single-node environment, you have one account and are the only validator signing blocks for your private network.

1. Initialize your genesis file that will bootstrap the network. Replace the following variables with your own information:

   ```bash
     xplad init --chain-id=<testnet-name> <node-moniker>
   ```

1. Generate a XPLA Chain account. Replace the variable with your account name:

   ```bash
   xplad keys add <account-name>
   ```

{{< hint info >}}
**Get tokens**
In order for xplad to recognize a wallet address, it must contain tokens. For the testnet, use [the faucet](https://faucet.xpla.io/) to send XPLA to your wallet. If you are on mainnet, send funds from an existing wallet. 1-3 XPLA are sufficient for most setup processes.
{{< /hint >}}

### Add Your Account to the Genesis

Run the following commands to add your account and set the initial balance:

```bash
xplad add-genesis-account $(xplad keys show <account-name> -a) 100000000000000000000axpla
xplad gentx <my-account> 10000000000000000000axpla --chain-id=<testnet-name>
xplad collect-gentxs
```

### Start Your Private XPLA Chain

Run the following command to start your private network:

```bash
xplad start
```

If the private XPLA Chain is set up correctly, your `xplad` node will be running on `tcp://localhost:26656`, listening for incoming transactions, and signing blocks.

## Join a public network

These instructions are for setting up a brand new full node from scratch. You can join a public XPLA Chain, such as the mainnet or testnet, by completing the following steps:

### 1. Select a network

Specify the network you want to join by choosing the corresponding **genesis file** and **seeds**:

| Network          | Type    | Genesis                                                                                            | Seed or Known Peers                                                     |
|:-----------------|:--------|:---------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------|
| `dimension_37-1` | Mainnet | [Genesis Link](https://raw.githubusercontent.com/xpladev/mainnet/main/dimension_37-1/genesis.json) | e7b6016ce5663a69ba71a982072315545eb0d5f6@seed.xpla.delightlabs.io:26656 |
| `cube_47-5`      | Testnet | [Genesis Link](https://raw.githubusercontent.com/xpladev/testnets/main/cube_47-5/genesis.json)     | 9ddfac28dc6b28601e3039902ee5a8915dc7891f@3.35.54.221:26656              |

{{< hint info >}}
**Selecting a network**
Note that the versions of the network listed above are the latest versions. To find earlier versions, please consult the [testnets repo](https://github.com/xpladev/testnets) or [mainnet repo](https://github.com/xpladev/mainnet).
{{< /hint >}}

### 2. Download genesis file and address book

**Genesis-transaction** specifies the account balances and parameters at the start of the network to use when replaying transactions and syncing.

**Addressbook** lists a selection of peers for your node to dial to in order to discover other nodes in the network. Public address books of peers are made available by the XPLA Chain community.

Choose a `testnet` or `mainnet` address type and download the appropriate genesis-transaction and addressbook. Links to these are posted in [Select-a-network]({{< ref "#1-select-a-network" >}}).

- For default `xplad` configurations, the `genesis` and `addressbook` files should be placed under `~/.xpla/config/genesis.json` and `~/.xpla/config/addrbook.json` respectively.

### 3. `xplad start`

Start the network and check that everything is running smoothly:

```bash
xplad start
xplad status
# It will take a few seconds for xplad to start.
```

{{< expand "Healthy Node Status Example" >}}
```json
{
  "NodeInfo": {
    "protocol_version": {
      "p2p": "8",
      "block": "11",
      "app": "0"
    },
    "id": "821dc1401fd0270487b3e615c652181b4d4566dd",
    "listen_addr": "18.157.84.154:26656",
    "network": "cube_47-5",
    "version": "v0.34.19-xpla.2",
    "channels": "40202122233038606100",
    "moniker": "xpladocs",
    "other": {
      "tx_index": "on",
      "rpc_address": "tcp://127.0.0.1:26657"
    }
  },
  "SyncInfo": {
    "latest_block_hash": "ED9F6D0855FD92A5BA2F91082CD49ADB18A07DCE3F747529D357071E5B7C0D4C",
    "latest_app_hash": "D621068882E7FC5045CDD957ADEABE9BF8E90F2092C9526E22BE4767940D128B",
    "latest_block_height": "260770",
    "latest_block_time": "2022-06-09T15:22:48.792283245Z",
    "earliest_block_hash": "F948EF10AA663D182309790C51E5A7A9125D7CF4D60D9E735994059DB7CAD4D4",
    "earliest_app_hash": "E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855",
    "earliest_block_height": "1",
    "earliest_block_time": "2022-05-23T06:00:00Z",
    "catching_up": false
  },
  "ValidatorInfo": {
    "Address": "04024A7F76485B0D2B99570EC7DA3E9A3B3735CC",
    "PubKey": {
      "type": "tendermint/PubKeyEd25519",
      "value": "KUqIsRD9yzPt7k9et+ClFp6h8wXwEIcb/TVZPrC57+I="
    },
    "VotingPower": "0"
  }
}
```
{{< /expand >}}

Your node is now syncing. This process will take a long time. Make sure you've set it up on a stable connection so this process is not interrupted.

{{< hint warning >}}
**Sync start times**
Nodes take at least an hour to start syncing. This wait time is normal. Before troubleshooting a sync, please wait an hour for the sync to start.
{{< /hint >}}

Continue to the [Sync]({{< ref "sync" >}}) page to find out more about syncing your node.
