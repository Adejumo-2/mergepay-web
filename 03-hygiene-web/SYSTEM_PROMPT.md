# Task: Repo hygiene for open-source legitimacy — mergepay-web

## Role
You are operating inside a local clone of `mergepay/mergepay-web`, on `main`,
with `gh` CLI authenticated with `repo` and `admin:org` scope sufficient to
set branch protection and topics. This task touches docs, repo settings, and
metadata only — no changes to `src/`.

Verified facts:
- Repo is already public, homepage already set to
  `https://mergepay-web.vercel.app`.
- No topics currently set.
- No `SECURITY.md` currently exists.
- No release has ever been published.
- A `.github/workflows/ci.yml` exists but its exact job name was **not**
  confirmed before this prompt was written — do not assume it matches the
  API repo's `build-test`. Read the file yourself first and use its real
  job `name:` value for Step 3.

## Step 0 — Read the CI workflow
```bash
cat .github/workflows/ci.yml
```
Note the literal job `name:` field(s). Use that exact string in Step 3, not
a guess.

## Step 1 — Repo topics
```bash
gh repo edit mergepay/mergepay-web --add-topic stellar --add-topic soroban --add-topic nextjs --add-topic typescript --add-topic freighter --add-topic defi --add-topic open-source
```

## Step 2 — SECURITY.md
Create `SECURITY.md` at repo root. This is a frontend, so the attack surface
is different from the API repo — do not copy that repo's file. Cover:

1. **Supported versions** — testnet-focused, `main` only.
2. **In-scope vulnerability classes**, specific to this app (read
   `README.md`'s "How the flows work" section first):
   - XSS in expense descriptions, group names, or any user-supplied text
     rendered without sanitization
   - Invite-link (`/join/[code]`) handling — code enumeration, open
     redirect, or auto-accepting an invite without confirmation
   - Client-side JWT storage/exposure (check how the token is stored —
     flag if it's `localStorage` vs. httpOnly cookie, since that's a real
     XSS-exfiltration difference)
   - Wallet-connection phishing — anything that could trick a user into
     signing an unintended transaction (mismatched amount/destination shown
     vs. what's actually signed)
   - CSRF on state-changing requests to the API
3. **Out of scope** — vulnerabilities in Freighter itself, vulnerabilities
   in `mergepay-api` (report those on that repo instead).
4. **Audit status** — unaudited, testnet-focused, use at your own risk.
5. **Reporting** — GitHub Security Advisory only, no public issues for live
   vulnerabilities.

Commit:
```
docs(security): add SECURITY.md
```
Push.

## Step 3 — Branch protection on `main`
Use the real job name from Step 0 in place of `<JOB_NAME>` below:
```bash
gh api -X PUT repos/mergepay/mergepay-web/branches/main/protection \
  -H "Accept: application/vnd.github+json" \
  -f required_status_checks[strict]=true \
  -f 'required_status_checks[contexts][]=<JOB_NAME>' \
  -f enforce_admins=true \
  -f 'required_pull_request_reviews[required_approving_review_count]=1' \
  -f required_linear_history=false \
  -f allow_force_pushes=false \
  -f allow_deletions=false
```
Verify with:
```bash
gh api repos/mergepay/mergepay-web/branches/main/protection
```
If it fails on permissions, stop and report — don't retry with elevated
scopes you weren't given.

## Step 4 — README additions
Same three additions as the API repo, adjusted for this repo:

1. Badges:
   ```markdown
   ![CI](https://github.com/mergepay/mergepay-web/actions/workflows/ci.yml/badge.svg)
   ![License](https://img.shields.io/github/license/mergepay/mergepay-web)
   ![Vercel](https://img.shields.io/badge/deployed-vercel-black)
   ```
2. Maintainer table (identical to API repo):
   ```markdown
   | Maintainer | Role | GitHub | Telegram |
   |---|---|---|---|
   | Fuhad (K1NGD4VID) | Maintainer | [@K1NGD4VID](https://github.com/K1NGD4VID) | [FUHAD: add your Telegram handle] |
   ```
3. Contributors credits at the bottom:
   ```markdown
   ## Contributors

   [![Contributors](https://contrib.rocks/image?repo=mergepay/mergepay-web)](https://github.com/mergepay/mergepay-web/graphs/contributors)
   ```

Commit:
```
docs(readme): add badges, maintainer table, contributor credits
```
Push.

## Step 5 — Release tag
```bash
gh release create v0.1.0 --repo mergepay/mergepay-web \
  --title "v0.1.0 — Initial public release" \
  --notes "First public release. Neobrutalist Next.js 14 frontend: SEP-10 wallet login, group/expense management, settlement, treasury mode, SEP-24 anchor flows, history export. Pairs with mergepay-api."
```

## Step 6 — Flag, don't resolve, the open strategic question
Do **not** decide this — just surface it clearly in your final report:
this repo (`mergepay-web`) carries its own `FUNDING.json`/`CONTRIBUTING.md`
so it could be claimed independently later. Confirm in your report that this
repo's `FUNDING.json` is unchanged and ask Fuhad directly whether that's still
the actual plan or whether these funding-facing files should be trimmed since
only the API repo is being submitted right now.

## Constraints
- Do not create labels or issues on this repo — the bounty queue is scoped to
  the API repo only.
- Do not touch `src/`, `.env.example`, `package.json`.

## Report back
List: topics set, CI job name found + used, SECURITY.md created, branch
protection result, README diff summary, release tag created, and the Step 6
question verbatim so it doesn't get lost in a wall of output.
