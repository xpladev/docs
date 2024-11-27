---
title: Mobile Vault
weight: 40
type: docs
---

XPLA mobile vault is an application that enables users to interact with XPLA Chain Core.

XPLA mobile vault allows users to:

- Create vaults and send tokens
- Get involved with staking by browsing through validator information and delegating XPLA tokens.
- Use QRCodes for easy interactions when sending assets and recovering vaults

## URL Scheme

XPLA mobile vault includes a custom [URL Scheme](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app) that lets developers trigger different actions in the app.

These URL handlers can be opened by scanning a QR code or opening the link directly.

### Send

The send function allows a user to send a specified amount of funds to a recipient. This function can be used to accept payment for goods and other point-of-sale configurations.

#### URL

```
xplavault://send/?payload=${base64 json}
```

#### Payload Format

| Key     | Description                                   | Required? |
|---------|-----------------------------------------------|-----------|
| address | XPLA Chain address to send funds to           |           |
| token   | Native token denom or cw20 contract address   | ✔️        |
| amount  | Amount of tokens in micro format              |           |
| memo    | Specific memo to include with the transaction |           |

#### Example

**Payload:**

```
{
  "address": "xpla1dcegyrekltswvyy0xy69ydgxn9x8x32zdtapd8",
  "token": "axpla",
  "amount": "250000",
  "memo": "Order #1122"
}
```

**Base64 encoded payload:**

```
ewogICJhZGRyZXNzIjogInhwbGExZGNlZ3lyZWtsdHN3dnl5MHh5Njl5ZGd4bjl4OHgzMnpkdGFwZDgiLAogICJ0b2tlbiI6ICJheHBsYSIsCiAgImFtb3VudCI6ICIyNTAwMDAiLAogICJtZW1vIjogIk9yZGVyICMxMTIyIgp9
```

**Full URL with encoded payload:**

```
xplavault://send/?payload=ewogICJhZGRyZXNzIjogInhwbGExZGNlZ3lyZWtsdHN3dnl5MHh5Njl5ZGd4bjl4OHgzMnpkdGFwZDgiLAogICJ0b2tlbiI6ICJheHBsYSIsCiAgImFtb3VudCI6ICIyNTAwMDAiLAogICJtZW1vIjogIk9yZGVyICMxMTIyIgp9
```
