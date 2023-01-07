---
title: Manage Staking
weight: 70
---

Use this guide to manage your staking delegations in XPLA Vault. To learn how to stake or withdraw rewards, visit the [XPLA mobile vault tutorials]({{< ref "vaults/mobile-vault" >}}).

If this is your first time using XPLA Vault, follow the [XPLA Vault tutorial]({{< ref "web-vault" >}}).

## Stake XPLA

Stake your XPLA to a validator to start earning rewards. Before you stake, make sure you have XPLA in your wallet. You can transfer XPLA from an [exchange]({{< ref "wallet" >}}).

1. Open XPLA Vault and click **Stake**.

2. Select a Validator and click on their name in the **Moniker** column of the validator list.

3. In the **My delegations** section, click **Delegate**. A new window will appear.

4. In the **Amount** field, specify the amount of XPLA you want to delegate.

   {{< hint warning >}}
   **Keep coins for fees**
   Always keep some coins to pay fees with. Never stake your entire wallet amount. Without money for fees, you can't make any transactions.
   {{< /hint >}}

5. Double check the amounts and fees. Enter your password and click **Submit**.

Congratulations, you've just delegated XPLA!

## Withdraw Staking Rewards

Rewards start accruing the moment you stake XPLA. Monitor your rewards in the staking section of XPLA Vault. Once you have sufficient rewards, follow these steps to withdraw them:

1. Open XPLA Vault and click **Stake**.

2. To claim all rewards, click **Withdraw all rewards** in the upper right corner of the staking page. To withdraw rewards only from a single validator, click on their name in the list and click **Withdraw rewards** on their page.  A new window will appear.

3. Review the amounts and click **Submit**.

Congratulations, you've just withdrawn your staking rewards!

## Redelegate

Redelegating lets you transfer staked XPLA from one validator to another without waiting the 21-day unstaking period. Redelegating happens instantly.

{{< hint warning >}}
**Warning**
When a user redelegates staked XPLA from one validator to another, the validator receiving the staked XPLA is barred from making further redelegation transactions for 21 days. This requirement only applies to the wallet that made the redelegation transaction.
{{< /hint >}}

1. Open XPLA Vault and connect your wallet. Click **Stake**.

2. Click on the validator you want to redelegate to.

3. Under the **My delegations** section, click **Redelegate**.

4. Select the validator you would like to redelegate from.

5. Enter the amount of XPLA you want to redelegate.

6. Confirm the amounts and click **Submit**.

Your staked XPLA will be transferred to the new validator.

## Undelegate

Undelegate XPLA to unstake it from a validator. The unstaking period takes 21 days to complete.

{{< hint warning >}}
**Warning**
Once started, the delegating or undelegating processes can't be stopped.
Undelegating takes 21 days to complete. The only way to undo a delegating or undelegating transaction is to wait for the unbonding process to pass. Alternatively, you can redelegate staked XPLA to a different validator without waiting 21 days.
{{< /hint >}}

1. Open XPLA Vault and connect your wallet. Click **Stake**.

2. Click on the validator you want to unstake from.

3. Click **Undelegate** under the **My delegations** section.

4. Enter the amount of XPLA you want to undelegate. Click **Submit**.

Your staked XPLA is unbonding. Please check back in 21 days to complete the process.
