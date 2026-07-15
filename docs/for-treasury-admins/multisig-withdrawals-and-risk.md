# Multisig Withdrawals and Risk

A treasury withdrawal is signed **from the treasury account**, not from
an individual member's wallet, and only a group admin can start one. The
API builds the unsigned withdrawal payment; the required signers sign it
in their own wallets; the API submits once it has enough signatures.

If `requiredSigners` is 1, any one registered signer's signature is
enough and the API submits immediately. If it's higher, the withdrawal
waits — the API collects signatures from different signers on the same
transaction until the count is met, then submits. A single signer can't
sign twice to reach the threshold; the signatures have to come from
distinct keys.

**What this protects against:** one compromised or careless signer
draining the account. A 2-of-3 or 3-of-5 setup means no single person —
including whoever proposed the withdrawal — can move funds alone.

**What this doesn't protect against:** collusion among enough signers to
meet the threshold, or losing access to enough signer keys to *ever* meet
it. If you require 3-of-3 and one signer loses their key, the treasury is
permanently stuck — Mergepay has no recovery mechanism, and neither does
anyone else. That's how Stellar multisig works: the keys are the only
authority, and nobody can override them. Pick a threshold with real
slack: 2-of-3 tolerates one lost key, 3-of-3 tolerates none.

**Setting it up correctly matters more than using the app correctly.**
The signer count is chosen once and is awkward to change later — changing
the account's signer configuration on-chain is itself a transaction that
needs the *current* threshold's signatures to authorize. If you're not
sure what to use, start with 2-of-3 and a small balance until you're
confident, rather than defaulting to 1-of-1 "for now" on an account that
will later hold real group funds.
