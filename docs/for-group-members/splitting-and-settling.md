# Splitting and Settling an Expense

## Splitting an expense
When you log an expense you choose one of three split types (the API
enforces each rule and rejects anything that doesn't add up):

- **Equal** (`equal`) — the amount divides evenly across selected
  members. Stellar amounts carry 7 decimal places, so an amount that
  doesn't divide cleanly leaves a remainder of a few stroops; Mergepay
  gives that remainder to the first participant so the shares sum to the
  total exactly, to the stroop.
- **Custom** (`custom`) — you type what each person owes. The amounts
  must sum to the total, or the API rejects it with `invalid_split` and
  tells you what they summed to instead.
- **Percentage** (`percentage`) — you assign a percentage per member.
  They must sum to 100 (within a 0.001 tolerance) or it's rejected.

The person who paid has their own share marked settled automatically —
you never owe yourself.

## Settling what you owe
Open the group's Balances tab. It shows the netted suggestions described
in [Settlement Mechanics](../protocol/settlement-mechanics.md) — not a
raw list per expense, the reduced set of payments that clears everyone.
Tap **Settle** next to a suggestion:

1. The app requests an unsigned payment from the API
   (`POST /expenses/:id/settle` for a single share, or
   `POST /groups/:id/settlements` to batch several).
2. Freighter pops up showing the exact destination, asset, and amount —
   check it matches what you expect before signing. This is the one
   moment you're actually authorizing money to move; everything before
   it was just bookkeeping.
3. After signing, the app posts the signed transaction back
   (`POST /settlements/:id/confirm`). The API checks the signature is
   attached to exactly the payment it built, submits it, and shows a
   **pending** state, then **settled** with a link to the transaction on
   stellar.expert once Horizon confirms it — usually 3-5 seconds.

If a settlement fails (most often a missing trustline for USDC, or an
underfunded account), the API surfaces the reason and the settlement is
marked `failed` so you can fix the underlying issue — add a trustline in
Freighter, or fund the account — and try again.
