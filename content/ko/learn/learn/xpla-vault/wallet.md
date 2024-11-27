---
title: 지갑
weight: 90
type: docs
---

이 가이드는 XPLA 볼트의 고급 기능에 대한 설명입니다. XPLA 볼트를 처음 사용하시는 경우, [XPLA 볼트 튜토리얼]({{< ref "web-vault" >}})을 따르세요.

XPLA 볼트에서 다중서명 지갑을 사용하는 방법에 대한 자세한 내용은 [다중서명 지갑]({{< ref "multisig-wallets" >}})을 참조하세요.

## 지갑 만들기

1. XPLA 웹 볼트를 열고 **New wallet**을 클릭합니다.

2. 보안 지갑 이름과 비밀번호를 입력합니다.

3. 비밀번호를 확인합니다.

4. 펜과 종이를 사용해 24단어 니모닉을 보이는 대로 정확하게 적습니다. 각 단어에 번호를 매겨 쉽게 확인할 수 있도록 합니다.

   {{< alert context="danger" >}}
   **위험**

   니모닉 문구를 아는 사람은 누구나 내 돈에 엑세스할 수 있으며, 누군가 니모닉 문구를 훔쳐가도 구제할 방법이 없습니다. 니모닉 문구를 보호하려면 다음 팁을 참고하세요:
   - 니모닉 문구를 어떤 디바이스에도 디지털 파일로 저장하지 마세요.
   - 항상 펜과 종이를 사용하여 니모닉 문구를 적으세요.
   - 니모닉 문구가 적힌 종이는 안전한 곳에 보관하세요.
   - 직원을 포함하여 누구에게도 니모닉 문구를 알려주지 마세요.
   {{< /alert >}}

1. Verify your writing to make sure every word is spelled correctly and in the right order. If you numbered your phrase, it can be helpful to verify it backward.

2. Check the box ensuring you wrote down your mnemonic phrase, and click **Next**.

3. Confirm your mnemonic phrase by typing or selecting the correct words in each prompt.

4. Click **Create a wallet**.

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
