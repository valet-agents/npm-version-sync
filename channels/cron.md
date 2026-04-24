# Daily Version Sync Check

The cron channel fires this agent on a schedule. There is no
webhook payload — the trigger itself is the signal. Do not look
for JSON input.

## What to do

Run the workflow in `SOUL.md`:

1. Parse the `Monitored Packages` JSON block from `SOUL.md`.
2. If `packages` is empty or every entry is a `<placeholder>`,
   run **Phase 2: Cold-start discovery** and post the proposed
   starter JSON block to Slack. Then stop.
3. Otherwise, run **Phase 3: Drift check** — compare each monitored
   package's GitHub version to its npm dist-tag version, applying
   the edge-case rules in `SOUL.md` (check `versions[]`, handle
   404 as first publish, skip pre-release bumps when dist-tag is
   `latest`).
4. If any drift rows exist, post a single grouped Slack message
   per alert channel. If not, stay silent.

## Guardrails for this run

- One run = one check. Do not retry on transient failures within
  the same run; the next daily cron will re-check.
- If the `gh` CLI fails to authenticate, post a single error
  message to the default alert channel and stop.
- Never publish, never edit, never open PRs (see `SOUL.md`
  guardrails).
