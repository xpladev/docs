---
title: Wallet
weight: 90
---

This guide is for advanced features of the XPLA Vault. If this is your first time using XPLA Vault, follow the [XPLA Vault tutorial]({{< ref "web-vault" >}}).

For information on using multisig wallets in XPLA Vault, see [Multisig wallets]({{< ref "multisig-wallets" >}})

## Create a wallet

1. Open the XPLA web vault and click **New wallet**.

1. Type in a secure wallet name and password.

1. Confirm your password.

1. Using a pen and paper, write down your 24-word mnemonic exactly as it appears. Number each word to make verifying easier.

   {{< alert context="danger" >}}
   **Danger**

   Anyone with your mnemonic phrase can access your money, and there is no recourse for someone stealing your mnemonic phrase. To protect your mnemonic phrase, consider the following tips:
   - Never save or store your mnemonic phrase as a digital file on any device.
   - Always write down your mnemonic phrase with a pen and paper.
   - Store the paper with your mnemonic phrase on it somewhere safe.
   - Never give your mnemonic phrase to anyone, not even support staff.
   {{< /alert >}}

1. Verify your writing to make sure every word is spelled correctly and in the right order. If you numbered your phrase, it can be helpful to verify it backward.

1. Check the box ensuring you wrote down your mnemonic phrase, and click **Next**.

1. Confirm your mnemonic phrase by typing or selecting the correct words in each prompt.

1. Click **Create a wallet**.

Congratulations! You have just created a XPLA Vault.

## Select a Wallet

Follow these steps to connect to a wallet previously accessed on your device.

1. Open XPLA Vault and click **Connect**.

1. Click **Select wallet** and select the wallet you want to connect to.

1. Enter the password of the wallet and click **Next**.

XPLA Vault is now connected to your selected wallet. To change wallets, [disconnect your wallet](#disconnect-a-wallet) and follow these steps again.

## Connect to a Wallet Using a Private Key

Use a private key to access your wallet from other devices. Unlike recovering your wallet using a mnemonic phrase, private keys allow you to keep your wallet name and password. Follow these steps to connect to an existing wallet using a private key. You will need access to your existing wallet.

### Export Your Private Key

1. Open XPLA Vault and connect to your wallet.

1. Locate your wallet address on XPLA Vault. Click the gear icon next to your wallet address.

1. Click **Export private key**.

1. Enter your password and click **Generate key**.

You can now access your private key. Do not share your private key with anyone. Anyone with your private key and password can access your account.

### Import Your Private Key

1. Open XPLA Vault and click **Connect**.

1. Click **Import private key**.

1. Enter your private key and password.

1. Click **Submit**.

Your private key has been imported to the device you are using. You can now use your wallet name and password to access your wallet on your device. Repeat this process for any device you wish to access your wallet.

## Connect to a Wallet Using a QR Code

Use a QR code to access your wallet on a mobile device. Unlike recovering your wallet using a mnemonic phrase, QR codes allow you to keep your wallet name and password. Follow these steps to connect to an existing wallet using a private key QR code. You will need access to your existing wallet.

### Export Your QR Code Using Desktop

1. Open XPLA Vault and connect to your wallet.

1. Locate your wallet address on XPLA Vault. Click the gear icon next to your wallet address.

1. Click **Export with QR code**.

1. Enter your password and click **Generate QR code**.

### Export Your QR Code Using the Mobile App

1. Open the XPLA Vault app and connect to your wallet.

1. Tap the gear icon in the upper right corner of the app.

1. Click **Export wallet with QR code**.

1. Enter your password.

You can now access your private key QR code. Do not share your private key with anyone. Anyone with your private key and password can access your account.

### Import Your QR Code

1. Open the XPLA Vault and tap **Recover wallet**.

1. Tap **Scan QR code**.

1. Scan your QR code using your device's camera and enter your password.

1. Tap **Next**.

Your private key has been imported to the device you are using. You can now use your wallet name and password to access your wallet on your device. Repeat this process for any device you wish to access your wallet.

## Recover a Wallet Using a mnemonic phrase

If you forgot or deleted your login info, you can recover a wallet using your mnemonic phrase. You can also use this method to change your wallet name.

1. Open XPLA Vault and click **Connect**.

1. Click **Recover existing wallet**.

1. Enter a wallet name and password. Confirm your password.

1. Enter your mnemonic phrase and click **Next**.

You can now access your wallet with your login and password.

## Disconnect a Wallet

1. Locate your wallet address on XPLA Vault. Click the gear icon next to your wallet address.

1. Select **disconnect** from the options.

Your wallet is now disconnected.

## Delete a Wallet

Deleting a wallet deletes the wallet name, password, and private key from your device. You can access the wallet again by entering your [mnemonic phrase](#recover-a-wallet-using-a-mnemonic-phrase) or [private key and password](#connect-to-a-wallet-using-a-private-key). Deleting a wallet from one device does not delete it from other devices.

{{< alert context="danger" >}}
**Write down your mnemonic phrase**
Before you delete your wallet, always make sure you have your mnemonic phrase and private key. Never store your mnemonic phrase on a digital device. Without a mnemonic phrase or private key and password, your wallet and funds will be permanently inaccessible. Always store your mnemonic phrase in a secure location.
{{< /alert >}}

1. Open XPLA Vault and connect to your wallet.

1. Locate your wallet address on XPLA Vault. Click the gear icon next to your wallet address.

1. Select **Delete wallet**.

1. Follow the prompt and click **Delete**.

Your wallet is now deleted. You can only access it again by entering your [mnemonic phrase](#recover-a-wallet-using-a-mnemonic-phrase) or [private key and password](#connect-to-a-wallet-using-a-private-key).

## Change Your Password

Follow these steps to change your password. Changing your password only changes your wallet password on a single device. Repeat these steps to change your wallet password on other devices.

1. Open XPLA Vault and connect to your wallet.

1. Locate your wallet address on XPLA Vault. Click the gear icon next to your wallet address.

1. Select **Change password** from the options.

1. Enter your current password and your new password. Confirm your new password.

1. Click **Change password**.

Your wallet password is now changed on your device. Repeat these steps to change your wallet password on other devices.
