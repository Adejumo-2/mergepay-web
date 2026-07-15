# How It Works

Four steps, no exceptions:

1. **Sign in with a wallet.** No email/password. Mergepay uses SEP-10:
   the API issues a challenge transaction, the user's Freighter wallet
   signs it, and that signature *is* the login. The user's Stellar public
   key is their identity in the app.
2. **Add expenses to a group.** Anyone in the group logs an expense —
   amount, payer, and how it's split (equal, custom amounts, or
   percentages). The API computes each member's share and stores it as a
   pending share. The payer's own share is marked settled on creation —
   you don't owe yourself.
3. **Settle.** When a member is ready to pay what they owe, the API
   builds an *unsigned* Stellar payment (correct destination, asset,
   amount, and a memo tying it to that specific expense). The member's
   wallet signs it. The API checks the signed transaction matches
   exactly what it built, then submits it to the Stellar network. Full
   mechanics: [Settlement Mechanics](../protocol/settlement-mechanics.md).
4. **Everyone sees the same ledger.** Every settlement's transaction hash
   is stored and shown with a link to stellar.expert. The group's
   balances update from confirmed on-chain state, not from someone's word.

Two optional layers sit on top of this: **treasury mode**, for a group
that wants a shared, multisig-protected account instead of settling
peer-to-peer (see [For Treasury Admins](../for-treasury-admins/getting-started.md)),
and **anchor on/off-ramp** via SEP-24, for moving between fiat and Stellar
without leaving the app.

## The one rule behind all of it

The API never holds a private key that can move a user's money. Every
money-moving endpoint returns an unsigned transaction envelope; the
wallet signs locally; the API validates the returned signature matches
the exact transaction it built before submitting. This is why Mergepay
can move real value without custodying it — and why the validation step
in [Expense Share Lifecycle](../protocol/expense-share-lifecycle.md)
matters as much as it does.
