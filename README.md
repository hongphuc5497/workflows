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

## Secrets Required

| Secret | Required | Purpose |
|--------|----------|---------|
| `TELEGRAM_BOT_TOKEN` | ✅ | Telegram bot token for failure alerts |
| `TELEGRAM_CHAT_ID` | ✅ | Telegram chat/group ID to send alerts to |
| `SLACK_WEBHOOK_URL` | ❌ | Slack webhook URL (skipped if not set) |
