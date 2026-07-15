# Understanding Fees and Balances

**Fees you pay:** the Stellar network fee only (see
[Economic Model](../protocol/economic-model.md) — a settlement is built at
200 stroops, a fraction of a US cent). Mergepay itself charges nothing and
takes no cut of the amount you send.

**Why your balance sometimes doesn't match "what I think I owe."** The
app shows netted balances, not per-expense debts — see the worked example
in [Settlement Mechanics](../protocol/settlement-mechanics.md). If you
paid for dinner and someone else paid for the Uber, your balance reflects
the *difference*, not two separate line items. This is intentional: it's
fewer actual payments for the group to make, but it means "who do I owe
for the Uber" isn't a question the balance screen answers directly — the
History tab, not Balances, shows per-expense detail (and can export it to
CSV or PDF).

**Why the person you pay isn't always the person who paid.** The
suggestion engine minimizes the total number of transfers across the
whole group, so you may be asked to pay someone you never directly
transacted with. The money still nets out correctly — it's just routed
for fewer transfers, not for per-expense tracing.

**Why a balance can be non-zero even after you settle.** Settling clears
the specific suggestion you acted on, and only a *confirmed* settlement
reduces a debt — a `pending` or `failed` one doesn't. If a new expense is
added by someone else after you settle, the netted balance recalculates
and a new suggestion may appear; that's a new debt, not the old one
reappearing. The source of truth is the chain: the transaction hash on
stellar.expert confirms a specific settlement actually happened,
regardless of what the balance screen shows at any given moment.
