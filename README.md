# tony-hub-cron

Scheduled health/ops pings. This repo is **public** so ordinary GitHub Actions
minutes are not *supposed* to bill — but the account billing ledger has
attributed a large July charge here (see monorepo
`data/outputs/GITHUB_CRON_BILLING_ATTRIBUTION_2026-07-31.md`). Treat public as
"should be free," not as proof of $0, until that ledger is reconciled.

## What's here

Nothing but workflow YAML. Every workflow does exactly one thing: on a schedule
(or on `workflow_dispatch`), make a token-gated HTTPS `POST` to an already-
public production endpoint. The endpoint does all the real work; this repo holds
no application code, no data, and no plaintext secrets.

All target endpoints are **bearer-token gated** (they return `401` without the
correct token). The tokens are stored as **GitHub encrypted secrets** on this
repo and are never printed to logs.

## Secrets (set via `gh secret set` — encrypted, not in source)

| Secret | Used by |
| --- | --- |
| `TBONE_BRIDGE_TOKEN` | brain-cadence-*, brain-focus-cache-refresh, brain-morning-brief, gcal-cloud-sync |
| `TAI_CRON_SECRET` | combined-tai-http-30min, cubs/email/proactive twins (dispatch), opportunity-refresh, gmail-watch-renew, dining/memory twins |
| `COGNITION_BRAIN_TOKEN` | brain-cloud-ingest |
| `TAI_APP_URL` | gmail-watch-renew (base URL; optional) |

Repo variable `TAI_CRON_URL` still points at `https://heytai.app` for some older
workflows. The combined 30-min Tai HTTP sweep **hard-pins**
`https://tai-web.fly.dev` (Fly production DONE gate) because heytai lags.

## 30-min Tai HTTP consolidation (PR #1 / 2026-07-31)

Four formerly independent `*/30` workflows are now **one** scheduled job
(`combined-tai-http-30min.yml`):

| Former workflow | Status |
| --- | --- |
| `email-watch-check` | schedule → combined (step 2) |
| `email-watch-fast` | schedule **retired** (subsumed by full check at equal cadence) |
| `cubs-pregame-push` | schedule → combined (step 1, first — 10-min window) |
| `tai-proactive-refresh` | schedule → combined (step 3) |

Each former file keeps `workflow_dispatch` for manual one-off pings. Do **not**
re-enable their `schedule:` blocks while the combined workflow is live.

Cadence stays `*/30`. Auth stays `Bearer ${TAI_CRON_SECRET}`. Server-side
idempotency / skip windows / reach-out rules are untouched.

## Rollback

1. Comment out `schedule:` in `combined-tai-http-30min.yml`.
2. Uncomment `schedule: - cron: '*/30 * * * *'` on the individual twins you
   want back. Re-enable `email-watch-fast` only if you restore a *faster*
   cadence than the full check.
3. Never run the same job in both the combined workflow and an individual
   schedule at once.
