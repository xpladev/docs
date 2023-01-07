---
title: Configure General Settings
weight: 40
---

The following information describes the most important node configuration settings found in the `~/.xpla/config/` directory. It is recommended that you update these settings with your own information.

{{< expand "Structure of .xpla/config" >}}
```bash
~/.xpla/config
│-- addrbook.json                       # a registry of peers to connect to
│-- app.toml                            # xplad configuration file
│-- client.toml                         # configurations for the cli wallet (ex xplacli)
│-- config.toml                         # Tendermint configuration  file
│-- genesis.json                        # genesis transactions
│-- node_key.json                       # private key used for node authentication in the p2p protocol (its corresponding public key is the nodeid)
└-- priv_validator_key.json             # key used by the validator on the node to sign blocks
```
{{< /expand >}}

## Initialize and Configure Moniker

Initialize the node with a human-readable name:

```bash
xplad init <your_custom_moniker> # ex., xplad init validator-joes-node
```

{{< hint warning >}}
**Moniker characters**
Monikers can only contain ASCII characters; using Unicode characters will render your node unreachable by other peers in the network.
{{< /hint >}}

You can update your node's moniker by editing the `moniker` field in `~/.xpla/config/config.toml`

## Update Minimum Gas Prices

1. Open `~/.xpla/config/app.toml`.

2. Modify `minimum-gas-prices` and set the minimum price of gas a validator will accept to validate a transaction and to prevent spam.

Recommended setting is:
`minimum-gas-prices = "850000000000axpla"`

**Example**:

````toml
# The minimum gas prices a validator is willing to accept for processing a
# transaction. A transaction's fees must meet the minimum of any denomination
# specified in this config (e.g. 0.25token1;0.0001token2).
minimum-gas-prices = "850000000000axpla"
````

## Start the Light Client Daemon (LCD)

For information about the available XPLA Chain REST API endpoints, see the [Swagger documentation](https://cube-lcd.xpla.dev/swagger/). To enable the REST API and Swagger, and to start the LCD, complete the following steps:

1. Open `~/.xpla/config/app.toml`.

2. Locate the `API Configuration` section (`[api]`).

3. Change `enable = false` to `enable = true`.

   ```toml
   # Enable defines if the API server should be enabled.
   enable = true

4. Optional: Swagger defines if swagger documentation should automatically be registered. To enable Swagger, change `swagger = false` to `swagger = true`.

   ```toml
   swagger = true
   ```

5. Restart the service via `systemctl restart xplad`. Once restarted, the LCD will be available (by default on port `127.0.0.1:1317`)

## Set-up `external_address` in `config.toml`

In order to be added to the address book in seed nodes, you need to configure `external_address` in `config.toml`. This addition will prevent continuous reconnections. The default P2P_PORT is `26656`.

```sh
sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':26656\"/g' ~/.xpla/config/config.toml
```
