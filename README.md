# workflows

Shared GitHub Actions workflows for [@hongphuc5497](https://github.com/hongphuc5497) repos.

## Available Workflows

### `notify-failure.yml`

Sends Telegram (and optionally Slack) notifications on workflow failure.

**Usage:**

```yaml
notify-failure:
  name: Notify failure
  needs: [<your-job-name>]
  if: ${{ always() && needs.<your-job-name>.result == 'failure' }}
  uses: hongphuc5497/workflows/.github/workflows/notify-failure.yml@main
  with:
    status: failure
    repo: ${{ github.repository }}
    workflow: ${{ github.workflow }}
    job: <your-job-name>
    branch: ${{ github.ref_name }}
    sha: ${{ github.sha }}
    actor: ${{ github.actor }}
    event_name: ${{ github.event_name }}
    run_number: ${{ github.run_number }}
    run_url: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
  secrets:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    TELEGRAM_BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
    TELEGRAM_CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
```

For multiple jobs or to avoid listing each one individually:

```yaml
notify-failure:
  name: Notify failure
  needs: [job1, job2]
  if: ${{ always() && contains(needs.*.result, 'failure') }}
  uses: hongphuc5497/workflows/.github/workflows/notify-failure.yml@main
  ...with/secrets same as above
```

### `upstream-sync.yml`

Runs a "sync" command in the caller repo and, if it produces file changes (i.e.
upstream drift), opens or updates a pull request with the diff. Generic — works
with any command that mutates the working tree.

**Exit-code contract:** the sync command may exit non-zero to signal "drift
found" (e.g. `sync_upstreams.py --apply` exits 1 when changes exist). Success or
failure is decided from the git working tree, not the command's exit code:

- changes present → open/update PR, job succeeds
- no changes, exit 0 → nothing to do, job succeeds
- no changes, exit non-zero → treated as a real failure, job fails (so a
  `notify-failure` job downstream can fire)

**Usage:** drive it from a scheduled caller workflow:

```yaml
on:
  schedule:
    - cron: "0 6 * * 1" # Mondays 06:00 UTC
  workflow_dispatch:

jobs:
  sync:
    permissions:
      contents: write
      pull-requests: write
    uses: hongphuc5497/workflows/.github/workflows/upstream-sync.yml@main
    with:
      sync-command: "python3 scripts/sync_upstreams.py --apply --pr"
      # optional, with defaults shown:
      python-version: "3.11"
      setup-python: true
      pip-packages: "" # e.g. "pyyaml requests"
      pr-base: "main"
      pr-branch: "chore/upstream-sync"
      commit-message: "chore(sync): refresh upstream sources"
      pr-title: "chore(sync): upstream drift detected"
      pr-labels: "automated,upstream-sync"
    secrets:
      GH_PAT: ${{ secrets.GH_PAT }} # optional
```

**Inputs:** only `sync-command` is required; everything else has a default (see
the snippet above).

**Notes:**

- The job needs `contents: write` and `pull-requests: write` — set them on the
  caller `uses:` job as shown.
- If you rely on the default `GITHUB_TOKEN` (no `GH_PAT`), enable **Settings →
  Actions → General → Allow GitHub Actions to create and approve pull requests**
  in the caller repo, or PR creation will fail.
- PRs opened with `GITHUB_TOKEN` do **not** trigger other workflows (e.g.
  validation). Supply a `GH_PAT` secret (a fine-grained PAT with `contents` +
  `pull-requests` write) if you want the sync PR's checks to run.

## Secrets Required

| Secret | Required | Purpose |
|--------|----------|---------|
| `TELEGRAM_BOT_TOKEN` | ✅ | Telegram bot token for failure alerts |
| `TELEGRAM_CHAT_ID` | ✅ | Telegram chat/group ID to send alerts to |
| `SLACK_WEBHOOK_URL` | ❌ | Slack webhook URL (skipped if not set) |
| `GH_PAT` | ❌ | PAT for `upstream-sync.yml` PRs (falls back to `GITHUB_TOKEN`) |
