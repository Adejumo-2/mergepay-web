# Expense Share Lifecycle

An `ExpenseShare` is one member's owed amount on one expense. In the
database its `status` field moves through three states — `pending`,
`settling`, `settled` — with a fourth state (`failed`) living on the
`Settlement` record, not the share. The names below are the exact string
values stored in Postgres (`expense_shares.status` in
[`schema.prisma`](https://github.com/mergepay/mergepay-api/blob/main/prisma/schema.prisma)).

## `pending`
An expense is created in a group (`POST /groups/:id/expenses`) with an
amount, a payer, and a split. The API computes each participant's owed
amount with `computeShares` and writes one `ExpenseShare` row per
participant. Every non-payer share starts `pending`. The payer's own
share is written straight to `settled` — you don't owe yourself. No money
has moved. Any group member can create an expense; only the payer or a
group admin can edit or delete it, and a delete is refused once any
non-payer share is `settled`.

## `settling`
A member calls `POST /expenses/:id/settle`. The API creates a
`Settlement` row (status `pending`), builds an unsigned payment XDR —
exact source, destination, asset, amount, and an `MP:<shortCode>` memo —
and flips the member's share to `settling`. Nothing has been submitted to
Stellar yet; the XDR just exists, waiting for a signature. The endpoint
refuses if you have no share, if your share is already settled, or if
you're the payer.

## The confirm step (where a signed XDR is validated)
The member's wallet signs the XDR and the client posts it to
`POST /settlements/:id/confirm`. Before anything is submitted, the API
re-parses the signed XDR and checks it matches the stored intent exactly:
same source account, exactly one operation, that operation is a payment,
same destination, same asset, same amount (compared at 7-decimal
precision), and same memo. Any mismatch is rejected with `xdr_mismatch`
and nothing is submitted.

{% hint style="warning" %}
This is the step where a bug would matter most. The check exists because
the API never holds the key that signs the transaction — the signature
could in principle be attached to a *different* transaction, so the
server must prove the thing it's about to submit is byte-for-byte the
payment the user agreed to. The validation lives in `validatePaymentTx`
in `src/services/stellar.ts`. If you're picking a backend issue to start
with, read that function and `src/routes/settlements.ts` before anything
else.
{% endhint %}

## `settled`
Once the signed XDR passes validation, the API submits it to Horizon,
stores the returned transaction hash on the `Settlement`, sets that
settlement's status to `confirmed`, and — in the same database
transaction — sets the linked `ExpenseShare` to `settled`. This is
terminal: a settled share is never reopened. The group's balance for that
member updates immediately, because `computeNetBalances` only counts
unsettled shares and confirmed settlements.

## `failed` (on the settlement, not the share)
Horizon submission can fail after a valid signature — insufficient
balance, a missing trustline for the settlement asset, a sequence-number
conflict. When `submitPayment` throws, the API sets the `Settlement`
status to `failed` and the error propagates to the client. The share
stays `settling`; the user fixes the underlying issue (for example,
establishes a trustline in their wallet) and settles again. Note the
confirm endpoint is idempotent on success: calling it again on an
already-`confirmed` settlement returns the stored result without
re-submitting.
