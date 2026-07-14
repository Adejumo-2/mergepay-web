# The Problem

Group expense apps solve the math problem — who owes who, how much — but
not the payment problem. Splitwise, the category leader, tracks balances
but doesn't move money; users still settle outside the app via bank
transfer, cash, or Venmo, and the app has no way to confirm it actually
happened. That gap is where balances go stale and groups stop using the
app.

Cross-border groups make this worse. A trip split between people in
Nigeria, the US, and the UK hits three different payment rails, three
sets of fees, and multi-day settlement times if anyone uses a bank
transfer. A $30 Uber split can cost more in wire fees than the debt
itself to actually collect from someone abroad.

Mergepay closes both gaps with one primitive: a Stellar payment. It's
borderless by default, settles in a few seconds regardless of where
either party is, and the on-chain transaction hash is the receipt — no
"did you actually pay?" thread.

## Why this hasn't been the default already

Moving real money needs custody, compliance, and payment-rail
integrations — expensive to build and a liability to hold. Mergepay sidesteps
custody entirely: it never holds anyone's funds or keys. It builds a
payment, hands it to the user's wallet to sign, checks the signature
matches what it built, and submits it. The value moves directly between
two people's Stellar accounts. That's the design choice that makes an
app like this buildable without becoming a money transmitter that
warehouses balances.
