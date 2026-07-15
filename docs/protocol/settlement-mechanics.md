# Settlement Mechanics

A group of three — Alice, Bob, Carol — logs three expenses over a
weekend, each paid in full by one person and split equally three ways:

| Expense | Amount | Paid by | Each member's share |
|---|---|---|---|
| Dinner | $120 | Alice | $40 |
| Rideshare | $30 | Bob | $10 |
| Groceries | $60 | Carol | $20 |

Naively, that's up to 6 possible one-directional debts (each person could
owe each other person from each expense). Mergepay's settlement engine
nets them into a single balance per person instead. This is
`computeNetBalances` in `src/services/settlement.ts`: for every unsettled
share where the owner isn't the payer, it credits the payer and debits
the owner; confirmed settlements then move value from payer to payee.

- **Alice** paid $120, owes a total share of $40 + $10 + $20 = $70 → net
  **+$50** (the group owes her $50)
- **Bob** paid $30, owes $70 → net **−$40** (he owes $40)
- **Carol** paid $60, owes $70 → net **−$10** (she owes $10)

Check: +50 − 40 − 10 = 0. Net balances always sum to zero — money doesn't
appear or disappear, it only moves between members. All of this arithmetic
runs in BigInt stroops (integer value × 10⁷), not floating-point dollars,
so the sums are exact to Stellar's 7-decimal precision and never drift
(see `src/services/money.ts`).

From there, `suggestSettlements` turns net balances into the minimum
number of actual payments with a greedy match: sort debtors and creditors
by size, then repeatedly settle the largest debtor against the largest
creditor.

1. Bob (owes $40) pays Alice $40. Bob is now settled. Alice's net drops
   from +$50 to +$10.
2. Carol (owes $10) pays Alice $10. Both are now settled.

Two payments instead of up to six. The algorithm produces at most (n − 1)
transfers to zero out n members. This is what `GET /groups/:id/balances`
returns — not a raw list of who-owes-who-per-expense, but the reduced set
of suggested settlements. Each suggestion becomes one real Stellar
payment when a member acts on it, following the states in
[Expense Share Lifecycle](expense-share-lifecycle.md).

This is pure netting, not a payment pool or an internal balance the app
holds — Mergepay never custodies funds. Every dollar in the table above
corresponds to money that already moved directly between two people's
wallets: once in the original expense payment, and again in the
settlement payment. The app only ever computes who should pay whom next.

One detail worth stating plainly: the suggested pairing is a
math-minimal set of transfers, so the person you pay to settle up may not
be the person who paid for the specific expense you're thinking of. Carol
pays Alice, even though Carol's largest single share was the dinner Alice
bought and the groceries Carol herself bought. That's expected — the
engine optimizes the whole group's transfer count, not per-expense
tracing. Per-expense detail lives in the History view, not the balance.
