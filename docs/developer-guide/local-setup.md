# Local Setup

You run two repos side by side: the API (`mergepay-api`) and the web app
(`mergepay-web`). Start the API first — the web app is useless without it.

## Backend

```bash
git clone https://github.com/mergepay/mergepay-api.git
cd mergepay-api
npm install
cp .env.example .env
npm run gen:sep10key        # generates a signing key — paste it into .env as SEP10_SIGNING_SECRET
npm run prisma:generate
npm run prisma:migrate      # needs DATABASE_URL pointing at a running Postgres
npm run db:seed             # optional demo data
npm run dev                 # API on :4000
npm run worker              # separate shell — background reconciliation of submitted transactions
```

The worker is a separate process from the API. It polls Horizon for
transactions that were submitted but not yet confirmed and updates their
status — if you skip it, settlements can stay in `settling` instead of
advancing to `settled`.

## Frontend, in a second terminal

```bash
git clone https://github.com/mergepay/mergepay-web.git
cd mergepay-web
npm install
cp .env.example .env.local
npm run dev                 # http://localhost:3000, talks to NEXT_PUBLIC_API_URL (:4000 by default)
```

## What you need installed

- **Node.js 20+** and npm.
- **PostgreSQL 14+** running locally, or a hosted instance (Neon,
  Supabase) with its connection string in `DATABASE_URL`.
- The [Freighter](https://www.freighter.app/) browser extension.

## A note on networks

Both repos default to Stellar **mainnet** (`public`), not testnet — check
`STELLAR_NETWORK` in the API's `.env` and `NEXT_PUBLIC_STELLAR_NETWORK` in
the web app's `.env.local`. For local development with no real money, set
both to `testnet` and fund your Freighter account from
[friendbot.stellar.org](https://friendbot.stellar.org). Do this before you
try a settlement, or you'll be signing real mainnet payments against a
local build.

Full variable reference: [Environment Variables](environment-variables.md).
