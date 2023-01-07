---
title: Multisig Wallets
weight: 40
---

Multisig wallets are an advanced feature of XPLA Vault. If you’re using XPLA Vault for the first time, follow the [XPLA Vault tutorial]({{< ref "xpla-vault" >}}).

Multisig wallets enable a wallet to be controlled by multiple parties. A wallet manager creates a transaction and sends an encoded transaction message to the wallet signers. The signers sign the transaction and send back their signatures. The wallet manager then inputs the encoded transaction message along with the received signatures to complete the transaction.

## Prerequisites

- Download the [XPLA extension vault]({{< ref "vaults/extension-vault" >}})

## Create a Multisig Wallet

1. Open the XPLA extension vault and click **New multisig wallet**.

2. Enter the wallet addresses of each multisig user in the correct order.

   {{< alert context="warning" >}}
   **Warning**

   Each wallet added must have a transaction history of at least one transaction before it can be added to a multisig wallet.
   {{< /alert >}}

3. Enter the number of signatures needed to post a transaction in the **Threshold** field.

4. Click **Submit**.

5. Enter a wallet name and click **Submit**.

## Create a Multisig Transaction

Multisig wallet managers initiate transactions and send coded strings for multisig participants to sign.

1. Open the XPLA extension vault and connect to your multisig wallet.

2. Make a transaction using your multisig wallet.

   {{< alert >}}
   **Note**

   Brand new multisig wallets have no transaction histories. After your first transaction, you will be prompted to provide the wallet addresses again for the multisig wallet. Provide the wallet addresses used to create the wallet to proceed with your transaction.
   {{< /alert >}}

3. After submitting your transaction, you will be taken to the **Post a multisig tx** page.

4. Copy the multisig wallet address and the encoded string in the **Tx** box and send both to each of the multisig wallet signers.

   {{< alert >}}
   **Sending multisig strings**

   Encoded multisig transaction strings can be sent using a regular messenger, as they are not sensitive information. They contain a simple description of the transaction.
   {{< /alert >}}

## Sign a Multisig Transaction

Upon receiving an encoded transaction string, multisig signers can use the following steps to sign:

1. Open the XPLA extension vault and connect to the wallet you provided to create the multisig wallet.

2. Click the three vertical dots located to the right of your wallet name.

3. Click **Sign a multisig tx**.

4. Enter the multisig wallet address and the encoded multisig transaction string you received from the multisig wallet manager.

5. Enter your password and click **Submit**.

6. Copy the signature string provided and send it to the multisig wallet manager.

## Post a Multisig Transaction

The final step in a multisig transaction is for the wallet manager to input the signatures:

1. Open the XPLA extension vault and connect to the multisig wallet you created.

2. Click the three vertical dots located to the right of your wallet name.

3. Click **Post a multisig tx**.

4. If this is your first transaction for the wallet, enter the wallet addresses used to create the multisig wallet. Otherwise, proceed to step 5.

5. Enter the multisig wallet address, the initial encoded transaction string, and the signatures sent to you by the multisig signers.

6. Click **Submit**. Your transaction should begin processing.

Congratulations, you just completed a multisig transaction!
