---
title: Restore A Validator
weight: 70
type: docs
---

A validator can be completely restored on a new XPLA Chain node with the following set of keys:

- The Consensus key, stored in `~/.xpla/config/priv_validator.json`
- The mnemonic to the validator wallet

{{< alert context="danger" >}}
**Danger**

Before proceeding, ensure that the existing validator is not active. Double voting has severe slashing consequences.
{{< /alert >}}

To Restore a Validator:

1. Setup a full XPLA Chain node synced up to the latest block.
2. Replace the `~/.xpla/config/priv_validator.json` file of the new node with the associated file from the old node, then restart your node.
