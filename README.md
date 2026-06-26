# tony-hub-cron

Free, scheduled health/ops pings. This repo is **public** only so that GitHub
Actions minutes are not billed — public repos get unlimited Actions minutes.

## What's here

Nothing but workflow YAML. Every workflow does exactly one thing: on a schedule,
make a single token-gated HTTPS `POST` to an already-public production endpoint.
The endpoint does all the real work; this repo holds no application code, no data,
and no plaintext secrets.

All target endpoints are **bearer-token gated** (they return `401` without the
correct token). The tokens are stored as **GitHub encrypted secrets** on this repo
and are never printed to logs.

## Secrets (set via `gh secret set` — encrypted, not in source)

| Secret | Used by |
| --- | --- |
| `TBONE_BRIDGE_TOKEN` | brain-cadence-*, brain-focus-cache-refresh, brain-morning-brief, gcal-cloud-sync |
| `TAI_CRON_SECRET` | cubs-pregame-push, email-watch-*, opportunity-refresh, gmail-watch-renew |
| `COGNITION_BRAIN_TOKEN` | brain-cloud-ingest |
| `TAI_APP_URL` | gmail-watch-renew (base URL; optional, defaults to production) |

Variable: `TAI_CRON_URL` (base URL override; defaults to production).

## Status

**Active (scheduled, verified):** the 8 bridge/cognition workflows — `brain-cadence-*`,
`brain-focus-cache-refresh`, `brain-morning-brief`, `gcal-cloud-sync`, `brain-cloud-ingest`.
Their `TBONE_BRIDGE_TOKEN` / `COGNITION_BRAIN_TOKEN` secrets are set and validated (HTTP 200).

**Pending one secret (schedule commented, dispatch-only):** the 5 `TAI_CRON_SECRET`
workflows — `cubs-pregame-push`, `email-watch-fast`, `email-watch-check`,
`opportunity-refresh`, `gmail-watch-renew`. The correct `TAI_CRON_SECRET` is stored as a
Vercel *sensitive* env var and is not machine-readable, so it must be set by hand:

```
gh secret set TAI_CRON_SECRET --repo tonf461-hue/tony-hub-cron
# paste the value used by the private repo's TAI_CRON_SECRET secret
```

Then uncomment the `schedule:` block in each of those 5 files. Until then, their private-repo
twins keep running on schedule, so nothing stops working.

## Rollback

Each workflow here has an identical-behavior twin in the private hub repo whose
`schedule:` trigger was disabled when this repo went live (the twin file and its
`workflow_dispatch` were kept). To move a job back: re-add its `schedule:` block to
the private twin and either delete or disable the workflow here. Never run a job in
both places at once.
