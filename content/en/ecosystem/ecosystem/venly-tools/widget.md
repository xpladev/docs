---
title: Widget
weight: 30
type: docs
---

# Venly Widget

Venly Widget is a powerful javascript SDK created to streamline everyday blockchain tasks. Its purpose is to enable functionalities otherwise restricted due to security implications, such as creating signatures. By encapsulating Venly's extensive capabilities within a user-friendly JavaScript layer, Venly Widget empowers developers and simplifies the development process.

> If you are new to Web3 and don't have experience with blockchain technologies, we recommend you use the Venly [Widget](https://docs.venly.io/docs/widget-overview) natively for a better developer experience.

## Create XPLA wallets with Venly widget

Venly Wallet allows you to create and manage wallets on the XPLA network. You can send and receive native XPLA and ERC20 tokens. Apart from this, the Venly Widget also supports:

![create-xpla-wallet](https://github.com/xpladev/docs/assets/139292301/47143857-c296-4b3e-8a36-cac20c323023)

## Features
- Multiple blockchains
- Native token swap functionality
- Signing messages (including EIP712 message)
- Calling smart contracts
- Importing wallets
- Support web and mobile
- Offers social logins

## Look and feel

As the Widget is a product that incorporates a user interface (UI), let's look at how some of the more regular flows would appear for an end user.

### Token transfer

The application prompts the user to transfer a token from their wallet to a different destination in this flow.

![transferring-xpla-tokens](https://github.com/xpladev/docs/assets/139292301/acb03eb3-6f41-417a-a39e-9335ae9e2941)

## Different integration options

Multiple integration options are available to incorporate the Venly Widget into your application. Here is a brief overview of some of these options:

1. [Native Integration with Venly SDK](https://docs.venly.io/docs/widget-overview): This approach involves utilizing the Venly SDK directly within your application's codebase. It allows you to access the full functionality of the Venly Widget and customize its behavior according to your requirements.
2. [Ethers.js Integration](https://docs.venly.io/docs/ethersjs): You can integrate the Venly Widget with your application using the popular Ethers.js library. This involves utilizing the Ethers.js API to interact with the Venly Widget and manage user wallets, transactions, and other blockchain-related operations.
3. [Web3Modal Integration](https://docs.venly.io/docs/web3modal-walletconnect) (WalletConnect): Web3Modal is a library that simplifies connecting to different wallet providers using standard protocols like WalletConnect. Integrating Web3Modal and WalletConnect enables users to interact with the Venly Widget and connect their wallets to your application seamlessly.
4. [Wagmi](https://docs.venly.io/docs/wagmi): @wagmi/core is a comprehensive VanillaJS library that provides all the essential tools for initiating Ethereum-related tasks. Simplifying processes such as "Connect Wallet," showcasing ENS and balance details, message signing, contract interaction, and more has never been easier with this library.
5. [Web3-react](https://docs.venly.io/docs/web3-react): web3-react offers abstractions that facilitate the connection of your decentralized application (dApp) to web3 connectors. It also exposes methods for seamless interaction with these connections.

These are just a few integration options to incorporate the Venly Widget into your application. The choice of integration method depends on your specific requirements, preferences, and the existing infrastructure of your application.

![integration-options](https://github.com/xpladev/docs/assets/139292301/1bbaadfa-dd9c-40ff-8dfe-c0513893f72e)

## When to choose what?

If you want to build your wallet app for users to interact with, you should use the [Wallet API](https://venly.readme.io/docs/overview).

If you want to integrate an existing, full-fletched wallet solution, you can use the Venly Widget.  
There are multiple ways to integrate it - natively or by using another library (which also uses the Venly Widget in the background):

![choosing-integration-options](https://github.com/xpladev/docs/assets/139292301/f6f554d0-21fe-4a89-9e04-2c42a6bccb3c)

| Integration Type                                                    | Description                                                                                                                                                     | UI Flexibility                                                                                                                                                                                                | Blockchains          |
| :------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------- |
| **[Native](https://docs.venly.io/docs/widget-getting-started)**     | A JavaScript SDK that seamlessly integrates with various API functionalities, empowering users to execute diverse blockchain operations effortlessly.           | The Widget delivers pre-designed screens tailored explicitly for end users, offering a ready-to-use solution. These screens are not customizable, ensuring consistency and simplicity in the user experience. | All supported chains |
| **[Ethers.js](https://docs.venly.io/docs/ethersjs)**                | A JavaScript library is used to interact with the EVM blockchains. It provides a wide range of functionality for developers to build decentralized applications | This integration ensures that the Widget is invoked when needed, allowing users to conveniently and securely perform the required actions within the context of your application.                             | Only EVM chains      |
| **[Wagmi](https://docs.venly.io/docs/wagmi)**                       | A collection of React Hooks containing everything to work with EVMs.                                                                                            | This integration ensures that the Widget is invoked when needed, allowing users to conveniently and securely perform the required actions within the context of your application.                             | Only EVM chains      |
| **[Web3-React](https://docs.venly.io/docs/web3-react)**             | A JavaScript SDK based on ethers.js.                                                                                                                            | This integration ensures that the Widget is invoked when needed, allowing users to conveniently and securely perform the required actions within the context of your application.                             | Only EVM chains      |
| **[Web3Modal](https://docs.venly.io/docs/web3modal-walletconnect)** | Web3Modal is a library that simplifies the process of connecting to different wallet providers using standard protocols like WalletConnect                      | When users opt to log in with Venly, the modal will initiate the Venly Widget upon various user actions, facilitating seamless integration between your application and the Venly platform.                   | Only EVM chains      |

> 👍 
> 
> Ready to try out the Venly Widget? [Click here to read the getting started guide](https://docs.venly.io/docs/widget-getting-started).
