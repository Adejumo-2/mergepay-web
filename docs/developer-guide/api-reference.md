# API Reference

Base URL is `NEXT_PUBLIC_API_URL` (default `http://localhost:4000`). Every
endpoint except the SEP-10 auth pair and the anchor webhook requires an
`Authorization: Bearer <jwt>` header. Request bodies are validated with
Zod; every group action checks membership (and admin rights where noted).

The standard error shape, returned on any failure:

```json
{ "error": { "code": "invalid_split", "message": "Custom amounts must sum to 42.5 (got 40)" } }
```

## Endpoint index

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/challenge` · `/auth/verify` · `/auth/logout` | SEP-10 auth |
| GET/PATCH | `/me` | Current user |
| POST/GET | `/groups` · `/groups/:id` | Create / list / detail |
| POST | `/groups/:id/invite` · `/groups/join` · `/groups/:id/leave` · `/groups/:id/archive` | Membership |
| POST/GET/PATCH/DELETE | `/groups/:id/expenses` · `/expenses/:id` | Expenses |
| POST | `/expenses/:id/settle` · `/groups/:id/settlements` · `/settlements/:id/confirm` | Settlement |
| GET | `/groups/:id/balances` · `/groups/:id/ledger` | Balances & ledger |
| POST/GET | `/groups/:id/treasury/enable` · `/groups/:id/treasury` · `/groups/:id/treasury/deposit` · `/groups/:id/treasury/withdraw` · `/treasury-transactions/:id/confirm` · `/groups/:id/treasury/history` | Treasury |
| GET/POST | `/anchors` · `/anchors/deposit` · `/anchors/withdraw` · `/anchors/sessions/:id/complete` · `/anchors/sessions` · `/anchors/webhook` | Anchors (SEP-24) |
| GET | `/history` | Cross-group history |
| POST | `/uploads` | Receipt upload (multipart) |

Auth endpoints are rate-limited to 10 requests per minute.

## Worked examples

### `POST /auth/challenge`
Request:
```json
{ "account": "Greally...VALIDKEY" }
```
Response:
```json
{
  "transaction": "AAAAAgAAAAB...base64-XDR...",
  "networkPassphrase": "Public Global Stellar Network ; September 2015"
}
```
The wallet signs `transaction`, then calls verify. An invalid public key
returns `{ "error": { "code": "invalid_account", ... } }`.

### `POST /auth/verify`
Request:
```json
{ "transaction": "AAAAAgAAAAB...signed-XDR..." }
```
Response:
```json
{
  "token": "eyJhbGciOi...jwt...",
  "user": {
    "id": "clx0user1",
    "stellarPublicKey": "GREALLY...VALIDKEY",
    "displayName": "GREA…IKEY",
    "avatarUrl": null,
    "createdAt": "2026-07-14T10:00:00.000Z"
  }
}
```
On first login the user is created with a shortened key as the display
name. The token is valid for 12 hours.

### `POST /groups/:id/expenses`
Request (equal split of 45 XLM across three members):
```json
{
  "title": "Dinner",
  "amount": "45.0000000",
  "assetCode": "XLM",
  "splitType": "equal",
  "shares": [
    { "userId": "clx0user1" },
    { "userId": "clx0user2" },
    { "userId": "clx0user3" }
  ]
}
```
Response (abridged): an `expense` object with one `shares` entry per
participant. The payer defaults to the caller; the payer's own share is
returned with `status: "settled"`, the others `status: "pending"`. A split
that doesn't add up returns `{ "error": { "code": "invalid_split", ... } }`;
a non-member participant returns `invalid_participant`.

### `POST /expenses/:id/settle`
Request (optional body — omit to settle in the expense's own asset):
```json
{ "assetCode": "USDC", "assetIssuer": "GA5ZSEJY...KZVN" }
```
Response:
```json
{
  "settlement": {
    "id": "clx0settle1",
    "fromUserId": "clx0user2",
    "toUserId": "clx0user1",
    "amount": "15.0000000",
    "assetCode": "USDC",
    "status": "pending",
    "stellarTxHash": null,
    "memo": "MP:AB12CD",
    "expenseId": "clx0expense1",
    "createdAt": "2026-07-14T10:05:00.000Z"
  },
  "xdr": "AAAAAgAAAAB...unsigned-payment-XDR...",
  "networkPassphrase": "Public Global Stellar Network ; September 2015"
}
```
Errors: `no_share` (you're not in the split), `already_settled`,
`payer_share` (you paid, so you owe nothing), or `account_unfunded` (your
Stellar account doesn't exist yet).

### `POST /settlements/:id/confirm`
Request (the wallet-signed version of the `xdr` above):
```json
{ "signedXdr": "AAAAAgAAAAB...signed-payment-XDR..." }
```
Success response:
```json
{
  "settlement": {
    "id": "clx0settle1",
    "status": "confirmed",
    "stellarTxHash": "3389e9f...c2b7",
    "amount": "15.0000000",
    "assetCode": "USDC"
  }
}
```
If the signed transaction doesn't match what the API built, it's rejected
**before** submission:
```json
{ "error": { "code": "xdr_mismatch", "message": "Payment amount does not match" } }
```
The same `xdr_mismatch` code covers a wrong source, wrong destination,
wrong asset, wrong memo, or more than one operation. Only the settlement's
payer may confirm it (`403` otherwise). Confirming an already-`confirmed`
settlement returns the stored result without re-submitting.

### `GET /groups/:id/balances`
Response:
```json
{
  "balances": [
    { "userId": "clx0user1", "user": { }, "net": "30.0000000", "assetCode": "XLM" },
    { "userId": "clx0user2", "user": { }, "net": "-15.0000000", "assetCode": "XLM" },
    { "userId": "clx0user3", "user": { }, "net": "-15.0000000", "assetCode": "XLM" }
  ],
  "suggestions": [
    { "fromUserId": "clx0user2", "from": { }, "toUserId": "clx0user1", "to": { }, "amount": "15.0000000", "assetCode": "XLM", "assetIssuer": null },
    { "fromUserId": "clx0user3", "from": { }, "toUserId": "clx0user1", "to": { }, "amount": "15.0000000", "assetCode": "XLM", "assetIssuer": null }
  ]
}
```
`net` is a signed decimal string — positive means the group owes this
member, negative means they owe. `suggestions` is the minimal set of
transfers that zeroes every balance (see
[Settlement Mechanics](../protocol/settlement-mechanics.md)).

## Notes for the rest of the table

- **Treasury** endpoints mirror settlement: enable (admin only), deposit
  and withdraw both return an unsigned `xdr` plus a `treasuryTransaction`,
  and `/treasury-transactions/:id/confirm` submits the signed XDR. A
  multisig withdrawal is created with status `awaiting_signatures` (see
  [Multisig Withdrawals & Risk](../for-treasury-admins/multisig-withdrawals-and-risk.md)).
- **Anchors** implement SEP-24: `deposit`/`withdraw` start a session and
  return a SEP-10 challenge from the anchor; `/anchors/sessions/:id/complete`
  exchanges the signed challenge for the interactive URL. `/anchors/webhook`
  is unauthenticated but verifies a constant-time secret and always
  returns `200` so it never reveals whether the secret matched.
- **History** (`GET /history`) returns the caller's expenses and
  settlements across all their groups, for the CSV/PDF export in the web
  app.
