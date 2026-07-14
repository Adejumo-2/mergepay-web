# Stellar Integration Overview

Every Stellar primitive Mergepay uses, and exactly where:

| Primitive | Used for | Where in the code |
|---|---|---|
| SEP-10 | Wallet login — the user's public key is their identity | `POST /auth/challenge`, `POST /auth/verify` (`src/routes/auth.ts`, `src/services/sep10.ts`) |
| Payments | The actual settlement transaction | `POST /expenses/:id/settle`, `POST /settlements/:id/confirm` (`src/routes/settlements.ts`) |
| Memos | Binding a payment to a specific expense (`MP:<shortCode>`, text memo, capped at 28 bytes) | `memoText` in `src/services/stellar.ts` |
| Trustlines | Enabling USDC (or any non-native asset) settlement | Checked implicitly at submission; a missing trustline fails the payment |
| Multisig | Shared treasury accounts requiring 2+ signers for withdrawal | `POST /groups/:id/treasury/*` (`src/routes/treasury.ts`) |
| SEP-24 | Fiat on/off-ramp via an anchor | `POST /anchors/deposit`, `POST /anchors/withdraw`, `POST /anchors/sessions/:id/complete` (`src/routes/anchors.ts`) |

**The rule that matters most:** the API never holds a user's private key.
Every endpoint that moves money returns an *unsigned* transaction
envelope (XDR). The client's wallet — Freighter — signs it locally. The
API's only private key is its own SEP-10 signing key (`SEP10_SIGNING_SECRET`),
used to build the login challenge, never to sign a payment on a user's
behalf.

## Network and configuration

The network is set by `STELLAR_NETWORK`, which defaults to `public`
(mainnet) in both `mergepay-api` (`src/config.ts`) and `mergepay-web`
(`src/lib/constants.ts`). Set it to `testnet` for a no-real-money dry run;
that switches the network passphrase, the Horizon URL, and the
stellar.expert explorer links together. Transactions are built with a
300-second timeout.

## Request flow for a settlement, end to end

```
Client              API                    Horizon
  |  POST /expenses/:id/settle |
  | ---------------------------> |
  |  { settlement, xdr }        |   (share -> "settling")
  | <--------------------------- |
  |  [wallet signs XDR]         |
  |  POST /settlements/:id/confirm { signedXdr }
  | ---------------------------> |
  |                              |  validatePaymentTx: signed XDR must match
  |                              |  stored intent exactly, else xdr_mismatch
  |                              |  submit ------------------->
  |                              |  <------------------------- tx hash
  |  { settlement: confirmed, txHash }   (share -> "settled")
  | <--------------------------- |
```

The same challenge/sign/submit shape recurs for SEP-10 login (sign a
challenge instead of a payment) and for SEP-24 anchor sessions (the API
fetches a SEP-10 challenge *from the anchor*, the wallet signs it, and the
API exchanges it for the anchor's interactive URL).
