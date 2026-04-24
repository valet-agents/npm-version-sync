This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## What this agent does

Every day at 9am, the agent checks each repo listed in the
`Monitored Packages` table in `SOUL.md`. For each row, it reads
`package.json` at the configured branch and compares the `version`
field to the matching dist-tag on the npm registry. If they drift,
it posts a grouped alert to Slack. On the first run, if the table
is empty, the agent discovers publishable packages across the
user's GitHub repos and proposes a starter config to paste back
into `SOUL.md`.

## Setup

### Connectors

- **gh**: Provides the `gh` CLI on the agent's PATH with
  `GITHUB_TOKEN` injected. The agent uses `gh repo list` and
  `gh api .../contents/package.json` to read package manifests.
  Note: the connector **must be named `gh`** (not the
  catalog's default `github-cli`) so the wrapper name matches
  the CLI command the agent types. A wrapper named
  `github-cli` puts `/var/lib/valet/bin/github-cli` on PATH —
  when the agent types `gh` it falls through to the
  unauthenticated `/usr/bin/gh`. The `valetdotdev` org already
  has a `gh`-named instance from the catalog.

### Channels

- **cron** (daily 9am America/Los_Angeles, declared inline in
  `valet.yaml`): fires the workflow once per day. Adjust the
  `schedule` / `timezone` fields in `valet.yaml` to change when
  it runs.
- **slack**: gives the agent a dedicated Slack bot identity so it
  can post drift alerts. Attach the org's existing `slack`
  channel to this agent — no new workspace connection required.

### Secrets

- **GITHUB_TOKEN**: Personal access token or fine-grained token
  with `repo` scope (or `public_repo` for public-only). Used by
  the `github-cli` connector. Already set at the org level for
  `valetdotdev`, so no action needed for that org.
- **(optional) NPM_TOKEN**: Only needed if monitoring private
  npm packages. Public packages do not require auth against
  `registry.npmjs.org`.

### External setup

- **Invite the bot to a Slack channel.** After the agent is
  deployed, `valet channels attach slack` prints the bot name
  and workspace. Invite that bot to `#releases` (or whichever
  channel is listed in the `Alert Channel` column of the
  `Monitored Packages` table). Without an invite, the bot can't
  post.
- **Fill in `Monitored Packages` in `SOUL.md`.** The config is a
  JSON block — edit it directly. The agent stays in discovery
  mode until you replace the `<placeholder>` entry with real
  package info. On the first run (or on-demand via
  `valet run "discover packages"`), it will DM a ready-to-paste
  JSON block.

## Bootstrap flow (cold start)

1. `valet agents create --from .` in this directory
2. `valet connectors attach gh --agent version-sync`
3. `valet channels attach slack --agent version-sync`
   (the cron channel is auto-created from `valet.yaml`)
4. Invite the printed bot to your alert channel in Slack
5. Either wait for the next 9am run or prime the discovery with
   `valet run "discover packages now" --agent version-sync`
6. Paste the proposed table into `SOUL.md`, remove the
   `<placeholder>` row, and `valet agents deploy`

## Skipping discovery

If you already know which packages to watch, edit the
`Monitored Packages` table in `SOUL.md` before the first deploy.
The presence of any non-`<placeholder>` row switches the agent
into fast-path mode and discovery is skipped entirely.
