# Environment Variables

Taken from each repo's `.env.example` and its config parser. Re-check
against the live files before trusting this page — it's a snapshot and can
drift. Note that both repos default to Stellar **mainnet** (`public`), not
testnet.

## `mergepay-api`

Parsed and defaulted in `src/config.ts` (`.env.example` at the repo root).

| Variable | Description | Default |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://postgres:postgres@localhost:5432/mergepay` |
| `PORT` | Port the API listens on | `4000` |
| `API_PUBLIC_URL` | Public base URL of the API (SEP-10 home domain is derived from its host) | `http://localhost:4000` |
| `WEB_URL` | Frontend origin for CORS + invite links; `*` allows all origins | `*` |
| `JWT_SECRET` | Secret for signing session JWTs (12h expiry) | `change-me-in-production` |
| `STELLAR_NETWORK` | `testnet` or `public` | `public` |
| `HORIZON_URL` | Horizon server | `https://horizon.stellar.org` |
| `SEP10_SIGNING_SECRET` | Server's SEP-10 signing key (`npm run gen:sep10key`) | *(unset)* |
| `SEP10_HOME_DOMAIN` | SEP-10 home domain | derived from `API_PUBLIC_URL` |
| `WEB_AUTH_DOMAIN` | SEP-10 web-auth domain | derived from `API_PUBLIC_URL` |
| `ANCHOR_HOME_DOMAIN` | SEP-24 anchor home domain | `testanchor.stellar.org` |
| `ANCHOR_NAME` | Display name for the anchor | `Stellar Test Anchor` |
| `ANCHOR_WEBHOOK_SECRET` | Shared secret verifying the anchor webhook | `change-me` |
| `STABLE_ASSET_CODE` | Stable asset code for settlement | `USDC` |
| `STABLE_ASSET_ISSUER` | Issuer of the stable asset | Circle mainnet USDC (`GA5ZSEJY…KZVN`) |
| `UPLOADS_DIR` | Directory for receipt uploads | `./uploads` |

The testnet USDC issuer, for when you set `STELLAR_NETWORK=testnet`, is
`GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5`.

## `mergepay-web`

Read in `src/lib/constants.ts` (`.env.example` at the repo root). All are
`NEXT_PUBLIC_*` because the browser needs them.

| Variable | Description | Default |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | Base URL of mergepay-api | `http://localhost:4000` |
| `NEXT_PUBLIC_STELLAR_NETWORK` | `testnet` or `public` | `public` |
| `NEXT_PUBLIC_HORIZON_URL` | Horizon server URL | `https://horizon.stellar.org` |
| `NEXT_PUBLIC_STABLE_ASSET_CODE` | Stable asset offered for settlement | `USDC` |
| `NEXT_PUBLIC_STABLE_ASSET_ISSUER` | Issuer of the stable asset | Circle mainnet USDC (`GA5ZSEJY…KZVN`) |

The network passphrase and the stellar.expert explorer base URL are both
derived from `NEXT_PUBLIC_STELLAR_NETWORK` — you don't set them directly.
