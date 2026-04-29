# Version Sync Watchdog

Every morning, Version Sync Watchdog compares your repo's
package.json to the live npm registry. Posts a Slack alert the
moment they drift — silent otherwise.

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> <code>cron</code></td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>github-cli</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://dashboard.valet.dev/setup/configure?from=github.com/valet-agents/npm-version-sync">
        <img src="https://raw.githubusercontent.com/valet-agents/npm-version-sync/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>

## How it works

Runs once a day on a cron. For each repo listed in the
`Monitored Packages` JSON block in `SOUL.md`:

1. Reads `package.json` at the configured branch via `gh`
2. Hits `https://registry.npmjs.org/<package>` and compares the
   local `version` against `dist-tags[<tag>]`
3. Checks the `versions[]` array to avoid false alarms on
   republish scenarios
4. Treats a 404 as "first publish needed", not an error
5. Groups any drift into a single Slack message per alert channel

Silent when everything is in sync — that's the success signal.

## Two modes

**Fast path.** Once the `Monitored Packages` JSON block in
`SOUL.md` has at least one real entry, every daily run just
checks those packages and posts drift alerts.

**Cold start.** When the config block still has the
`<placeholder>` entry, the agent uses `gh repo list` to find your
repos, reads each `package.json`, probes the npm registry, and
posts a ready-to-paste JSON block back to Slack. Paste it into
`SOUL.md`, remove the placeholder, redeploy — you're now in fast
path.

## Configure

The single piece of config you edit is the JSON block in
`SOUL.md`:

```json
{
  "packages": [
    {
      "repo": "acme/widget-sdk",
      "package": "@acme/widget-sdk",
      "branch": "main",
      "dist_tag": "latest",
      "alert_channel": "#releases"
    }
  ],
  "settings": {
    "default_alert_channel": "#releases",
    "silent_on_success": true,
    "alert_on_first_publish_needed": true,
    "skip_prerelease_when_dist_tag_latest": true
  }
}
```

## Setup

See [AGENTS.md](AGENTS.md) for connector, channel, and secret
setup. Short version: attach `gh` and `slack` from your org,
deploy, invite the bot to your alert channel.

## Credits

The drift-detection logic — per-dist-tag comparison, `versions[]`
membership check, 404-as-first-publish — is adapted from the
[JS-DevTools/npm-publish](https://github.com/JS-DevTools/npm-publish)
GitHub Action. This agent only *detects* drift; it never
publishes.
