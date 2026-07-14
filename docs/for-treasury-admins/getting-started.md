# Getting Started with Treasury Mode

Treasury mode is for groups that want a shared pot instead of settling
peer-to-peer every time — a house account, a trip fund people top up in
advance. It's a real Stellar account, created in someone's wallet (the
API never holds its key), then registered to the group.

1. In the group's **Treasury** tab, click **Enable treasury mode**. Only
   a group admin can do this.
2. Create a new Stellar account in Freighter (or use an existing one) —
   do this outside Mergepay, in your wallet, since the account's key
   needs to live somewhere Mergepay never touches. Fund it so it meets
   the network's minimum balance reserve.
3. Register the account's public key to the group and set the number of
   required signers (`requiredSigners`, 1–20). This is the count of
   distinct signatures a withdrawal needs before the API will submit it —
   read [Multisig Withdrawals & Risk](multisig-withdrawals-and-risk.md)
   before picking a number above 1.
4. Members deposit into the treasury account directly. A deposit is a
   normal payment built by the API and signed from the depositor's own
   wallet — the same build-then-sign flow as a settlement.

Deposits and withdrawals both follow the build-unsigned → wallet-signs →
confirm pattern, but they differ in who can start them and how many
signatures they need. Withdrawals are the part with real risk — read the
next page before you rely on this for anything holding real money.
