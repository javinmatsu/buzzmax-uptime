# buzzmax-uptime

Buzzmax's dead-man's switch, run from outside Vercel (BM-128). If Vercel cron or
Inngest stops, every monitor inside the app stops with it and reports nothing.
This repo watches from GitHub Actions instead.

## What it checks

`.github/workflows/uptime-watchdog.yml` runs every 30 minutes. It fails when:

- `https://buzzmax.ai` is not HTTP 200.
- `https://app.buzzmax.ai/login` is not HTTP 200.
- `https://app.buzzmax.ai/api/heartbeat` is not HTTP 200, or its
  `heartbeatAgeMin` is over 150.
  - The app's client heartbeat cron runs hourly at :50, so 150 minutes means
    about two missed runs.

Each URL gets 3 tries, 20 seconds apart. `/api/heartbeat` is public and returns
only `{ok, heartbeatAgeMin}`, with no client data.

Nothing in this repo is secret, so it can be public. Public repos get free
Actions minutes.

## Who is told when it fails

A failed scheduled run sends an email. GitHub sends it to "the user who last
modified the cron syntax in the workflow file"
([GitHub docs](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule)).
Here that is `javinmatsu`, who created the file.

Keep two things in mind:

- If someone else edits the `cron:` line, the emails go to them instead.
- Emails follow that account's notification settings. Check github.com >
  Settings > Notifications > Actions: email should be on, and "Only notify for
  failed workflows" should be checked.

## Staying switched on

GitHub disables scheduled workflows in a public repo after 60 days with no
repository activity. `.github/workflows/keepalive.yml` prevents that: once a
month it commits a timestamp to `.keepalive`.

If the Actions tab ever shows the watchdog as disabled, re-enable it there.

## Running it by hand

Actions tab > uptime-watchdog > Run workflow.
