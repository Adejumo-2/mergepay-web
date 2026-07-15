# Economic Model

Mergepay charges no protocol fee. The only cost of using it is the
Stellar network's own transaction fee. Mergepay builds each payment at
twice the network base fee — `BASE_FEE` is 100 stroops, so a settlement
transaction is set to 200 stroops (0.00002 XLM), a small premium over the
minimum to help the transaction clear during surge pricing. At a few
cents per XLM that's a tiny fraction of a US cent per settlement —
functionally free compared to a bank wire or a card network's interchange
fee. A settlement is a single payment operation, and Stellar charges per
operation.

## Asset choice
Groups settle in native XLM or a configured stable asset (USDC by
default, set via `STABLE_ASSET_CODE` / `STABLE_ASSET_ISSUER`; the default
issuer is Circle's mainnet USDC). XLM has no counterparty risk but its
price moves; USDC removes price risk but requires the receiving account
to hold a trustline to the issuer before it can receive a payment in that
asset. If the trustline is missing, the payment fails at submission —
Horizon rejects it and the settlement is marked `failed`, so the payer
sees the failure rather than a silent hang.

The asset a group "settles in" isn't fixed globally — `groupPrimaryAsset`
derives it from the group's most recent expense, defaulting to XLM if the
group has no expenses yet. An individual settling a share can also
override the asset on `POST /expenses/:id/settle`.

## Multisig cost
Treasury accounts requiring multiple signatures don't add on-chain fee
cost — Stellar charges per operation, not per signature — but they add
latency. A withdrawal from a multisig treasury is created with status
`awaiting_signatures` and stays there until enough of the required signers
have signed, which could be minutes or days depending on how responsive
the group's other signers are. See
[Multisig Withdrawals & Risk](../for-treasury-admins/multisig-withdrawals-and-risk.md).

## What Mergepay doesn't do
No yield, no lending, no liquidity pool, no Soroban smart contracts —
settlement only. Mergepay holds no funds and runs no contract on-chain;
it's an off-chain coordinator that builds ordinary Stellar payments. If a
future version adds treasury yield (say, parking idle group funds in a
lending protocol), that's a materially different trust model than what's
documented here and would need its own economic model page, not a patch
to this one.
