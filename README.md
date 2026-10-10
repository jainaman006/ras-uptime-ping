# ras-uptime-ping

A scheduled GitHub Action that pings the RAS Associates API so Render's free
instance does not go to sleep during business hours.

**This repository is public on purpose, and contains nothing private.** GitHub
Actions minutes are unlimited on public repos but metered on private ones
(2,000/month on the free plan) — a ping every 10 minutes would need roughly
4,300 minutes a month, so running this from the private application repo would
blow through the allowance in the first week. The only thing here is a public
health-check URL and the workflow that calls it. No credentials, no app code.

## What it does

`.github/workflows/keep-warm.yml` calls `GET /api/health` every 10 minutes
between 02:00 and 15:59 UTC (07:30–21:29 IST).

## Why business hours only

Render's free plan allows **750 instance-hours per month**, consumed while the
service is awake. Keeping it warm around the clock is ~744 hours in a 31-day
month: it technically fits, but it uses the whole allowance and leaves nothing
for any other free service. Business hours only costs ~434 hours and covers the
times leads actually arrive.

Outside those hours the first visitor may wait ~50 seconds for a cold start. The
website form falls back to WhatsApp with their details prefilled if the request
times out, so a lead is never silently lost.

## Known limitations

- **GitHub's scheduled workflows are best-effort.** Runs are frequently delayed
  during peak load, sometimes past the 15-minute sleep window, so this reduces
  cold starts rather than eliminating them. A dedicated uptime service
  (cron-job.org, UptimeRobot — both free) fires more punctually if that matters.
- **GitHub disables scheduled workflows in public repos after 60 days without
  activity.** If pings quietly stop, check the Actions tab first; re-enabling or
  any commit restarts it.
- **This is not a monitor.** It does not alert on failure, and deliberately does
  not fail the job on a bad status — a red X on every Render blip would just
  train everyone to ignore it.

## Changing the schedule

Edit the `cron` line in the workflow. Times are UTC; IST is UTC+5:30.

To run it around the clock, use `*/10 * * * *` — but check the instance-hours
maths above first.
