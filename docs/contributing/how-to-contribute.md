# How to Contribute

Mergepay is two open-source repos, both taking contributions:

- [`mergepay-api`](https://github.com/mergepay/mergepay-api) — the Fastify
  backend: transaction building, XDR validation, the settlement engine,
  Horizon submission.
- [`mergepay-web`](https://github.com/mergepay/mergepay-web) — the Next.js
  frontend and this docs folder.

To contribute:

1. Fork the relevant repo — `mergepay-api` for backend work, `mergepay-web`
   for frontend or docs.
2. Read that repo's `CONTRIBUTING.md` first. It has the branch naming
   (`feat/`, `fix/`, `docs/`), the conventional commit format, and the PR
   checklist you'll be held to (`typecheck`, `lint`, `build` all pass; no
   secrets committed; contract changes mirrored across both repos).
3. Pick an open issue labelled `good first issue` if it's your first
   contribution. If nothing's labelled that way, open an issue describing
   what you want to change and get agreement on the approach before writing
   code.
4. Open a PR against `main`. CI must pass before review. Link the PR to its
   issue with `Closes #NN`.

## Where the risky code is

If you're touching the backend, the file that matters most is the XDR
validation in `mergepay-api`'s `src/services/stellar.ts` — specifically
`validatePaymentTx`. That function is the only thing standing between "the
user signed what they agreed to" and "the user signed something else."
A gap there is the highest-impact bug in the codebase. Read
[Expense Share Lifecycle](../protocol/expense-share-lifecycle.md) and
[Settlement Mechanics](../protocol/settlement-mechanics.md) before changing
anything in the settlement path.

## Docs contributions

This docs folder is open to contribution too. Typos, unclear explanations,
and missing pages are all fair game as PRs against `mergepay-web`. Docs
live in `docs/` as plain Markdown; the table of contents is
[`docs/SUMMARY.md`](../SUMMARY.md). If you add a page, add its line to
`SUMMARY.md` in the same PR or it won't appear in the rendered docs.
