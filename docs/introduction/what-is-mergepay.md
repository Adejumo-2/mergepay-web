# What is Mergepay

Mergepay is a Stellar-native group settlement app. It replaces the "I'll
pay you back" spreadsheet or IOU thread with real, on-chain payments:
every settlement is an actual Stellar transaction with a transaction hash
you can look up on stellar.expert, not an internal ledger entry inside
someone's database.

Groups use it for shared spending — rent, trips, dinners, recurring
household costs — the same use case as apps like Splitwise. The
difference is what happens when someone hits "settle": instead of a bank
transfer that takes a day and costs a few dollars, Mergepay builds a
Stellar payment (XLM or a stablecoin like USDC), the payer signs it in
their own wallet, and it clears in seconds for a fraction of a cent.

Two repos make up the product:
[`mergepay-web`](https://github.com/mergepay/mergepay-web) is the Next.js
14 frontend, [`mergepay-api`](https://github.com/mergepay/mergepay-api) is
the Fastify backend that builds transactions, validates signed XDRs, and
talks to Horizon. Keys never leave the user's wallet — the API only ever
builds *unsigned* transactions. The one private key the server holds is
its own SEP-10 signing key, used to build the login challenge, never to
move a user's money.

## What's actually on-chain, and what isn't

On-chain: the settlement payments themselves, their memos, and the
transaction hashes. Off-chain, in Postgres: groups, membership, the
expense records, each member's computed share, and the *intent* of a
settlement before it's signed and submitted. The chain is the record of
money that moved; the database is the record of who agreed to what. The
two meet at the moment a signed payment is submitted and its hash is
stored against the settlement.
